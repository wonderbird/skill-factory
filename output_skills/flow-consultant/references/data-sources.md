# Data sources

Field paths below were checked against Go beads (`bd`) 1.3.4 and Claude Code
2.1.x transcripts in November 2026; formats change between versions. All
commands are read-only.

## Measured repositories

The tracker with the flow-improvement epic may live in one repository and the
work it measures in others. The epic description names the measured
repositories: read each one's absolute path, and its `owner/name` when you
need GitHub data. Read every source by path and stay in the working
directory: `git -C <path>`, `gh --repo <owner/name>`, and `jq` on
`<path>/.beads/issues.jsonl`. `<path>` below is a measured repository's
absolute path, without a trailing slash.

## Beads

Read beads data through the `bd` CLI or its JSONL export, never through the
store directly: the store can be embedded Dolt or a Dolt server.

- `<path>/.beads/issues.jsonl` holds every record only when exported with
  `bd export --all`; a plain export drops gates and memories. Check
  `<path>/AGENTS.md` for its export rule; the export is current only if
  every agent follows it.
- `started_at` is set by `bd update <id> --claim`. Coverage varies, because
  not every item gets claimed.
- `bd history <id> --events` shows status changes (claims, closes) but is
  not durable: a re-import resets it. It is no source for history.
- Count and aggregate from the JSONL export. `bd` reads the tracker of the
  working directory, not of a measured repository. `bd list` stops at 50
  results by default and hides gates unless given `--include-gates`. Wrap any
  `--limit 0` listing in `timeout`: unbounded listings can hang on large
  databases.

Field coverage, as percent of records:

```bash
jq -s 'length as $n | [.[]|keys[]] | group_by(.)
  | map({k:.[0], pct:(length*100/$n|floor)})
  | map(select(.k|IN("status","issue_type","created_at","started_at",
      "closed_at","close_reason","labels","dependencies")))' \
  <path>/.beads/issues.jsonl
```

### Status transitions from git snapshots

The only durable status history, when `issues.jsonl` is tracked. Resolution
is the commit cadence: items created and closed between two commits appear
only as `closed`.

```python
"""Print id, status, first commit time it was seen. Args: <repo-path> [file]."""
import json, subprocess, sys
repo = sys.argv[1]
path = sys.argv[2] if len(sys.argv) > 2 else ".beads/issues.jsonl"
log = subprocess.run(
    ["git", "-C", repo, "log", "--reverse", "--format=%H %cI", "--", path],
    capture_output=True, text=True, check=True).stdout.split("\n")
last = {}
for line in filter(None, log):
    sha, when = line.split(" ", 1)
    blob = subprocess.run(["git", "-C", repo, "show", f"{sha}:{path}"],
                          capture_output=True, text=True)
    if blob.returncode:
        continue
    for rec in blob.stdout.splitlines():
        try:
            r = json.loads(rec)
        except ValueError:
            continue
        i, s = r.get("id"), r.get("status")
        if i and last.get(i) != s:
            last[i] = s
            print(f"{i}\t{s}\t{when}")
```

Reopens are `closed` followed by any other status for the same id.

## Git

- Agent-guidance changes, to mark on every time axis:
  `git -C <path> log --format='%cs %h %s' -- CLAUDE.md AGENTS.md .claude`,
  plus every other rule file the repository has, such as a constitution.
- Commit-to-item join: issue ids in trailers such as `Refs: <id>`.
- Commit types: the conventional-commit prefix, e.g. `fix:`, in the subject.
- Branch-to-item join: the repository's branch naming rule, if any.

## Claude Code transcripts

Directory: `~/.claude/projects/$(echo <path> | sed 's|[^A-Za-z0-9]|-|g')/`,
derived from the measured repository, not from the working directory. Each
session is `<uuid>.jsonl`; subagents are in `<uuid>/subagents/*.jsonl`.
Worktrees get their own directory; match them on the `cwd` field.
Retention follows `cleanupPeriodDays` in `~/.claude/settings.json`, so
history is often only days long.

Signals and the records that carry them:

- Tokens: `type=="assistant"`, `.message.usage.{input_tokens,
  output_tokens, cache_read_input_tokens, cache_creation_input_tokens}`.
  One API response spans several lines with the same usage: deduplicate on
  `.message.id`, or totals double or worse.
- Human prompt: `type=="user"` and `.origin.kind=="human"`.
- Interrupt: user content contains `Request interrupted by user`.
- Denied tool call: `tool_result` with `is_error` and the text
  `The user doesn't want to proceed`.
- Human decision: assistant `tool_use` named `AskUserQuestion`.
- Turn end: `type=="system"`, `subtype=="turn_duration"`, `.durationMs`.
- Compaction: `subtype=="compact_boundary"`,
  `.compactMetadata.{trigger, preTokens, postTokens}`.
- Session cost: `type=="cost-state"`, `.totalCostUSD` (API-equivalent; keep
  the last per session).
- Branch: `.gitBranch` on most records.

Raw token sums overweight cache reads. To compare against usage windows,
weight by relative API price (cache read about 0.1x input, cache write
1.25x, output 5x) or use `cost-state` totals. Both are proxies.

## Usage windows

The status-line command receives JSON on stdin with
`rate_limits.five_hour` and `rate_limits.seven_day`, each with
`used_percentage` (0 to 100, may be fractional) and `resets_at` (epoch
seconds). It exists only for subscribers and only after the session's first
API response. Nothing stores it over time: a metric based on it needs a
status-line logger first. The value is account-wide.
