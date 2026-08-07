# Integração Git — GitLab vs GitHub

Mapa rápido das operações que o orquestrador e os subagents precisam. Escolha o mecanismo de auth no início (`gh` CLI, `glab` CLI, MCP dedicado, ou API + token) e use consistentemente.

## Operações necessárias

| Operação | Quem usa | GitHub | GitLab |
|---|---|---|---|
| Ler issue | triage-pm, spec-builder | `gh issue view N --json ...` | `glab issue view N --output json` |
| Comentar na issue | triage-pm | `gh issue comment N -b "..."` | `glab issue note N -m "..."` |
| Aplicar label | orquestrador | `gh issue edit N --add-label X` | `glab issue update N --label X` |
| Criar branch | coder-dev | `git checkout -b ...` (local) | idem |
| Push + abrir MR/PR | coder-dev | `gh pr create --base main --title ... --body ...` | `glab mr create --source-branch ... --target-branch ...` |
| Ler diff do MR/PR | code-reviewer | `gh pr diff N` | `glab mr diff N` |
| Comentar em linha de diff | code-reviewer | `gh api` (endpoint `pulls/comments`) | `glab api` (endpoint `merge_requests/notes` com `position`) |
| Ler approvals | orquestrador (HITL 2) | `gh pr view N --json reviews` | `glab api projects/:id/merge_requests/N/approvals` |
| Checar labels | orquestrador (HITL 1/3) | `gh issue view N --json labels` | `glab issue view N -F json` |

## Autenticação

- **GitHub**: `gh auth login` grava token em keyring. Em CI, `GH_TOKEN` env.
- **GitLab (self-hosted)**: `glab auth login --hostname <host>`. Em CI, `GITLAB_TOKEN` env.
- **MCP dedicado**: se houver server MCP conectado para o host, prefira — tokens ficam fora do processo.

## Detectando aprovação (para HITL)

### HITL 1 (SDD) — label based
```bash
# GitHub
gh issue view $ISSUE --json labels -q '.labels[].name' | grep -q '^SDD-Approved$'
# GitLab
glab issue view $ISSUE -F json | jq -r '.labels[]' | grep -q '^SDD-Approved$'
```

### HITL 2 (Code Review)
```bash
# GitHub — pelo menos 1 APPROVED e nenhum CHANGES_REQUESTED pendente
gh pr view $PR --json reviews -q '[.reviews[] | .state] | any(. == "APPROVED") and (all(. != "CHANGES_REQUESTED"))'
# GitLab — approvals_left == 0
glab api projects/:id/merge_requests/$MR/approvals | jq '.approvals_left == 0'
```

### HITL 3 (QA) — label based
```bash
# análogo ao HITL 1, checando 'qa-passed' vs 'qa-failed'
```

## Rate limiting

- GitHub: 5000 req/h autenticado. `gh api rate_limit` para checar.
- GitLab.com: 300 req/min. Self-hosted: depende da config.

Regra prática: uma checagem por invocação de orquestrador. Se o usuário quer monitoramento contínuo, use `scheduled-tasks` MCP com intervalo ≥ 15 min.

## Segredos

Nunca ecoe tokens em log/evento. Nunca coloque token em URL de comentário. Se precisar mostrar um link, mostre a URL pública do MR/issue — auth vem do ambiente.
