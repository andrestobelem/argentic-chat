# gh — GitHub CLI

CLI oficial de GitHub. Maneja la capa de plataforma (PRs, issues, releases, Actions) sin salir de la terminal.

## Autenticación

```bash
gh auth login                    # browser flow
gh auth login --with-token < token.txt
export GH_TOKEN=ghp_xxxxx        # para CI/scripts (sin gh auth login)
```

## PRs

```bash
gh pr create                     # interactivo
gh pr create --fill              # título/cuerpo del último commit
gh pr create --fill-verbose      # descripción detallada del diff
gh pr list --author "@me"
gh pr checkout 42
gh pr review 42 --approve
gh pr merge 42 --squash --delete-branch
gh pr merge 42 --auto --squash --delete-branch  # merge automático al pasar CI
gh pr checks                     # estado de CI del PR actual
```

## Issues

```bash
gh issue create -t "titulo" -b "cuerpo" -l bug -a @me
gh issue develop 123 -c          # crea branch vinculada al issue + checkout
gh issue list --assignee @me --state open
gh issue close 123 --reason completed
```

## Releases

```bash
gh release create v1.2.0 --generate-notes     # notas automáticas desde commits/PRs
gh release create v1.2.0 dist/*.tar.gz        # con assets
gh release download v1.2.0 -p "*.tar.gz"
```

## Actions/CI

```bash
gh workflow run deploy.yml --ref main -f environment=prod
gh run list --limit 10
gh run watch                     # live streaming del run
gh run watch --exit-status && notify-send "Deploy OK"
gh run rerun 1234567890 --failed-only
gh run download 1234567890       # descarga artifacts
```

## API (REST y GraphQL)

```bash
gh api repos/owner/repo/releases/latest --jq '.tag_name'
gh api orgs/myorg/repos --paginate --jq '.[].name'
gh api graphql -f query='{ viewer { login } }' --jq '.data.viewer.login'

# Flags clave:
# --jq          filtra con jq
# --paginate    sigue paginación automáticamente
# -X POST       método HTTP
# -f key=val    campo string en el body
# -F key=val    campo con type inference
```

## Scripting

```bash
# Alias personalizados
gh alias set ship 'pr merge --auto --squash --delete-branch'
gh alias set prc 'pr create --fill'

# Cerrar issues en lote
gh issue list --label wontfix --json number --jq '.[].number' | \
  xargs -I{} gh issue close {}
```

## Flujo completo desde terminal

```bash
gh issue develop 123 -c          # branch + checkout desde issue
# ... código ...
gh pr create --fill              # PR con contexto del commit
gh run watch --exit-status       # espera CI
gh pr merge --auto --squash --delete-branch
```
