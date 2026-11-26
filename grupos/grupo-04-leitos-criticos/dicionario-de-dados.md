# Dicionário de Dados — G4 Leitos Críticos e Reserva

> Artefato de **E4 · Modelagem Estrutural**, alinhado ao diagrama [`diagramas/classes-dominio.puml`](diagramas/classes-dominio.puml), ao glossário do projeto (`docs/glossario-base.md`) e aos contratos em `docs/arquitetura-modular.md`.

---

## Princípios de modelagem adotados

1. **Fronteira de propriedade:** entidades pertencentes a outros módulos (`Chamado` do G1, `Prontuario`/`PreAdmissao` do G3 e `EventoAuditoria` do G6) **não são duplicadas** no G4; são referenciadas apenas por identificador (`chamado_id`, `prontuario_id`).
2. **Minimização de dados (LGPD / `RNF-G04-06`):** `ReservaLeito` e `AlertaTransferencia` armazenam **0 campos de dados pessoais ou clínicos** do paciente além de `chamado_id`, `prontuario_id` e `prioridade`.
3. **Sem recálculo de prioridade:** `prioridade` é recebida no contrato C3 (`pedido_pre_admissao`) e preservada intacta pelo G4.

---

## Classes de domínio

### 1. `Setor`
Representa uma unidade crítica do hospital gerenciada pelo módulo (Sala Vermelha ou Observação).

| Atributo | Tipo | Obrigatório | Restrições / Domínio | Significado |
| --- | --- | --- | --- | --- |
| `setor_id` | UUID | sim | único, imutável | Identificador único do setor |
| `nome` | String | sim | único, 3–80 caracteres | Nome operacional exibido nos painéis (ex.: "Sala Vermelha — Adulto") |
| `tipo` | `TipoSetor` | sim | `SALA_VERMELHA`, `OBSERVACAO` | Classificação crítica compatível com o campo `necessidade` do contrato C3 |
| `capacidade` | Integer | sim | `>= 1` | Quantidade nominal de leitos operacionais do setor |
| `ativo` | Boolean | sim | padrão `true` | Indica se o setor está habilitado para receber reservas |

- **Relacionamentos:** compõe `1..*` instâncias de `Leito`; recebe `0..*` instâncias de `AlertaTransferencia` exibidas no painel do setor.
- **Requisito/Indicador:** suporta `RF-G04-01` e o indicador do G6 *Ocupação de Sala Vermelha* (`leitos ocupados ÷ leitos operacionais`).

---

### 2. `Leito`
Recurso hospitalar crítico individualizado com ciclo de vida explícito. Nunca é modelado como contador inteiro.

| Atributo | Tipo | Obrigatório | Restrições / Domínio | Significado |
| --- | --- | --- | --- | --- |
| `leito_id` | UUID | sim | único, imutável | Identificador único do leito |
| `codigo` | String | sim | único por setor (ex.: `SV-01`, `OBS-03`) | Código físico visível à equipe assistencial |
| `estado` | `EstadoLeito` | sim | `VAGO`, `RESERVADO`, `OCUPADO`, `EM_HIGIENIZACAO`, `BLOQUEADO` | Estado atual do leito na máquina de estados |
| `estado_anterior` | `EstadoLeito` | não | preenchido em transições | Último estado antes da transição atual (apoia auditoria e desbloqueio) |
| `atualizado_em` | DateTime | sim | UTC, timestamp da transição | Data/hora da última mudança de estado |

- **Restrições de negócio:**
  - `RN-05`: um `Leito` só pode ter **no máximo 1** `ReservaLeito` com estado `ATIVA`.
  - `RN-06`: leito em `EM_HIGIENIZACAO` ou `BLOQUEADO` nunca é ofertado em consultas de disponibilidade (`estaDisponivel() == false`).
  - Um leito em `OCUPADO` não pode ser bloqueado diretamente (`UC-G04-09`, exceção E1).
- **Relacionamentos:** pertence a `1` `Setor`; associa-se a `0..*` `ReservaLeito`, `0..*` `Higienizacao` e `0..*` `BloqueioLeito`.

---

### 3. `ReservaLeito`
Vínculo temporário, transacional e com prazo de expiração entre um chamado pré-hospitalar e um leito crítico.

| Atributo | Tipo | Obrigatório | Restrições / Domínio | Significado |
| --- | --- | --- | --- | --- |
| `reserva_id` | UUID | sim | único, imutável | Identificador único da reserva |
| `chamado_id` | UUID | sim | referência externa (dono: G1); único entre reservas `ATIVA` | Chamado ao qual o leito está reservado |
| `prontuario_id` | UUID | sim | referência externa (dono: G3) | Ficha pré-hospitalar originadora do pedido C3 |
| `prioridade` | `PrioridadeManchester` | sim | herdada do G1 via C3; imutável no G4 | Classificação de risco do paciente |
| `estado` | `EstadoReserva` | sim | `ATIVA`, `CONCLUIDA`, `CANCELADA`, `EXPIRADA` | Situação atual da reserva |
| `criada_em` | DateTime | sim | UTC | Instante de criação da reserva |
| `expira_em` | DateTime | sim | `> criada_em`; calculado conforme `ADR-G04-01` | Prazo-limite para chegada do paciente antes da expiração automática |
| `encerrada_em` | DateTime | não | obrigatório quando `estado != ATIVA` | Instante de ocupação, cancelamento ou expiração |
| `motivo_encerramento` | String | não | obrigatório se `CANCELADA` | Justificativa do cancelamento (ex.: "leito bloqueado", "desvio de hospital") |
| `responsavel_id` | UUID | não | obrigatório em cancelamento ou ocupação manual | Identificador do ator autenticado que encerrou a reserva |

- **Restrições de negócio:**
  - `RN-05`: unicidade de reserva `ATIVA` por `leito_id` e por `chamado_id`.
  - Idempotência (`UC-G04-02`, fluxo A3): novo pedido de pré-admissão para um `chamado_id` que já possui reserva `ATIVA` retorna a reserva existente.
  - `RNF-G04-06`: proibido incluir nome, CPF, queixa ou sinais vitais do paciente.
- **Relacionamentos:** vinculada a `1` `Leito`; origina `0..1` `AlertaTransferencia`.

---

### 4. `AlertaTransferencia`
Notificação operacional exibida no painel da `EquipeReceptora` para preparar o setor antes da chegada da viatura.

| Atributo | Tipo | Obrigatório | Restrições / Domínio | Significado |
| --- | --- | --- | --- | --- |
| `alerta_id` | UUID | sim | único, imutável | Identificador único do alerta |
| `chamado_id` | UUID | sim | referência externa (dono: G1) | Identifica a ocorrência a caminho |
| `prioridade` | `PrioridadeManchester` | sim | herdada de `ReservaLeito` | Destaque visual de urgência no painel do setor |
| `estado` | `EstadoAlerta` | sim | `PENDENTE`, `CIENTE`, `ENCERRADO` | Situação do alerta no painel |
| `emitido_em` | DateTime | sim | UTC | Momento da geração do alerta (`RNF-G04-03`: p95 ≤ 5 s) |
| `ciente_em` | DateTime | não | obrigatório quando `estado == CIENTE` | Horário em que a equipe confirmou ciência |
| `ciente_por` | UUID | não | obrigatório quando `estado == CIENTE` | Integrante da `EquipeReceptora` que acusou recebimento |

- **Relacionamentos:** gerado por `1` `ReservaLeito`; exibido em `1` `Setor`.

---

### 5. `Higienizacao`
Registro formal do ciclo de descontaminação/limpeza de um leito após a saída do paciente, antes de devolvê-lo ao estado `VAGO`.

| Atributo | Tipo | Obrigatório | Restrições / Domínio | Significado |
| --- | --- | --- | --- | --- |
| `higienizacao_id` | UUID | sim | único, imutável | Identificador único do registro |
| `iniciada_em` | DateTime | sim | UTC | Momento em que o leito foi liberado (`OCUPADO → EM_HIGIENIZACAO`) |
| `concluida_em` | DateTime | não | `>= iniciada_em`; obrigatório se `resultado != EM_ANDAMENTO` | Momento da conclusão ou reprovação |
| `resultado` | `ResultadoHigienizacao` | sim | `EM_ANDAMENTO`, `APROVADA`, `REPROVADA` | Parecer final da higienização |
| `responsavel_id` | UUID | sim | perfil `EquipeReceptora` (`ADR-G04-03`) | Usuário que registrou a etapa |
| `observacao` | String | não | obrigatório se `REPROVADA` | Justificativa em caso de reprovação ou risco sanitário |

- **Restrições de negócio:**
  - `RN-06` / `RF-G04-10`: o leito só retorna a `VAGO` se `resultado == APROVADA`. Se `REPROVADA`, o leito transita para `BLOQUEADO` (`UC-G04-09`).
- **Relacionamentos:** pertence a `1` `Leito`.

---

### 6. `BloqueioLeito`
Registro de indisponibilização temporária de um leito por risco sanitário, manutenção ou falta de equipamento.

| Atributo | Tipo | Obrigatório | Restrições / Domínio | Significado |
| --- | --- | --- | --- | --- |
| `bloqueio_id` | UUID | sim | único, imutável | Identificador único do bloqueio |
| `motivo` | `MotivoBloqueio` | sim | lista parametrizável (`RNF-G04-08`) | Categoria do bloqueio |
| `descricao` | String | não | obrigatório quando `motivo == OUTRO` | Detalhamento operacional do bloqueio |
| `estado_origem` | `EstadoLeito` | sim | `VAGO`, `RESERVADO` ou `EM_HIGIENIZACAO` | Estado do leito antes do bloqueio; define o retorno no desbloqueio (`UC-G04-09` A2) |
| `bloqueado_por` | UUID | sim | perfil autorizado (`EquipeReceptora`) | Responsável pelo bloqueio |
| `bloqueado_em` | DateTime | sim | UTC | Instante do bloqueio |
| `desbloqueado_por` | UUID | não | obrigatório após desbloqueio | Responsável pelo desbloqueio (`UC-G04-10`) |
| `desbloqueado_em` | DateTime | não | `>= bloqueado_em` | Instante do desbloqueio |

- **Relacionamentos:** pertence a `1` `Leito`.

---

### 7. `ParametroOperacional`
Configuração dinâmica de regras operacionais do G4 sem necessidade de alteração de código ou deploy (`RNF-G04-08`).

| Atributo | Tipo | Obrigatório | Restrições / Domínio | Significado |
| --- | --- | --- | --- | --- |
| `chave` | String | sim | único (ex.: `prazo_expiracao_padrao_min`, `margem_eta_min`, `motivos_bloqueio`) | Identificador do parâmetro |
| `valor` | String | sim | não vazio; validado conforme tipo esperado | Valor vigente (ex.: `"15"`, `"5"`) |
| `descricao` | String | sim | — | Explicação operacional do parâmetro |
| `atualizado_em` | DateTime | sim | UTC | Data da última alteração |
| `atualizado_por` | UUID | sim | perfil administrativo autorizado | Responsável pela última alteração |

---

### 8. `EventoPendenteG6`
Estrutura de buffer de saída para o contrato **C5 (`evento_sensivel`)**, garantindo entrega eventual sem bloquear operações críticas caso o G6 esteja indisponível (`ADR-G04-02`, `RNF-G04-04`).

| Atributo | Tipo | Obrigatório | Restrições / Domínio | Significado |
| --- | --- | --- | --- | --- |
| `evento_id` | UUID | sim | único, imutável (chave de idempotência C5) | Identificador único do evento |
| `tipo` | String | sim | conforme contrato C5 (ex.: `RESERVA`, `ACESSO`) | Categoria do evento sensível |
| `ator_id` | UUID | sim | — | Usuário ou sistema (`Temporizador`) que executou a ação |
| `papel` | String | sim | glossário (`EquipeReceptora`, `MedicoRegulador`, `Sistema`) | Papel do ator no momento do fato |
| `entidade` | String | sim | `Leito` ou `ReservaLeito` | Tipo da entidade afetada |
| `entidade_id` | UUID | sim | — | Identificador do leito ou reserva afetada |
| `ocorrido_em` | DateTime | sim | UTC, horário do fato (não do reenvio) | Timestamp original da transição |
| `resultado` | String | sim | `SUCESSO`, `NEGADO`, `ERRO` | Desfecho da operação |
| `justificativa` | String | não | obrigatório em exceções, cancelamentos e bloqueios | Motivo operacional registrado |
| `enviado` | Boolean | sim | padrão `false` | Indica se o G6 confirmou o recebimento do evento |

---

## Enumerações

| Enumeração | Valores | Origem / Referência |
| --- | --- | --- |
| `TipoSetor` | `SALA_VERMELHA`, `OBSERVACAO` | Premissa P1 e campo `necessidade` do contrato C3 |
| `EstadoLeito` | `VAGO`, `RESERVADO`, `OCUPADO`, `EM_HIGIENIZACAO`, `BLOQUEADO` | `docs/temas-grupos.md` (G4) e `diagramas/estados-leito.puml` |
| `EstadoReserva` | `ATIVA`, `CONCLUIDA`, `CANCELADA`, `EXPIRADA` | `UC-G04-02`, `UC-G04-05`, `UC-G04-06`, `UC-G04-07` |
| `EstadoAlerta` | `PENDENTE`, `CIENTE`, `ENCERRADO` | `RF-G04-06`, `UC-G04-02`, `UC-G04-05` |
| `ResultadoHigienizacao` | `EM_ANDAMENTO`, `APROVADA`, `REPROVADA` | `RF-G04-10`, `UC-G04-08` |
| `MotivoBloqueio` | `MANUTENCAO`, `RISCO_SANITARIO`, `FALTA_EQUIPAMENTO`, `OUTRO` | `RF-G04-11`, `UC-G04-09`, `RNF-G04-08` |
| `PrioridadeManchester` | `VERMELHO`, `LARANJA`, `AMARELO`, `VERDE`, `AZUL` | `docs/glossario-base.md` e contrato C1/C3 |
