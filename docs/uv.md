# uv

Package manager de Python por Astral — reemplaza pip + virtualenv + pyenv + poetry con un binario Rust (10-100x más rápido).

## Comandos esenciales

```bash
uv init                        # inicializar proyecto
uv add <pkg>                   # agregar dependencia
uv add --dev <pkg>             # agregar dev dependency
uv add --group <name> <pkg>    # agregar a grupo nombrado
uv remove <pkg>                # eliminar dependencia
uv sync                        # sincronizar entorno con lockfile
uv sync --locked               # falla si lockfile desactualizado (ideal CI)
uv run <cmd>                   # ejecutar en el entorno del proyecto
uv run pytest                  # correr tests
uv python pin 3.12             # fijar versión de Python
uv tool install ruff           # instalar herramienta CLI aislada
uvx ruff check .               # ejecutar herramienta sin instalar
uv audit                       # escanear vulnerabilidades
uv tree                        # árbol de dependencias
uv export --format requirements.txt  # exportar para tools legacy
```

## pyproject.toml

```toml
[project]
name = "mi-proyecto"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["httpx>=0.27"]

[dependency-groups]  # PEP 735 — estándar para dev deps
dev = ["pytest>=8", "ruff>=0.4"]

[tool.uv]
default-groups = ["dev"]

# Fuentes alternativas (útil para PyTorch + CUDA)
[tool.uv.sources]
torch = [
    { index = "pytorch-cpu",   marker = "sys_platform != 'linux'" },
    { index = "pytorch-cu128", marker = "sys_platform == 'linux'" },
]

[[tool.uv.index]]
name = "pytorch-cu128"
url = "https://download.pytorch.org/whl/cu128"
explicit = true
```

## Workspaces (monorepos)

```
monorepo/
├── pyproject.toml        # workspace root
├── uv.lock               # lockfile ÚNICO para todo el workspace
└── packages/
    ├── core/pyproject.toml
    └── api/pyproject.toml
```

```toml
# root pyproject.toml
[tool.uv.workspace]
members = ["packages/*"]

[tool.uv.sources]
core = { workspace = true }  # referencia a miembro local
```

## Scripts inline (PEP 723)

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.12"
# dependencies = ["httpx", "rich"]
# ///

import httpx
# uv run script.py — instala deps en entorno cacheado sin tocar el global
```

## Lockfile

- Formato TOML, cross-platform (Linux/macOS/Windows en un solo archivo)
- Commitear al repo para reproducibilidad
- `uv sync --frozen` — usa lockfile sin verificar (CI rápido)
