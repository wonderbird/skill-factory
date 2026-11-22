# Fake executables

A fake executable stands in for a slow or external tool in a unit test. The
code under test runs unchanged, through its real `subprocess` calls. The test
therefore checks what the code asks of the tool, not how the code calls
Python. A test that mocks `subprocess.run` checks the call instead, and it
breaks when the call changes even though the behaviour stays the same.

## The pattern

- The test writes a short script, named like the tool, into a temporary
  directory. A `sh` script is enough for most fakes. Use a Python script with
  the shebang `#!{sys.executable}` when the fake must parse input, such as a
  manifest.
- The fake logs its arguments, one line per call. It then succeeds or fails
  as the test requires. An environment variable such as `FAKE_GH_MODE` can
  select the behaviour, so one fake serves several tests.
- The fake reaches the code in one of two ways:
  - A tool that the code looks up on `PATH` comes from the temporary
    directory, which is placed first on `PATH`.
  - A tool that the code receives as a path, such as a venv's `python`, is
    passed as that path.
- Each test states what its fake does. The plumbing that every fake shares
  lives in a shared test module, e.g. `fakes.py`.
- Fast, local tools such as git stay real. They run in a temporary
  repository, with `HOME` and the `XDG_*` directories redirected, so that no
  test touches the developer's checkout or configuration.

## Worked example

These illustrate the principle. Consider what fits your context.

The shared plumbing, `fakes.py`. `write_executable` is public because a test
whose fake is a Python script calls it directly:

```python
"""Fake executables that stand in for external tools in the unit tests.

A test states what its fake does; these helpers do the plumbing that every
fake shares: writing the script, logging its calls, and putting it on PATH.
"""

import os
import textwrap


def write_fake(path, behaviour="", log=None):
    # A shell script that logs the arguments of each call, if given a log,
    # then runs the behaviour.
    script = "#!/bin/sh\n"
    if log is not None:
        script += f'echo "$@" >> "{log}"\n'
    script += textwrap.dedent(behaviour)

    return write_executable(path, script)


def write_executable(path, script):
    path.write_text(script)
    path.chmod(0o755)
    return path


def logged_calls(log):
    # One line per call. No log means no call.
    return log.read_text().splitlines() if log.exists() else []


def first_on_path(bin_dir):
    # The environment that makes the fakes in bin_dir win over the real tools.
    return {"PATH": f"{bin_dir}{os.pathsep}{os.environ['PATH']}"}
```

A test module for a simplified `release.publish()` that reads `VERSION` in a
git repository and runs `gh release create`. The class lists `setUp`, then the
tests, then the helpers. The module-level fixture builder follows the class:

```python
import os
import subprocess
import tempfile
import unittest
from pathlib import Path
from unittest import mock

from . import release
from .errors import ReleaseError
from .fakes import first_on_path, logged_calls, write_fake


class PublishTest(unittest.TestCase):
    def setUp(self):
        # An isolated environment: the fake gh first on PATH, HOME and the
        # XDG configuration in a temporary directory.
        tmp = Path(self.enterContext(tempfile.TemporaryDirectory()))
        bin_dir = tmp / "bin"
        bin_dir.mkdir()
        self.log = tmp / "gh-calls"
        self.fake_gh = write_fake(bin_dir / "gh", log=self.log)
        env = {
            **first_on_path(bin_dir),
            "HOME": str(tmp / "home"),
            "XDG_CONFIG_HOME": str(tmp / "config"),
        }
        self.enterContext(mock.patch.dict(os.environ, env))

        # A real git repository, because git is fast and local.
        self.repo = make_git_repo(tmp / "repo", version="1.2.0")

    def test_creates_release_for_version(self):
        self.run_release_publish_with()
        self.assertEqual(logged_calls(self.log), ["release create v1.2.0"])

    def test_fails_when_gh_exits_nonzero(self):
        write_fake(self.fake_gh, behaviour="exit 1", log=self.log)
        with self.assertRaises(subprocess.CalledProcessError):
            self.run_release_publish_with()

    def test_fails_without_version_file(self):
        (self.repo / "VERSION").unlink()
        with self.assertRaisesRegex(ReleaseError, "Cannot read version"):
            self.run_release_publish_with()
        self.assert_gh_not_called()

    def run_release_publish_with(self):
        release.publish(repo_root=self.repo)

    def assert_gh_not_called(self):
        self.assertEqual(logged_calls(self.log), [], "gh ran despite failed check")


def make_git_repo(path, version):
    path.mkdir()
    subprocess.run(["git", "init", "-q"], cwd=path, check=True)
    (path / "VERSION").write_text(f"{version}\n")
    return path
```

What the example shows:

- `setUp` shows the data flow. `make_git_repo` takes its inputs as
  arguments and returns the repository.
- `setUp` patches the process environment once, so git and the fake both
  run with a temporary `HOME`, and every test inherits the isolation.
- The tests assert outcomes: what `gh` received, the error raised, and that
  `gh` never ran after a failed precondition.
- `run_release_publish_with` is the only place that calls the function under
  test, so every test body reads as a specification.
