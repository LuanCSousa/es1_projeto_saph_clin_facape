# UC-G04-06 — Cancelar Reserva

| Campo | Valor |
|---|---|
| Ator primário | EquipeReceptora, MedicoRegulador |
| RFs rastreados | RF-G04-08, RF-G04-12 |
| Status | resumo no M1; especificação textual completa só se for um dos 2 UCs detalhados |

**Fluxo principal**
1. O ator seleciona a reserva ativa e informa o motivo. 2. O sistema marca a reserva Cancelada e o leito volta a Vago. 3. Notifica o G3 e emite evento.

**Fluxo de exceção**
Reserva já Ocupada: recusa o cancelamento (o caminho é UC-08, liberar leito).
