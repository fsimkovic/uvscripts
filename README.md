# uv-script

[![PyPI](https://img.shields.io/pypi/v/uv-script)](https://pypi.org/project/uv-script/)

A lightweight, zero-dependency script runner for [uv](https://docs.astral.sh/uv/). Define project scripts in `pyproject.toml` and run them through `uv run`.

```
$ uvs --list
  check   lint -> test
  format  ruff format src/
  lint    ruff check src/
  test    pytest tests/ -v
```

## Why?

uv is a fantastic package manager but has no built-in task runner. If you've been using [Hatch](https://hatch.pypa.io/) just for its scripts, or reaching for [Poe the Poet](https://poethepoet.natn.io/) / [Taskipy](https://github.com/taskipy/taskipy) to fill the gap, `uvs` offers a simpler alternative:

- **Zero runtime dependencies** — stdlib only (`tomllib`, `argparse`, `subprocess`)
- **uv-native** — every command runs through `uv run`, so your venv, lockfile, and Python version are always in sync
- **Familiar** — if you've used npm scripts or Hatch scripts, you already know how this works

## Installation

```bash
# As a dev dependency in your project
uv add --dev uv-script

# Or run without installing
uvx uv-script --list
```

## Quick start

Add a `[tool.uvs.scripts]` section to your `pyproject.toml`:

```toml
[tool.uvs.scripts]
test = "pytest tests/ -v"
lint = "ruff check src/"
format = "ruff format src/"
check = ["lint", "test"]
```

Then run:

```bash
uvs test       # runs: uv run pytest tests/ -v
uvs check      # runs lint, then test (stops on first failure)
uvs --list     # shows all available scripts
```

## Configuration

Scripts are defined under `[tool.uvs.scripts]` in `pyproject.toml`. Three formats are supported:

### Simple command

A string value runs a single command through `uv run`:

```toml
[tool.uvs.scripts]
test = "pytest tests/ -v"
lint = "ruff check ."
```

### Composite script

An array of strings runs multiple steps sequentially. Items can reference other script names or be raw commands. Execution stops on the first non-zero exit code.

```toml
[tool.uvs.scripts]
lint = "ruff check ."
test = "pytest tests/"
check = ["lint", "test"]
full = ["ruff format --check .", "lint", "test"]
```

### Table with options

A table gives you control over environment variables and help text:

```toml
[tool.uvs.scripts.serve]
cmd = "python -m flask run"
env = { FLASK_DEBUG = "1", FLASK_APP = "app.py" }
help = "Start the development server"

[tool.uvs.scripts.deploy]
cmd = "python scripts/deploy.py"
env = { ENV = "production" }
help = "Deploy to production"
```

| Key    | Type              | Required | Description                              |
|--------|-------------------|----------|------------------------------------------|
| `cmd`  | string            | yes      | The command to run                       |
| `env`  | table of strings  | no       | Environment variables (overlays current) |
| `help` | string            | no       | Description shown in `--list` output     |

### Editable installs

For monorepos with multiple packages, you can configure editable installs under `[tool.uvs]`. These are passed as `--with-editable` flags to every `uv run` invocation, so local packages take priority over PyPI versions:

```toml
[tool.uvs]
editable = ["../shared-lib", "../auth-service"]

[tool.uvs.scripts]
test = "pytest tests/ -v"
```

Paths are resolved relative to the `pyproject.toml` directory. To skip editable installs for a single run, use `--no-editable`:

```bash
uvs --no-editable test
```

## Usage

```
uvs [options] <script> [-- extra-args...]
```

| Flag              | Description                           |
|-------------------|---------------------------------------|
| `-l`, `--list`    | List all available scripts            |
| `-v`, `--verbose` | Print each command before running     |
| `--no-editable`   | Ignore editable installs from config  |
| `-V`, `--version` | Show version and exit                 |

### Passing extra arguments

Use `--` to forward arguments to the underlying command:

```bash
uvs test -- -k "test_parse" --no-header
# runs: uv run pytest tests/ -v -k test_parse --no-header
```

For composite scripts, extra arguments are appended to the last command in the chain.

### Running from subdirectories

`uvs` walks up from your current directory to find the nearest `pyproject.toml`, just like `uv` does. You can run `uvs test` from anywhere inside your project.

## How it works

`uvs` is a thin orchestration layer. Every command is prefixed with `uv run`, which means:

1. Your virtual environment is automatically activated
2. Dependencies are synced from the lockfile if needed
3. The correct Python version is used

There is no magic — `uvs test` is equivalent to typing `uv run pytest tests/ -v` yourself.

## Development

This project uses `uvs` to manage its own scripts:

```bash
git clone <repo-url> && cd uv-script
uv sync
uvs test       # run tests
uvs lint       # run linter
uvs check      # lint + test
```

## License

MIT
