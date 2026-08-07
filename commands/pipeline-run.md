---
description: Dispara o pipeline SDLC completo para uma issue (triagem → SDD → implementação → review → QA).
argument-hint: <issue-id-ou-url>
---

Invoque o subagent `pipeline-orchestrator` para tocar a work item **$ARGUMENTS** do início ao fim, respeitando as três paradas HITL (SDD, Code Review, QA).

Antes de invocar, confirme que:
1. `gh auth status` está autenticado.
2. Estamos no repositório certo (`gh repo view` mostra o repo esperado).

Se a issue já estiver parada em uma HITL, o orquestrador vai detectar o estado atual pelo Git e continuar de onde parou — não recomece do zero.
