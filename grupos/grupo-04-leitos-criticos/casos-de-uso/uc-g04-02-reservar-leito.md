# UC-G04-02 — Reservar Leito

| Campo | Valor |
|---|---|
| Ator primário | Módulo G3 (envia a `PreAdmissao`) |
| Atores secundários | `EquipeReceptora` (recebe o alerta), Módulo G6 (recebe o evento) |
| Relacionamentos | `<<include>>` UC-G04-03 Verificar Conflito de Reserva · estendido por UC-G04-04 Negar Reserva por Indisponibilidade |
| RFs rastreados | RF-G04-02, 03, 04, 05, 06, 12 |
| Gatilho | G3 envia `PreAdmissao` com `chamado_id` |

**Pré-condições**
1. `PreAdmissao` recebida do G3 contendo `chamado_id`, prioridade e setor requerido.
2. Existe ao menos um `Setor` ativo compatível com a prioridade.

**Fluxo principal**
1. O sistema valida os campos obrigatórios da `PreAdmissao`.
2. O sistema lista os leitos em estado Vago do setor requerido.
3. O sistema escolhe um leito pelo critério definido em `ADR-G04-01`.
4. O sistema executa **UC-G04-03** (conflito de leito e de chamado) e não encontra conflito.
5. O sistema, em uma única operação atômica, cria a `ReservaLeito` (Ativa, `expira_em` = agora + prazo) e muda o leito para Reservado.
6. O sistema gera o `AlertaTransferencia` para a `EquipeReceptora` do setor.
7. O sistema responde ao G3 "leito reservado" com `leito_id`, setor e `expira_em`.
8. O sistema emite o evento de transição ao G6.

**Fluxos alternativos**
- **A1 · Sem leito vago (passo 2):** nenhum leito vago no setor → **UC-G04-04** (`<<extend>>`): responde "negado" com motivo, nenhum leito é alterado, evento ao G6.
- **A2 · Conflito detectado (passo 4):** o leito escolhido foi reservado por outra solicitação → volta ao passo 3 com o próximo leito; se esgotar, segue A1.
- **A3 · Pedido repetido (passo 1):** já existe reserva ativa para o `chamado_id` → devolve a reserva existente, sem criar outra.
- **A4 · Setor lotado, outro setor com vaga (passo 2):** o sistema apenas **sugere** o setor alternativo ao `MedicoRegulador`; não reserva sem confirmação _(hipótese a validar com o professor)_.

**Fluxos de exceção**
- **E1 · Pedido inválido (passo 1):** falta `chamado_id` ou prioridade → rejeita com erro; nenhuma alteração.
- **E2 · Falha de gravação/concorrência (passo 5):** rollback completo, sem reserva parcial; devolve erro ao G3 para nova tentativa.
- **E3 · Alerta não entregue em 5 s (passo 6):** a reserva é mantida; o sistema tenta de novo, sinaliza no painel do setor e registra o evento.
- **E4 · G6 indisponível (passo 8):** a reserva segue; o evento fica em fila e é reenviado (não bloquear atendimento crítico).

**Pós-condições**
- Sucesso: leito Reservado, `ReservaLeito` Ativa, alerta pendente de ciência, evento registrado.
- Negado: nenhum leito alterado e motivo devolvido ao G3.
