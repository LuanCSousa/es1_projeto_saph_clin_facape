# UC-G04-08 — Liberar Leito

| Campo | Valor |
|---|---|
| Ator primário | EquipeReceptora |
| RFs rastreados | RF-G04-10, RF-G04-12 |
| Status | resumo no M1; especificação textual completa só se for um dos 2 UCs detalhados |

**Fluxo principal**
1. A equipe registra a saída do paciente. 2. O leito vai a Em Higienização. 3. Ao concluir, registra a `Higienizacao` e o leito volta a Vago. 4. Evento ao G6 a cada transição.

**Fluxo de exceção**
Higienização reprovada ou risco sanitário: o leito vai a Bloqueado via UC-09.
