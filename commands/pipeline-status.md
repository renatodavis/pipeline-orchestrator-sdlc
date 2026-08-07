---
description: Mostra o estado atual de uma issue no pipeline (fase, HITL pendente, links relevantes).
argument-hint: <issue-id>
---

Sem invocar nenhum subagent, monte um snapshot do estado da issue **$ARGUMENTS**:

1. Rode `gh issue view $ARGUMENTS --json title,state,labels,comments,url`.
2. Identifique a fase atual olhando labels e comentários:
   - Sem label de SDD → provável Fase 1 ou 2.
   - Label `sdd-approved` + sem PR aberto → Fase 3.
   - PR aberto sem approval humano → Fase 4 (HITL 2).
   - PR mergeado sem label `qa-passed`/`qa-failed` → Fase 5 (HITL 3).
   - Label `qa-passed` + issue fechada → Done.
3. Se existir `pipeline-events.jsonl` no repo, mostre os últimos 5 eventos dessa `issue_id`:
   ```bash
   grep "\"issue_id\":\"$ARGUMENTS\"" pipeline-events.jsonl | tail -n 5
   ```
4. Liste os próximos passos concretos: qual HITL está pendente, quem precisa agir, comando `gh` para desbloquear.

Não escreva nada no Git — este comando é somente leitura.
