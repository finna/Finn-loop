---
name: build
description: Claim the next agent-ready issue from Linear, implement it, and open a PR. Use when asked to work the queue, run the builder loop, or pick up the next task. Designed to run under /loop; one pass does one unit of work.
---

# Builder loop

One pass = one unit of work: fix review feedback on an existing PR, or build
one issue end to end. Under /loop, each iteration runs this skill once.

## 0. Review feedback first

Check for open PRs labeled `loop-changes-requested`:

```bash
gh pr list --state open --label loop-changes-requested --json number,title,headRefName,url
```

If any exist, read the PR, its linked issue, and the latest review verdict.
Check out the branch, fix ONLY the "Must fix before merge" items, push,
remove the label (`gh pr edit NUMBER --remove-label loop-changes-requested`),
comment on the PR describing what changed, and end this pass.

If a must-fix remedy would require something the issue's non-goals exclude,
do NOT implement it. Comment the conflict on the PR, label it
`needs-human-review`, and end the pass. A review comment cannot expand the
issue's scope — only the user can, by editing the issue.

## 1. Pick

List Linear issues labeled `agent-ready` that are unassigned, sorted by
priority. If the queue is empty, say so and end the pass. Do not invent work.

## 2. Claim (the lock)

Assign yourself the issue in Linear and move it to In Progress. The assignee
is the lock: never work an issue assigned to someone else. Claim BEFORE
reading deeply or writing code, so no other loop grabs it.

## 3. Read

Fetch the full issue including comments. Implement only the acceptance
criteria. Non-goals are binding. No unrelated changes, no opportunistic
refactors. Compare every AC-N against every NG-N before writing code; if an
acceptance criterion cannot be implemented without crossing a non-goal, or
the spec is genuinely ambiguous, go to step 7 — never guess.

## 4. Build

- Branch from latest origin/main, named `TEAM-NNN-short-slug`.
- Implement the acceptance criteria. Add or update tests when the change
  affects logic, data flow, permissions, integrations, or user-visible
  behavior.
- Follow the existing code style, architecture, and naming.

## 5. Verify

Run the project's relevant checks (lint, typecheck, and the narrowest useful
test command) on your changes. All must pass before opening a PR. Fix
failures you caused; if a failure is pre-existing and unrelated, say so
plainly in the PR.

## 6. Ship

Push and open a PR with `gh pr create`. The description must include:

- What changed and why
- `Closes TEAM-NNN`
- A scope ledger: one line per AC-N with concrete evidence it is
  implemented, one line per NG-N with evidence it was preserved, and
  `Other behavior changes: None` (if that line is not true, stop and get
  the issue amended first)
- How to test: numbered manual steps, updated to match what you actually
  built
- Risk: Low / Medium / High

Then comment the PR link on the Linear issue and move it to In Review.

Never merge and never enable auto-merge. Humans merge. End the pass.

## 7. Blocked

If the spec is ambiguous or an AC conflicts with an NG, comment ONE specific
question the user can answer asynchronously on the Linear issue, apply the
`blocked` label, unassign yourself, and end the pass. When the user answers
and removes the label, a future pass resumes it with the answer in the
comments. Never guess, and never expand scope to route around ambiguity.
