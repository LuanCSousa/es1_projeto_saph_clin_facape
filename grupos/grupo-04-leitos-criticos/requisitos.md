# Catálogo de Requisitos — G4 Leitos Críticos e Reserva

## Atores
| Ator | Papel no módulo |
|---|---|
| `EquipeReceptora` | consulta, aceita o paciente, libera, higieniza, bloqueia/desbloqueia e cancela |
| `MedicoRegulador` | consulta disponibilidade e cancela reserva de chamado |
| Módulo G3 (sistema) | envia a `PreAdmissao` e recebe a resposta |
| Módulo G6 (sistema) | recebe os eventos de transição |
| Temporizador (sistema) | dispara a expiração de reservas |

## Requisitos Funcionais
| ID | Ator | Descrição | Critério de aceite | UC |
|---|---|---|---|---|
| RF-G04-01 | EquipeReceptora, MedicoRegulador | Consultar a disponibilidade de leitos por setor, com o estado atual de cada leito | Lista os leitos do setor com estado; leito Reservado, Ocupado, Em Higienização ou Bloqueado nunca aparece como vago | UC-G04-01 |
| RF-G04-02 | Módulo G3 | Receber o pedido de pré-admissão e abrir a solicitação de reserva vinculada ao `chamado_id` | Pedido sem `chamado_id` ou sem prioridade é rejeitado; pedido repetido do mesmo chamado não cria segunda reserva | UC-G04-02 |
| RF-G04-03 | Sistema | Reservar um leito vago de setor compatível, mudando-o para Reservado e fixando `expira_em` | Após a reserva o leito não aparece como vago; `expira_em` = hora da reserva + prazo configurado | UC-G04-02 |
| RF-G04-04 | Sistema | Verificar conflito antes de reservar: um leito tem no máximo uma reserva ativa e um chamado tem no máximo uma reserva ativa | Duas solicitações simultâneas ao mesmo leito geram exatamente uma reserva; a outra recebe outro leito ou negativa | UC-G04-03 |
| RF-G04-05 | Sistema | Negar a reserva com motivo quando não há leito vago compatível | Resposta "negado" ao G3 com motivo (ex.: setor lotado); nenhum leito alterado | UC-G04-04 |
| RF-G04-06 | Sistema | Emitir `AlertaTransferencia` à EquipeReceptora ao confirmar a reserva | Alerta com `chamado_id`, prioridade, setor e leito aparece no painel do setor e fica pendente até a ciência da equipe | UC-G04-02 |
| RF-G04-07 | EquipeReceptora | Confirmar a chegada e ocupar o leito reservado (aceite do paciente na porta) | Reservado → Ocupado só com usuário identificado; registra responsável e hora | UC-G04-05 |
| RF-G04-08 | EquipeReceptora, MedicoRegulador | Cancelar reserva ativa informando o motivo | Reserva fica Cancelada, leito volta a Vago e o G3 é notificado | UC-G04-06 |
| RF-G04-09 | Temporizador | Expirar a reserva sem chegada dentro do prazo | Reserva ativa além de `expira_em` vira Expirada, leito volta a Vago; EquipeReceptora e G3 são notificados | UC-G04-07 |
| RF-G04-10 | EquipeReceptora | Liberar leito ocupado, enviá-lo a Em Higienização e registrar a `Higienizacao` ao concluir | Leito só volta a Vago após higienização registrada como concluída | UC-G04-08 |
| RF-G04-11 | EquipeReceptora | Bloquear leito com motivo obrigatório e desbloqueá-lo | Leito Bloqueado sai da disponibilidade; leito Ocupado não pode ser bloqueado; bloquear leito Reservado cancela a reserva e notifica o G3 | UC-G04-09, UC-G04-10 |
| RF-G04-12 | Sistema | Enviar ao G6 um evento por transição de estado de leito e de reserva | Evento com `leito_id`, estado anterior e novo, `chamado_id` (se houver), ator e hora | UC-G04-02 a UC-G04-10 |

## Requisitos Não Funcionais
| ID | Categoria | Descrição | Métrica e alvo |
|---|---|---|---|
| RNF-G04-01 | Desempenho | Resposta da reserva (confirmada ou negada) | p95 ≤ 2 s com até 20 req/min |
| RNF-G04-02 | Desempenho (concorrência) | Ausência de reserva duplicada em pico | 0 reservas duplicadas em teste com 50 solicitações simultâneas sobre o mesmo leito |
| RNF-G04-03 | Desempenho | Entrega do alerta à equipe receptora | p95 ≤ 5 s entre a confirmação da reserva e a exibição no painel |
| RNF-G04-04 | Disponibilidade | Consulta e reserva disponíveis; falha do G6 não bloqueia a reserva (eventos ficam em fila e são reenviados) | ≥ 99,5% ao mês (≈ 3,6 h de indisponibilidade por mês) |
| RNF-G04-05 | Segurança | Controle de acesso por perfil nas operações que alteram estado | 100% das alterações exigem usuário autenticado e autorizado; tentativas negadas geram evento ao G6 |
| RNF-G04-06 | Conformidade (LGPD) | Minimização de dados de saúde | `ReservaLeito` e `AlertaTransferencia` guardam 0 campos de dados pessoais ou clínicos além de `chamado_id` e prioridade |
| RNF-G04-07 | Usabilidade | Painel de leitos | Estado dos leitos de um setor em uma única tela, a ≤ 2 interações do painel inicial |
| RNF-G04-08 | Manutenibilidade | Prazo de expiração e motivos de bloqueio parametrizáveis | 0 alterações de código ou deploys para mudar o prazo ou a lista de motivos |
