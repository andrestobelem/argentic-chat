# Commits: Convencionales y Atómicos

## Conventional Commits

### Formato

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Tipos

| Tipo | SemVer | Uso |
|------|--------|-----|
| `feat` | MINOR | Nueva funcionalidad |
| `fix` | PATCH | Corrección de bug |
| `refactor` | — | Reescritura sin fix ni feature |
| `perf` | PATCH | Mejora de performance |
| `test` | — | Agregar/corregir tests |
| `docs` | — | Solo documentación |
| `style` | — | Formato, whitespace (sin lógica) |
| `build` | — | Build system, dependencias |
| `ci` | — | Configuración CI/CD |
| `chore` | — | Mantenimiento general |

### Breaking Changes

```bash
# Con ! en el header
feat(api)!: remove /v1/users endpoint

# Con footer BREAKING CHANGE:
feat(api): add /v2/users endpoint

BREAKING CHANGE: /v1/users has been removed. Migrate to /v2/users.
Closes #890
```

### Ejemplos

```bash
feat(cart): add item quantity selector
fix(auth): prevent token refresh loop on 401 response
refactor(cart): extract price calculation to CartPricingService
perf(search): replace linear scan with binary search
chore(deps): bump lodash from 4.17.20 to 4.17.21
ci(github-actions): add Node 20 to test matrix
revert: feat(payments): add Stripe integration
```

### Errores comunes

- `fix: Fixed the bug` — usar imperativo: `fix: prevent X`
- `feat: add feature.` — sin punto final
- `Update stuff` — siempre incluir tipo
- Scope para issue: `fix(#123):` ❌ → issues van en footer: `Closes #123` ✓

---

## Commits Atómicos

Un commit = un cambio lógico que deja el repo en estado funcional.

**Test de atomicidad**: `git revert <sha>` debe dejar el repo coherente sin efectos colaterales.

### Si el mensaje necesita "y", son dos commits

```bash
# Mal
git commit -m "fix login and update navbar styles"

# Bien
git commit -m "fix(auth): prevent session expiry on page reload"
git commit -m "style(navbar): normalize border-radius to 4px"
```

### git add -p — herramienta central

```bash
git add -p              # hunk por hunk para todo el working tree
git add -p archivo.py   # solo un archivo

# Opciones por hunk:
# y — stagear
# n — no stagear
# s — dividir en hunks más pequeños
# e — editar el hunk manualmente
# q — salir y commitear lo staged
```

### Rebase interactivo para limpiar antes de push

```bash
git rebase -i HEAD~5

# Opciones:
# pick    — mantener
# squash  — fusionar con anterior (concatena mensajes)
# fixup   — fusionar con anterior (descarta mensaje)
# reword  — solo cambiar el mensaje
# drop    — eliminar el commit
```

### Workflow con --fixup

```bash
# Fix rápido de un commit anterior durante desarrollo
git commit --fixup <sha-del-commit-a-corregir>

# Limpiar antes de push
git rebase -i --autosquash HEAD~N
```

### git stash con granularidad

```bash
git stash push -m "WIP: feature X sin terminar"
git add -p              # stagear solo los cambios listos
git commit -m "fix: ..."
git stash pop

git stash push -p       # stashear hunks específicos
```

---

## Changelogs automáticos

| Tool | Stack | Control |
|------|-------|---------|
| `git-cliff` | Cualquiera (Rust, sin Node) | Total — solo genera CHANGELOG.md |
| `semantic-release` | Node/NPM | Automático — versiona, tagea, publica |
| `release-please` | Cualquiera | Medio — crea PR de release para aprobar |

```bash
# git-cliff
git cliff --output CHANGELOG.md
git cliff --latest --output CHANGELOG.md
```

## Enforcement con pre-commit (Python/cualquier stack)

```yaml
# .pre-commit-config.yaml
default_install_hook_types: [pre-commit, commit-msg]
repos:
  - repo: https://github.com/alessandrojcm/commitlint-pre-commit-hook
    rev: v9.18.0
    hooks:
      - id: commitlint
        stages: [commit-msg]
        additional_dependencies: ['@commitlint/config-conventional']
```

```bash
uv add --dev pre-commit
uv run pre-commit install -t commit-msg
```
