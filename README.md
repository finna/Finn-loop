# Finn-loop

Three Claude Code skills that turn Linear + GitHub into an AI software
factory:

**idea → `/spec` interviews you and files the issue → you label it
`agent-ready` → `/build` claims it and opens a PR → `/review` posts a
verdict → you merge.**

Three skills, one label, one rule: **humans merge**.

- `skills/spec` — interviews you about a raw idea until it's unambiguous,
  then files a Linear issue with acceptance criteria (AC-N) and non-goals
  (NG-N).
- `skills/build` — claims the top `agent-ready` issue, implements only the
  acceptance criteria, opens a PR. Runs on repeat with `/loop /build`.
- `skills/review` — reviews open PRs against their linked issue only, posts
  a three-group verdict, labels the PR. Runs on repeat with `/loop /review`.

## Install

Paste this into Claude Code inside the repo where you want the factory:

```
Set up the Finn-loop software factory from https://github.com/finna/Finn-loop

1. Copy the three skills from that repo into this repo:
   .claude/skills/spec/SKILL.md
   .claude/skills/build/SKILL.md
   .claude/skills/review/SKILL.md
   Fetch each file's raw content from GitHub and write it verbatim — do not
   rewrite or summarize the skills.

2. Ask me for my Linear team key (e.g. ENG) and replace every TEAM
   placeholder in the copied skills with it.

3. Check that the Linear connector is available. If it isn't, tell me to
   connect it (Claude Code → Connectors → Linear) and wait.

4. Check `gh auth status` works. If not, tell me to run `gh auth login`.

5. Create these labels: in Linear, `agent-ready` and `blocked`; in the
   GitHub repo, `loop-approved`, `loop-changes-requested`, and
   `needs-human-review` (`gh label create ...`).

6. Smoke test: list Linear issues labeled agent-ready and list open PRs.
   Both commands succeeding = installed. Then tell me how to run my first
   loop.
```

## Daily rhythm (~15 minutes)

1. Dump ideas with `/spec` whenever they hit you. Read the filed issue; if
   you like it, apply `agent-ready` in Linear. That label is the approval
   gate — only you apply it.
2. Kick off `/loop /build` (and `/loop /review` in a second session) and
   walk away.
3. Come back to reviewed PRs: merge the `loop-approved` ones you're happy
   with, answer any `blocked` questions in Linear.

## The rules that make it work

- If it's not in the Linear issue, it doesn't exist. No side-channel
  instructions.
- One issue per PR, sized to a day of agent work or less.
- Acceptance criteria are observable outcomes; non-goals are binding. A PR
  comment or review can never silently expand scope — only editing the
  issue can.
- Spec quality is the bottleneck. Vague acceptance criteria produce
  confident wrong PRs; let `/spec` ask as many questions as it wants.
- Agents never merge. Ever.
