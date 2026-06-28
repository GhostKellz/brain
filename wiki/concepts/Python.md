---
type: concept
title: "Python"
created: 2026-06-28
updated: 2026-06-28
tags:
  - language
  - python
  - programming
status: developing
related:
  - "[[Go]]"
  - "[[JavaScript]]"
  - "[[Rust]]"
  - "[[Local LLM Inference]]"
---

# Python

Python is a dynamically typed, interpreted, "batteries-included" language prized
for **readability and speed of development**. It's the lingua franca of scripting,
automation, data science, and ML — including the local-LLM tooling in this vault
(`retrieve.py`, embedding scripts, see [[Local LLM Inference]]).

> [!key-insight]
> Python's superpower is the ecosystem (PyPI) and how little ceremony it takes to
> get something working. The cost is runtime performance and packaging/environment
> management — which is why **virtual environments are non-negotiable** on Linux.

## Where it fits vs Go / Rust

| | Python | [[Go]] | [[Rust]] |
|--|--------|--------|----------|
| Typing | dynamic (+ optional hints) | static | static |
| Speed | slow (interpreted) | fast | fastest |
| Best at | scripting, glue, data/ML, prototyping | services, CLIs | safety-critical apps |
| Deploy | interpreter + venv/deps | single binary | single binary |

Reach for Python for automation, data work, ML, and quick prototypes; reach for
[[Go]]/[[Rust]] when you need a shippable binary or raw performance.

## Virtual environments on Linux (do this first)

Never `pip install` into the system Python on Linux — distros mark it
"externally managed" (PEP 668) and you'll break system packages. Always isolate.

```bash
python -m venv .venv            # create an env in ./.venv
source .venv/bin/activate       # activate (deactivate with `deactivate`)
python -m pip install -U pip    # upgrade pip inside the env
pip install -r requirements.txt # install pinned deps
pip freeze > requirements.txt   # snapshot exact versions
```

> [!key-insight]
> The venv is just a directory with its own `bin/python` and `site-packages`.
> Activating it prepends `.venv/bin` to `PATH`. Commit `requirements.txt` (or
> `pyproject.toml`), never the `.venv/` directory itself.

### The tooling ladder

| Tool | Use it for |
|------|------------|
| `venv` + `pip` | stdlib baseline — always available |
| **`pipx`** | install *applications* (black, ruff, httpie) in isolated envs, on PATH globally |
| **`uv`** | ultra-fast (Rust-based) pip/venv replacement: `uv venv`, `uv pip install`, `uv run`, `uv sync` — increasingly the default |
| **`poetry`** / **`pdm`** | project + dependency + build management via `pyproject.toml` with a lockfile |
| **`pyenv`** | install/switch multiple Python *versions* per project |

```bash
# uv — the modern fast path
uv venv && uv pip install flask
uv run python app.py            # runs in the project env, no manual activate
```

## Web: Flask and FastAPI

**Flask** — minimal WSGI microframework; you add what you need.

```python
from flask import Flask, jsonify
app = Flask(__name__)

@app.route("/health")
def health():
    return jsonify(status="ok")

# dev:  flask --app app run --debug
# prod: gunicorn -w 4 'app:app'   (behind nginx)
```

**FastAPI** — modern async framework with Pydantic validation + automatic
OpenAPI docs; the default choice for new APIs.

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/health")
async def health():
    return {"status": "ok"}
# run: uvicorn app:app --reload
```

> [!note] Flask = simple/sync/ubiquitous; FastAPI = async/typed/auto-docs.
> Both sit behind a real WSGI/ASGI server (`gunicorn`/`uvicorn`) + nginx in prod,
> never the dev server.

## Must-have libraries

| Area | Libraries |
|------|-----------|
| HTTP | **`requests`** (sync), **`httpx`** (sync+async) |
| CLI | **`typer`** (type-hint based), **`click`**, **`argparse`** (stdlib) |
| Terminal UX | **`rich`** (tables, color, progress), **`textual`** (TUI) |
| Validation/config | **`pydantic`** (v2), **`pydantic-settings`**, **`python-dotenv`** |
| Data | **`numpy`**, **`pandas`**, **`polars`** (fast), **`matplotlib`** |
| ML/AI | **`pytorch`**, **`transformers`**, **`sentence-transformers`**, **`ollama`** |
| DB | **`sqlalchemy`** (ORM/core), **`psycopg`** (Postgres), **`sqlite3`** (stdlib) |
| Quality | **`ruff`** (lint+format, Rust-fast), **`black`**, **`mypy`** (types), **`pytest`** |
| Async/jobs | **`asyncio`** (stdlib), **`celery`**, **`apscheduler`** |

## Strengths / trade-offs

**Strengths**
- Fastest path from idea to working code; enormous ecosystem (PyPI).
- Dominant in data/ML/AI; first-class for sysadmin scripting and glue.
- Readable, gentle learning curve; optional type hints + `mypy` scale it up.

**Trade-offs**
- Slow at CPU-bound work; the **GIL** limits true CPU parallelism (use
  multiprocessing, or offload to native libs/`asyncio` for I/O). Free-threaded
  builds (3.13+ experimental) are changing this.
- Packaging/deployment is historically painful — venvs and pinned deps are
  mandatory discipline. `uv` is closing the gap.
- Dynamic typing trades early safety for speed of writing.

## Related

- [[Go]] · [[JavaScript]] · [[Rust]] — the other languages here
- [[Local LLM Inference]] — Python drives the embedding/retrieval scripts
