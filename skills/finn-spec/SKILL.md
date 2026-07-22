---
name: finn-spec
description: Interview the user about a raw idea until confident, then file a build-ready issue in Linear. Use when asked to run Finn-loop's spec interview, draft a queue-ready issue, or plan a feature. Interactive — requires the user present; never run unattended.
---

# Spec interview

Turns a raw idea into a Linear issue so complete that a build agent needs
nothing beyond the issue. Works like plan mode: research the codebase,
interview the user in rounds until confident, draft, confirm, file. The user
is the product brain; you are the codebase brain. Never guess product
decisions.

## 1. Research before asking

Read the relevant code first. Find which files are involved, what patterns
already exist, and what constraints apply. Never ask the user something the
codebase can answer.

## 2. Interview in rounds

Ask 1-4 questions per round, each with concrete options and your recommended
option first. Ask only genuine product decisions:

- Behavior forks: who sees it, what exactly happens, where does it live
- Scope boundaries: what is explicitly out of this issue
- Edge cases that change acceptance criteria: empty states, permissions,
  failure handling
- Data implications: existing records, migrations

After each round, fold the answers in and apply the confidence test:

> Could two different engineers read this spec and ship the same observable
> behavior?

If any fork remains, ask another round. There is NO cap on rounds: a small
fix might need two questions; a big feature legitimately needs 10-20+. Never
stop early because it feels like a lot of questions. Once the test passes,
stop — no filler questions.

## 3. Draft the issue

Use exactly this shape:

```md
## Problem

What user or business problem does this solve? One or two sentences.

## Acceptance Criteria

- [ ] AC-1 — Observable, testable outcome one
- [ ] AC-2 — Observable, testable outcome two

## Non-goals

- NG-1 — What must NOT change in this task
- NG-2 — What is explicitly excluded or saved for later

## Relevant files

- path/to/file.ts — why it matters

## Test expectations

- What should be tested, manually or automatically

## How to verify

1. Numbered manual steps anyone can follow to confirm the work: where to
   go, what to do, exactly what should happen. Cover every AC.
```

Rules for the draft:

- Every acceptance criterion is an observable outcome with a stable `AC-N`
  id. Every non-goal has a stable `NG-N` id. These ids are the contract the
  build and review skills enforce.
- No acceptance criterion may require a non-goal. If one does, resolve it
  with the user before filing.
- Size the issue to one day of agent work or less. Bigger work becomes a
  chain of small issues, ordered so each is buildable using only merged
  code from the ones before it.

## 4. Confirm and file

Show the full draft in chat and get the user's go-ahead. Then create the
issue on the configured `TEAM` Linear team (via the Linear connector) with
the draft as the body. Report the exact issue identifier and URL returned by
Linear; later skills use that identifier rather than guessing it.

## Hard rule

Never apply the `agent-ready` label. The user applies it in Linear after a
final read — that label is the approval gate between "idea" and "an agent
builds it".
