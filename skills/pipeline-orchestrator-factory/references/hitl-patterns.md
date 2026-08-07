# Padrões HITL (Human-in-the-Loop)

Uma HITL bem feita tem três propriedades: (1) o orquestrador **para de verdade**; (2) o sinal de retomada é **inequívoco**; (3) a inferência do estado vem de **evento real do Git**, não de suposição.

## As três paradas

### HITL 1 — SDD aprovado
Momento: fim da Fase 2.
Quem aprova: Arquiteto / Tech Lead.
Sinais aceitáveis (escolher um por projeto):

- **Label na issue**: `SDD-Approved` aplicada pelo Arquiteto.
- **Comentário estruturado**: `/approve-sdd` ou `LGTM SDD` de usuário com role permitida.
- **Merge de MR de spec**: se o SDD é publicado como MR em `docs/sdd/`, o próprio merge é o sinal.

Rejeição: label `SDD-Rejected` ou comentário `/reject-sdd <motivo>`. Ao detectar, volte para Fase 2 com os comentários como input.

### HITL 2 — Code Review aprovado
Momento: fim da Fase 4.
Quem aprova: outro desenvolvedor humano (não o autor do PR).
Sinais aceitáveis:

- **GitLab**: `approvals_left == 0` na API do MR + estado `can_be_merged`.
- **GitHub**: review com state `APPROVED` de usuário com permissão + checks verdes.

Rejeição: review `CHANGES_REQUESTED` (GitHub) ou `unapprove` + comentário (GitLab). Ao detectar, deixe o CoderDev reagir aos comentários no próprio MR — **não** feche o MR nem crie novo ciclo automaticamente; espere o CoderDev ser reinvocado.

### HITL 3 — QA aprovado
Momento: Fase 5.
Quem aprova: QA humano validando comportamento contra critérios de aceite.
Sinais aceitáveis:

- Label `qa-passed` na issue.
- Comentário estruturado `/qa-approve` com referência aos critérios validados.
- Transição de status em sistema externo (Jira/Linear) — mais frágil, evite se possível.

Rejeição: label `qa-failed` + comentário descrevendo o bug reproduzível. Orquestrador volta para Fase 3 passando o comentário como feedback.

## Como implementar "parar de verdade"

O subagent/orquestrador emite `{status:"waiting_human"}` e **termina**. Nenhum loop de polling ativo. A retomada acontece porque:

1. O usuário reinvoca o orquestrador: `use pipeline-orchestrator para checar issue #42`.
2. O orquestrador lê o estado atual no Git e detecta o sinal.
3. Se sinal presente → avança. Se ausente → emite `waiting_human` de novo e para.

Alternativa (avançada, opcional): agendar checagem periódica via `mcp__scheduled-tasks__create_scheduled_task` a cada 15-30 min. Mas isso consome tokens — use só se o usuário pediu explicitamente monitoramento automático.

## Anti-padrões (nunca faça)

- **Sleep loop**: `while not approved: sleep(60)` queima contexto/tokens sem parar.
- **Auto-aprovação por tempo**: "se 24h sem review, aprova" — quebra a semântica de HITL.
- **Inferência frouxa**: "o dev comentou 'ok', deve estar aprovado" — só sinal exato conta.
- **Simular aprovação em teste**: se está testando, use uma issue de teste real e faça as ações reais no Git. Simular gera dependência de código de mock que vaza para produção.
