# Contribuindo

Obrigado por considerar contribuir. Este plugin é pequeno e focado — mudanças bem-vindas seguem os princípios abaixo.

## Setup local

```bash
git clone https://github.com/renatodavis/pipeline-orchestrator-sdlc.git
cd pipeline-orchestrator-sdlc
# Sem build, sem npm install. Arquivos versionados são o produto.
```

Para testar mudanças no seu Claude Code sem publicar:

```
/plugin marketplace add /caminho/absoluto/para/pipeline-orchestrator-sdlc
/plugin install pipeline-orchestrator-sdlc@pipeline-orchestrator-sdlc
```

Aponta o marketplace para um path local em vez de repo remoto — Claude Code aceita ambos.

## Regras de mudança

**Mudou template em `skills/pipeline-orchestrator-factory/references/`?**
Atualize também o subagent pré-gerado correspondente em `agents/` — os dois precisam ficar coerentes, senão quem instala o plugin vê um comportamento e quem regenera pela factory vê outro.

**Mudou o schema de eventos JSON?**
Incremente `schema_version` em todos os lugares (agents/, references/event-schema.md, exemplo em assets/) e documente no CHANGELOG. Consumidores externos (dashboards) dependem dessa versão.

**Adicionou subagent novo?**
- Crie tanto o template (`references/<nome>-template.md`) quanto a versão pré-gerada (`agents/<nome>.md`).
- Adicione uma linha na tabela de subagents do `agents/pipeline-orchestrator.md` — o orquestrador tem regra de nunca invocar subagent que não esteja listado lá.
- Atualize a fase correspondente no fluxo do orquestrador se for uma nova fase.

**Adicionou slash command?**
Adicione uma linha na tabela de comandos do README.

## CI

O workflow `.github/workflows/plugin-check.yml` valida em cada PR:
- `plugin.json` e `marketplace.json` com campos obrigatórios e versão coerente
- Frontmatter YAML de todos os agents e skills
- `name` do frontmatter batendo com o filename (Task tool exige)
- Nenhum placeholder `{{...}}` esquecido
- JSONL de exemplo bem formado

Rode localmente antes de abrir PR:

```bash
python3 -c "
import subprocess, sys
r = subprocess.run(['bash', '-c', 'grep -c \"python3\" .github/workflows/plugin-check.yml'], capture_output=True, text=True)
print('Steps de validação:', r.stdout.strip())
"
```

(Ou simplesmente abra o PR — o CI roda em ~30s.)

## Versionamento

[SemVer](https://semver.org/lang/pt-BR/). Bump correto:
- **PATCH**: correção que não muda comportamento observável dos agents.
- **MINOR**: novo subagent, novo comando, nova opção nos templates. Retrocompatível.
- **MAJOR**: mudança no schema de eventos, remoção de subagent, incompatibilidade com marketplace.

Ao bumpar, atualize simultaneamente:
- `.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json` (campo `version` do plugin listado)
- `CHANGELOG.md` (nova seção)
- Depois: `git tag -a vX.Y.Z -m "X.Y.Z"` + `git push origin vX.Y.Z`

## Código de conduta

Assuma boa fé. Descreva reprodução. Discorde no mérito, não na pessoa.
