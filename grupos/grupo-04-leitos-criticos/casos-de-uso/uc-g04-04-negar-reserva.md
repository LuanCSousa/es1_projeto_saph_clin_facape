# UC-G04-04 — Negar Reserva por Indisponibilidade (extensão de UC-02)

| Campo | Valor |
|---|---|
| Ator primário | Sistema (estende UC-G04-02) |
| RFs rastreados | RF-G04-05, RF-G04-12 |
| Status | resumo no M1; especificação textual completa só se for um dos 2 UCs detalhados |

**Fluxo principal**
1. Condição: nenhum leito vago compatível. 2. O sistema responde 'negado' ao G3 com o motivo. 3. Emite evento ao G6.

**Fluxo de exceção**
G3 indisponível para receber a resposta: reenvia com nova tentativa e mantém o registro da negativa.
