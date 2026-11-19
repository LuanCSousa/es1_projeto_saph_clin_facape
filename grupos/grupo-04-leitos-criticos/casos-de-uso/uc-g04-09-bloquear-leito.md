# UC-G04-09 — Bloquear Leito

| Campo | Valor |
|---|---|
| Ator primário | `EquipeReceptora` |
| Atores secundários | Módulo G3 (notificado se houver reserva), Módulo G6 |
| Relacionamentos | pode acionar UC-G04-06 Cancelar Reserva e UC-G04-02 Reservar Leito |
| RFs rastreados | RF-G04-11, 08, 12 |
| Gatilho | a equipe identifica risco sanitário, manutenção ou falta de equipamento |

**Pré-condições**
1. Usuário autenticado com perfil autorizado.
2. Leito em estado Vago, Reservado ou Em Higienização.

**Fluxo principal**
1. A `EquipeReceptora` seleciona o leito.
2. Informa o motivo (lista parametrizável: manutenção, risco sanitário, falta de equipamento, outro com descrição).
3. O sistema valida o estado do leito e o perfil do usuário.
4. O sistema muda o leito para Bloqueado, guardando estado anterior, motivo, responsável e hora.
5. O leito sai da disponibilidade.
6. O sistema emite o evento ao G6 e confirma a operação.

**Fluxos alternativos**
- **A1 · Leito Reservado (passo 3):** o sistema executa UC-G04-06 (motivo "leito bloqueado"), notifica o G3 e o `MedicoRegulador`, e tenta nova reserva para o chamado via UC-G04-02 _(hipótese a validar)_.
- **A2 · Leito Em Higienização (passo 3):** a higienização é suspensa; no desbloqueio o leito volta a Em Higienização.

**Fluxos de exceção**
- **E1 · Leito Ocupado:** recusa e orienta a liberar o leito antes de bloquear.
- **E2 · Motivo ausente:** recusa; nenhuma alteração.
- **E3 · Usuário sem permissão:** nega a operação e registra evento sensível ao G6.
- **E4 · Falha de gravação:** rollback; o leito mantém o estado anterior.

**Pós-condições**
- Leito Bloqueado com motivo, responsável e hora; reserva afetada tratada (cancelada ou realocada).
