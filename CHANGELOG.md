# Changelog

Todas as mudanças notáveis deste plugin são documentadas aqui.

O formato segue [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/) e o versionamento adota [SemVer](https://semver.org/lang/pt-BR/).

## [Unreleased]

## [0.1.0] - 2026-08-06

### Adicionado
- Skill `pipeline-orchestrator-factory` — fábrica de subagents SDLC com descoberta guiada, templates parametrizáveis e evolução iterativa.
- Subagents pré-configurados com defaults GitHub + `gh` CLI:
  - `pipeline-orchestrator` — governa o fluxo, impõe HITL, emite eventos.
  - `triage-pm` — decide se a Work Item está pronta para especificação.
  - `spec-builder` — lê o código e escreve o SDD.
  - `coder-dev` — implementa o SDD aprovado, abre PR.
  - `code-reviewer` — posta findings automáticos no PR.
- Slash commands: `/pipeline-run`, `/pipeline-factory`, `/pipeline-status`.
- Contrato de eventos JSON `state_transition` versionado (`schema_version: 1`).
- Documentação de padrões HITL (SDD, Code Review, QA) e integração GitLab/GitHub.
- Trilha de eventos de exemplo (`assets/event-log-example.jsonl`).

[Unreleased]: https://github.com/renatodavis/pipeline-orchestrator-sdlc/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/renatodavis/pipeline-orchestrator-sdlc/releases/tag/v0.1.0
