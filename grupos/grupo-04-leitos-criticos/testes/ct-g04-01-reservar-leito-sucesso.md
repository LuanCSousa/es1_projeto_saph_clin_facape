# CT-G04-01 — Reservar leito vago com sucesso e emitir alerta à equipe receptora

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | CT-G04-01 |
| **Requisito rastreado** | RF-G04-02, RF-G04-03, RF-G04-06, RF-G04-12, RNF-G04-01, RNF-G04-03, RNF-G04-06 |
| **Caso de uso** | UC-G04-02, UC-G04-03 |
| **Tipo** | funcional |
| **Prioridade** | alta |

## Objetivo

Garantir que um pedido válido de pré-admissão (`C3`) reserva um leito vago compatível, calcula `expira_em` conforme `ADR-G04-01`, emite `AlertaTransferencia` minimizado (LGPD) e enfileira o evento `LEITO_RESERVADO` ao G6.

## Pré-condições

- Setor `Sala Vermelha` ativo com os leitos `SV-01` (estado `VAGO` há 40 min) e `SV-02` (estado `VAGO` há 10 min).
- Parâmetro operacional `margem_eta_min = 10`.
- Nenhuma reserva `ATIVA` associada ao `chamado_id = CH-2026-001`.

## Dados de entrada

| Campo | Valor |
| --- | --- |
| `chamado_id` | `CH-2026-001` |
| `prontuario_id` | `PR-2026-001` |
| `prioridade` | `VERMELHO` |
| `necessidade` | `SALA_VERMELHA` |
| `eta_hospital` | `8 min` |

## Passos

1. Enviar o contrato C3 `pedido_pre_admissao` com os dados de entrada acima.
2. Verificar o leito selecionado e o estado atualizado na entidade `Leito`.
3. Verificar os atributos da `ReservaLeito` criada e o valor de `expira_em`.
4. Verificar o `AlertaTransferencia` exibido no painel da `EquipeReceptora`.
5. Verificar o registro gerado em `EventoPendenteG6` para o contrato C5.

## Resultado esperado

- O leito `SV-01` (maior tempo ocioso) muda de `VAGO` para `RESERVADO`; `SV-02` permanece `VAGO`.
- É criada uma `ReservaLeito` com `estado = ATIVA`, `prioridade = VERMELHO` (sem recálculo) e `expira_em = criada_em + 18 min` (`eta_hospital` 8 min + margem 10 min).
- O `AlertaTransferencia` aparece no painel do setor com `estado = PENDENTE`, contendo apenas `chamado_id`, `prioridade`, `setor_id` e `leito_id` (0 dados clínicos/pessoais adicionais — `RNF-G04-06`).
- O G3 recebe resposta `RESERVADO` com `leito_id = SV-01`, `setor = SALA_VERMELHA` e `expira_em`.
- É enfileirado 1 evento `LEITO_RESERVADO` (`resultado = SUCESSO`) para entrega assíncrona ao G6.

## Resultado obtido

`não executado` (validação sobre a modelagem prevista para o M3).

## Critério de aceite

Passou se e somente se o leito `SV-01` mudar atomicamente para `RESERVADO`, a resposta ao G3 contiver `expira_em` correto, o alerta respeitar a minimização LGPD e o evento C5 for gerado sem bloquear a resposta.

## Observações

Rastreia diretamente `DS-G04-01` (`diagramas/seq-reservar-leito.puml`), `ADR-G04-01` e `ADR-G04-02`.
