---
name: engineer-flow
description: Conduct the pack's full pipeline end-to-end — a pre-run interview writes the score (which gates stop for a human, which models play which roles), then subagents play each stage in clean contexts.
disable-model-invocation: true
---

# Engineer-flow

Run the pack's pipeline — grilling → PRD → gaps → slices → build → review — as one performance. You are the **conductor**: you hold the long-horizon context, the judgment calls, and the relationship with the human. You play the interview-shaped stages yourself — grilling, the PRD interview, the slice quiz — because the human relationship is never delegated. Everything with a file-in/file-out shape — scouting, cold reads, building, reviewing, chores — is done by a **player**, a subagent in a clean context, cast on the cheapest model that can play the part well. The **score**, written in a pre-run interview, decides where the human enters.

The run at a glance — each stage produces an artifact and ends at a gate:

| # | Stage | The human sees | Artifact | Gate (default) |
|---|---|---|---|---|
| 1 | grilling | interview, one question at a time | `./grill-<slug>.md` | goal-locked (human) |
| 2 | write-a-prd | interview + module check; then a silent gaps loop | `prds/<slug>.prd.md` | gaps-clean (auto) → prd-approved (human) |
| 3 | prd-to-slices | the breakdown quiz | `slices/<slug>/` | slices-approved (human) |
| 4 | build | a run branch growing commits | `run/<slug>` branch | slice-done, per slice (conductor) |
| 5 | code-review | nothing until findings land | review reports | review-clean (conductor) |
| 6 | ship | run summary + a landing recommendation | merge or PR | ship (human) |

The score itself lives at `runs/<slug>/score.md` — written before stage 1, logged at every gate.

Run the conductor on the strongest model available (Fable/Opus-class). Casting economizes on players, never on the conductor. Spawning players and judging their reports is conducting, not playing.

## 1. Write the score

Before anything runs, interview the user using the `grilling` skill's interview loop — one question at a time, recommended answer included. (The loop only: this interview's deliverable is the score, not a grill doc.) Cover, about the RUN, not yet the feature:

1. **The goal** — one sentence is enough; stage 1 grounds it properly. Derive the run's `<slug>` from it and reuse that slug everywhere a stage wants one.
2. **Where to enter** — full pipeline, or mid-way: a PRD already exists → start at stage 2's gaps loop; slices exist → start at build.
3. **The gate map** — present the defaults below; let them tighten or loosen each gate.
4. **Casting** — present the default casting; let them set the cost posture: **thrifty** drops the frontier-cast players (Spec reviewer, Diagnostician) to mid, **premium** lifts mid players to frontier, **balanced** keeps the table as-is — no posture touches the conductor. Map tiers to the models the harness offers today and record the mapping in the score.
5. **AFK behavior** — when the human is needed and nobody answers, whether at a gate or mid-interview: **park** (write a **Paused at** block into the score — the stage, everything in flight, the open question — plus a log line, so a fresh conductor resumes from the score alone) or **proceed-and-flag** (continue on conductor judgment, flag the decision loudly in the log).

Write the answers to `runs/<slug>/score.md` (template below). The score is the run's single source of truth — a fresh conductor must be able to resume the performance from the score and the stage artifacts alone.

## 2. Gates

A **gate** sits after every stage. Three settings:

- **human** — stop; present the stage artifact, your one-paragraph verdict, and a recommendation; wait.
- **conductor** — judge the exit criterion yourself, proceed, log the decision.
- **auto** — proceed the moment the mechanical exit check passes. Auto is only available where the check IS mechanical — a judgment call floors at conductor. And it governs the stage's exit only: a human question surfacing mid-stage routes per Perform rule 3, it doesn't inherit the auto.

Where a stage's skill says "iterate until the user approves" — or checks a decision with the user mid-stage — the gate's setting decides who that approver is: human, or the conductor answering per Perform rule 3.

| Gate | After | Default | Exit check |
|---|---|---|---|
| goal-locked | grilling | human | grill doc complete and self-contained |
| gaps-clean | the PRD's gaps loop | auto | fresh cold reader says NO GAPS |
| prd-approved | write-a-prd | human | PRD in `prds/`, gaps-clean already passed |
| slices-approved | prd-to-slices | human | slice-folder gaps loop empty AND breakdown quiz passed |
| slice-done | each slice built | conductor | merged, tests green, acceptance criteria checked |
| review-clean | code-review | conductor | no finding the conductor judges must-fix (documented-standard breaches always are) |
| ship | end | human | — |

A human gate that answers with edits hasn't passed: fold the edits in, re-run the stage's exit check — an edited PRD meets a fresh cold reader — and present again.

Two overrides sit above every gate and AFK setting — nothing relaxes them, proceed-and-flag included:

- A slice typed **HITL** in its slice file always gets its human moment — a decision goes to the human BEFORE its Builder plays, a design review when the build lands — and its slice-done gate is human.
- **Escalation exhaustion** — the ladder in §3 running out — and anything destructive or irreversible always go to the human.

## 3. Casting

Cast each role on the cheapest tier that can play it. Two tiers: **frontier** (Fable/Opus-class — taste and judgment) and **mid** (Sonnet-class — strong execution where the taste already lives in the handoff files). Nothing casts below mid — small-model savings aren't worth the rework.

| Role | Plays in | Tier | Why |
|---|---|---|---|
| Conductor | everything | frontier | judgment, long horizon, the human |
| Interviewer | grilling, PRD interview, slice quiz | the conductor itself | never delegate the human relationship |
| Scout | codebase exploration for PRD / slices | mid | breadth, not taste |
| Cold reader | every gaps-report round | mid | skeptical comprehension, cheap enough to loop |
| Builder | one per slice, running `tdd` | mid | the slice + standards carry the taste; the model supplies capability |
| Standards reviewer | code-review, axis 1 | mid | matching documented rules and labelled smells |
| Spec reviewer | code-review, axis 2 | frontier | judging intent, not text |
| Diagnostician | a Builder stuck past escalation | frontier | hard bugs are judgment |
| Stagehand | scaffolding, verify runs, formatting, merges, file moves | mid | mechanical, but still touches the tree |

**Escalation.** A player *fails* when it returns without doing its job — tests red, task incomplete, report unusable — or its gate rejects the work. (Review findings sent back to a Builder are new work, not a failure — no rung climbed.) The ladder:

1. ONE re-run, one tier up, with the failure pasted into the brief. Frontier players skip this rung.
2. Builders only: a failed re-run goes to the Diagnostician (see Off-score skills); only a failed diagnosis climbs on.
3. The human — always the last rung, regardless of the gate map. No ladder ends anywhere else.

The Cold reader stands off the ladder: its brief is fixed, so an unusable report just gets another fresh reader — and a gap that keeps returning across rounds goes to the human, as its own skill says.

## 4. Perform

Walk the stages the score lists, in order. Four rules bind every player:

1. **Files are the only handoff.** Brief a player with paths — never a summary of what the file already says. (Rule 3's pre-answers are the one sanctioned extract: they answer a named checkpoint, they don't replace the files.) That is the pack's whole design; don't defeat it from the podium.
2. **Clean context per player.** Spawn via the subagent (Task) tool with its model option set from the score's tier mapping; if the harness can't set a per-agent model, the briefs still carry the role. Name every player `<model>-<role>` — the cast model as a short slug prefixing the role, e.g. `son-5-builder-03`, `opus-4.8-spec-reviewer` — so the run's transcript shows at a glance who played what on which model; an escalation re-run gets the new tier's slug, which is how a climbed ladder stays visible. Name the pack skill the player must run (e.g. "run the `tdd` skill on this slice"). End every brief with: state your confidence and anything you could not verify. Exception: the Cold reader — `gaps-report` fixes its brief verbatim, and its NO GAPS reply stays untouched.
3. **The conductor is the player's user.** A running player has no live channel back to you, so:
   - Pre-answer checkpoints you can see coming: a Builder's brief answers what it can of tdd's planning checklist — the interface changes and the behaviors to test, extracted from the slice, overview, and PRD — and states that the brief IS the plan approval tdd asks for.
   - Tell every player: a "confirm with the user" your files don't answer means stop and return the question with your state.
   - Answer a returned question from the run's artifacts; one they can't answer is a human question — ask if the human is around, otherwise route per AFK behavior. Respawn a fresh player with the answer folded into the brief. That chain — artifacts → human → AFK — is how every mid-stage question routes, a gaps loop's decision gaps included.
   - Interview-shaped stages talk to the human directly — no chain needed while the human is already in the room.
4. **Stage boundaries beat embedded steps.** When a stage's skill embeds work another stage owns, run it once, where the score puts it: `write-a-prd`'s grilling step is already satisfied by stage 1's grill doc; its exit gaps loop and `prd-to-slices`'s exit gaps loop run with cast Cold readers under their gates above.

Stage by stage:

1. **grilling** — conductor with the human, per the skill; cast a Scout for any explore-the-codebase questions → `./grill-<slug>.md`. Gate: goal-locked.
2. **write-a-prd** — conductor plays every interview-shaped step: the interview from the grill doc AND the skill's module-sketch checks (do the modules match expectations, which modules get tests — those answers feed every Builder's brief); cast a Scout for the codebase exploration and fold its report in → `prds/<slug>.prd.md`. Its exit gaps loop: a fresh Cold reader each round (never reuse one — it's warm), fold every gap into the document yourself — folding is authorship, and authorship is conducting — loop until NO GAPS. Gates: gaps-clean, then prd-approved — the human approves the post-gaps PRD, so the loop never mutates an approved document.
3. **prd-to-slices** — conductor drafts and quizzes per the skill; a Scout can pre-read the code → `slices/<slug>/`. Run its exit gaps loop on the slice folder the same way. Gate: slices-approved.
4. **build** — cut a run branch `run/<slug>` from `HEAD`, `git rev-parse` it, and log the ref AND the branch it was cut from: the ref is the review's fixed point, the base is ship's merge target, and every slice lands on the run branch. One Builder per slice, in dependency order from `00-overview.md`, each committing to the run branch. Unblocked slices may run in parallel ONLY in isolated worktrees, each on a per-slice branch (`run/<slug>-NN`) cut from the run branch — git won't share one branch across worktrees. A Stagehand owns that plumbing: it sets up each worktree and branch, merges finished branches back as each lands, and sweeps both once merged. A textual conflict is a Stagehand failure — the §3 ladder takes it; the run branch going red after a clean merge is the landing slice's Builder failing its gate — that Builder's ladder takes it. "Merged" in slice-done means the slice's commits are on the run branch with tests green there. Each Builder's brief hands over paths — its slice file, the overview, the parent PRD, the repo's standards sources (found the way `code-review` locates them; none documented → proceed and flag it in the log, that's a failed agent-ready pre-flight talking) — and names the vocabulary skills (see Off-score skills). Gate per slice: slice-done (HITL slices → human, always).
5. **code-review** — check out `run/<slug>` first: the skill diffs the fixed point against HEAD. The conductor runs the skill itself — Standards and Spec players in parallel, cast per §3 on the score's tier mapping (the model is what casting sets, wherever the harness allows), fixed point = the logged build ref. The Spec player's spec is the whole `slices/<slug>/` folder plus the parent PRD; brief it to tag each finding with the slice it falls under. Standards findings arrive by file/hunk — attribute them to slices yourself via the run branch's per-slice commits. Aggregate the reports into `runs/<slug>/review.md` (that's the gate's artifact path), send findings worth fixing back to a Builder on the responsible slice, then re-run both review players against the same fixed point — review-clean judges the latest reports.
6. **ship** — present a run summary built from the score's log, plus a landing recommendation: merge `run/<slug>` into its logged base, or open a PR from it. The human picks; a Stagehand executes; log the result. Edits at this gate are code, not prose — route them like review findings: a Builder on the responsible slice, then back through stage 5. Gate: ship.

After every gate decision — and every escalation, park, or override — append one line to the score's **Log**: stage or gate, who decided (human / conductor / auto), verdict, artifact path. A human returning from AFK reads the log, not the transcript.

## Off-score skills

Skills the score doesn't list as stages — the sidelines — and when the conductor reaches for each:

- **agent-ready** — pre-flight. Cast a Scout to audit its five checks (read-only) alongside the score interview; take failures to the human — scaffolding is their call, and before stage 1 beats mid-run. Every player plays better in a repo that passes.
- **prototype** — when the PRD or slicing interview hits a design question words can't settle. Cast a mid player to build the throwaway; log the answer in the score AND fold it into the PRD or slice it settles — the skill wants the answer captured durably, and the score alone isn't the repo. A Stagehand sees the delete-or-absorb step through.
- **diagnosing-bugs** — the Builder's post-re-run rung on the escalation ladder (§3). The Diagnostician replaces the Builder; scope its brief to the diagnosis loop and a verified fix, not the skill's closing architecture handoff.
- **codebase-design** — the deep-module vocabulary (module, interface, depth, seam). Name it wherever design gets sketched: the PRD's module step, Builder briefs.
- **domain-modeling** — keeper of `CONTEXT.md` and ADRs. Scouts just read those; name the skill in Builder briefs so model-changing decisions get recorded as they crystallize.
- **handoff** — the human's pause button, fired by name mid-stage. Point its document into `runs/<slug>/` and log it — the resume story lives in the run's files, not the OS temp dir. Between stages the score's log already is the handoff; park writes the Paused-at block for exactly that reason.

## Score template

The header, tier mapping, AFK behavior, and Log are always filled; the two override sections record only deviations — the defaults' single source of truth is this file. Paused at exists only while parked.

```markdown
# Run: <feature> — score

> Goal: <one sentence> · Slug: <slug> · Entered at: <stage> · Cost posture: <thrifty|balanced|premium>

## Tier mapping

frontier → <model> · mid → <model>

## Gate overrides

<a |Gate|Setting| table of only the gates that differ from the defaults — "none" if none>

## Casting overrides

<only roles cast off-default — "none" if none>

## AFK behavior

<park | proceed-and-flag>

## Paused at

<only while parked: the stage, everything in flight, the open question>

## Log

| When | Stage / gate | Decided by | Verdict | Artifact |
|---|---|---|---|---|
```
