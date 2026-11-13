# Especificação

## Visão Geral

**Problema:** em muitas Unidades Básicas de Saúde (UBS), pacientes chegam de madrugada para conseguir uma ficha e esperam horas na recepção sem saber sua posição na fila nem quanto tempo falta. Isso gera aglomeração, desgaste físico (principalmente para idosos, gestantes e pessoas doentes) e conflitos sobre a ordem de atendimento.

**Objetivo:** o FUBS permite que o paciente retire uma senha virtual, acompanhe a fila à distância e seja avisado quando estiver perto de ser chamado. A recepção mantém o controle da ordem de atendimento, das prioridades legais e da inclusão de quem chega pessoalmente.

## Usuários

- **Paciente:** pessoa que precisa de atendimento na UBS; pode ter pouca familiaridade com tecnologia.
- **Recepcionista:** profissional da UBS que opera a fila e confirma prioridades.
- **Gestor da unidade:** acompanha indicadores da fila para planejar equipe e horários.

## Histórias de Usuário

- **HU-01:** Como paciente, quero retirar uma senha sem sair de casa para não precisar madrugar na porta da UBS.
- **HU-02:** Como paciente, quero ver minha posição e o tempo estimado de espera para decidir quando ir para a UBS.
- **HU-03:** Como paciente, quero ser avisado quando estiver perto da minha vez para não perder a chamada.
- **HU-04:** Como paciente, quero cancelar minha senha para liberar a vaga se eu não puder ir.
- **HU-05:** Como recepcionista, quero chamar o próximo paciente com um clique para atender na ordem correta e sem discussões.
- **HU-06:** Como recepcionista, quero confirmar ou recusar a prioridade declarada para que a ordem da fila seja justa.
- **HU-07:** Como recepcionista, quero marcar quem não compareceu para seguir a fila sem prejudicar quem está presente.
- **HU-08:** Como recepcionista, quero incluir na fila quem chegou pessoalmente para que ninguém fique de fora por não ter celular.
- **HU-09:** Como gestor, quero ver indicadores da fila para planejar equipe e horários.

## Requisitos Funcionais

- **RF-01:** O paciente deve poder retirar uma senha informando nome (ou apelido) e telefone.
- **RF-02:** O sistema deve limitar a quantidade de senhas por turno conforme as vagas definidas pela unidade. `[PRECISA ESCLARECER: o limite é por turno ou por tipo de atendimento?]`
- **RF-03:** O sistema deve permitir apenas 1 senha ativa por telefone por turno.
- **RF-04:** O paciente deve poder se declarar prioritário, indicando a categoria (60+, 80+, gestante, lactante, pessoa com deficiência, pessoa com criança de colo, entre outras previstas em lei).
- **RF-05:** A fila deve ser ordenada pela prioridade **confirmada pela recepção**: pessoas com 80 anos ou mais antes das demais prioridades legais; em empate, por ordem de chegada.
- **RF-06:** O sistema deve intercalar pacientes normais entre prioritários conforme proporção configurável pela unidade. `[PRECISA ESCLARECER: qual proporção? Proposta: 1 normal a cada 2 prioritários]`
- **RF-07:** O paciente deve ver sua posição e uma faixa de tempo estimado, atualizadas automaticamente.
- **RF-08:** O paciente deve ser avisado quando faltarem 3 pessoas e quando for chamado.
- **RF-09:** A recepção deve poder chamar o próximo, rechamar, marcar ausência, finalizar atendimento e encerrar o turno.
- **RF-10:** Paciente ausente na primeira vez volta ao fim da fila; na segunda, a senha é encerrada. `[PRECISA ESCLARECER: tempo de tolerância antes de marcar ausência? Proposta: 5 minutos]`
- **RF-11:** A recepção deve poder incluir pacientes manualmente, sem telefone. `[PRECISA ESCLARECER: existe cota de vagas reservada para atendimento presencial? Proposta: 30%]`
- **RF-12:** A recepção deve poder confirmar ou recusar prioridades declaradas.
- **RF-13:** Toda ação da recepção deve ficar registrada com autor e horário.
- **RF-14:** O paciente deve poder cancelar a própria senha enquanto ela estiver aguardando.
- **RF-15:** O painel da recepção deve exigir login.
- **RF-16:** O sistema **não deve** fazer triagem clínica nem permitir que o paciente se declare caso de urgência.
- **RF-17:** O gestor deve poder consultar indicadores agregados: total de senhas, tempo médio de espera, taxa de ausência e distribuição por prioridade.
- **RF-18:** A senha virtual deve ser liberada a partir de um horário definido pela unidade. `[PRECISA ESCLARECER: quanto tempo antes da abertura? Proposta: 1 hora]`

## Requisitos Não Funcionais

- **RNF-01 (Privacidade/LGPD):** coletar apenas nome (ou apelido) e telefone; nenhum dado clínico.
- **RNF-02 (Privacidade/LGPD):** nome e telefone devem ser anonimizados 30 dias após o fim do turno.
- **RNF-03 (Privacidade):** indicadores e registros de ação não podem expor nome nem telefone.
- **RNF-04 (Acessibilidade):** as telas devem atender às diretrizes WCAG 2.1 nível AA, com letras grandes e linguagem simples.
- **RNF-05 (Usabilidade):** um paciente sem experiência deve conseguir retirar a senha em menos de 1 minuto.
- **RNF-06 (Desempenho):** mudanças na fila devem aparecer para o paciente em até 2 segundos.
- **RNF-07 (Conectividade):** a tela do paciente deve funcionar em celulares simples e internet lenta (3G).
- **RNF-08 (Confiabilidade):** duas pessoas nunca podem receber a mesma senha, nem a mesma vaga.
- **RNF-09 (Confiabilidade):** falha no envio de um aviso não pode interromper o funcionamento da fila.
- **RNF-10 (Segurança):** o sistema deve bloquear temporariamente excesso de pedidos de senha vindos da mesma origem.
- **RNF-11 (Inclusão):** o sistema nunca pode ser a única forma de entrar na fila; o atendimento presencial deve continuar possível.

## Critérios de Aceitação

**Retirar senha (HU-01)**
- Dado que o turno está aberto e há vagas, quando o paciente informa nome e telefone, então recebe um código de senha e um link de acompanhamento.
- Dado que o paciente se declara prioritário, quando pede a senha, então ela é emitida como prioritária pendente de confirmação e ele é avisado de que precisará comprovar na recepção.
- Dado que as vagas acabaram, quando o paciente pede uma senha, então vê a mensagem de vagas esgotadas e o horário do próximo turno.
- Dado que o paciente já tem senha ativa no turno, quando pede outra com o mesmo telefone, então o sistema recusa e mostra o link da senha existente.
- Dado que duas pessoas pedem a última vaga ao mesmo tempo, quando os pedidos são processados, então só uma recebe a senha.

**Acompanhar a fila (HU-02, HU-03)**
- Dado que o paciente tem senha aguardando, quando abre o link, então vê código, posição e uma faixa de tempo estimado (ex.: "entre 20 e 35 minutos").
- Dado que o paciente está com a página aberta, quando outra pessoa é chamada, então sua posição se atualiza sem recarregar a página.
- Dado que faltam 3 pessoas antes do paciente, quando isso acontece, então ele recebe um aviso para se dirigir à UBS.
- Dado que a senha foi chamada, quando a recepção faz a chamada, então o paciente recebe um aviso com o local de atendimento.
- Dado que a conexão está instável, quando a atualização falha, então a página mostra a última informação e avisa que pode estar desatualizada.

**Ordem de chamada (HU-05)**
- Dado uma fila só com pacientes normais, quando a recepção chama o próximo, então é chamado quem chegou primeiro.
- Dado um prioritário confirmado e um normal que chegou antes, quando a recepção chama o próximo, então o prioritário é chamado.
- Dado um paciente de 65 anos e um de 82 anos, ambos confirmados, com o de 82 chegando depois, quando a recepção chama o próximo, então o de 82 anos é chamado.
- Dado um paciente com prioridade ainda não confirmada, quando a recepção chama o próximo, então ele é tratado como normal na ordem, mas aparece destacado no painel.
- Dado que o limite de prioritários seguidos foi atingido e há normais aguardando, quando a recepção chama o próximo, então é chamado um paciente normal.
- Dado que a fila está vazia, quando a recepção tenta chamar, então é informada de que não há pacientes aguardando.

**Prioridades (HU-06)**
- Dado um paciente com prioridade pendente, quando a recepção confirma, então ele passa a ser ordenado como prioritário.
- Dado uma prioridade que não se confirma, quando a recepção recusa, então o paciente continua como normal, mantendo a ordem de chegada.
- Dado qualquer alteração de prioridade, quando é feita, então fica registrado quem alterou e quando.

**Ausência (HU-07)**
- Dado um paciente chamado que não compareceu, quando é marcado ausente pela primeira vez, então volta ao fim da fila e é avisado.
- Dado um paciente que já voltou à fila uma vez, quando não comparece de novo, então a senha é encerrada como ausente e ele é avisado.
- Dado um paciente chamado, quando a recepção rechama, então ele recebe um novo aviso.

**Inclusão presencial (HU-08)**
- Dado um paciente que chegou pessoalmente, quando a recepção o inclui só com o nome, então ele recebe um código e entra na fila pela ordem de chegada.
- Dado um paciente presencial prioritário, quando a recepção o inclui, então a prioridade já entra confirmada.

**Cancelamento (HU-04)**
- Dado uma senha aguardando, quando o paciente cancela pelo link, então a senha sai da fila e a vaga volta a ficar disponível.
- Dado uma senha já chamada, quando o paciente tenta cancelar, então é orientado a falar com a recepção.

**Indicadores (HU-09)**
- Dado um turno encerrado, quando o gestor consulta, então vê total de senhas, tempo médio de espera, taxa de ausência e distribuição por prioridade, sem nenhum nome ou telefone.

## Fora de Escopo

- Agendamento de consultas com dia e hora marcados.
- Prontuário eletrônico ou integração com o e-SUS.
- Triagem clínica ou classificação de risco.
- Teleconsulta.
- Painel público de chamadas em TV (avaliar em versão futura).
- Retirada de senha pelo WhatsApp (versão futura; o MVP começa pela web).