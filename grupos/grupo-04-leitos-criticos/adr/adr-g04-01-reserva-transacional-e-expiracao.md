# ADR-G04-01 — Reserva transacional com critério de seleção e prazo de expiração parametrizável

## Status

`aceito`

Data: 2026-11-26
Decisores: Luan Carlos de Sousa, José Guilherme de Souza Costa Alencar, Kerlon de Moura Marinho

## Contexto

Em horários de pico, duas solicitações de pré-admissão (`C3 pedido_pre_admissao`) podem chegar simultaneamente para o último leito vago da Sala Vermelha ou Observação (`RN-05`, `RNF-G04-02`: 0 reservas duplicadas em 50 requisições concorrentes). Além disso, o controle informal por telefone não define por quanto tempo um leito permanece retido caso a viatura atrase ou desvie, nem qual leito deve ser escolhido quando há mais de uma vaga no setor (pendência registrada no M1 para `UC-G04-02` passos 3 e 5).

## Decisão

O sistema modela `ReservaLeito` como uma **operação atômica com verificação de conflito na criação** (no máximo 1 reserva `ATIVA` por `leito_id` e por `chamado_id`), escolhe o leito vago pelo critério determinístico de **maior tempo ocioso no setor** (desempate por `codigo`) e calcula `expira_em = agora + eta_hospital + margem_eta` (com margem inicial parametrizável de 10 min e prazo padrão de 15 min via `ParametroOperacional` caso `eta_hospital` esteja ausente ou zerado).

## Alternativas consideradas

| Alternativa | Prós | Contras | Por que foi descartada |
| --- | --- | --- | --- |
| A · Controle manual pela recepção sem trava transacional | Simples de operar em baixo volume | Permite reserva dupla sob concorrência | Viola `RN-05` e `RNF-G04-02` |
| B · Prazo fixo em código (ex.: 15 min sem considerar `eta_hospital`) | Cálculo trivial | Expira reservas de viaturas distantes ou retém leito demais para viaturas próximas; exige deploy para ajustar | Desperdiça o campo `eta_hospital` do contrato C3 e viola `RNF-G04-08` |
| C · Realocação automática para outro setor sem confirmação | Reduz taxa de negativa imediata | Pode alocar paciente de Sala Vermelha em Observação sem aval clínico | Contraria o princípio de que decisão clínica é do `MedicoRegulador` (`docs/arquitetura-modular.md`) |

## Consequências

**Positivas**

- Elimina condição de corrida e garante `RN-05` tanto por leito quanto por chamado.
- Alinha o prazo de retenção do leito ao tempo real previsto de chegada (`eta_hospital`), reduzindo leitos ociosos.
- Permite ajustar margem e prazo padrão sem alteração de código (`RNF-G04-08`).

**Negativas / custos**

- Exige controle de concorrência (trava transacional) na tabela/entidade `Leito` e `ReservaLeito`, adicionando pequena contenção sob pico (mitigada pelo alvo `p95 <= 2 s` de `RNF-G04-01`).
- Requer manutenção dos parâmetros operacionais pela gestão do hospital.

**Impactos em outros grupos**

- **G3:** nenhum campo novo no contrato C3; confirma o uso efetivo de `eta_hospital` já previsto em `docs/arquitetura-modular.md`.
- **G6:** recebe os eventos `LEITO_RESERVADO`, `LEITO_NEGADO` e `RESERVA_EXPIRADA` via contrato C5.

## Como verificar

- Diagrama de classes [`diagramas/classes-dominio.puml`](../diagramas/classes-dominio.puml) (`ReservaLeito`, `ParametroOperacional`).
- Diagrama de sequência [`diagramas/seq-excecao-reserva-e-indisponibilidade.puml`](../diagramas/seq-excecao-reserva-e-indisponibilidade.puml) (`DS-G04-02`).
- Caso de teste de concorrência [`testes/ct-g04-02-concorrencia-reserva-dupla.md`](../testes/ct-g04-02-concorrencia-reserva-dupla.md) (`CT-G04-02`).
