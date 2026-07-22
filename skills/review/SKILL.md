---
name: review
description: Review open pull requests against their linked Linear issues and post a three-group verdict with labels. Use when asked to review the PR queue or run the reviewer loop. Designed to run under /loop; never merges.
---

# Reviewer loop

One pass = one PR reviewed. Under /loop, each iteration runs this skill once.

## 1. Find a PR needing review

```bash
gh pr list --state open --json number,title,labels,isDraft,url
```

Skip drafts. Skip PRs already labeled `loop-approved` or
`loop-changes-requested` unless new commits landed after the last review
comment. If nothing needs review, say so and end the pass.

## 2. Review it

- Parse the linked issue from the PR body (`Closes TEAM-NNN`) and fetch it
  from Linear, including comments. No linked issue is itself a must-fix
  finding.
- Review against the linked issue ONLY. Look for: acceptance criteria gaps,
  bugs, broken data flow, unnecessary scope expansion, security issues, bad
  abstractions, missing loading/error states, and code future agents will
  find hard to modify. Do not suggest unrelated improvements unless severe.
- Read the full diff (`gh pr diff NUMBER`) and the changed files in context.
- Every must-fix finding starts with one of:
  - `[AC-N]` — the PR does not satisfy that acceptance criterion
  - `[DEFECT]` — the implementation is broken while staying inside scope
  - `[SECURITY]` — a severe security issue blocks shipping
- Non-goals are binding. If fixing a finding would require something an NG-N
  excludes, do not prescribe code: write `[SCOPE-CONFLICT AC-N ↔ NG-N]` with
  the exact contradiction, add the `needs-human-review` label, and leave the
  decision to the user. A reviewer finding never expands the issue.

## 3. Post the verdict

Post ONE comment (`gh pr comment NUMBER --body-file ...`) in exactly this
structure:

```md
## Review

Summary: one or two plain-language sentences on what this PR does for the
user or business.

## 1. Must fix before merge

None.

## 2. Should fix soon

None.

## 3. Safe to merge

Yes.
```

Then set labels:

- Must-fix empty: `gh pr edit NUMBER --add-label loop-approved --remove-label loop-changes-requested`
- Must-fix non-empty: `gh pr edit NUMBER --add-label loop-changes-requested --remove-label loop-approved`

(`--remove-label` errors if the label is absent — drop the flag in that
case.) Use comments plus labels, not formal GitHub reviews: the loop often
runs on the PR author's own token, and GitHub rejects self-reviews.

## 4. Hard limits

- Never merge. Never push commits to the PR branch.
- Never approve or request changes via a formal GitHub review.
- A `loop-approved` label means "no must-fix findings" — the human still
  makes the merge call.
