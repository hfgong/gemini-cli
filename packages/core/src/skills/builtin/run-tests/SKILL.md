---
name: run-tests
description: Discover and run tests to validate code changes. Use this skill after modifying source code to verify correctness — especially before declaring a task complete. Supports Python (pytest, unittest), Node.js (jest, vitest, mocha), Rust (cargo test), and Go (go test). Helps catch regressions, broken imports, and incorrect implementations early.
---

# Run Tests

Validate code changes by discovering and running the right tests.

## Step 1: Find the Right Tests

If the task specifies a test file, use that. Otherwise, discover tests:

```bash
# Python: find test files related to the module you changed
find . -name "test_*.py" -o -name "*_test.py" | grep -i <module_name> | head -5

# Node.js
find . -name "*.test.ts" -o -name "*.test.js" -o -name "*.spec.ts" | grep -i <module_name> | head -5

# Check if the project has a test config
ls pytest.ini pyproject.toml tox.ini jest.config.* vitest.config.* 2>/dev/null
```

## Step 2: Run Tests

### Python
```bash
# Activate venv if it exists
source .venv/bin/activate 2>/dev/null

# Run specific test file (preferred — fast, focused)
python -m pytest path/to/test_file.py -x -q

# Run a single test
python -m pytest path/to/test_file.py::TestClass::test_name -xvs

# If pytest not installed
python -m unittest path.to.test_module

# If imports fail, add source to path
PYTHONPATH=src:lib:. python -m pytest path/to/test_file.py -x -q
```

### Node.js
```bash
# Check package.json for test command
npm test
# Or run specific file
npx jest path/to/test.ts
npx vitest run path/to/test.ts
```

### Rust
```bash
cargo test <test_name>
cargo test --lib  # unit tests only
```

### Go
```bash
go test ./path/to/package/ -run TestName -v
```

## Step 3: Interpret Results

**All passed** — you're done. Proceed to complete the task.

**Import error / NameError** — you have a missing import or undefined name. Fix it and rerun.

**Assertion error** — your implementation produces wrong output. Read the assertion to understand what's expected vs. what you returned.

**Timeout** — your implementation may have an infinite loop or is too slow. Check for unbounded recursion or O(n^2) where O(n) is expected.

**Test passed but others regressed** — your change broke something else. Review what you changed and minimize the blast radius.

## Key Principles

- **Run tests early and often** — don't wait until you think you're done. Run after each significant edit.
- **Run the specific test first** — don't run the entire test suite. Run only the test file relevant to your change.
- **Read test failures carefully** — the error message usually tells you exactly what's wrong.
- **Never modify test files** unless the task explicitly asks you to. Fix the source code to make tests pass.
- **If a test expects a specific type**, return that exact type (e.g., `pd.Series` not `np.ndarray`).
