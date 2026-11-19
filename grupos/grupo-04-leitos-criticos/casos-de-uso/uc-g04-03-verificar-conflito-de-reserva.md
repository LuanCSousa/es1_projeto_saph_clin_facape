# UC-G04-03 — Verificar Conflito de Reserva (incluído por UC-02)

| Campo | Valor |
|---|---|
| Ator primário | Sistema (incluído por UC-G04-02) |
| RFs rastreados | RF-G04-04 |
| Status | resumo no M1; especificação textual completa só se for um dos 2 UCs detalhados |

**Fluxo principal**
1. Recebe leito candidato e `chamado_id`. 2. Confirma que o leito está Vago e sem reserva ativa. 3. Confirma que o chamado não tem outra reserva ativa. 4. Devolve 'sem conflito'.

**Fluxo de exceção**
Conflito encontrado: devolve 'conflito' com o tipo (leito ou chamado) para UC-02 decidir (próximo leito ou reserva existente).
