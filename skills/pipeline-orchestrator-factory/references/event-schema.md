# Contrato de Eventos JSON

Toda mudança de estado da pipeline vira uma linha JSON em stdout (JSON Lines / `.jsonl`). É esse fluxo que um dashboard futuro consome — sem ele, você tem um pipeline invisível.

## Schema v1

```json
{
  "event": "state_transition",
  "phase": "triage | spec | implementation | review | qa | done",
  "status": "in_progress | waiting_human | approved | rejected",
  "issue_id": "string",
  "message": "string curto (< 200 chars)",
  "ts": "ISO-8601 UTC, ex: 2026-08-06T14:32:00Z",
  "schema_version": 1
}
```

Campos opcionais permitidos (não quebram schema):
- `actor`: subagent que emitiu (`triage-pm`, `spec-builder`, ...)
- `refs`: `{ mr_url, sdd_url, branch }` — links úteis pro dashboard
- `duration_ms`: quando emitido no fim de fase

Eventos específicos de subagent (não são transições de estado, mas complementam a trilha):

```json
{"event":"triage_result","issue_id":"...","status":"ready|not_ready","missing":[...]}
{"event":"sdd_published","issue_id":"...","location":"...","open_questions":0}
{"event":"mr_opened","issue_id":"...","mr_url":"...","branch":"...","tests_pass":true}
{"event":"review_posted","issue_id":"...","mr_url":"...","blockers":0,"majors":1,"minors":3}
{"event":"scope_expansion_detected","issue_id":"...","details":"..."}
{"event":"blocked","issue_id":"...","reason":"..."}
```

## Por que cada campo importa

- **`phase` + `status`** = célula única na matriz de estado do dashboard. Não invente valor novo sem versionar.
- **`issue_id`** = chave para agrupar toda a trilha de uma work item. Sem ele, o dashboard não consegue montar timeline.
- **`ts`** ISO-8601 UTC = sortável lexicograficamente. Nunca use timezone local.
- **`schema_version`** = permite migrar dashboards sem quebrar consumidores antigos.

## Regras
- Uma linha JSON por evento. Nada de pretty-print.
- `message` é human-readable, não é estrutura — não faça o dashboard parsear ela.
- Emita **antes** e **depois** de cada fase. `waiting_human` conta como "depois" (a fase pausou).
- Se persistir também em arquivo, use `pipeline-events.jsonl` na raiz do projeto (ou path que o usuário definiu). Sempre **append**, nunca sobrescreva.

## Exemplo mínimo de trilha (issue #42, happy path)

Ver `assets/event-log-example.jsonl` para trilha completa.
