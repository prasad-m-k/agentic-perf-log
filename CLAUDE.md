# Agent instructions: self-logging to the correction tracker

If you are an AI coding agent working in this repo, log every completed task
to `agent-correction-tracker.xlsx` once a human has reviewed your output and
either accepted it as-is or asked for a fix.

Run this after each reviewed task, not before. Logging your own work before
a human has looked at it defeats the point.

```bash
python3 update_log.py \
  --task-id "<short id or branch name>" \
  --category "<Test Generation|New Feature|Refactor|Bug Fix|Documentation|Config/Boilerplate|Other>" \
  --agent "Claude Code" \
  --corrected "<yes|no>" \
  --correction-type "<only if corrected=yes, see below>" \
  --notes "<one sentence, what happened>" \
  --logged-by agent
```

Correction types, pick the closest one:
- Wrong Assumption: you guessed at something ambiguous in the task and guessed wrong
- Missed Edge Case: the main path worked, an edge case did not
- Scope Creep: you changed more than the task asked for
- Style Mismatch: functionally correct, did not match existing conventions
- Logic Error: the core logic was wrong on the main path, not an edge case

Be honest here. The point of this log is an accurate picture of where you
need tighter task specs or closer review, not a clean record. A correction
you log yourself is worth more than one a human has to go dig up later.

If a human tells you directly "no changes needed, ship it," log
`--corrected no`. If they correct anything before merging, even something
small, log `--corrected yes` with the closest matching type.
