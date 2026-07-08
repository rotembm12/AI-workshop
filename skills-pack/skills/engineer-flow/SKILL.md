---
name: engineer-flow
description: Conduct the pack's full pipeline end-to-end — a pre-run interview writes the score (which gates stop for a human, which models play which roles), then subagents play each stage in clean contexts.
disable-model-invocation: true
---

# Engineer-flow

Run the pack's pipeline — grilling → PRD → gaps → slices → build → review — as one performance. You are the **conductor**: you hold the long-horizon context, the judgment calls, and the relationship with the human. You play the interview-shaped stages yourself — grilling, the PRD interview, the slice quiz — because the human relationship is never delegated. Everything with a file-in/file-out shape — scouting, cold reads, building, reviewing, chores — is done by a **player**, a subagent in a clean context, cast on the cheapest model that can play the part well. The **score**, written in a pre-run interview, decides where the human enters.

Run the conductor on the strongest model available (Fable/Opus-class). Casting economizes on players, never on the podium. Spawning players and judging their reports is conducting, not playing.

## 1. Write the score

Before anything runs, interview the user using the `grilling` skill's interview loop — one question at a time, recommended answer included. (The loop only: this interview's deliverable is the score, not a grill doc.) Cover, about the RUN, not yet the feature:

1. **The goal** — one sentence is enough; stage 1 grounds it properly. Derive the run's `<slug>` from it and reuse that slug everywhere a stage wants one.
2. **Where to enter** — full pipeline, or mid-way: a PRD already exists → start at stage 2's gaps loop; slices exist → start at build.
3. **The gate map** — present the defaults below; let them tighten or loosen each gate.
4. **Casting** — present the default casting; let them set the cost posture (thrifty / balanced / premium shifts players one tier down or up, where a tier exists to shift to — never the conductor). Map tiers to the models the harness offers today and record the mapping in the score.
5. **AFK behavior** — when the human is needed and nobody answers, whether at a gate or mid-interview: **park** (append a paused-at log entry capturing everything in flight, so a fresh conductor resumes from the score alone) or **proceed-and-flag** (continue on conductor judgment, flag the decision loudly in the log).

Write the answers to `./runs/<slug>/score.md` (template below). The score is the run's single source of truth — a fresh conductor must be able to resume the performance from the score and the stage artifacts alone.

## 2. Gates

A **gate** sits after every stage. Three settings:

- **human** — stop; present the stage artifact, your one-paragraph verdict, and a recommendation; wait.
- **conductor** — judge the exit criterion yourself, proceed, log the decision.
- **auto** — proceed the moment the mechanical exit check passes. Auto governs the loop's *exit* only — a human question surfacing mid-stage (Perform rule 3) doesn't inherit the auto.

Where a stage's skill says "iterate until the user approves", the gate's setting decides who that approver is: human, or the conductor answering per Perform rule 3.

| Gate | After | Default | Exit check |
|---|---|---|---|
| goal-locked | grilling | human | grill doc complete and self-contained |
| gaps-clean | the PRD's gaps loop | auto | fresh cold reader says NO GAPS |
| prd-approved | write-a-prd | human | PRD in `prds/`, gaps-clean already passed |
| slices-approved | prd-to-slices | human | slice-folder gaps loop empty AND breakdown quiz passed |
| slice-done | each slice built | conductor | merged, tests green, acceptance criteria checked |
| review-clean | code-review | conductor | no finding the conductor judges must-fix (documented-standard breaches always are) |
| ship | end | human | — |

Two overrides no gate setting — and no AFK setting, including proceed-and-flag — can relax:

- A slice typed **HITL** in its slice file always gets its human interaction: take the decision or design review the slice needs to the human BEFORE its Builder plays, and set its slice-done gate to human.
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
| Standards reviewer | code-review, axis 1 | mid | pattern-matching documented rules |
| Spec reviewer | code-review, axis 2 | frontier | judging intent, not text |
| Diagnostician | a Builder stuck past escalation | frontier | hard bugs are judgment |
| Stagehand | scaffolding, verify runs, formatting, merges, file moves | mid | mechanical, but still touches the tree |

**Escalation**: a player that fails or reports low confidence gets ONE re-run, one tier up, with the failure pasted into its brief; frontier players skip this rung. The next rung after that is the human, regardless of the gate map — with one insertion: a Builder that fails its re-run goes to the Diagnostician first (see the sidelines), and only a failed diagnosis reaches the human. The Cold reader stands off the ladder — its brief is fixed, so a failed round just gets another fresh reader, and a gap that keeps returning across rounds goes to the human, as its own skill says. Wherever a ladder runs out, the human is the last rung, no exceptions.

## 4. Perform

Walk the stages the score lists, in order. Four rules bind every player:

- **Files are the only handoff.** Brief a player with paths — never a summary of what the file already says. That is the pack's whole design; don't defeat it from the podium.
- **Clean context per player.** Spawn via the subagent (Task) tool with its model option set from the score's tier mapping; if the harness can't set a per-agent model, the briefs still carry the role. Name the pack skill the player must run (e.g. "run the `tdd` skill on this slice"), and end every brief with: state your confidence and anything you could not verify — except players whose sibling skill fixes their brief verbatim: the Cold reader's brief, and its NO GAPS reply, stay untouched.
- **The conductor is the player's user.** A running player has no live channel to the podium, so the traffic moves in two ways. Pre-answer checkpoints you can see coming: a Builder's brief answers tdd's planning checklist up front — the interface changes and the behaviors to test, extracted from the slice, overview, and PRD. For the rest, tell every player: a "confirm with the user" your files don't answer means stop and return the question with your state — the conductor answers it (from the run's artifacts if they can; a question they can't answer is a human question — ask if the human is around, otherwise route per AFK behavior) and respawns a fresh player with the answer folded into the brief. (Interview-shaped stages the conductor plays talk to the human directly; this rule is for players.)
- **Stage boundaries beat embedded steps.** When a stage's skill embeds work another stage owns, run it once, where the score puts it: `write-a-prd`'s grilling step is already satisfied by stage 1's grill doc; its exit gaps loop and `prd-to-slices`'s exit gaps loop run with cast Cold readers under their gates above.

Stage by stage:

1. **grilling** — conductor with the human, per the skill; cast a Scout for any explore-the-codebase questions → `./grill-<slug>.md`. Gate: goal-locked.
2. **write-a-prd** — conductor runs the interview from the grill doc; cast a Scout for the codebase exploration and fold its report in → `prds/<slug>.prd.md`. Its exit gaps loop: a fresh Cold reader each round (never reuse one — it's warm), fold every gap into the document, loop until NO GAPS. Gates: gaps-clean, then prd-approved — the human approves the post-gaps PRD, so the loop never mutates an approved document.
3. **prd-to-slices** — conductor drafts and quizzes per the skill; a Scout can pre-read the code → `slices/<slug>/`. Run its exit gaps loop on the slice folder the same way. Gate: slices-approved.
4. **build** — cut a run branch `run/<slug>` from `HEAD`, `git rev-parse` it, and log the ref: that is the review's fixed point, and every slice lands on that branch. One Builder per slice, in dependency order from `00-overview.md`; each commits its slice to the run branch. Unblocked slices may run in parallel ONLY in isolated worktrees; a Stagehand merges each finished worktree back into the run branch in dependency order. "Merged" in slice-done means the slice's commits are on the run branch with tests green there. Each Builder gets its slice file, the overview, the parent PRD, and the repo's standards sources (found the way `code-review`'s step 3 finds them — if the repo documents none, that's the agent-ready pre-flight talking). Gate per slice: slice-done (HITL slices → human, always).
5. **code-review** — the conductor runs the skill from the podium: Standards and Spec players in parallel with the casting above (overriding the skill's default subagent type where the harness allows), fixed point = the logged build ref. The Spec player's spec is the whole `slices/<slug>/` folder plus the parent PRD; brief it to tag each finding with the slice it falls under. Standards findings arrive by file/hunk — attribute them to slices yourself via the run branch's per-slice commits. Aggregate the reports, send findings worth fixing back to a Builder on the responsible slice, then re-run both review players against the same fixed point — review-clean judges the latest reports.
6. **ship** — present a run summary built from the score's log. Gate: ship.

After every stage and gate, append one line to the score's **Log**: stage, who decided (human / conductor / auto), verdict, artifact path. A human returning from AFK reads the log, not the transcript.

## The sidelines

Skills the score doesn't list as stages, and when the conductor reaches for each:

- **agent-ready** — pre-flight. If the repo would fail its five checks, run it with the human before stage 1; every player plays better in a repo that passes.
- **prototype** — when the PRD or slicing interview hits a design question words can't settle. Cast a mid player to build the throwaway; log the answer in the score, and see the skill's delete-or-absorb step through — a Stagehand can do the deleting.
- **diagnosing-bugs** — the Builder's post-re-run rung on the escalation ladder (§3). The Diagnostician replaces the Builder; scope its brief to the diagnosis loop and a verified fix, not the skill's architecture exit.
- **domain-modeling** / **codebase-design** — vocabulary keepers. Name them in Scout and Builder briefs so `CONTEXT.md` and ADRs stay current as decisions crystallize.
- **handoff** — the human's pause button, fired by name mid-stage. Between stages the score's log already is the handoff; park writes the paused-at entry for exactly that reason.

## Score template

Record only deviations from this skill's defaults — the defaults' single source of truth is this file.

```markdown
# Run: <feature> — score

> Goal: <one sentence> · Slug: <slug> · Entered at: <stage> · Cost posture: <thrifty|balanced|premium>

## Tier mapping

frontier → <model> · mid → <model>

## Gate overrides

| Gate | Setting |
|---|---|
| <only gates that differ from the defaults — "none" if none> | |

## Casting overrides

<only roles cast off-default — "none" if none>

## AFK behavior

<park | proceed-and-flag>

## Log

| When | Stage / gate | Decided by | Verdict | Artifact |
|---|---|---|---|---|
```
