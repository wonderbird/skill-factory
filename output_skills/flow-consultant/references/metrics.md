# Metrics catalogue

Candidates for one increment, mapped to an agentic engineering workflow.
Each entry names the question it answers, its data, a chart, a balancing
metric and the main pitfall in reading it. Coverage and traps of each data
source live in [data-sources.md](data-sources.md). These illustrate the
options; consider what fits the user's question and data.

Closed items exclude the issue types `gate`, `epic` and `molecule`: they
would count the same work twice.

## Flow

- **Lead time.** How long does committed work take, and how predictable is
  it? Data: `started_at` → `closed_at` per closed item. Chart: scatterplot
  per item with p50 and p85 lines. Balance: failure-demand share. Pitfall: a
  claim made seconds before work starts turns lead time into agent touch
  time.
- **Throughput.** How many items finish per week? Data: `closed_at`. Chart:
  run chart. Balance: failure-demand share, so splitting work or skipping
  review does not look like progress. Pitfall: item sizes vary; read the
  trend, not single weeks.
- **Aging WIP.** Which items in progress are getting old? Data: today's
  `in_progress` items and their `started_at`, against the lead-time p85; no
  history needed. Chart: dot per item by age. Balance: throughput. Pitfall:
  items claimed as "next" but not started look old.
- **WIP over time.** Is work started faster than it finishes? Data: status
  per day from git snapshots. Chart: run chart. Balance: throughput; Little's
  law ties both to lead time. Pitfall: fewer items in progress can mean
  idle agents, not better flow.
- **Cumulative flow diagram.** Where does work pile up? Data: status
  transitions from git snapshots. Chart: stacked area per status. Balance:
  lead time. Pitfall: with only open, in progress and closed, it shows
  little; it needs a status per stage.
- **Flow efficiency.** How much of the lead time is active work? Data: agent
  turn durations on the item's branch ÷ lead time. Chart: run chart per
  week. Balance: failure-demand share. Pitfall: active agent time is not the
  user's time; high efficiency can hide long reviews.

## The user's attention

- **Review wait.** How long does finished work wait for the user's review?
  Data: human-review gates, `created_at` → `closed_at`. Chart: scatterplot
  with p50 and p85. Balance: failure-demand share, against rubber-stamp
  reviews. Pitfall: a gate that also spans the fixes measures the review
  loop, not the wait.
- **Response gap.** How long do agents wait for the user's next prompt?
  Data: transcript turn end → next human prompt. Chart: scatterplot with p50
  and p85. Balance: interruptions per item. Pitfall: mixes waiting with time
  away; read it as an upper bound.
- **Interruptions per item.** How often does an agent need the user? Data:
  `AskUserQuestion` calls, permission requests, interrupts and denied tool
  calls in transcripts, per branch. Chart: run chart per week. Balance:
  failure-demand share, since fewer questions can mean more wrong guesses.
  Pitfall: one long session per item inflates the count.

## Quality

- **Failure-demand share.** How much work exists because earlier work was
  wrong? Data: children of findings gates, issues labelled `rework`,
  reopens, and `fix:` commits among all commits. Chart: run chart per week.
  Balance: throughput. Pitfall: self-applied labels under-count; `fix:`
  commits also fix external breakage, so treat them as candidates.

## Economics

- **Usage per closed item.** How much subscription does one finished item
  cost? Data: status-line `rate_limits`, logged over time; the logger is its
  own first increment. Chart: run chart per week. Balance: failure-demand
  share. Pitfall: account-wide, so other projects inflate it; read it as an
  upper bound.
- **Tokens or cost per item.** A proxy for usage that has history. Data:
  deduplicated transcript usage or `cost-state` totals per branch. Chart:
  run chart per week. Balance: throughput. Pitfall: a proxy; compare it with
  usage per closed item once both exist.
