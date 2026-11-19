# UC-G04-01 — Consultar Disponibilidade

| Campo | Valor |
|---|---|
| Ator primário | EquipeReceptora, MedicoRegulador |
| RFs rastreados | RF-G04-01 |
| Status | resumo no M1; especificação textual completa só se for um dos 2 UCs detalhados |

**Fluxo principal**
1. O ator escolhe o setor. 2. O sistema lista os leitos do setor com o estado atual. 3. Apenas leitos Vago são marcados como disponíveis.

**Fluxo de exceção**
Falha ao ler o estado dos leitos: o sistema informa indisponibilidade e **não** exibe dado em cache como se fosse atual.
