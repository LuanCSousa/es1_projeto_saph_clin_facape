# Matriz de Rastreabilidade — G04

> Modelo alinhado a `docs/templates/matriz-rastreabilidade.md`, ao catálogo em [`requisitos.md`](requisitos.md) e aos diagramas de E4 e E5.
> Células planejadas para o próximo marco são indicadas com `— (M3)`.

## Requisitos funcionais

| RF | Descrição curta | Caso de uso | Classe(s) | Sequência / Estado | Caso de teste | Status |
| --- | --- | --- | --- | --- | --- | --- |
| RF-G04-01 | Consultar disponibilidade de leitos por setor com estado explícito (RN-06) | UC-G04-01 | `Setor`, `Leito` | DS-G04-01, `estados-leito` | CT-G04-02, — (M3) | modelado (M2) |
| RF-G04-02 | Receber pedido de pré-admissão (`PreAdmissao`) e abrir solicitação vinculada ao `chamado_id` | UC-G04-02 | `ReservaLeito`, `Setor` | DS-G04-01, `atividades-reserva-leito` | CT-G04-01 | modelado (M2) |
| RF-G04-03 | Reservar leito vago compatível e fixar prazo `expira_em` | UC-G04-02 | `Leito`, `ReservaLeito`, `ParametroOperacional` | DS-G04-01, `estados-leito` | CT-G04-01 | modelado (M2) |
| RF-G04-04 | Verificar conflito de reserva ativa por leito e por chamado (RN-05) | UC-G04-02, UC-G04-03 | `Leito`, `ReservaLeito` | DS-G04-01, DS-G04-02 | CT-G04-02 | modelado (M2) |
| RF-G04-05 | Negar reserva com motivo quando não houver leito vago compatível | UC-G04-02, UC-G04-04 | `Setor`, `Leito`, `EventoPendenteG6` | DS-G04-02, `atividades-reserva-leito` | CT-G04-02 | modelado (M2) |
| RF-G04-06 | Emitir `AlertaTransferencia` à `EquipeReceptora` ao confirmar a reserva | UC-G04-02 | `AlertaTransferencia`, `ReservaLeito`, `Setor` | DS-G04-01 | CT-G04-01 | modelado (M2) |
| RF-G04-07 | Confirmar chegada do paciente na porta e ocupar o leito reservado | UC-G04-05 | `Leito`, `ReservaLeito`, `AlertaTransferencia` | DS-G04-03, `estados-leito` | — (M3) | modelado (M2) |
| RF-G04-08 | Cancelar reserva ativa informando motivo | UC-G04-06, UC-G04-09 | `ReservaLeito`, `Leito`, `AlertaTransferencia` | DS-G04-03, `estados-leito` | — (M3) | modelado (M2) |
| RF-G04-09 | Expirar reserva ativa sem chegada dentro do prazo (`expira_em`) | UC-G04-07 | `ReservaLeito`, `Leito`, `ParametroOperacional` | `estados-leito`, ADR-G04-01 | — (M3) | modelado (M2) |
| RF-G04-10 | Liberar leito ocupado, enviar a Em Higienização e registrar `Higienizacao` | UC-G04-08 | `Leito`, `Higienizacao` | DS-G04-03, `estados-leito` | — (M3) | modelado (M2) |
| RF-G04-11 | Bloquear leito com motivo obrigatório e desbloqueá-lo (RN-06) | UC-G04-09, UC-G04-10 | `Leito`, `BloqueioLeito`, `ParametroOperacional` | DS-G04-03, `estados-leito` | — (M3) | modelado (M2) |
| RF-G04-12 | Publicar evento sensível ao G6 por transição de estado de leito e reserva (RN-09) | UC-G04-02 a UC-G04-10 | `EventoPendenteG6`, `Leito`, `ReservaLeito` | DS-G04-01, DS-G04-02, DS-G04-03 | CT-G04-01, CT-G04-02 | modelado (M2) |

## Requisitos não funcionais

| RNF | Métrica e alvo | Como é verificado | Status |
| --- | --- | --- | --- |
| RNF-G04-01 | Resposta da reserva (confirmada ou negada) com p95 ≤ 2 s até 20 req/min | `DS-G04-01` + `ADR-G04-01` + `ADR-G04-02` + `CT-G04-01` | modelado (M2) |
| RNF-G04-02 | 0 reservas duplicadas em teste com 50 solicitações simultâneas sobre o mesmo leito | `DS-G04-02` + `ADR-G04-01` + `CT-G04-02` | modelado (M2) |
| RNF-G04-03 | Entrega do `AlertaTransferencia` no painel do setor com p95 ≤ 5 s após confirmação | `DS-G04-01` + `CT-G04-01` | modelado (M2) |
| RNF-G04-04 | Disponibilidade ≥ 99,5% ao mês; falha do G6 não bloqueia reserva (fila assíncrona) | `DS-G04-02` (Cenário 2) + `ADR-G04-02` + `EventoPendenteG6` | modelado (M2) |
| RNF-G04-05 | 100% das alterações de estado exigem usuário autenticado/autorizado e geram evento em negativa | `UC-G04-09` (E3) + `DS-G04-03` + caso de teste de permissão (M3) | modelado (M2) |
| RNF-G04-06 | 0 campos de dados pessoais/clínicos em `ReservaLeito` e `AlertaTransferencia` além de `chamado_id`, `prontuario_id` e prioridade (LGPD) | `classes-dominio.puml` + `dicionario-de-dados.md` + `CT-G04-01` | modelado (M2) |
| RNF-G04-07 | Estado dos leitos de um setor visível em uma única tela, a ≤ 2 interações do painel inicial | inspeção dos wireframes de baixa fidelidade (M3) | especificado (M1) |
| RNF-G04-08 | 0 alterações de código/deploy para mudar prazo de expiração ou lista de motivos de bloqueio | `ParametroOperacional` (`classes-dominio.puml`, `dicionario-de-dados.md`) + `ADR-G04-01` | modelado (M2) |

## Contratos de módulo consumidos

| Contrato | Origem | Uso no módulo | Campo crítico |
| --- | --- | --- | --- |
| C3 `pedido_pre_admissao` | G3 | gatilho e dados de entrada de UC-G04-02 (`Reservar Leito`) | `chamado_id`, `prontuario_id`, `prioridade`, `necessidade`, `eta_hospital` |

## Contratos de módulo fornecidos

| Contrato | Destino | Campo crítico | Estabilidade |
| --- | --- | --- | --- |
| Resposta de pré-admissão (reserva ativa ou negativa) | G3 / `MedicoRegulador` | `leito_id`, `setor`, `expira_em` / `motivo` | estável |
| `AlertaTransferencia` | `EquipeReceptora` | `chamado_id`, `prioridade`, `setor`, `leito_id` | estável |
| C5 `evento_sensivel` (`LEITO_RESERVADO`, `LEITO_NEGADO`, `LEITO_BLOQUEADO`, etc.) | G6 | `evento_id`, `tipo`, `entidade_id`, `resultado` | estável |

## Rastreio a indicadores do G6 (bônus)

| Indicador G6 (`docs/arquitetura-modular.md`) | Requisitos / Entidades do G4 que alimentam o indicador |
| --- | --- |
| Taxa de reserva negada (`pedidos negados ÷ pedidos`) | `RF-G04-05`, `RF-G04-12` (`LEITO_RESERVADO` e `LEITO_NEGADO` via `EventoPendenteG6`) |
| Ocupação de Sala Vermelha (`leitos ocupados ÷ leitos operacionais`) | `RF-G04-01`, `RF-G04-07`, `RF-G04-10`, `RF-G04-11` (`Setor.calcularTaxaOcupacao()`, `Leito.estado`) |
| Tempo porta-leito (`chegada ao hospital → leito ocupado`) | `RF-G04-07`, `RF-G04-12` (`ReservaLeito.encerrada_em` + evento `LEITO_OCUPADO`) |

## Pendências e riscos

| Item | Impacto | Prazo interno |
| --- | --- | --- |
| Completar casos de teste (`CT-G04-03` e `CT-G04-04`) para ocupação, higienização e bloqueio | completa coluna de CT na matriz | M3 |
| Elaborar wireframes de baixa fidelidade (E7) e relatório técnico no `README.md` | fecha E7 e valida `RNF-G04-07` | M3 |
| Executar verificação completa sobre a modelagem e atualizar campo "Resultado obtido" nos CTs | zera pendências antes do PR final | M3 |
