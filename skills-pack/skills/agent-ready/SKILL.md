---
name: agent-ready
description: Audit a repo against the agent-ready checklist — onboarding file, one-command verification, conventions + golden path, territory map, guardrails — then scaffold what's missing. Use when the user wants to make a repo agent-ready, set up a CLAUDE.md / AGENTS.md, or asks why the agent keeps getting the repo wrong.
---

An agent starts every task cold — no memory of the last session, no tribal knowledge. Agent-ready means the repo itself answers what a new senior hire would ask in week one: what is this, how do I check my work, how do we do things here, where does everything live, what must I never touch.

Five checks. Audit before you scaffold — a wrong file is worse than a missing one, because the agent believes the repo.

## The five checks

Each check reads *what passes* → *how to verify it*.

1. **Onboarding file** — a root-level `CLAUDE.md` (or `AGENTS.md`) the agent reads on every task. Passes when it exists, stays around a screen in length, and every claim in it is true. → Read it, then spot-check each claim against the repo: the commands run, the paths exist, the described stack matches the lockfile. Short-and-true beats long-and-stale — one stale line fails the check even if the file is beautiful.

2. **One-command verification** — a single command that tells the agent whether the repo is broken, floor to ceiling: floor = compiles / lints / runs, ceiling = tests. Passes when the onboarding file names one command and that command actually discriminates. → Run it on the clean tree (must pass), then read what it runs — a command that can't fail on a type error or a broken test is a lie the loop will believe.

3. **Conventions + golden path** — "our way" written down, plus one real example to copy. The patterns you'd correct in code review are the ones the agent can't guess. Passes when a short conventions doc exists AND points at one golden-path file or flow in the repo that actually follows it. → Open the golden path and check it against its own conventions doc.

4. **Territory map** — where things live and where a change usually starts. A few lines, not a wiki. Passes when the onboarding file names the key directories and entry points, and they exist. → Resolve every named path.

5. **Guardrails** — the off-limits list: how secrets are handled (never committed), which files are generated (regenerate, don't hand-edit), what the agent must never do (migrations, prod config, force-push…). Passes when the onboarding file states them explicitly. → Some of these only the team knows — this check is allowed to end as "ask the user".

## Process

### 1. Audit

Explore the repo and score each check **pass / fail**, with the evidence that decided it — the claim you spot-checked, the command you ran, the path that didn't resolve. No check passes on intention; each passes on what you verified.

### 2. Scorecard

Present the five checks as a table: pass/fail · evidence · proposed fix for each fail. Where a fix needs knowledge only the team holds (which verify command is canonical, what's off-limits), ask — one question per gap, with your recommended answer.

### 3. Scaffold

With the user's approval, fix every fail:

- **Onboarding file** — write or prune it. When in doubt, cut: every line is context the agent pays for on every task.
- **Verification** — wire the one command (a `verify` script in the project's task runner is usually enough) and name it in the onboarding file.
- **Conventions + golden path** — derive candidates from the code: the pattern the repo already repeats most is "our way" until the user says otherwise. Confirm, write the short doc, and link the single best existing example as the golden path. Don't invent conventions the repo doesn't show — with one exception: seed the doc with a **simplicity bar** as a house rule, since it's a default that holds regardless of what the repo currently does. State it plainly: *the simplest thing that works — stdlib before a dependency, native platform features before custom code, one line before fifty, and question whether the change needs to exist at all (YAGNI).* Every agent run reads the onboarding file, so this rule lands on every code-generating task without needing a skill to fire. (For a harder per-change pass, point the team at the external `ponytail` skill — `npx skills add DietrichGebert/ponytail` — which enforces exactly this bar.)
- **Territory map and guardrails** — sections in the onboarding file.

### 4. Verify the fix

Run the verification command. Then run the `/gaps-report` skill (it ships in this pack) on the onboarding file — a cold reader who gets only that file should know how to start a change and how to check it. Done when all five checks pass and the gaps report comes back empty.
