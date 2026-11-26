# ADR-G04-03 — Papel da EquipeReceptora na higienização e tratamento de bloqueio em leito reservado

## Status

`aceito`

Data: 2026-11-26
Decisores: Luan Carlos de Sousa, José Guilherme de Souza Costa Alencar, Kerlon de Moura Marinho

## Contexto

Duas ambiguidades operacionais ficaram em aberto no M1 e no documento de contexto (`docs/contexto-do-sistema.md`, pergunta 4; `README.md`, premissa P6):
1. O glossário oficial (`docs/glossario-base.md`) não define um ator separado de "Equipe de Higienização", exigindo aderência aos 6 atores padronizados.
2. Em `UC-G04-09`, quando um leito já `RESERVADO` precisa ser bloqueado por risco sanitário ou falha de equipamento antes de a viatura chegar, era necessário definir se o sistema realoca silenciosamente outro leito ou cancela a reserva e devolve a decisão ao G3 / `MedicoRegulador`.

## Decisão

O sistema atribui à **`EquipeReceptora`** o registro formal do início e da conclusão/reprovação da `Higienizacao` (`UC-G04-08`) e determina que o **bloqueio de um leito em estado `RESERVADO` cancela imediatamente a `ReservaLeito` com motivo `LEITO_BLOQUEADO` e notifica o G3 e o `MedicoRegulador`**, sem realizar realocação automática silenciosa.

## Alternativas consideradas

| Alternativa | Prós | Contras | Por que foi descartada |
| --- | --- | --- | --- |
| A · Criar um sétimo ator `EquipeHigienizacao` no módulo G4 | Separa operacionalmente limpeza de enfermagem | Diverge da lista fechada de atores do projeto | Viola a regra de coerência nº 3 (`docs/requisitos-entrega-av2.md`) |
| B · Realocar automaticamente para outro leito vago ao bloquear leito reservado | Reduz uma interação do regulador quando há vaga no mesmo setor | Altera o destino acordado com a equipe em trânsito sem confirmação explícita e pode esconder indisponibilidade do setor | Conflita com o princípio de que o sistema registra e notifica, enquanto a decisão assistencial pertence ao ator (`docs/arquitetura-modular.md`) |
| C · Impedir o bloqueio de leito reservado | Evita cancelar reserva em trânsito | Força o recebimento de paciente em leito contaminado ou sem equipamento | Risco assistencial grave; viola `RN-06` |

## Consequências

**Positivas**

- Mantém 100% de conformidade com os atores do glossário oficial sem alterar contratos de outros grupos.
- Garante resposta explícita e rastreável para a pergunta 4 de `docs/contexto-do-sistema.md` (viatura em trânsito com leito bloqueado: a reserva é cancelada, o G3/`MedicoRegulador` recebe alerta imediato e pode disparar novo `pedido_pre_admissao` ou desvio).
- Preserva a segurança sanitária (`RN-06`) ao permitir bloqueio imediato de leito reservado com registro de motivo e responsável.

**Negativas / custos**

- A `EquipeReceptora` assume o registro no sistema após a equipe física de limpeza finalizar o leito.
- O `MedicoRegulador` / G3 precisa acionar nova solicitação de reserva caso o leito original seja bloqueado durante o deslocamento.

**Impactos em outros grupos**

- **G3 / G2:** recebem notificação de cancelamento de reserva com motivo `LEITO_BLOQUEADO` para decidir nova reserva ou desvio.
- **G6:** recebe os eventos `RESERVA_CANCELADA` e `LEITO_BLOQUEADO` (ambos com `justificativa` preenchida).

## Como verificar

- Máquina de estados [`diagramas/estados-leito.puml`](../diagramas/estados-leito.puml) (transições `Reservado -> Bloqueado` e `EmHigienizacao -> Vago / Bloqueado`).
- Diagrama de sequência [`diagramas/seq-ciclo-ocupacao-bloqueio.puml`](../diagramas/seq-ciclo-ocupacao-bloqueio.puml) (`DS-G04-03`, Fluxos 2 e 3).
- Dicionário de dados [`dicionario-de-dados.md`](../dicionario-de-dados.md) (`Higienizacao`, `BloqueioLeito`).
