# Matriz de Rastreabilidade — G04

> Modelo alinhado a `docs/templates/matriz-rastreabilidade.md` e ao catálogo em `requisitos.md`.
> Células de fases futuras são indicadas com `— (M2)` ou `— (M3)`.

## Requisitos funcionais

| RF | Descrição curta | Caso de uso | Classe(s) | Sequência | Caso de teste | Status |
| --- | --- | --- | --- | --- | --- | --- |
| RF-G04-01 | Consultar disponibilidade de leitos por setor com estado explícito (RN-06) | UC-G04-01 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-02 | Receber pedido de pré-admissão (`PreAdmissao`) e abrir solicitação vinculada ao `chamado_id` | UC-G04-02 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-03 | Reservar leito vago compatível e fixar prazo `expira_em` | UC-G04-02 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-04 | Verificar conflito de reserva ativa por leito e por chamado (RN-05) | UC-G04-02, UC-G04-03 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-05 | Negar reserva com motivo quando não houver leito vago compatível | UC-G04-02, UC-G04-04 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-06 | Emitir `AlertaTransferencia` à `EquipeReceptora` ao confirmar a reserva | UC-G04-02 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-07 | Confirmar chegada do paciente na porta e ocupar o leito reservado | UC-G04-05 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-08 | Cancelar reserva ativa informando motivo | UC-G04-06, UC-G04-09 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-09 | Expirar reserva ativa sem chegada dentro do prazo (`expira_em`) | UC-G04-07 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-10 | Liberar leito ocupado, enviar a Em Higienização e registrar `Higienizacao` | UC-G04-08 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-11 | Bloquear leito com motivo obrigatório e desbloqueá-lo (RN-06) | UC-G04-09, UC-G04-10 | — (M2) | — (M2) | — (M3) | especificado (M1) |
| RF-G04-12 | Publicar evento sensível ao G6 por transição de estado de leito e reserva (RN-09) | UC-G04-02 a UC-G04-10 | — (M2) | — (M2) | — (M3) | especificado (M1) |

## Requisitos não funcionais

| RNF | Métrica e alvo | Como é verificado | Status |
| --- | --- | --- | --- |
| RNF-G04-01 | Resposta da reserva (confirmada ou negada) com p95 ≤ 2 s até 20 req/min | análise do fluxo de reserva + caso de teste de desempenho (M3) | especificado (M1) |
| RNF-G04-02 | 0 reservas duplicadas em teste com 50 solicitações simultâneas sobre o mesmo leito | caso de teste de fronteira/concorrência + diagrama de sequência (M2/M3) | especificado (M1) |
| RNF-G04-03 | Entrega do `AlertaTransferencia` no painel do setor com p95 ≤ 5 s após confirmação | diagrama de sequência + caso de teste de alerta (M2/M3) | especificado (M1) |
| RNF-G04-04 | Disponibilidade ≥ 99,5% ao mês; falha do G6 não bloqueia reserva (fila assíncrona) | fluxo de exceção E4 de UC-G04-02 + ADR de integração | especificado (M1) |
| RNF-G04-05 | 100% das alterações de estado exigem usuário autenticado/autorizado e geram evento em negativa | fluxo de exceção E3 de UC-G04-09 + caso de teste de permissão (M3) | especificado (M1) |
| RNF-G04-06 | 0 campos de dados pessoais/clínicos em `ReservaLeito` e `AlertaTransferencia` além de `chamado_id` e prioridade (LGPD) | inspeção do diagrama de classes e dicionário de dados (M2) | especificado (M1) |
| RNF-G04-07 | Estado dos leitos de um setor visível em uma única tela, a ≤ 2 interações do painel inicial | inspeção dos wireframes de baixa fidelidade (M3) | especificado (M1) |
| RNF-G04-08 | 0 alterações de código/deploy para mudar prazo de expiração ou lista de motivos de bloqueio | inspeção de atributos/parametrização no modelo estrutural e ADR (M2) | especificado (M1) |

## Contratos de módulo consumidos

| Contrato | Origem | Uso no módulo | Campo crítico |
| --- | --- | --- | --- |
| C3 `pedido_pre_admissao` | G3 | gatilho e dados de entrada de UC-G04-02 (`Reservar Leito`) | `chamado_id`, `prioridade`, `necessidade`, `eta_hospital` |

## Contratos de módulo fornecidos

| Contrato | Destino | Campo crítico | Estabilidade |
| --- | --- | --- | --- |
| Resposta de pré-admissão (reserva ativa ou negativa) | G3 / `MedicoRegulador` | `leito_id`, `setor`, `expira_em` / `motivo` | estável |
| `AlertaTransferencia` | `EquipeReceptora` | `chamado_id`, `prioridade`, `setor`, `leito_id` | estável |
| C5 `evento_sensivel` (`LEITO_RESERVADO`, `LEITO_NEGADO`, `LEITO_BLOQUEADO`, etc.) | G6 | `evento_id`, `tipo`, `entidade_id`, `resultado` | estável |

## Pendências e riscos

| Item | Impacto | Prazo interno |
| --- | --- | --- |
| Definir em `ADR-G04-01` regra de cálculo de `expira_em` com `eta_hospital` e critério de escolha de leito | afeta passo 3 e 5 de UC-G04-02 | M2 |
| Modelar classes de domínio, diagrama de sequência e máquina de estados do leito | completa colunas de M2 na matriz | M2 |
| Elaborar casos de teste (`CT-G04-NN`) e protótipos | completa colunas de M3 na matriz | M3 |
