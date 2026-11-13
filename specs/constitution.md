# Constituição do Projeto

## Stack

- Python, FastAPI
- SQLAlchemy + PostgreSQL
- WebSockets
- Jinja2 + HTMX
- WhatsApp Cloud API
- pytest + pytest-bdd
- uv (pacotes), ruff (lint/format), mypy (tipos)

## Padrões de Qualidade

### Spec primeiro
- Nenhum código de funcionalidade sem `spec.md` com critérios de aceite no formato Dado / Quando / Então.
- A spec descreve o quê e por quê, nunca como.
- Se o comportamento muda, a spec muda primeiro. Código que contradiz a spec é bug.

### Domínio puro
- Regras de negócio ficam em `src/minhavez/domain/`, sem importar FastAPI, SQLAlchemy ou HTTP.
- Dependências externas (banco, notificação, relógio) entram por interfaces (`Protocol`).
- Nunca usar `datetime.now()` direto no domínio; o relógio é injetado.

### Testes
- Todo critério de aceite vira um cenário `.feature` executado com pytest-bdd.
- Regras de domínio desenvolvidas com TDD.
- Cobertura mínima: 95% em `domain/` (linhas e branches) e 80% no projeto.
- Todo bug corrigido ganha um teste que o reproduz.
- Emissão de senha tem teste de concorrência.

### Código
- Devem passar sem erro: `ruff format --check`, `ruff check`, `mypy --strict src/`, `pytest`.
- Toda função pública tipada; `Any` só com justificativa.
- Termos do domínio em português, iguais aos da spec (`Senha`, `Fila`, `chamar_proximo`).
- Funções com até ~30 linhas e complexidade ciclomática ≤ 10.
- Sem números mágicos: limites vêm de configuração.
- Erros de domínio são exceções próprias, traduzidas em status HTTP na camada da API (422 validação, 404 não encontrado, 409 conflito de estado).

### Observabilidade
- Logs estruturados em JSON com `request_id`.
- Métricas: senhas emitidas, tempo médio de espera, taxa de ausência, falhas de notificação.

### Fluxo de trabalho
- Uma branch por spec (`001-fila-senhas`).
- Conventional Commits (`feat:`, `fix:`, `test:`, `docs:`...).
- pre-commit com ruff e mypy; CI bloqueia merge se algo falhar.
- Decisões técnicas relevantes registradas como ADR em `docs/adr/`.

### Definição de Pronto
- Critérios de aceite passando como testes.
- ruff, mypy e pytest sem erros; cobertura nos mínimos.
- Nenhum dado pessoal em logs.
- Documentação atualizada e PR revisado com CI verde.

## Restrições

### Regras de negócio inegociáveis
- O sistema não faz triagem clínica: urgência é decidida pela equipe de saúde, nunca autodeclarada.
- Prioridades seguem a Lei 10.048/2000 e o Estatuto da Pessoa Idosa (80+ antes de 60+).
- Prioridade declarada é confirmada pela recepção.
- O app nunca é barreira de acesso: a recepção sempre pode incluir pacientes manualmente.

### LGPD e segurança
- Coletar apenas nome (ou apelido) e telefone; nenhum dado clínico.
- Dados pessoais de senhas encerradas anonimizados em até 30 dias.
- Telefones mascarados em logs.
- Segredos apenas em variáveis de ambiente; `.env` fora do repositório.
- `pip-audit` no CI; vulnerabilidade alta ou crítica bloqueia merge.
- Painel da recepção autenticado; toda ação registra quem fez e quando.

### Acessibilidade
- WCAG 2.1 nível AA.
- Fonte base mínima de 18px na tela do paciente; linguagem simples.
- Página do paciente com no máximo 200 KB no primeiro carregamento, funcional em 3G.
- Se o WebSocket cair, volta para atualização periódica e avisa o usuário.

### Desempenho e confiabilidade
- Operações da fila com p95 < 300 ms.
- Atualização de posição chega ao paciente em até 2 segundos.
- Emissão de senha atômica (transação + restrição de unicidade no banco).
- Falha de notificação nunca quebra a fila; é registrada e reenviada.

## Governança

- Esta constituição prevalece sobre qualquer outra prática do projeto.
- Alterações exigem justificativa escrita, nova versão (semver) e revisão das specs afetadas.
- Exceções pontuais são registradas no PR com motivo e plano para removê-las.
- Versão: 1.0.0 · Ratificada em: 2026-11-13