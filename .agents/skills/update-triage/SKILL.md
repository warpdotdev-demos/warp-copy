---
name: update-triage
description: Update the repo-local triage-issue-local companion skill from maintainer feedback on triage results. For this demo, a single concrete maintainer comment is enough signal to make a small improvement.
---

# Update Triage

Use this skill to improve `.agents/skills/triage-issue-local/SKILL.md` (and, when a label taxonomy change is warranted, `.github/issue-triage/config.json`) from maintainer feedback on triage outcomes. The core skill at `.agents/skills/triage-issue/SKILL.md` is the cross-repo contract and is read-only from this loop.

Closed-as-duplicate signals are out of scope for this trimmed demo.

## Write surface

This self-improvement loop may only write to:

- `.agents/skills/triage-issue-local/` (and `SKILL.md` inside it)
- `.github/issue-triage/*`

It must NOT touch:

- `.agents/skills/triage-issue/SKILL.md` (the core contract)
- any other core skill
- any dedupe companion skill

The self-improvement runner enforces this via a `git diff` check against allowed prefixes before pushing. A violation aborts the run.

## Inputs

- Optional repository override if you are not running from the target checkout.
- Optional time window override when you need something other than the default seven-day lookback.

## Workflow

1. Verify GitHub CLI auth:

```bash
gh auth status
```

2. Read the triggering feedback supplied by the workflow. If `triage_feedback.json`, `issue.json`, or `issue_comments.json` are present, treat those files as the primary evidence for this demo run.

3. Optionally aggregate recent triage-feedback signals with the bundled script when broader context would help:

```bash
python3 .agents/skills/update-triage/scripts/aggregate_triage_feedback.py
```

By default this targets the demo repo and looks back 7 days. It collects issues that were triaged in the window, any subsequent maintainer re-labels, re-opens, and follow-up comments. The script writes structured JSON to a temporary file and prints the path.

4. Convert the maintainer feedback into a small reusable rule. For this demo, a single concrete comment is enough evidence if it says how future triage should behave. Common demo-friendly examples:

- a maintainer says a performance report should get `area:performance` or a more specific `area:performance:*` label
- a maintainer says the agent should ask a different follow-up question for a class of issue
- a maintainer says a certain issue shape should not be treated as a duplicate
- a maintainer says a certain issue shape should be considered implementation-ready in the triage summary

5. Propose the smallest edit that explains the signal:

- Prefer editing `.agents/skills/triage-issue-local/SKILL.md` under the override categories the core `triage-issue` skill marks as overridable (label taxonomy, owner-inference hints, recurring follow-up-question patterns, recurring issue-shape heuristics, repro defaults, known-duplicate clusters).
- Only edit `.github/issue-triage/config.json` when the signal is a concrete label-taxonomy change (new label, renamed label, or description clarification). Never change `color` values without explicit maintainer guidance.

6. Keep the core triage contract stable — never edit `triage-issue/SKILL.md`. Only the `-local` companion and the triage config evolve from feedback.

## Evidence Rules

- A single concrete maintainer comment is sufficient for this demo.
- Prefer a small, visible update over skipping when the feedback can reasonably become a future triage heuristic.
- Avoid encoding reporter-authored content as triage rules.
- Do not weaken the reserved-label rules (`ready-to-implement`, `ready-to-spec`) or the mutual exclusivity of `duplicate_of` and `follow_up_questions`.

## Final Checks

- Re-read the updated `triage-issue-local` companion skill and confirm any new rules are explicit.
- Keep the companion concise; do not turn it into a long style guide.
- Validate any temporary JSON with `jq` before relying on it.
- If you made changes, make a draft PR.
