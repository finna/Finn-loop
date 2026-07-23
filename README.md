# Finn-loop

Three Claude Code skills that turn Linear + GitHub into a small, human-gated
AI software factory:

**idea → `/finn-spec` interviews you and files the issue → you label it
`agent-ready` → `/finn-build` claims it and opens a PR → `/finn-review` posts
a verdict → you merge.**

Three skills, one approval label, one rule: **humans merge**.

- [`skills/finn-spec`](skills/finn-spec/SKILL.md) — researches the repo,
  interviews you until the behavior is unambiguous, then files a Linear issue
  with acceptance criteria (`AC-N`) and non-goals (`NG-N`).
- [`skills/finn-build`](skills/finn-build/SKILL.md) — claims the next safe
  `agent-ready` issue, implements only its contract, verifies it, and opens a
  PR. Runs repeatedly with `/loop /finn-build`.
- [`skills/finn-review`](skills/finn-review/SKILL.md) — reviews open PRs against
  their linked issue and required GitHub checks, then posts a three-group
  verdict. Runs repeatedly with `/loop /finn-review`.

The `finn-` prefix avoids collisions with Claude Code's bundled commands and
with generic personal skills such as `/review` or `/build`.

## Requirements

- A Git repository hosted on GitHub, with a working `origin` remote
- [Claude Code](https://code.claude.com/docs/en/overview) 2.1.71 or newer
  (`/loop` was added in that release)
- A Linear workspace and team
- The Linear connector enabled in Claude Code
- The GitHub CLI (`gh`) authenticated with write access to the target repo
- At least one required GitHub status check if you want fully automated
  `loop-approved` verdicts; without required CI, Finn-loop escalates the PR for
  human review

Recommended: connect Linear's
[GitHub integration](https://linear.app/docs/github-integration) so linked PRs
can update issue status when they open and merge.

## Install

Paste this into Claude Code inside the repo where you want the factory:

```text
Set up Finn-loop from https://github.com/finna/Finn-loop.

1. Copy these files from that repo into this repo, preserving their contents:
   skills/finn-spec/SKILL.md   → .claude/skills/finn-spec/SKILL.md
   skills/finn-build/SKILL.md  → .claude/skills/finn-build/SKILL.md
   skills/finn-review/SKILL.md → .claude/skills/finn-review/SKILL.md

2. Ask for my Linear team key (for example ENG), then replace every TEAM
   placeholder in the copied skills with that exact key.

3. Check `claude --version` is 2.1.71 or newer. Check that the Linear
   connector is available and can list the chosen team's labels and workflow
   states. If it is unavailable, tell me to connect it and wait.

4. Check `gh auth status` and `gh repo view` both work. Detect the repository's
   real default branch; do not assume it is main. Confirm the authenticated
   account can push to this repository.

5. Create missing labels idempotently:
   - Linear: agent-ready, blocked
   - GitHub: loop-approved, loop-changes-requested, needs-human-review
   Do not fail if a label already exists.

6. Confirm the Linear team has a workflow state of type "started". Finn-loop
   will prefer a state named "In Progress" but can use any started state. A
   separate review state is optional.

7. Recommend enabling Linear's GitHub integration if it is not already
   connected. Explain that without it, Finn-loop can post the PR link but a
   merge may not automatically move the Linear issue to Done.

8. Validate that all three copied SKILL.md files have valid YAML frontmatter.
   Tell me to run `/reload-skills` (or restart Claude Code), then have me
   confirm `/skills` lists finn-spec, finn-build, and finn-review.

9. Smoke test by listing:
   - unassigned Linear issues labeled agent-ready but not blocked
   - the target repo's default branch and required GitHub checks
   - open pull requests and their Finn-loop labels
   All reads succeeding and all three skills appearing in `/skills` means the
   installation is ready. Then tell me how to run my first spec and loop.
```

## Daily rhythm (~15 minutes)

1. Run `/finn-spec` whenever an idea hits you. Read the filed issue; if you
   approve the exact contract, apply `agent-ready` in Linear. Only a human
   applies that label.
2. Start `/loop /finn-build`. If you want reviews to happen continuously, run
   `/loop /finn-review` in a second session.
3. Merge only PRs that are `loop-approved`, conflict-free, and green on all
   required checks. A `needs-human-review` PR requires you to read and resolve
   the reason for escalation before merging.
4. Answer concrete questions on `blocked` Linear issues, then remove the
   `blocked` label so a future build pass can resume them.

Run only one builder loop per Linear team. The Linear assignee is a cooperative
lock between people, but two simultaneous sessions authenticated as the same
person cannot reliably lock each other. Use separate clean worktrees if you
intentionally operate on more than one repository task at once.

## What `loop-approved` means

`loop-approved` means the reviewer found no must-fix issue against the Linear
contract, all required GitHub checks passed, and the PR was not conflicting at
the reviewed commit. It is evidence for the human merge decision, not
permission for an agent to merge.

Finn-loop does not create CI for the target project. The target repository owns
its build, test, security, and deployment checks.

## The rules that make it work

- If it is not in the Linear issue, it does not exist. No side-channel
  instructions.
- One issue per PR, sized to a day of agent work or less.
- Acceptance criteria are observable outcomes; non-goals are binding. A PR
  comment or review cannot expand scope — only editing the Linear issue can.
- Blocked issues and human-escalated PRs leave the automated queue until a
  human resolves them.
- Spec quality is the bottleneck. Vague acceptance criteria produce confident
  wrong PRs; let `/finn-spec` ask as many questions as it needs.
- Agents never merge or enable auto-merge.

`/loop` runs only while its Claude Code session remains open. Watch the first
few passes and your usage before leaving a new installation unattended.

## From the starter loop to a full software factory

Finn-loop deliberately ships the smallest useful version: spec, build, review,
and a human merge. The larger factory that inspired this repository added the
layers below only after that core loop was stable.

These are **architecture patterns and next steps**, not features bundled in
this repository. Add them one at a time. Keep Linear and GitHub as the durable
sources of truth; use Slack and other interfaces as control surfaces, never as
shadow state.

| Layer | What it adds | Human boundary |
| --- | --- | --- |
| Self-converging PRs | A fresh reviewer checks each builder PR and the builder fixes must-fix findings | Stop after a small, fixed retry budget |
| Slack control plane | Blocked questions and merge-ready PRs reach founders where they already work | Slack actions are accepted only from approved users and rechecked against live state |
| Risk-aware merging | Narrow safe changes can merge automatically; sensitive changes always escalate | Policy decides what is reversible and safe |
| Verification gates | UI changes are tested on a real preview and behavior changes update architecture docs | Founders remain the final taste and risk check |
| Direction and status | A morning planner replenishes the queue and a read-only status skill returns the exact action list | A founder approves the work packet |
| Factory watchdog | Silent stalls, red CI, and unhealthy queues create actionable alerts | The watchdog alerts; it does not invent fixes |

### 1. Make build and review self-converging

In the starter, `/finn-build` and `/finn-review` can run independently. The
next step is to have the builder open its PR and then launch a **fresh reviewer
with clean context**. Do not let the builder review its own work from the same
conversation.

A proven convergence policy is:

1. The builder opens one PR for one Linear issue.
2. A fresh reviewer checks the exact head commit against the issue and required
   CI.
3. If the verdict is `loop-changes-requested`, the builder fixes only the
   must-fix findings.
4. A new fresh reviewer checks the new head commit.
5. After two failed fix rounds, label the PR `loop-stuck` and stop for a human.

The retry cap matters. An agent should not argue with a reviewer forever or
silently broaden the Linear contract to make a test pass.

### 2. Use Slack as a human control plane

Slack becomes useful when it removes trips to Linear and GitHub without
replacing either one.

| Channel | Event | Human action | Durable result |
| --- | --- | --- | --- |
| `#notifs-linear-blocked` | A builder blocks an issue with one concrete question | Reply in the message thread | Copy the answer to Linear, remove `blocked`, and return the unassigned issue to the queue |
| `#notifs-merge-ready` | A PR is `loop-approved`, CI-green, conflict-free, and still at the reviewed SHA | Review the preview/test steps and react with 🚀 | Re-read the PR from GitHub, then squash-merge only if every gate is still true |
| `#notifs-digest` | A scheduled daily snapshot summarizes merges, approvals, blockers, and queue depth | Use it to choose the day's founder actions | Claim one durable post per date so retries cannot create duplicate digests |
| `#notifs-alerts` | The watchdog detects an unhealthy pipeline | Investigate the named condition | Post once when opened and once when resolved; stay quiet while unchanged |

A safe Slack integration needs more than a webhook that calls merge:

- Verify Slack and GitHub webhook signatures before processing an event.
- Allow only explicitly configured founder Slack user IDs to approve an action.
- Store event IDs, message timestamps, PR numbers, and reviewed head SHAs so
  duplicate deliveries are harmless.
- Claim a post or reaction in durable storage before making an external call.
- Re-read Linear or GitHub at action time. Never trust an old Slack message as
  proof that an issue is still blocked or a PR is still merge-ready.
- Mark old merge messages as superseded when the PR head changes.
- Retry transient failures, but repeat the live safety checks before every
  retry.
- Provide environment-level kill switches for automated posting and merging.

The useful division of responsibility is:

- **Linear:** what should be built and whether an issue is ready or blocked.
- **GitHub:** the code, review commit, CI, conflicts, and merge state.
- **Slack:** human-facing questions, approvals, and alerts.
- **The repository:** how the factory behaves—skills, templates, and policy.

### 3. Add risk-aware merge policy carefully

The public starter's rule is intentionally simple: agents never merge. In a
more mature factory, automation can execute a narrowly pre-authorized merge,
but only after the team explicitly changes that governance rule.

A conservative policy has three lanes:

- **Human merge:** application code and anything involving schema, auth,
  permissions, billing, deployment, or provisioning.
- **Founder-authorized merge:** a founder's 🚀 reaction authorizes one exact PR
  head; the system re-verifies labels, CI, conflicts, and SHA before merging.
- **Safe auto-merge:** an opt-in allowlist such as docs and tests, plus an
  explicit override label and a global kill switch. Never infer safety merely
  from a small diff.

Keep a `needs-human-review` label for sensitive paths and a
`safe-auto-merge` label for deliberate exceptions. A PR that changes after
approval must earn approval again.

### 4. Use Linear relations for multi-part features

Large features should become a chain of one-day issues, connected with
Linear's blocked-by relations. All parts may be approved up front, but the
queue should hide a downstream issue until its blocker is Done, and the claim
operation should enforce the same rule.

This lets several agents work from a durable plan without allowing part three
to start before part one has established the contract it depends on.

### 5. Turn previews and documentation into merge gates

Unit tests are not enough for user-facing work. For a PR that changes rendered
UI, require the builder to open the deployment preview, sign in with a
non-production test account, run every founder-verification step from the
Linear issue, and record per-step `PASS` or `FAIL` evidence with screenshots.
The reviewer should treat missing or failed evidence as a must-fix finding.

Behavior-changing PRs should also update the matching architecture document.
A lightweight CI check can require the PR body to name the docs changed, or to
state why no docs change is justified. This makes documentation part of done
instead of cleanup that never happens.

### 6. Add direction and status skills

Two read-mostly skills remove a surprising amount of founder overhead:

- A **morning director** reads product goals plus live Linear and GitHub state,
  proposes a small daily packet, and files or promotes only the exact issues a
  founder approves in that session. It plans; it never starts builders.
- A **status inspector** reads open PRs and the `agent-ready`, `spec-drafted`,
  `needs-spec`, and `blocked` queues, then returns an ordered list of what the
  founder must merge, approve, or answer. It never mutates either system.

The separation is important: planning decides what enters the factory;
building should remain a boring execution step.

### 7. Add a factory watchdog

Event-driven notifications cover things that happen. A watchdog covers things
that silently stop happening. First normalize Linear and GitHub events into one
durable pipeline record per issue/PR, with derived stages such as awaiting
approval, ready, building, review, merge-ready, blocked, and done. Let the
status view, daily digest, and watchdog read that same model.

Run the watchdog on a schedule against the pipeline model plus current
default-branch CI, and alert on conditions such as:

- work stuck in building or review beyond a threshold;
- changes requested with no follow-up;
- ready work but no recent merges;
- an empty build queue or a growing spec/blocked backlog; and
- red required checks on the default branch.

Alert once when a condition opens, stay silent while it persists, and post a
short resolved message when it clears. A watchdog that repeats the same alert
every few minutes becomes noise and gets muted.

### 8. Run persistent workers with leases

`/loop` is a good first scheduler, but it still depends on an open interactive
session. When you move to always-on machines or a worker fleet, make jobs
durable instead of treating a process named "running" as proof of life.

- Give every job explicit queued, leased, running, succeeded, and failed state.
- Have workers renew short leases while they work. Expire abandoned leases,
  requeue up to an attempt limit, then fail visibly.
- Give each job a clean clone or worktree. Never let concurrent builders share
  a working directory.
- Keep worker, dispatcher, and human-operator credentials separate and
  least-privileged.
- Record heartbeats and terminal results so the watchdog can distinguish an
  empty queue from a dead scheduler.
- Keep the worker boundary conservative: agents may research, draft, code,
  test, and propose, but publishing, spending, production mutation, and merges
  still require the policy gates you chose above.

### 9. Build a post-merge learning loop

The highest-signal lessons often appear as reviewer must-fix findings and then
disappear once the PR is corrected. A learning loop can scan recently merged
PRs, read **every** earlier reviewer verdict, inspect the eventual fix, and
distill one reusable rule.

Store proposed rules as version-controlled files with applicability tags and
source PRs. Deduplicate against active rules; reinforce an existing rule when
the same failure class recurs. The agent opens a learning proposal PR, but a
founder decides whether that rule becomes factory policy by merging it. Agents
should never silently rewrite the instructions that govern them.

### 10. Next experiment: approve specs from Slack

This is the next logical control-plane extension, but it is not part of the
public starter. Treat `spec-drafted && !agent-ready` as "awaiting approval"
and post a versioned spec summary to a Slack review inbox. A founder can either
approve during the `/finn-spec` session or defer the decision and approve the
unchanged issue later with a Slack reaction.

Before applying `agent-ready`, re-read the Linear issue and confirm its version
matches the Slack message. If the spec changed, supersede the old message and
require a new approval. Start with approval-only reactions; a full
changes-request conversation is a separate workflow.

## Suggested implementation order

1. Run the three-skill starter for several real PRs and tune spec quality.
2. Add fresh-reviewer convergence and a `loop-stuck` escape hatch.
3. Add blocked-by issue chains and a read-only status skill.
4. Connect the blocked-issue Slack lane.
5. Connect the merge-ready Slack lane with live re-verification.
6. Add preview and documentation CI gates.
7. Introduce risk-tiered merging only after the earlier evidence is reliable.
8. Add the morning director and watchdog once queue volume justifies them.
9. Add leased persistent workers when open sessions become the throughput
   bottleneck.
10. Add the post-merge learning loop once reviews produce recurring findings.
11. Add versioned Slack spec approval last; it changes the most important human
   gate in the system.

The goal is not "no humans." It is for humans to handle product judgment,
policy, and irreversible exceptions while agents handle repeatable execution.
