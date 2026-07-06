# Part 8 live runbook: Team Spaces, planned e2e on stage — grill handoff

## Goal

The Part 8 demo is redefined, and it runs TODAY at the hackathon with zero rehearsal.
Old plan: walk pre-made checkpoint files of the half-built share-records feature.
New plan: Daybook is presented as **already built and agent-ready**, and the facilitator
runs the full AI-engineering pipeline (grill → spec → verify → architect → slice →
build+review) **fully live**, end to end, on a genuinely new massive feature:
**Team Spaces** — shared workspaces where members plan together.

The room (hackathon attendees) never touches Daybook; they run the same stages on their
own projects between the facilitator's gates.

## Context (verified facts)

- **Daybook repo:** `~/.superset/projects/AI-workshop/daybook` (private nested git repo;
  bare-mirror origin at `../daybook-origin.git`, relative remote — dirs must stay siblings).
- Stack: Hono API — all route groups registered in `src/api/index.ts`; `node:sqlite`
  (`src/api/db.ts`); Vite+React; vitest; `npm run verify`. Mock identity = `x-user-id`
  header, seeded users `alex`/`sam` (middleware in `src/api/records.ts:22-33`).
- Schema (`src/api/db.ts:16-36`): `users(id, name)` ·
  `records(id, owner_id, title, days JSON, created_at, updated_at)` ·
  `shares(token, record_id, created_at, revoked_at, expires_at)`.
- **Ownership check is copy-pasted inline per handler** (`src/api/records.ts:75`,
  `row.owner_id !== userId`) — deliberate. Team-spaces forces it into a real
  access-control seam; the existing seam precedent is `resolveLiveShare` in
  `src/api/shares.ts` (from the share-records architecture stage).
- share-records: slice 1 of 5 merged (`72d3370`, ~6 min gate-to-gate); slices 2–5 open
  in `docs/features/share-records/slices/`. Checkpoint files live in
  `docs/features/share-records/` (grounded-context.md §3.5 = the 410→404 grill-catch story).
- Agent-ready pass is already committed: `868bc28` (CLAUDE.md, conventions,
  add-endpoint-our-way skill). The public skills-pack ships the `agent-ready` skill
  (`skills-pack/skills/agent-ready/SKILL.md`): onboarding file, one-command verify,
  conventions+golden path, territory map, guardrails.
- Pipeline runner: Archon v0.5.0. Workflows: `daybook/.archon/workflows/plan-feature.yaml`
  (worktree DISABLED — docs commit straight to main under `docs/features/<slug>/`) and
  `build-slice.yaml` (worktree enabled, local `--no-ff` merge, no GitHub dependency).
  Stage disciplines in `.archon/commands/*.md` (self-contained).
- Old demo script (now superseded for Part 8): `~/.superset/projects/AI-workshop/guides/demo-plan.md`
  line 278. Its fallback table already says: "archon breaks → the checkpoint files ARE
  the demo" — that fallback now points at share-records' files.

## Decisions

1. **Feature = Team Spaces.** Shared workspaces: create a space, invite existing users,
   records can live in a space, members view/edit, owner-only destructive actions.
   Why: massive, rich grill ambiguity, and it forces the copy-pasted ownership check
   into a real access seam — cashes in the ownership-debt storyline planted in Part 2.
   Rejected: templates/recurring (no permissions drama), comments-on-shares (too small),
   weekly review/streaks (read-only, weakest pipeline fit).

2. **Fully live, interleaved gates.** plan-feature runs live on stage, kicked off at the
   top of Part 8. Each stage gate that fires = that stage's teaching beat (cut to
   terminal, read artifact aloud, approve/reject live). Between gates the room runs
   their relay on their own repos. No rehearsal happens (facilitator is already at the
   hackathon). Rejected: pre-day run (no time), hand-authored checkpoints (fiction).
   Known risk accepted: prior run took ~55 min on opus with 8 gates; concurrency with
   the room relay is what makes it fit.

3. **share-records stays as-is (1/5 merged) and is REFRAMED, not hidden.** Line:
   "Every artifact I'm about to make live — this feature already has them; slice 1 is
   merged. Today we do it again, live, on something bigger." Open slices 2–5 =
   evidence the pipeline is real, not embarrassing debt. Rejected: finishing them
   off-screen (no time), reverting sharing (destroys authentic history).

4. **Scope dial = tight core.** The facilitator's live grill answers deliberately bound
   the feature: IN — create space, invite alex/sam, move a record into a space, members
   edit records in the space, owner-only delete/invite. OUT — multiple roles beyond
   owner/member, notifications, space-level share links, leaving/ownership transfer.
   Why: keeps the live PRD coherent and the slice count teachable (~5). The OUT list is
   the Out-of-scope dwell beat at the PRD gate.

5. **Build beat: live build-slice on team-spaces slice-1 if the clock allows.** When the
   plan run completes, background `build-slice` on the new slice-1 (~6 min gate-to-gate
   historically) and teach over it; fresh-context review → APPROVE → local merge →
   tests green is the closer. Fallback: `git log` of `72d3370` (share-records slice-1)
   as canned proof of stage ⑥.

## The live answer script (facilitator's own product intent)

Opening ask to the grill (the fuzzy sentence, on purpose):

> I want teams to plan together in Daybook — give people shared spaces.

Planted contradictions — surface them in YOUR answers so the room watches the grill
catch them:

1. **Edit vs delete:** "everyone in the space can edit the records in it" …then later…
   "actually, deleting — only I should delete." Freeze when the grill pushes:
   *"That push is the whole stage."*
2. **Private vs moved:** "personal records stay private, spaces don't change that"
   …then… "but I want to move my existing record into the space." (What happens to
   privacy when it moves? Who owns it now?)

Scope answers when asked (from Decision 4): owner/member only; invites = add an existing
user (alex/sam — mock identity, no email flow); no notifications; no space share-links;
no leaving/transfer — all explicitly OUT.

Expected slice-1 shape (sanity check when reading the slices gate): the ugly column that
round-trips — create a space + list my spaces, end to end through API + UI, no members yet.

## Run commands (from the recorded run mechanics)

Every archon call needs the env prefix; `--detach` is broken (child re-execs and dies) —
run foreground in a dedicated terminal or shell-background it.

```sh
cd ~/.superset/projects/AI-workshop/daybook

# kick off (confirm exact arg form against .archon/workflows/plan-feature.yaml before running):
CLAUDE_BIN_PATH=/Users/rotembarmaimon/.local/bin/claude CLAUDE_USE_GLOBAL_AUTH=1 \
  archon workflow run plan-feature "I want teams to plan together in Daybook — give people shared spaces"

# gates:
archon workflow approve <run-id> "msg"     # or reject
archon workflow get <run-id> --json        # poll `status` ONLY (approval metadata goes stale)
# interactive-loop input arrives via $LOOP_USER_INPUT; loops end on <promise>SIGNAL</promise>
```

build-slice afterwards (Decision 5):

```sh
CLAUDE_BIN_PATH=/Users/rotembarmaimon/.local/bin/claude CLAUDE_USE_GLOBAL_AUTH=1 \
  archon workflow run build-slice docs/features/team-spaces/slices/slice-1.md --branch spaces-slice-1
# gotchas: --branch X becomes branch archon/task-X; CLI exiting 0 at a pause is normal
# (approve resumes in a new process); after merge-local: git push origin main (bare mirror sync);
# cleanup beat: archon complete <branch> --force
```

Pre-flight (do before Part 8 starts): main checkout clean, no `docs/features/team-spaces/`
dir exists, `npm run verify` green, bare mirror in sync, no other dev servers on the
default ports (`scripts/demo` kills strays; `scripts/reset-demos.sh` exists if worktree
states are dirty).

## Fallbacks (in order)

1. Run hangs mid-stage → the share-records checkpoint files in
   `docs/features/share-records/` become the spine (the ORIGINAL Part 8 plan — §3.5
   grill-catch, PRD out-of-scope dwell, architecture 410 collapse, slices, `72d3370` log).
   Narrate: "here's the same trail from the last feature."
2. Run produces weak drama → same fallback for the affected stage only; the live
   artifacts still exist and are real.
3. No time for live build-slice → `git log --oneline` around `72d3370`: "6 minutes
   gate-to-gate, reviewed by a fresh context that never saw the builder's rationalizations."

## Open questions

- Exact plan-feature CLI arg form (positional ask vs flag) — check
  `.archon/workflows/plan-feature.yaml` at pre-flight; 30 seconds.
- Interleave timing vs the room relay (grilling 10' → PRD 7' → peer gate 3' →
  prd-to-slices 5' → peer gate 2') — improvised on the day around whenever gates fire;
  no rehearsal data exists for team-spaces gate spacing.
- Deck (`index.html` Part 8 beats) and `guides/demo-plan.md` still describe the
  share-records spine — update AFTER the hackathon to match whatever actually happened.

## Out of scope

- Finishing share-records slices 2–5 (left open deliberately; reframed as evidence).
- Any deck/demo-plan edits before the hackathon.
- Team-spaces full vision: roles matrix, notifications, space share-links, ownership
  transfer, leaving a space (these are the PRD's out-of-scope list, on purpose).
- Attendee-facing changes — the room's relay and skills-pack flow are unchanged.

## Suggested first steps (for the facilitator, right now)

1. Run the pre-flight block above (clean main, verify green, confirm plan-feature arg form).
2. Read the live answer script once; the two contradictions must come from YOUR mouth.
3. Open two terminals: ⌘5 = archon run, ⌘6 = main checkout for reading artifacts/git log.
4. At the top of Part 8: deliver the reframe line (Decision 3), kick off the run, and
   send the room into `/grilling <their feature>` while the agent grills you back.
