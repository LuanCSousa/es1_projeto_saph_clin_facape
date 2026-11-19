# UC-G04-10 — Desbloquear Leito

| Campo | Valor |
|---|---|
| Ator primário | EquipeReceptora |
| RFs rastreados | RF-G04-11, RF-G04-12 |
| Status | resumo no M1; especificação textual completa só se for um dos 2 UCs detalhados |

**Fluxo principal**
1. A equipe seleciona o leito Bloqueado. 2. Informa a justificativa da liberação. 3. O sistema devolve o leito ao estado anterior (Vago ou Em Higienização) e emite evento.

**Fluxo de exceção**
Usuário sem permissão: nega e registra evento sensível.
