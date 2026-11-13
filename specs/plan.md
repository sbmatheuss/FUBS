# Plano Técnico

## Resumo

Fila virtual para Unidades Básicas de Saúde. O paciente retira uma senha pela web (e futuramente pelo WhatsApp), acompanha a posição e o tempo estimado em tempo real e é avisado quando está perto de ser chamado. A recepção opera a fila por um painel: chama o próximo, marca ausência, confirma prioridades e inclui manualmente quem não tem celular.

Monólito modular em FastAPI com arquitetura hexagonal: as regras de negócio ficam num domínio em Python puro, e banco, telas e notificações são adaptadores.

Escala do MVP: 1 a 5 unidades, até ~150 senhas por unidade por dia, 1 instância da aplicação.

## Stack Tecnológica

- **Linguagem:** Python 3.12
- **Framework web:** FastAPI (assíncrono)
- **Banco:** PostgreSQL 16 + SQLAlchemy 2.0 (async, `asyncpg`) + Alembic (migrações)
- **Validação e configuração:** Pydantic v2 + `pydantic-settings`
- **Tempo real:** WebSockets nativos do FastAPI, com fallback para polling via HTMX
- **Telas:** Jinja2 + HTMX, CSS próprio e leve
- **Autenticação do painel:** sessão por cookie (`HttpOnly`, `Secure`) + hash Argon2
- **Notificações:** Web Push (MVP) e WhatsApp Cloud API (fase posterior)
- **Testes:** pytest, pytest-bdd, pytest-asyncio, httpx, testcontainers
- **Qualidade:** uv, ruff, mypy `--strict`, pre-commit, pip-audit
- **Deploy:** container Docker + Postgres gerenciado

## Arquitetura

Arquitetura hexagonal. As dependências sempre apontam para o domínio, nunca o contrário.

```
Adaptadores de entrada:  api/ (REST)  ·  web/ (Jinja2+HTMX)  ·  realtime/ (WebSockets)
                                          │
                                          ▼
Casos de uso:            application/  (emitir_senha, chamar_proximo, marcar_ausente...)
                                          │
                                          ▼
Domínio (Python puro):   domain/  (Fila, Senha, Prioridade, regras de ordenação)
                                          │ define portas (Protocol)
                                          ▼
Adaptadores de saída:    db/ (SQLAlchemy)  ·  notifications/ (WebPush, WhatsApp)  ·  clock
```

**Portas definidas no domínio:** `RepositorioSenhas`, `Notificador`, `Relogio`, `PublicadorEventos`.

**Decisões principais:**
- **Ordem da fila:** usa a prioridade **confirmada** pela recepção. `IDOSO_80` vem antes de `LEGAL`; empate por ordem de chegada. A cada N prioritários seguidos (padrão 2, configurável), chama 1 normal.
- **Emissão atômica:** transação com `SELECT ... FOR UPDATE` no turno + `UNIQUE (turno_id, codigo)` no banco.
- **Estimativa de espera:** posição × média móvel das últimas 10 chamadas, exibida como faixa ("entre 20 e 35 min").
- **Erros:** formato RFC 9457 (Problem Details). 422 validação, 404 não encontrado, 409 conflito de estado, 429 excesso de requisições.

**Principais rotas:**
- Paciente: `POST /unidades/{id}/senhas` · `GET /senhas/{token}` · `DELETE /senhas/{token}` · `WS /ws/senhas/{token}`
- Recepção: `POST /painel/turnos/{id}/chamar-proximo` · `POST /painel/senhas/{id}/ausente` · `POST /painel/senhas/{id}/atendida` · `POST /painel/senhas/{id}/confirmar-prioridade` · `POST /painel/turnos/{id}/senhas` (inclusão manual) · `WS /ws/painel/turnos/{id}`

**Estrutura de pastas:**
```
src/minhavez/
├── domain/          # regras puras + portas
├── application/     # casos de uso
├── adapters/
│   ├── db/  api/  web/  realtime/  notifications/
├── config.py
└── main.py          # composição e injeção de dependências
```

## Modelo de Dados

| Entidade | Campos principais |
|---|---|
| **Unidade** | `id`, `nome`, `endereco`, `ativa` |
| **Turno** | `id`, `unidade_id`, `data`, `periodo`, `vagas_totais`, `contador_normal`, `contador_prioritario`, `status` |
| **Senha** | `id`, `turno_id`, `codigo` (ex.: `P003`), `token_acesso`, `nome_exibicao`, `telefone`, `prioridade_declarada`, `prioridade_confirmada`, `origem`, `status`, `emitida_em`, `chamada_em`, `finalizada_em` |
| **Atendente** | `id`, `unidade_id`, `nome`, `email`, `hash_senha`, `ativo` |
| **EventoAuditoria** | `id`, `atendente_id`, `senha_id`, `acao`, `ocorrido_em` |
| **NotificacaoPendente** | `id`, `senha_id`, `canal`, `tipo`, `tentativas`, `proxima_tentativa_em`, `status` |

**Estados da senha:**
```
AGUARDANDO ──chamar──► CHAMADA ──atender──► ATENDIDA
    │  ▲                  │
    │  └──voltar_fila─────┤ (1ª ausência)
    │                     └──marcar_ausente──► AUSENTE (2ª ausência)
    └──cancelar──► CANCELADA
```

**Restrições:** `UNIQUE (turno_id, codigo)`, `UNIQUE (token_acesso)`, índice em `(turno_id, status, emitida_em)`.

**LGPD:** `nome_exibicao` e `telefone` são anonimizados 30 dias após o fim do turno (comando CLI agendado via cron).

## Integrações

| Integração | Uso | Observações |
|---|---|---|
| **Web Push** | Avisos "faltam 3" e "chamada" no navegador do paciente | Gratuito; canal principal do MVP |
| **WhatsApp Cloud API** | Retirada de senha e avisos pelo WhatsApp | Fase posterior. O paciente inicia a conversa (abre a janela de 24h) e as respostas usam mensagens de serviço: 1.000 grátis por mês por número, depois cobradas |
| **WebSockets** | Atualização em tempo real de posição e painel | Em memória no MVP; PostgreSQL LISTEN/NOTIFY se houver mais de uma instância |
| **PostgreSQL** | Fonte da verdade | Gerenciado em produção; testcontainers nos testes |

**Padrão para notificações (outbox):**
1. O caso de uso grava a notificação na mesma transação da mudança na fila.
2. Um worker em background envia e marca como enviada.
3. Em caso de falha, tenta de novo com backoff (30s, 2min, 10min), no máximo 3 vezes.
4. A falha de um canal **nunca** afeta a fila.

Todas as integrações implementam a porta `Notificador`; trocar ou somar canais não altera o domínio.

## Riscos e Mitigações

| Risco | Mitigação |
|---|---|
| Paciente sem celular fica em desvantagem | Inclusão manual pela recepção e cota de vagas reservada para atendimento presencial |
| Autodeclaração falsa de prioridade | A ordem da fila usa apenas a prioridade confirmada pela recepção |
| Pacientes normais nunca chamados com muitos prioritários | Intercalação configurável (padrão: 1 normal a cada 2 prioritários) |
| Duas pessoas recebem a mesma senha | Transação com `FOR UPDATE` + restrição `UNIQUE` no banco + teste de concorrência |
| Internet da UBS instável | Reconexão automática do WebSocket, fallback para polling e ações idempotentes |
| Estimativa de espera errada gera frustração | Exibir faixa de tempo, não número exato; aviso "faltam 3" como sinal principal |
| Falha ou custo do WhatsApp | Web Push como canal principal; WhatsApp opcional por unidade; outbox com retentativa |
| Vazamento de dados pessoais | Coleta mínima, anonimização em 30 dias, telefone mascarado nos logs, auditoria das ações |