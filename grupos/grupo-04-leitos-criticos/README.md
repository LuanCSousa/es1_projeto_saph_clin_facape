# G4 — Leitos Críticos e Reserva (M3 · Recursos críticos)

**Projeto:** SAPH-Clin · Engenharia de Software I · FACAPE 2026.2
**Integrantes:** Luan Carlos de Sousa, José Guilherme de Souza Costa Alencar, Kerlon de Moura Marinho
**Estado atual:** M2 v1.0 — modelagem estrutural (E4), comportamental (E5), ADRs e testes parciais (E6)

---

## E1 · Visão e Escopo do Módulo

### Problema que o módulo resolve
Hoje a equipe do hospital só descobre que há um paciente crítico a caminho quando ele chega. O leito não é separado a tempo, o que atrasa a abertura de leito crítico. O módulo G4 mantém o **estado confiável de cada leito crítico** (Sala Vermelha e Observação) e **reserva um leito para o chamado antes da chegada**, avisando a equipe receptora.

### Onde começa e onde termina
| | |
|---|---|
| **Começa** | quando o G3 envia o pedido de pré-admissão (`PreAdmissao`) com `chamado_id` e a prioridade clínica definida pelo G1 |
| **Termina** | quando o leito foi ocupado pelo paciente e, depois da alta, higienizado e devolvido ao estado Vago (ou bloqueado com motivo) |

### Contrato do módulo
| Direção | Conteúdo | Com quem |
|---|---|---|
| Entrada | `PreAdmissao` (`chamado_id`, prioridade, setor requerido) | G3 |
| Saída | Resposta "leito reservado" (`leito_id`, setor, `expira_em`) ou "leito negado" (motivo) | G3 / MedicoRegulador |
| Saída | `AlertaTransferencia` (`chamado_id`, prioridade, setor, leito) | EquipeReceptora |
| Saída | Evento por transição de estado de leito e de reserva | G6 |

### Dentro do escopo
- Cadastro lógico de `Leito` e `Setor` (Sala Vermelha, Observação) e seus estados
- Consulta de disponibilidade; reserva, cancelamento e expiração de reserva
- Aceite do paciente (ocupação), liberação e ciclo de higienização
- Bloqueio e desbloqueio de leito com motivo
- Alerta à equipe receptora; emissão de eventos ao G6

### Fora do escopo
- Prontuário hospitalar definitivo e faturamento
- Cálculo ou alteração da prioridade clínica (é do **G1**)
- Escolha de viatura e deslocamento (**G2**); ficha pré-hospitalar (**G3**)
- Estoque e fármacos controlados (**G5**)
- Trilha de auditoria própria (o G4 só **emite** eventos; a trilha é do **G6**)
- Gestão de leitos comuns/enfermaria (só leitos críticos)

### Premissas adotadas
- **P1.** O hospital tem dois setores críticos: Sala Vermelha e Observação.
- **P2.** O `chamado_id` pertence ao G1; o G4 só o referencia.
- **P3.** A `PreAdmissao` chega com prioridade já definida; o G4 não a recalcula.
- **P4.** O prazo de expiração da reserva é parametrizável (valor inicial proposto: 15 min, a validar em `ADR-G04-01`).
- **P5.** A autenticação e os perfis de usuário são providos pelo hospital; o G4 só os consome.
- **P6.** A equipe de higienização não consta no glossário; no M1 suas ações são feitas pela `EquipeReceptora` (decisão a registrar em ADR).

### Perguntas em aberto
- Qual setor atende qual faixa de prioridade? (regra de seleção de setor)
- Se o setor requerido estiver lotado, o sistema só **sugere** outro setor ou reserva sozinho?
- Quem confirma a "ciência" do alerta na porta (enfermagem do setor ou recepção)?

## Decisões arquiteturais (ADRs)
- [`ADR-G04-01`](adr/adr-g04-01-reserva-transacional-e-expiracao.md) — Reserva transacional com verificação atômica de conflito (`RN-05`), seleção determinística de leito vago e `expira_em` parametrizável a partir de `eta_hospital`.
- [`ADR-G04-02`](adr/adr-g04-02-emissao-assincrona-eventos-g6.md) — Emissão assíncrona com fila local (`EventoPendenteG6`) para eventos C5 ao G6, evitando bloqueio de atendimento crítico em caso de indisponibilidade (`RNF-G04-04`).
- [`ADR-G04-03`](adr/adr-g04-03-higienizacao-e-bloqueio-com-reserva.md) — Registro de higienização atribuído à `EquipeReceptora` (glossário oficial) e bloqueio de leito reservado com cancelamento explícito + notificação ao G3/`MedicoRegulador`.

## Artefatos do módulo
- **E2 Requisitos:** [`requisitos.md`](requisitos.md)
- **E3 Casos de uso:** [`casos-de-uso/`](casos-de-uso/) · [`diagramas/uc-geral.puml`](diagramas/uc-geral.puml)
- **E4 Modelagem estrutural:** [`diagramas/classes-dominio.puml`](diagramas/classes-dominio.puml) · [`dicionario-de-dados.md`](dicionario-de-dados.md)
- **E5 Modelagem comportamental:** [`diagramas/seq-reservar-leito.puml`](diagramas/seq-reservar-leito.puml) · [`diagramas/seq-excecao-reserva-e-indisponibilidade.puml`](diagramas/seq-excecao-reserva-e-indisponibilidade.puml) · [`diagramas/seq-ciclo-ocupacao-bloqueio.puml`](diagramas/seq-ciclo-ocupacao-bloqueio.puml) · [`diagramas/estados-leito.puml`](diagramas/estados-leito.puml) · [`diagramas/atividades-reserva-leito.puml`](diagramas/atividades-reserva-leito.puml)
- **E6 ADRs, rastreabilidade e testes:** [`adr/`](adr/) · [`matriz-rastreabilidade.md`](matriz-rastreabilidade.md) · [`testes/`](testes/)

## Estado da entrega

| Artefato | M1 (Aula 18) | M2 (Aula 24) | M3 (Aula 27) |
| --- | --- | --- | --- |
| E1 Visão e escopo | entregue (v1.0) | — | — |
| E2 Catálogo RF/RNF | entregue (v1.0) | — | — |
| E3 Casos de uso | entregue (v1.0) | — | — |
| E4 Classes + dicionário | — | entregue (v1.0) | — |
| E5 Sequência + estados + atividades | — | entregue (v1.0) | — |
| E6 ADRs, matriz e testes | — | entregue (parcial: 3 ADRs, 2 CTs, matriz M2) | pendente |
| E7 Protótipos + relatório | — | — | pendente |

## Relatório técnico
_(a ser elaborado no M3)_
