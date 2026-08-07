---
description: Abre a skill-fábrica para (re)gerar ou customizar subagents do pipeline para o projeto atual.
argument-hint: [componente-opcional]
---

Ative a skill `pipeline-orchestrator-factory` para trabalhar sobre **$ARGUMENTS** (ou, se vazio, para revisar o pipeline inteiro).

Fluxo esperado:
1. A skill roda a descoberta (host Git, auth, stack, formato de work item, sinais HITL, sinks de evento).
2. Você decide quais componentes gerar/atualizar.
3. Os arquivos são escritos em `~/.claude/agents/` (global) — não no repositório atual, para não conflitar com os agents já instalados pelo plugin.

Use isso quando os defaults do plugin (GitHub + `gh` CLI + labels `sdd-approved`/`qa-passed`) não servirem para o seu projeto.
