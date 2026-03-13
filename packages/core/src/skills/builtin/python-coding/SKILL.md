---
name: python-coding
description: Python coding best practices for writing correct patches. Use this skill when modifying Python source code — especially when adding new functions, changing imports, or editing modules in a larger package. Helps avoid NameErrors, broken imports, type mismatches, and incomplete implementations.
---

# Python Coding

Best practices for producing correct Python patches. These address the most common failure modes when editing Python codebases.

## After Every Edit: Validate Imports

The single most common cause of broken patches is a missing import or undefined name. After editing any `.py` file, run:

```bash
python -c "import <package>.<module>"
```

If the module is in a nested package, import the specific module you changed. If this fails with `NameError` or `ImportError`, fix it before moving on.

## Checklist Before Declaring Done

1. **Every name you use is defined or imported** — grep for the function/class name in the file. If it's not defined there, it must be imported.
2. **New functions match expected signatures** — if the task specifies a function signature, match it exactly (name, parameters, return type, defaults).
3. **Edits are minimal** — prefer adding to existing files over rewriting them. When a file has 500 lines and you need to add a function, add the function — don't rewrite the file.
4. **Existing tests still import** — if you edited a module that other modules import, verify those imports still work:
   ```bash
   python -c "import <package>"
   ```

## Common Pitfalls

### Adding a function that calls an undefined helper
When implementing a new function, every helper it calls must exist. Before writing `result = my_helper(x)`, verify `my_helper` is defined or imported in scope.

### Editing `__init__.py` exports
If you add a new public function to a module, check whether `__init__.py` re-exports it. If it does `from .module import *`, your new function is automatically exported. If it lists explicit names, add yours.

### Breaking a module's existing API
When adding code to an existing module, never remove or rename existing functions, classes, or constants unless the task explicitly asks for it. Other modules depend on them.

### Type mismatches
If the codebase uses type hints, match them. If a function returns `pd.Series`, don't return `np.ndarray`. Check the return type of functions you're replacing or extending.

### Wrong test file
When running tests to verify your change, run the test file specified in the task. If the task says "test_concat.py", don't run "test_merge.py" even if it seems related.

## Debugging a Failing Test

```bash
# Run the specific failing test with verbose output
python -m pytest path/to/test_file.py::TestClass::test_name -xvs

# If import fails, check what's missing
python -c "from package.module import SomeClass" 2>&1

# If a function returns the wrong type
python -c "from package.module import func; print(type(func(args)))"
```

## When Stuck on the Same Error 3+ Times

Stop and reconsider. Common escape routes:
- Re-read the task description — you may have misunderstood what's needed
- Read the test file to understand what it actually asserts
- Check if there's an existing implementation you can extend instead of writing from scratch
- Try a fundamentally different approach
