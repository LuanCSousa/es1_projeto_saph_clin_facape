# ADR-G04-02 — Emissão assíncrona com fila local de reenvio para eventos sensíveis ao G6

## Status

`aceito`

Data: 2026-11-26
Decisores: Luan Carlos de Sousa, José Guilherme de Souza Costa Alencar, Kerlon de Moura Marinho

## Contexto

A regra de negócio `RN-09` e o requisito `RF-G04-12` exigem que cada transição de estado de `Leito` e `ReservaLeito` emita um evento sensível ao módulo G6 (contrato `C5 evento_sensivel`). Ao mesmo tempo, o atendimento pré-hospitalar crítico opera 24h com custo assimétrico de erro: o requisito `RNF-G04-04` estabelece disponibilidade `>= 99,5%` ao mês e determina que uma indisponibilidade temporária do G6 não pode bloquear a reserva nem a ocupação de um leito crítico (`UC-G04-02`, exceção E4; `docs/arquitetura-modular.md`, seção 6).

## Decisão

O módulo G4 grava cada evento do contrato C5 em uma **fila local de saída (`EventoPendenteG6`) dentro da mesma transação da mudança de estado** e o envia de forma assíncrona ao G6 com reenvio automático preservando o `evento_id` imutável e o timestamp `ocorrido_em` original do fato.

## Alternativas consideradas

| Alternativa | Prós | Contras | Por que foi descartada |
| --- | --- | --- | --- |
| A · Chamada síncrona bloqueante ao G6 antes de confirmar a reserva | Garante entrega imediata quando o G6 está no ar | Queda ou lentidão do G6 paralisa reservas de Sala Vermelha e viola `RNF-G04-01` e `RNF-G04-04` | Inaceitável em operação crítica de urgência; viola a regra de falha previsível de `arquitetura-modular.md` |
| B · Envio "melhor esforço" (fire-and-forget sem persistência local) | Baixa latência e implementação simples | Perde eventos sensíveis se o G6 estiver fora do ar no instante da operação | Viola `RN-09` (perda de prova legal) e compromete indicadores do G6 |
| C · Criar trilha de auditoria própria completa no G4 | Elimina dependência do G6 | Duplica responsabilidade arquitetural | Proibido pelas regras de coerência (`docs/requisitos-entrega-av2.md`, seção 6) |

## Consequências

**Positivas**

- Mantém a operação de leitos resiliente e dentro da latência `p95 <= 2 s` (`RNF-G04-01`) mesmo com falha do G6.
- Garante entrega eventual sem perda de eventos, respeitando idempotência por `evento_id` e ordem temporal por `ocorrido_em`.
- Mantém a fronteira limpa: `EventoPendenteG6` atua apenas como buffer de transporte, não como trilha de auditoria consultável.

**Negativas / custos**

- Introduz atraso eventual na visibilidade de eventos no painel do G6 durante janelas de indisponibilidade.
- Exige rotina de reenvio e monitoramento de eventos pendentes no G4.

**Impactos em outros grupos**

- **G6:** nenhum contrato alterado; depende apenas da idempotência por `evento_id` e do uso de `ocorrido_em` (já previstos na seção 6 de `docs/arquitetura-modular.md`).

## Como verificar

- Diagrama de classes [`diagramas/classes-dominio.puml`](../diagramas/classes-dominio.puml) (`EventoPendenteG6`).
- Diagrama de sequência [`diagramas/seq-excecao-reserva-e-indisponibilidade.puml`](../diagramas/seq-excecao-reserva-e-indisponibilidade.puml) (`DS-G04-02`, Cenário 2).
- Fluxo de exceção E4 de [`casos-de-uso/uc-g04-02-reservar-leito.md`](../casos-de-uso/uc-g04-02-reservar-leito.md).
