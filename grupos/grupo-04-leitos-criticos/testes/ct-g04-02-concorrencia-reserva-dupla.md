# CT-G04-02 — Concorrência de duas reservas simultâneas no último leito vago

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | CT-G04-02 |
| **Requisito rastreado** | RF-G04-04, RF-G04-05, RF-G04-12, RNF-G04-02 |
| **Caso de uso** | UC-G04-02, UC-G04-03, UC-G04-04 |
| **Tipo** | fronteira (concorrência / duplicidade) |
| **Prioridade** | alta |

## Objetivo

Garantir que duas solicitações simultâneas de pré-admissão disputando o único leito vago de um setor gerem exatamente **uma** reserva ativa e **uma** negativa com motivo `LEITO_INDISPONIVEL`, impedindo reserva dupla (`RN-05`).

## Pré-condições

- Setor `Sala Vermelha` ativo com apenas 1 leito em estado `VAGO`: `SV-01` (demais leitos em `OCUPADO` ou `EM_HIGIENIZACAO`).
- Nenhuma reserva ativa para `CH-2026-101` nem para `CH-2026-102`.

## Dados de entrada

| Solicitação | `chamado_id` | `prontuario_id` | `prioridade` | `necessidade` | `eta_hospital` |
| --- | --- | --- | --- | --- | --- |
| Pedido A | `CH-2026-101` | `PR-2026-101` | `VERMELHO` | `SALA_VERMELHA` | `8 min` |
| Pedido B | `CH-2026-102` | `PR-2026-102` | `LARANJA` | `SALA_VERMELHA` | `5 min` |

## Passos

1. Disparar simultaneamente os pedidos de pré-admissão **A** e **B** direcionados ao setor `SALA_VERMELHA`.
2. Inspecionar a resposta retornada ao G3 para o Pedido A e para o Pedido B.
3. Consultar a quantidade de registros `ReservaLeito` com `estado = ATIVA` associados ao leito `SV-01`.
4. Consultar a disponibilidade da `Sala Vermelha` (`UC-G04-01`).
5. Verificar os eventos C5 enfileirados em `EventoPendenteG6`.

## Resultado esperado

- Exatamente **uma** solicitação (ex.: Pedido A) é aceita e recebe resposta `RESERVADO` para `SV-01`.
- A outra solicitação (Pedido B) detecta conflito em `UC-G04-03`, não encontra outro leito vago e aciona `UC-G04-04`, recebendo resposta `NEGADO` com motivo `LEITO_INDISPONIVEL` e prioridade `LARANJA` preservada.
- O leito `SV-01` possui estado `RESERVADO` e **exatamente 1** `ReservaLeito` com `estado = ATIVA`.
- A consulta de disponibilidade da `Sala Vermelha` retorna 0 leitos vagos.
- São enfileirados 2 eventos ao G6: `LEITO_RESERVADO` (`resultado = SUCESSO`) para o pedido aceito e `LEITO_NEGADO` (`resultado = NEGADO`, `justificativa = "LEITO_INDISPONIVEL"`) para o pedido negado.

## Resultado obtido

`não executado` (validação sobre a modelagem prevista para o M3).

## Critério de aceite

Passou se e somente se existir exatamente **1** reserva ativa para `SV-01` (0 reservas duplicadas), 1 resposta `NEGADO` explícita com motivo e ambos os eventos registrados na fila do G6.

## Observações

Valida `RN-05`, `RNF-G04-02`, `ADR-G04-01` e o cenário 1 do diagrama [`diagramas/seq-excecao-reserva-e-indisponibilidade.puml`](../diagramas/seq-excecao-reserva-e-indisponibilidade.puml) (`DS-G04-02`).
