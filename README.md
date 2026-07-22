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
