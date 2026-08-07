# pipeline-orchestrator-sdlc

Pipeline SDLC **spec-driven** para Claude Code, com três paradas obrigatórias para humanos (HITL), integração GitHub/GitLab e eventos JSON estruturados prontos para um dashboard.

Ao invés de um único agente que "faz tudo e às vezes acerta", este plugin instala um **orquestrador** que coordena quatro especialistas — cada um com escopo travado — e uma **skill-fábrica** que regenera esses especialistas customizados para o seu projeto.

```
                                                         ┌─── HITL 1: SDD approved
Work Item ─▶ TriagePM ─▶ SpecBuilder ─┼─▶ CoderDev ─▶ CodeReviewer ─┼─▶ QA ─▶ Done
                                                         └──────────────────────────────── HITL 2: PR approved      │
                                                                                                                    HITL 3: qa-passed
```

## Instalação

Em um terminal do Claude Code, dois passos:

```
/plugin marketplace add renatodavis/pipeline-orchestrator-sdlc
/plugin install pipeline-orchestrator-sdlc@pipeline-orchestrator-sdlc
```

O primeiro comando registra este repositório como um marketplace (fonte de plugins). O segundo instala o plugin dessa fonte. Nome final: `<plugin-name>@<marketplace-name>` — nesse caso os dois coincidem porque o repo hospeda um único plugin.

Depois, para manter atualizado ou remover:

```
/plugin marketplace update pipeline-orchestrator-sdlc     # busca versões novas
/plugin uninstall pipeline-orchestrator-sdlc              # remove
```

Nenhuma dependência de Node/npm — o plugin é 100% arquivos versionados neste repo.

**Pré-requisitos operacionais** (do seu lado, não do plugin):

- [`gh`](https://cli.github.com/) autenticado (`gh auth login`). Os defaults deste plugin assumem GitHub — se o seu fluxo é GitLab ou outro, rode `/pipeline-factory` para regenerar os agents customizados.
- Repositório com Issues habilitadas e política de PRs com approval (para o HITL 2 ter algo que ler).

## Uso

Três slash commands ficam disponíveis assim que o plugin instala:

| Comando | O que faz |
|---|---|
| `/pipeline-run <issue>` | Toca a issue do início ao fim, parando em cada HITL. |
| `/pipeline-status <issue>` | Snapshot read-only: em que fase está, qual HITL está pendente, próximo passo. |
| `/pipeline-factory [componente]` | Abre a fábrica para (re)gerar subagents customizados ao seu projeto. |

Alternativamente, invoque diretamente pelo Task tool:

> use pipeline-orchestrator para tocar a issue #42

### Convenções default (podem ser trocadas via factory)

| Sinal | Default |
|---|---|
| HITL 1 (SDD) | label `sdd-approved` na issue |
| HITL 2 (Code Review) | review nativo `APPROVED` + checks verdes |
| HITL 3 (QA) | label `qa-passed` na issue |
| Persistência de eventos | stdout (opcional: append em `pipeline-events.jsonl`) |

## Contrato de eventos

Toda transição de estado emite uma linha JSON em stdout:

```json
{"event":"state_transition","phase":"spec","status":"waiting_human","issue_id":"42","message":"SDD publicado, aguardando HITL 1","ts":"2026-08-06T14:11:03Z","schema_version":1}
```

Um dashboard futuro consome esse stream (`schema_version: 1` protege contra breaking changes). Veja [`skills/pipeline-orchestrator-factory/references/event-schema.md`](skills/pipeline-orchestrator-factory/references/event-schema.md) para o contrato completo e [`skills/pipeline-orchestrator-factory/assets/event-log-example.jsonl`](skills/pipeline-orchestrator-factory/assets/event-log-example.jsonl) para uma trilha ponta a ponta.

## O que está no plugin

```
pipeline-orchestrator-sdlc/
├── .claude-plugin/
│   ├── plugin.json                   Manifesto do plugin
│   └── marketplace.json              Manifesto do marketplace (single-plugin)
├── agents/                           5 subagents pré-configurados (GitHub + gh CLI)
│   ├── pipeline-orchestrator.md
│   ├── triage-pm.md
│   ├── spec-builder.md
│   ├── coder-dev.md
│   └── code-reviewer.md
├── commands/                         3 slash commands
│   ├── pipeline-run.md
│   ├── pipeline-status.md
│   └── pipeline-factory.md
├── skills/pipeline-orchestrator-factory/
│   ├── SKILL.md                      Fábrica que regenera os subagents customizados
│   ├── references/                   Templates + docs de HITL, eventos, integração Git
│   └── assets/event-log-example.jsonl
├── CHANGELOG.md
├── LICENSE                           MIT
└── README.md
```

## Design

Cinco princípios estão embutidos nos subagents. Vale conhecê-los antes de customizar:

1. **Um subagent, uma fase.** Se o `coder-dev` começar a "revisar" o próprio código, algo está errado — puxe para o `code-reviewer`.
2. **HITL é sagrado.** Nenhum subagent decide sozinho que "está aprovado". Ele emite `waiting_human` e para. Quem detecta o sinal é o orquestrador, lendo evento real do Git.
3. **Evento antes, evento depois.** Início de fase emite `in_progress`; fim emite `approved` | `rejected` | `waiting_human`. Sem isso, dashboard não existe.
4. **Idempotência.** Reinvocar um subagent (ex: QA reprovou, volta pro `coder-dev`) nunca cria branches/PRs duplicados. Sempre checa antes de criar.
5. **Escopo travado pelo SDD.** O `coder-dev` recusa mudanças fora do SDD aprovado. Novo requisito no meio → retorna para Fase 2 explicitamente. Nada de expandir escopo silenciosamente.

## Customizando para o seu projeto

Os agents pré-instalados são defaults úteis, mas provavelmente sua realidade tem particularidades: outro host Git, outras labels, stack específica. Rode:

```
/pipeline-factory
```

A fábrica faz uma descoberta guiada (host, auth, stack, formato de work item, sinais HITL, sinks de evento) e regenera os agents em `~/.claude/agents/` — que **sobrescrevem** os do plugin sem tocar no repo. Quando você quiser voltar ao default, basta apagar os arquivos locais.

## Contribuindo

Issues e PRs bem-vindos. Se propuser mudança em template de subagent, atualize também o teste de sanidade em `.github/workflows/plugin-check.yml`.

Ao versionar mudanças que quebram formato de evento, incremente `schema_version`.

## Licença

MIT — veja [LICENSE](LICENSE).
