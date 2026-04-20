# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`argentic-chat` — a chat application for Argentine AI agents. Python 3.12, managed with `uv`.

## Reference Docs

Detailed guides in `docs/`: `uv.md`, `gh.md`, `dolt.md`, `duckdb.md`, `commits.md`

## Commands

```bash
uv run main.py          # run the app
uv run pytest           # run tests
uv add <package>        # add a dependency
uv add --dev <package>  # add a dev dependency
uv sync                 # sync dependencies from lockfile
rm -rf .venv && uv sync # recreate virtual environment
```

## Dependencies

- `duckdb` — in-process analytical DB (OLAP, Parquet, CSV, vector search via VSS extension)
- `doltcli` — Git-versioned SQL database client (requires Dolt binary running as `dolt sql-server`)
- `mysql-connector-python` — connects to Dolt's MySQL-compatible server on port 3306

## Gotchas

- **Python ≤ 3.12 required**: `doltcli` depends on `typed-ast` which doesn't build on Python 3.13+. Do not upgrade `requires-python` beyond `<3.13`.
- **Dolt needs a separate binary**: `doltcli` wraps the Dolt CLI. Install Dolt separately: `brew install dolt`, then run `dolt sql-server` before using `doltcli` or `mysql-connector-python`.
