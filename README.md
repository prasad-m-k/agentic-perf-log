# Agent Correction Tracker

A lightweight way to track whether an AI coding agent's output needed
correcting, and what kind of correction it needed. Not tokens, not
latency, not CPU cost. This tracks judgment quality: did the agent make
a good call on the task it was given.

Background on why this exists: [link to your Medium post here].

## Files

- `agent-correction-tracker.xlsx` — the tracker itself. `Log` tab is where
  entries go, `Dashboard` tab summarizes automatically, `README` tab has
  the same instructions as this file.
- `update_log.py` — lets a script or an AI agent append a row to the Log
  tab from the command line, instead of a human doing it by hand.
- `CLAUDE.md` — instructions for coding agents (Claude Code or similar) to
  log their own corrections after a human reviews their work.

## Quick start

1. Download `agent-correction-tracker.xlsx`, open it in Excel, LibreOffice,
   or upload it to Google Sheets.
2. Delete the four yellow example rows on the `Log` tab once you're ready
   to log real data.
3. Add one row per reviewed task. The `Dashboard` tab updates on its own.

## Having the agent log itself

Drop `update_log.py` and `CLAUDE.md` into your project. If you're using
Claude Code or a similar CLI agent, it will pick up `CLAUDE.md`
automatically and log its own corrections after each reviewed task:

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

The agent still only logs after a human has reviewed and either accepted
or corrected its work. This isn't the agent grading its own homework
unsupervised, it's the agent filling out the form after you've already
made the call.

## Correction types

| Type | Meaning |
|---|---|
| Wrong Assumption | Agent guessed at something ambiguous, guessed wrong |
| Missed Edge Case | Main path worked, an edge case did not |
| Scope Creep | Agent changed more than the task asked for |
| Style Mismatch | Functionally fine, did not match conventions |
| Logic Error | Core logic wrong on the main path |

## Why this instead of token or CPU metrics

Cost and speed metrics tell you what a task cost to run. They say nothing
about whether the output was good. Correction rate and correction type
are a closer proxy for what actually matters: is this agent's judgment
getting better over time, and on what kind of work specifically.

## License

MIT. Use it, fork it, change the categories to fit your own workflow.
