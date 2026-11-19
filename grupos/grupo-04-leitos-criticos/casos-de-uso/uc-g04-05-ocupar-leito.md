# UC-G04-05 — Ocupar Leito (aceite do paciente na porta)

| Campo | Valor |
|---|---|
| Ator primário | EquipeReceptora |
| RFs rastreados | RF-G04-07, RF-G04-12 |
| Status | resumo no M1; especificação textual completa só se for um dos 2 UCs detalhados |

**Fluxo principal**
1. A equipe identifica o chamado na reserva ativa. 2. Confirma a chegada. 3. O sistema muda Reservado → Ocupado, registra responsável e hora, encerra o alerta e emite evento.

**Fluxo de exceção**
Reserva já expirada ou cancelada: o sistema recusa a ocupação direta e oferece nova consulta de disponibilidade.
