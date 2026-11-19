# UC-G04-07 — Expirar Reserva

| Campo | Valor |
|---|---|
| Ator primário | Temporizador |
| RFs rastreados | RF-G04-09, RF-G04-12 |
| Status | resumo no M1; especificação textual completa só se for um dos 2 UCs detalhados |

**Fluxo principal**
1. O temporizador encontra reservas ativas com `expira_em` vencido. 2. Marca Expirada e devolve o leito a Vago. 3. Notifica EquipeReceptora e G3; emite evento.

**Fluxo de exceção**
Chegada confirmada no mesmo instante da expiração: vale a primeira operação gravada; a outra recebe erro de estado.
