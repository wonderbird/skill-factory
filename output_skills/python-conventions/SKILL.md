---
name: python-conventions
description: Code and test conventions for Python that differ from Python's defaults. Use when writing, changing or reviewing Python code or tests.
---

# Python Conventions

STARTER_CHARACTER = 🐍

Code is read far more often than it is written, and often by people who know
another language better than Python. Uniform names, blocks with a stated
intent, and explicit comparisons let such a reader follow any module without
knowing Python's idioms. Tests that read as specifications state what the code
promises before they show how it is checked. When a case is not covered by a
rule, decide it in the way that serves this reader.

Follow the **Write** branch when you write or change Python code or tests.
Follow the **Review** branch when you review them. Both branches use the
rules below, each named in bold.

## Code rules

These rules bind test modules too.

- **One public entry point.** A module exposes only the functions its callers
  use. Every internal helper has a `_` prefix.
- **Name by meaning.** A name states what it holds, e.g. `for user in users`.
  This applies in comprehensions and lambdas too. Single letters are allowed
  only for the idioms `e` (caught exception), `k, v` (dict items), `_`
  (ignored value) and `i` (index).
- **One block per intent.** In a function whose body has more than five lines
  of code (not counting the docstring, comments and blank lines), put an empty
  line between the blocks that each serve one intent. Start each block with a
  short comment that names its intent, the way a heading names a section,
  e.g. `# Check every precondition before the first write.` A body that serves
  one intent needs no such comment: the function name states it.
- **Comments state what the code cannot.** Any comment beyond the intent names
  gives a reason, a constraint, or a consequence of where the code sits. Use
  the deletion test: if a reader loses nothing when the comment is deleted,
  delete it. A comment that restates the code costs every reader time. It also
  drifts from the code, and nothing fails when the two disagree.
- **Explicit over implicit.** A condition tests a `bool`. Compare any other
  value explicitly: `name != ""`, `len(items) > 0`, `value is not None`. A
  call that returns a `bool`, such as `path.exists()`, complies. Conditions
  include `if`, `while`, conditional expressions, `and`/`or` operands and
  `assertTrue`. This rule overrides PEP 8's advice to rely on the truthiness
  of empty sequences. Every
  `subprocess.run` passes `check=`: `check=True`, or `check=False` when the
  code then reads `returncode`.
- **Fail loud.** When the code cannot fetch, parse or compute a value that it
  needs, it stops. Its error message names the source (variable, path, URL or
  command) and the fix. It validates the output of a tool before it uses that
  output. Anti-examples: `.get(key, "")` for a required key, a `None` result
  that the caller passes on, `except Exception: pass`, an early `return` when
  a required file is missing. A silent failure gives a system that looks
  healthy but is misconfigured. An error at the gap is faster to diagnose than
  the symptom that appears later.
- **One error type.** A command-line program raises one exception type of its
  own for expected failures. Only `main()` catches it. `main()` prints the
  message to stderr and returns a nonzero exit code, and the module runs it
  as `sys.exit(main())` under `if __name__ == "__main__":`. `main()` also
  catches `subprocess.CalledProcessError` from `check=True`, whose message
  already names the command and its exit status. Because `main()`
  returns the code instead of exiting, a test can call it. Library code raises
  and leaves printing and exiting to its caller. An unexpected exception stays
  uncaught, so it keeps its traceback.
- **Preconditions first.** A script that changes state checks every
  precondition that it can before its first change. Afterwards it verifies
  the result itself when a tool can fail silently, e.g. when the tool reports
  a failure only as a warning. A run that fails early leaves less half-done
  work.
- **Re-runnable.** A script that changes state is idempotent. Run again after
  a success, it changes nothing and succeeds. Run again after a partial
  failure, it finishes the work. Logs and timestamps may change.
- **Environment inputs as parameters.** A function takes filesystem roots, the
  home directory and similar inputs from the environment as parameters. Their
  defaults are the real values, e.g. `publish(repo_root=REPO_ROOT, home=HOME)`.
  Tests pass temporary directories instead of patching module globals. The
  process environment, which carries `PATH` and `HOME`, is the exception:
  tests patch it with `mock.patch.dict(os.environ, ...)`.
- **Simplicity.** Solve the current, confirmed need with the minimum working
  solution. Add a parameter, option or abstraction only for a caller that
  exists now, and justify any complexity you add. Tests are such callers (see
  **Environment inputs as parameters**). Code built for hypothetical needs is
  hard to reason about and to maintain.
- **One source of truth.** Logic, configuration and documentation are defined
  in one place. When code needs them a second time, extract a shared helper or
  module instead of a copy, and refer to the source instead of repeating it.
  Copies drift apart, and then the authoritative version is unclear.

## Test rules

- **Sibling test module.** A module ships with tests unless it is too trivial
  to test. The tests are `unittest.TestCase` classes in a sibling
  `test_<module>.py`, run under pytest. When several test modules share test
  support, such as fake executables, it lives in a separate module.
- **Test our own logic.** Mock the standard library and external tools only at
  their boundary. Internal helpers run for real. A test never checks the
  behaviour of the standard library or of the tool.
- **Fake executables.** Replace a slow or external tool with a fake
  executable. Fast local tools such as git stay real and run in a temporary
  directory. When a test touches any tool, read
  [references/fake-executables.md](references/fake-executables.md): it holds
  the pattern, how a fake reaches the code, and a worked example.
- **Outcomes, not call order.** Prove that a program runs all its parts
  through the outcomes of those parts, in an integration or acceptance test.
  A test that asserts the order of internal calls (`assert_has_calls` on
  helpers, a mock for each step) only cements the internal design. What an
  external tool received, or that it was not called, is an outcome at the
  boundary, and a test may assert it.
- **A test body reads as its specification.** Put the mechanics in helpers
  named by intent, e.g. `assert_release_published()`. The helper that runs the
  function under test is named after the call it makes, e.g.
  `run_release_publish_with` for `release.publish()`.
- **Top-down order.** A test class lists `setUp` first, then the tests, then
  the helpers that the tests call. Module-level fixture builders come after
  the class.
- **Fixture builders take explicit inputs.** A builder takes its inputs as
  arguments and returns what it builds, so `setUp` shows the data flow.
- **Acceptance tests run by name.** A test that needs the real tools, the
  network or minutes of run time lives in `acceptance_<module>.py`. That name
  does not match pytest's default pattern `test_*.py`, so the default run
  skips it. Run it by naming the file.
- **Guards proven by failure.** A test, check or assertion counts as proven
  only after you have seen it fail on known-bad input. Break the guarded code
  (delete a check, drop a flag), run the tests, see at least one test fail,
  then restore the code and see the tests pass. If no test catches the break,
  write a test that does. In a review, report a new or changed test as
  unproven, not as passing, unless its failing run is recorded (commit, pull
  request, CI log). A guard that cannot fail looks exactly like a guard that
  finds nothing wrong.

## Ruff gate

The gate is part of done in both branches. It enforces formatting, line
length, import order, current idioms and common bug patterns. The rules
above leave those topics to ruff.

- When the project has no ruff configuration (`ruff.toml`, `.ruff.toml` or
  `[tool.ruff]` in `pyproject.toml`), add `--extend-select E501,W,I,UP,B` to
  `ruff check`.
- When `ruff` is not on `PATH`, run it as `uvx ruff`.

## Write

1. Write the code and its tests to the rules. Done when every rule holds for
   every function and test that you added or changed.
2. Run the tests, then prove each new or changed test by failure. Done when
   all tests pass and you have seen every new or changed test fail once on
   broken code.
3. Run `ruff format` and then `ruff check` on the changed files. Fix what they
   report, then run the tests again. Done when `ruff format --check` and
   `ruff check` exit 0 and the tests pass.

## Review

The scope is the files you were given, or the changed lines of a diff
together with the modules, functions and test classes that contain them.

1. Run `ruff format --check` and `ruff check` on the files in scope. Each
   problem that they report is a finding that cites the ruff rule code. Leave
   the files unchanged.
2. Check every rule against everything in scope. Done when you have checked
   each rule against each module, function and test class in scope, not only
   the rules that the first violations suggest.
3. Report one line per finding, sorted by file and line:

   ```text
   app/cli.py:42 | Explicit over implicit | Write `if name != "":`
   ```

   The rule is a bold rule name from this skill or a ruff code. The fix is
   the concrete change. A finding about a whole module, such as a missing
   test module or unformatted code, gives the path without a line, e.g.
   `app/cli.py | ruff format | Run ruff format`. If you find a defect that no
   rule covers, list it
   after the findings, under the heading `Outside the conventions`. If you
   have no findings, report `No findings` and the ruff results.
