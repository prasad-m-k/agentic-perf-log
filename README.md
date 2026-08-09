# Agent Correction Tracker

A lightweight way to track whether an AI coding agent's output needed
correcting, what kind of correction it needed, and what that correction
actually cost you in review time. Not tokens, not latency, not CPU cost.
This tracks judgment quality and delegation cost: did the agent make a
good call on the task it was given, and was handing it off worth your
time.

Background on why this exists: [Level Up to Be a Great Manager of Your Agent Swarm](https://medium.com/stackademic/level-up-to-be-a-great-manager-of-your-agent-swarm-112df69977b5).
The v2.0 metrics below are covered in [The Delegation Gap](https://github.com/prasad-m-k/agentic-perf-log).

## What's new in v2.0

v1.0 tracked whether a task got corrected and what kind of correction it
needed. v2.0 adds eight columns that capture the cost and context around
that correction: how much review time it took against writing it by
hand, how much of the agent's diff survived into the final commit, and
whether the agent stayed inside the task's stated scope. Same workbook,
same CLI, same workflow. Every v1.0 call to `update_log.py` still works
unchanged, the new fields are optional.

## Files

- `agent-correction-tracker.xlsx`: the tracker itself. `Log` tab is where
  entries go, `Dashboard` tab summarizes automatically across four
  metric groups, `README` tab has the same instructions as this file.
- `update_log.py`: lets a script or an AI agent append a row to the Log
  tab from the command line, instead of a developer doing it by hand.
- `CLAUDE.md`: instructions for coding agents (Claude Code or similar) to
  log their own corrections after a developer reviews their work.

## Quick start

1. Download `agent-correction-tracker.xlsx`, open it in Excel, LibreOffice,
   or upload it to Google Sheets.
2. Delete the four yellow example rows on the `Log` tab once you're ready
   to log real data.
3. Add one row per reviewed task. The `Dashboard` tab updates on its own.

## Having the agent log itself

Drop `update_log.py` and `CLAUDE.md` into your project. If you're using
Claude Code or a similar CLI agent, it will pick up `CLAUDE.md`
automatically and log its own corrections after each reviewed task.

Minimum call, same as v1.0:

```bash
python3 update_log.py \
  --task-id TASK-042 \
  --category "Bug Fix" \
  --agent "Claude Code" \
  --corrected yes \
  --correction-type "Missed Edge Case" \
  --notes "Off-by-one on empty list input, fixed before merge" \
  --logged-by agent
```

With the v2.0 fields:

```bash
python3 update_log.py \
  --task-id TASK-042 \
  --category "Bug Fix" \
  --agent "Claude Code" \
  --corrected yes \
  --correction-type "Missed Edge Case" \
  --notes "Off-by-one on empty list input, fixed before merge" \
  --logged-by agent \
  --risk-tier Medium \
  --spec-completeness 4 \
  --prompt-turns 2 \
  --review-time 8 \
  --est-manual-time 25 \
  --diff-retention 78.5 \
  --test-modified no \
  --non-goal-violation no
```

The agent still only logs after a developer has reviewed and either accepted
or corrected its work. This isn't the agent grading its own homework
unsupervised, it's the agent filling out the form after you've already
made the call.

**Multiple agents, one tracker.** If you're running more than one agent
against the same codebase, they can all log to the same workbook. Each
call to `update_log.py` takes an exclusive lock on the file first, so
concurrent writes from different agents queue up instead of one
overwriting another's row. No extra dependency needed, it's a plain file
lock. Tested with 8 simultaneous writers, all 8 rows landed with none
lost or clobbered. If a write can't get the lock within 30 seconds
(default, adjustable with `--lock-timeout`), it fails loudly instead of
silently corrupting the file.

## Correction types

| Type | Meaning |
|---|---|
| Wrong Assumption | Agent guessed at something ambiguous, guessed wrong |
| Missed Edge Case | Main path worked, an edge case did not |
| Scope Creep | Agent changed more than the task asked for |
| Style Mismatch | Functionally fine, did not match conventions |
| Logic Error | Core logic wrong on the main path |

## v2.0 fields

| Field | Values | What it captures |
|---|---|---|
| `--risk-tier` | Low, Medium, High | Contextualizes every other number. A 10% correction rate means something different on a Low-tier boilerplate task than on a High-tier auth change |
| `--spec-completeness` | 1 to 5 | How complete the task spec was before the agent started (5 = Goal, Boundaries, Edge cases, Output format, and Non-goals all present) |
| `--prompt-turns` | integer | Follow-up messages needed before the output passed. High counts usually point at the spec, not the agent |
| `--review-time` | minutes | Time spent reviewing the agent's output |
| `--est-manual-time` | minutes | Your estimate of how long the task would've taken to write by hand. Divided by review time, this is the Review-to-Authored Ratio on the Dashboard: above 1.0 means the agent cost you time instead of saving it |
| `--diff-retention` | percent, 0 to 100 | How much of the agent's generated diff survived into the final commit |
| `--test-modified` | yes / no | Did the agent edit or relax an existing test to make its own code pass. This is the one to watch closest |
| `--non-goal-violation` | yes / no | Did the agent edit files or scope outside what the task spec allowed |

## Dashboard

Four sections, all formulas, all update automatically as you add rows:

- **Review & Cognitive Overhead**: average Review-to-Authored Ratio, First-Pass Acceptance Rate, average Diff Retention Rate.
- **Context & Spec Fidelity**: average prompt turns, average spec completeness score, count of non-goal violations.
- **By Risk Tier**: total, corrected, and correction rate broken out by Low, Medium, High, with a bar chart.
- **Test Integrity & Safety**: Test Modification Rate and a count of tasks where the test suite itself got touched.

## What's not in here yet

Tautological Test Detection, Dependency Churn, and Bloat Delta all need
something scanning the actual diff, not a number typed in by hand after
review. They're a different kind of tool than a manual log, so they're
being worked on separately rather than bolted onto this schema half
finished.

## Why this instead of token or CPU metrics

Cost and speed metrics tell you what a task cost to run. They say nothing
about whether the output was good, or whether reviewing it cost you more
time than writing it yourself would have. Correction rate, correction
type, and the review-cost fields in v2.0 are a closer proxy for what
actually matters: is this agent's judgment getting better over time, on
what kind of work specifically, and is delegating still worth it.

## License

MIT. Use it, fork it, change the categories to fit your own workflow.
