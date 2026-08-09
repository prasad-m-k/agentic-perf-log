# Agent instructions: self-logging to the correction tracker (v2.0)

If you are an AI coding agent working in this repo, log every completed task
to `agent-correction-tracker.xlsx` once a developer has reviewed your output
and either accepted it as-is or asked for a fix.

Run this after each reviewed task, not before. Logging your own work before
a developer has looked at it defeats the point.

## Minimum call (always required)

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

## Full v2.0 call (fill in what you can, see field guide below)

```bash
python3 update_log.py \
  --task-id "<short id or branch name>" \
  --category "<Test Generation|New Feature|Refactor|Bug Fix|Documentation|Config/Boilerplate|Other>" \
  --agent "Claude Code" \
  --corrected "<yes|no>" \
  --correction-type "<only if corrected=yes>" \
  --notes "<one sentence, what happened>" \
  --logged-by agent \
  --risk-tier "<Low|Medium|High>" \
  --spec-completeness "<1-5>" \
  --prompt-turns "<integer>" \
  --review-time "<minutes, ask the developer>" \
  --est-manual-time "<minutes, ask the developer>" \
  --diff-retention "<percent, 0-100>" \
  --test-modified "<yes|no>" \
  --non-goal-violation "<yes|no>"
```

All eight v2.0 flags are optional. Fill in what you can determine yourself.
Ask the developer for anything you can't.

## Correction types

Pick the closest one:
- Wrong Assumption: you guessed at something ambiguous in the task and guessed wrong
- Missed Edge Case: the main path worked, an edge case did not
- Scope Creep: you changed more than the task asked for
- Style Mismatch: functionally correct, did not match existing conventions
- Logic Error: the core logic was wrong on the main path, not an edge case

## v2.0 field guide

Some of these you can work out yourself from the repo and the conversation.
Others only the developer knows. Don't guess at the ones you can't determine,
leave them off the call instead.

**You can usually fill these in yourself:**
- `--prompt-turns`: count your own follow-up exchanges on this task before the
  output was accepted, including clarifying questions either direction.
- `--spec-completeness`: rate the task spec 1-5 as it was given to you, before
  you filled in any gaps yourself. 5 means Goal, Boundaries, Edge cases,
  Output format, and Non-goals were all stated up front. 1 means you had to
  infer most of it. Rate the spec you got, not how well you executed.
- `--diff-retention`: if you can see the final committed diff versus what you
  originally generated, compute the percentage of your lines that survived.
  If you don't have visibility into the final commit, leave this off rather
  than estimate it.
- `--test-modified`: check whether any file you touched matches an existing
  test path and whether you changed an assertion, not just added a new test.
  Answer honestly even if the modification was justified, this field tracks
  whether it happened, not whether it was wrong to do.
- `--non-goal-violation`: check your own diff against the task's stated
  scope or `Non-goals` section, if one existed. If you edited a file outside
  what was asked, or made an architecture decision that wasn't part of the
  task, this is yes.

**Ask the developer for these, don't estimate them yourself:**
- `--review-time`: how many minutes they spent reviewing your output. You
  have no way to know this from inside the session.
- `--est-manual-time`: their estimate of how long the task would have taken
  to do by hand. Also not something you can infer.
- `--risk-tier`: Low (boilerplate, docs), Medium (business logic), High
  (auth, schema, anything touching production data). The developer usually
  has better judgment on this than you do, ask rather than guess, especially
  for anything you'd rate High yourself, since that's exactly the case where
  a wrong self-rating matters most.

If the developer doesn't answer or the exchange is asynchronous, log the
minimum call and skip the fields you don't have. A partial v2.0 row beats
a guessed one.

## General rules

Be honest here. The point of this log is an accurate picture of where you
need tighter task specs or closer review, not a clean record. A correction
you log yourself is worth more than one a developer has to go dig up later.

If a developer tells you directly "no changes needed, ship it," log
`--corrected no`. If they correct anything before merging, even something
small, log `--corrected yes` with the closest matching type.

If you flag `--test-modified yes` or `--non-goal-violation yes`, say so
plainly in `--notes` too. Those two fields are the ones a developer is most
likely to scan for first.
