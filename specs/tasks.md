# Tarefas

> **Regra do projeto (constituição, princípio III):** em toda tarefa da Fase 2 marcada como **🧪 Teste**, o teste é escrito **antes** do código da tarefa seguinte e precisa falhar primeiro.
> A Fase 3 cuida da validação do sistema como um todo: integração, desempenho, acessibilidade, LGPD e revisão final.
>
> **[P]** = pode ser feita em paralelo · **(RF/RNF/HU)** = requisito da spec atendido

## Fase 1 — Setup

- [ ] T001: Criar o projeto com `uv init` e a estrutura de pastas da arquitetura (`src/minhavez/domain`, `application`, `adapters`, `tests/`)
- [ ] T002: Adicionar dependências: fastapi, uvicorn, sqlalchemy[asyncio], asyncpg, alembic, pydantic-settings, jinja2, argon2-cffi
- [ ] T003: Adicionar dependências de desenvolvimento: pytest, pytest-bdd, pytest-asyncio, pytest-cov, httpx, testcontainers[postgres], ruff, mypy, pre-commit, pip-audit
- [ ] T004: [P] Configurar ruff, mypy `--strict`, pytest e coverage no `pyproject.toml`
- [ ] T005: [P] Configurar o pre-commit (ruff + mypy) em `.pre-commit-config.yaml`
- [ ] T006: [P] Criar `docker-compose.yml` com PostgreSQL 16 para desenvolvimento
- [ ] T007: [P] Criar `.env.example` e incluir `.env` no `.gitignore`
- [ ] T008: [P] Criar a pipeline de CI (format, lint, tipos, testes, cobertura, pip-audit) em `.github/workflows/ci.yml`
- [ ] T009: Criar `config.py` com `pydantic-settings` (URL do banco, segredo de sessão, padrões da fila)
- [ ] T010: Criar app FastAPI mínima com `GET /health` e um teste para ela
- [ ] T011: [P] Configurar logs em JSON com `request_id` em `logging.py`
- [ ] T012: [P] Criar o `Dockerfile` da aplicação
- [ ] T013: Configurar Alembic com SQLAlchemy async
- [ ] T014: [P] Escrever o ADR 001 (arquitetura hexagonal + Jinja2/HTMX) em `docs/adr/`

**✅ Pronto quando:** `docker compose up` sobe o banco, `/health` responde 200 e o CI está verde.

## Fase 2 — Implementação

### 2.1 Domínio (`domain/`, Python puro, sem banco nem FastAPI)

- [ ] T015: [P] Criar as exceções de domínio: `VagasEsgotadasError`, `TurnoEncerradoError`, `TurnoNaoAbertoError`, `TransicaoInvalidaError`, `FilaVaziaError`, `SenhaAtivaExistenteError`
- [ ] T016: [P] Criar as portas (`Protocol`): `RepositorioSenhas`, `RepositorioTurnos`, `Notificador`, `Relogio`, `PublicadorEventos`
- [ ] T017: [P] Criar relógio fixo (para testes) e relógio real (`adapters/clock.py`)
- [ ] T018: 🧪 Teste: transições válidas e inválidas da senha (RF-09, RF-10, RF-14)
- [ ] T019: Criar a entidade `Senha` com máquina de estados: AGUARDANDO, CHAMADA, ATENDIDA, AUSENTE, CANCELADA
- [ ] T020: 🧪 Teste: turno com vagas esgotadas, encerrado, não aberto e geração de código N/P (RF-02, RF-18)
- [ ] T021: Criar a entidade `Turno`: vagas, contadores, código da senha, horário de liberação
- [ ] T022: 🧪 Teste BDD `ordem_fila.feature`: normal por chegada, prioritário antes de normal, 80+ antes de 60+, prioridade não confirmada conta como normal, fila vazia (RF-05, HU-05)
- [ ] T023: 🧪 Teste BDD `intercalacao.feature`: 1 normal a cada N prioritários (RF-06)
- [ ] T024: Criar o enum `Prioridade` (NORMAL, LEGAL, IDOSO_80), com as leis citadas no docstring e sem nenhuma categoria de urgência (RF-04, RF-05, RF-16)
- [ ] T025: Criar a função pura `proximo_da_fila()` com ordenação e intercalação (RF-05, RF-06)
- [ ] T026: 🧪 Teste BDD `ausencia.feature`: 1ª ausência volta ao fim, 2ª encerra (RF-10)
- [ ] T027: 🧪 Teste: estimativa de espera com e sem histórico, sempre em faixa (RF-07)
- [ ] T028: Criar o cálculo da estimativa de espera (RF-07)

### 2.2 Persistência (`adapters/db/`)

- [ ] T029: Criar os modelos SQLAlchemy: Unidade, Turno, Senha, Atendente, EventoAuditoria, NotificacaoPendente
- [ ] T030: Criar a migração inicial com `UNIQUE (turno_id, codigo)`, `UNIQUE (token_acesso)` e índice `(turno_id, status, emitida_em)` (RNF-08)
- [ ] T031: Criar a sessão async e o controle de transação
- [ ] T032: 🧪 Teste: repositórios salvam, buscam senhas aguardando e buscam por token
- [ ] T033: Criar os repositórios implementando as portas do domínio
- [ ] T034: Implementar a emissão atômica com `SELECT ... FOR UPDATE` no turno (RF-02, RNF-08)
- [ ] T035: [P] Criar o script de dados iniciais (1 unidade, 1 turno, 1 recepcionista)

### 2.3 Paciente (`application/` + `adapters/api/` + `adapters/web/`)

- [ ] T036: 🧪 Teste BDD `paciente.feature`: retirar senha, prioridade pendente, vagas esgotadas, senha já ativa, consultar posição, cancelar (HU-01, HU-02, HU-04)
- [ ] T037: Criar o caso de uso `emitir_senha` (RF-01, RF-02, RF-03, RF-04)
- [ ] T038: [P] Criar o caso de uso `consultar_senha` com posição e faixa de espera (RF-07)
- [ ] T039: [P] Criar o caso de uso `cancelar_senha` (RF-14)
- [ ] T040: Criar os schemas Pydantic com validação de telefone (RNF-01)
- [ ] T041: Criar os handlers que traduzem exceções de domínio em erros HTTP no formato RFC 9457 (404, 409)
- [ ] T042: Criar as rotas do paciente: `GET /unidades/{id}/turno-atual`, `POST /unidades/{id}/senhas`, `GET /senhas/{token}`, `DELETE /senhas/{token}`
- [ ] T043: Criar o limite de requisições por origem (429) (RNF-10)
- [ ] T044: Criar as telas de retirar e acompanhar senha (fonte ≥ 18px, linguagem simples) (RNF-04, RNF-05, RNF-07)
- [ ] T045: [P] Mascarar telefones nos logs (RNF-01)

### 2.4 Recepção (`application/` + `adapters/web/`)

- [ ] T046: 🧪 Teste BDD `recepcao.feature`: chamar próximo, rechamar, marcar ausente, finalizar, encerrar turno (HU-05, HU-07)
- [ ] T047: 🧪 Teste BDD `prioridade_inclusao.feature`: confirmar, recusar, incluir paciente presencial (HU-06, HU-08)
- [ ] T048: Criar o login da recepção com sessão por cookie e hash Argon2 (RF-15)
- [ ] T049: Criar os casos de uso `chamar_proximo`, `rechamar`, `marcar_ausente`, `finalizar_atendimento`, `encerrar_turno` (RF-09, RF-10)
- [ ] T050: [P] Criar os casos de uso `confirmar_prioridade` e `recusar_prioridade` (RF-12)
- [ ] T051: [P] Criar o caso de uso `incluir_manualmente`, sem telefone e com prioridade já confirmada (RF-11, RNF-11)
- [ ] T052: Registrar auditoria (quem e quando) em todas as ações da recepção (RF-13)
- [ ] T053: Criar as rotas do painel
- [ ] T054: Criar a tela do painel: fila atual, prioridades pendentes em destaque, botões de ação com HTMX (RNF-04)
- [ ] T055: [P] Criar a tela de inclusão manual (RF-11)

### 2.5 Tempo real (`adapters/realtime/`)

- [ ] T056: Criar o gerenciador de conexões WebSocket em memória (`turno_id → conexões`)
- [ ] T057: Criar o adaptador `PublicadorEventos`, publicando só após o commit
- [ ] T058: Criar os endpoints `WS /ws/senhas/{token}` e `WS /ws/painel/turnos/{id}` (RF-07)
- [ ] T059: Criar o fallback: polling a cada 15s e aviso de "informação pode estar desatualizada" (RNF-07)

### 2.6 Notificações (`adapters/notifications/`)

- [ ] T060: 🧪 Teste BDD `notificacoes.feature`: senha emitida, faltam 3, chamada, ausente (RF-08, HU-03)
- [ ] T061: Gravar os avisos na outbox na mesma transação da mudança na fila
- [ ] T062: Criar o worker da outbox (backoff 30s / 2min / 10min, máximo 3 tentativas) (RNF-09)
- [ ] T063: Criar o notificador Web Push (chaves VAPID em variável de ambiente) (RF-08)
- [ ] T064: Criar o service worker e o pedido de permissão na tela do paciente (RF-08)
- [ ] T065: [P] Escrever o ADR 002 (Web Push antes do WhatsApp, por custo)

### 2.7 LGPD e indicadores

- [ ] T066: Criar o comando CLI de anonimização de nome e telefone após 30 dias, para rodar via cron (RNF-02)
- [ ] T067: Criar o caso de uso e a tela de indicadores do gestor: total de senhas, espera média, ausência e distribuição por prioridade (RF-17, HU-09)

## Fase 3 — Testes e Validação

### 3.1 Integração e confiabilidade
- [ ] T068: Testar a API do paciente: status 201, 404, 409, 422 e 429 no formato RFC 9457
- [ ] T069: Testar o login e as permissões do painel: 401 sem sessão, 403 em outra unidade (RF-15)
- [ ] T070: **Testar concorrência:** 50 pedidos simultâneos geram 50 códigos únicos, sem ultrapassar as vagas. Rodar 10 vezes seguidas (RNF-08)
- [ ] T071: Testar que um evento não é publicado quando a transação sofre rollback
- [ ] T072: Testar que uma falha no canal de aviso gera retentativa e **não** trava a fila (RNF-09)

### 3.2 Desempenho e tempo real
- [ ] T073: Medir que mudanças na fila chegam ao paciente em até 2 segundos (RNF-06)
- [ ] T074: Derrubar o WebSocket e confirmar que o polling assume sem perda de informação (RNF-07)
- [ ] T075: Medir as operações da fila: p95 abaixo de 300 ms

### 3.3 Acessibilidade e usabilidade
- [ ] T076: Rodar o Lighthouse nas telas do paciente e do painel: acessibilidade ≥ 95 (RNF-04)
- [ ] T077: Confirmar que a página do paciente tem no máximo 200 KB no primeiro carregamento (RNF-07)
- [ ] T078: Navegar pelo painel usando só o teclado (RNF-04)
- [ ] T079: Pedir a 3 pessoas sem experiência que retirem uma senha e cronometrar: menos de 1 minuto (RNF-05)

### 3.4 LGPD e segurança
- [ ] T080: Testar que nenhum log contém telefone completo (RNF-01)
- [ ] T081: Testar que a auditoria e os indicadores não expõem nome nem telefone (RF-13, RF-17, RNF-03)
- [ ] T082: Testar a anonimização: turnos encerrados há mais de 30 dias ficam sem dados pessoais (RNF-02)
- [ ] T083: Rodar `pip-audit` e resolver vulnerabilidades altas e críticas

### 3.5 Validação final
- [ ] T084: Teste ponta a ponta: retirar senha → aguardar → ser avisado → ser chamado → ser atendido
- [ ] T085: Conferir cobertura: `domain/` ≥ 95% e projeto ≥ 80%
- [ ] T086: [P] Exportar o OpenAPI para `specs/001-fila-senhas/contracts/`
- [ ] T087: [P] Escrever o `quickstart.md` e o `README.md`
- [ ] T088: Revisar cada item da Definição de Pronto da constituição
- [ ] T089: Apresentar o sistema a alguém de uma UBS e validar as pendências da spec (intercalação, tolerância de ausência, cota presencial, horário de liberação)