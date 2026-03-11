---
name: setup-project
description: Set up a project's development environment so code can be run and tested. Use this skill when you need to install dependencies, build the project, or set up a virtual environment before running tests or executing code. Activate early in any task that requires running the project's code.
---

# Project Setup

Set up the project's development environment so tests and code can be executed.

## Strategy

1. **Detect** the project type from config files
2. **Install** dependencies in an isolated environment
3. **Verify** the setup by importing or running a quick smoke test

## Python Projects

### Detection

Check for these files (in priority order):
- `pyproject.toml` — modern Python packaging (pip, poetry, flit, hatch, meson-python)
- `setup.py` or `setup.cfg` — legacy setuptools
- `requirements.txt` — dependency list only
- `tox.ini` or `nox.ini` — test runner configs (often list deps)
- `Pipfile` — pipenv
- `conda.yml` or `environment.yml` — conda

### Setup Steps

```bash
# 1. Create a virtual environment (avoids polluting system Python)
python3 -m venv .venv
source .venv/bin/activate

# 2. Upgrade pip to avoid build issues
pip install --upgrade pip setuptools wheel

# 3. Install the project (pick based on what config exists)
```

**pyproject.toml (most common):**
```bash
# Check the build backend first
grep -A2 'build-system' pyproject.toml

# For setuptools/flit/hatch/pdm:
pip install -e ".[dev,test]" 2>/dev/null || pip install -e ".[test]" 2>/dev/null || pip install -e .

# For meson-python (matplotlib, scipy, etc.) — needs system build tools:
pip install -e . --no-build-isolation 2>/dev/null || pip install .

# For poetry:
pip install poetry && poetry install
```

**setup.py / setup.cfg:**
```bash
pip install -e ".[test]" 2>/dev/null || pip install -e .
```

**requirements.txt only:**
```bash
pip install -r requirements.txt
# Also install test deps if they exist
pip install -r requirements-dev.txt 2>/dev/null
pip install -r requirements-test.txt 2>/dev/null
```

### Common Build Failures and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Python.h not found` | Missing Python headers | `apt-get install python3-dev` or use `--no-build-isolation` |
| `ModuleNotFoundError: _c_internal_utils` | C extension not compiled | `pip install .` (non-editable) or build first |
| `No matching distribution` | Wrong Python version | Check `python_requires` in config, use correct Python |
| `error: subprocess-exited-with-error` | Build tool missing | Install build deps: `pip install cython numpy meson-python meson ninja` |

### Running Tests

After setup, find and run tests:
```bash
# Detect test runner from config
grep -l "pytest\|unittest\|nose" pyproject.toml setup.cfg tox.ini 2>/dev/null

# Common test commands (try in order):
python -m pytest tests/ -x -q 2>/dev/null \
  || python -m pytest test/ -x -q 2>/dev/null \
  || python -m unittest discover -s tests 2>/dev/null \
  || python -m pytest -x -q

# If tests fail with import errors, ensure PYTHONPATH includes the source:
PYTHONPATH=src:lib:. python -m pytest tests/ -x -q
```

### Import Verification

Quick check that the project is importable:
```bash
# Find the main package name
PACKAGE=$(find . -maxdepth 2 -name "__init__.py" -not -path "./.venv/*" -not -path "./build/*" | head -1 | xargs dirname | xargs basename)
python -c "import $PACKAGE; print(f'{$PACKAGE} imported successfully')"
```

## Node.js Projects

### Detection
- `package.json` — npm/yarn/pnpm
- `pnpm-lock.yaml` — pnpm
- `yarn.lock` — yarn

### Setup
```bash
# Detect package manager and install
if [ -f pnpm-lock.yaml ]; then
  pnpm install
elif [ -f yarn.lock ]; then
  yarn install
else
  npm install
fi

# Run tests
npm test
```

## Rust Projects

### Detection
- `Cargo.toml`

### Setup
```bash
cargo build
cargo test
```

## Go Projects

### Detection
- `go.mod`

### Setup
```bash
go mod download
go test ./...
```

## Key Principles

- **Always use a virtual environment** for Python to avoid conflicts
- **Try editable install first** (`pip install -e .`) so source changes take effect immediately
- **Fall back gracefully** — if `.[test]` extras fail, try without extras
- **Check PYTHONPATH** if imports fail after install — some projects need `src/` or `lib/` on the path
- **Don't modify the project's build config** unless the task specifically asks for it
- **If build requires system packages** you can't install, note this and proceed with what works
