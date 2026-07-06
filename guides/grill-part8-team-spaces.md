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
- **Pipeline runner: the skills-pack itself — the SAME relay the room runs.** Archon is
  deleted from daybook (2026-07-06: `git rm` commits `a205f70` on main + `e8192ba` on
  `demo/part8-plan`; both `demo-part8-*-start` tags re-pointed so `reset-demos.sh` can't
  resurrect it). Stage → skill mapping:
  ① grill = `/grilling` (writes `./grill-<slug>.md`) · ② spec = `/write-a-prd`
  (explores repo, sketches modules, writes `./prds/<slug>.prd.md`) · ③ verify =
  `/gaps-report` (write-a-prd's own exit check — a cold subagent reads the PRD, loop
  until NO GAPS) · ④ architect = `/improve-codebase-architecture` (HTML report of
  deepening candidates, grill-through to a winner; `/codebase-design` is its vocabulary) ·
  ⑤ slice = `/prd-to-slices` (writes `./slices/<slug>/00-overview.md` + slice files) ·
  ⑥ build+review = fresh `claude` session builds slice-1 on a branch, `/code-review`
  in a second fresh session, local `--no-ff` merge. Clean contexts come from `/clear` /
  new sessions between stages; the checkpoint files ARE the handoff — no runner, no
  approve CLI. Gates = the facilitator reading each artifact aloud before `/clear`.
- Note the layout shift: share-records artifacts (Archon-era) live under
  `docs/features/share-records/`; team-spaces artifacts land in the skills-pack layout —
  `grill-team-spaces.md` (repo root), `prds/`, `slices/team-spaces/`. Both are real
  trails; say so if anyone notices.
- Old demo script (now superseded for Part 8): `~/.superset/projects/AI-workshop/guides/demo-plan.md`
  line 278 — its Part 8 section still scripts Archon commands that no longer exist.
  Ignore it entirely today; its one still-true idea is the fallback: "the checkpoint
  files ARE the demo" → share-records' files.

## Decisions

1. **Feature = Team Spaces.** Shared workspaces: create a space, invite existing users,
   records can live in a space, members view/edit, owner-only destructive actions.
   Why: massive, rich grill ambiguity, and it forces the copy-pasted ownership check
   into a real access seam — cashes in the ownership-debt storyline planted in Part 2.
   Rejected: templates/recurring (no permissions drama), comments-on-shares (too small),
   weekly review/streaks (read-only, weakest pipeline fit).

2. **Fully live, interleaved gates.** The pipeline runs live on stage, kicked off at the
   top of Part 8. Each stage boundary = that stage's teaching beat (cut to terminal,
   read the artifact aloud, then `/clear` into the next stage). Between the facilitator's
   stages the room runs the same relay on their own repos. No rehearsal happens
   (facilitator is already at the hackathon). Rejected: pre-day run (no time),
   hand-authored checkpoints (fiction). *(2026-07-06 update: Archon is gone, so the
   ~55-min/8-gate risk profile no longer applies — every stage is facilitator-driven,
   so pacing is fully in your hands; the new risk is facilitator attention split between
   driving the run and working the room. The interleave structure absorbs it: your
   stage runs while they do their relay leg.)*

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

5. **Build beat: live build of team-spaces slice-1 if the clock allows.** When slicing
   completes, a FRESH claude session gets only `slices/team-spaces/00-overview.md` +
   slice-1 on a branch and builds while you teach over it (share-records slice-1 took
   ~6 min under the old runner — same ballpark). Then `/code-review main` in a second
   fresh session — the reviewer never saw the builder's rationalizations — fold or
   dismiss findings, local `--no-ff` merge, `npm run verify` green is the closer.
   Fallback: `git log` of `72d3370` (share-records slice-1) as canned proof of stage ⑥.

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

## Run script (skills-pack relay — one claude session, `/clear` at every stage boundary)

```sh
cd ~/.superset/projects/AI-workshop/daybook
claude
```

Inside the session, stage by stage — each `/clear` is a teaching line ("this context is
done producing; a clean one takes over"):

```
① /grilling I want teams to plan together in Daybook — give people shared spaces
     answer from the live answer script; the two contradictions come from YOUR mouth
     → ./grill-team-spaces.md            read the Decisions section aloud = the gate

  /clear

② /write-a-prd — open with: "The grill handoff is in grill-team-spaces.md — build the
     PRD from it; skip your own grilling step, it's done."
     → ./prds/team-spaces.prd.md
③    its exit check IS the verify stage: /gaps-report spawns a cold reader on the PRD
     and loops until NO GAPS — narrate that loop, don't skip it. Dwell on Out-of-scope.

  /clear

④ /improve-codebase-architecture — steer it at the friction: "team-spaces makes
     'can this user touch this record?' stop meaning 'are they the owner?' — the check
     is copy-pasted inline in records.ts. Where does the access decision live?"
     HTML candidate report → grill through to the winner (resolveLiveShare in shares.ts
     is the seam precedent). Fold the decision into the PRD's Implementation Decisions.

  /clear

⑤ /prd-to-slices — the slice quiz is the beat; sanity-check slice 1 against the
     expected shape below. → ./slices/team-spaces/00-overview.md + numbered slices
```

Build beat afterwards (Decision 5) — clean contexts enforced by fresh sessions:

```sh
git checkout -b spaces-slice-1
claude   # FRESH session: "Build slices/team-spaces/<slice-1 file> following
         #  slices/team-spaces/00-overview.md. npm run verify must pass."
# when it lands, a SECOND fresh session on the branch:
claude   # /code-review main        (two-axis: standards + spec, parallel subagents)
# fold/dismiss findings, then the closer:
git checkout main && git merge --no-ff spaces-slice-1 && npm run verify
git push origin main    # keeps the bare mirror in sync (reset-demos.sh assumes it)
```

Pre-flight (do before Part 8 starts):

1. **Install the pack into daybook** — it is NOT there yet (only `add-endpoint-our-way`
   is): `cd daybook && npx skills@latest add rotembm12/AI-workshop`, take all skills.
   Smoke test: `claude`, type `/grilling` — must autocomplete. THIS IS THE ONE STEP THAT
   NEEDS CONFERENCE WIFI — do it the moment you sit down, not at the top of Part 8.
2. Main checkout clean; `npm run verify` green.
3. No leftovers: no `grill-team-spaces.md`, no `prds/team-spaces*`, no
   `slices/team-spaces/`.
4. No stray dev servers on the default ports (`scripts/demo` kills strays;
   `scripts/reset-demos.sh` exists if worktree states are dirty).

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

- Does `npx skills add` install into the project's `.claude/skills/` alongside the
  existing `add-endpoint-our-way`, or somewhere global? Verify during pre-flight step 1;
  if it lands globally, copy `skills-pack/skills/*` into `daybook/.claude/skills/` by
  hand — same result, 30 seconds.
- `/improve-codebase-architecture` is repo-wide by design (deepening opportunities),
  not feature-scoped — the steer-prompt in stage ④ should focus it, but if it wanders,
  fall back to asking directly with `/codebase-design` vocabulary: "2–3 candidates for
  where the access decision lives, judged on locality and leverage, before/after sketch
  each, pick a winner." Same beat, no HTML report.
- Interleave timing vs the room relay (grilling 10' → PRD 7' → peer gate 3' →
  prd-to-slices 5' → peer gate 2') — now fully facilitator-paced (no runner to wait on);
  improvised on the day. No rehearsal data exists for team-spaces stage spacing.
- Deck (`index.html` Part 8 beats) and `guides/demo-plan.md` still describe the
  share-records spine and (demo-plan) dead Archon commands — update AFTER the hackathon
  to match whatever actually happened.

## Out of scope

- Finishing share-records slices 2–5 (left open deliberately; reframed as evidence).
- Any deck/demo-plan edits before the hackathon.
- Team-spaces full vision: roles matrix, notifications, space share-links, ownership
  transfer, leaving a space (these are the PRD's out-of-scope list, on purpose).
- Attendee-facing changes — the room's relay and skills-pack flow are unchanged.

## Suggested first steps (for the facilitator, right now)

1. Run the pre-flight block above — step 1 (install the pack into daybook) FIRST, while
   there's wifi and no audience.
2. Read the live answer script once; the two contradictions must come from YOUR mouth.
3. Open two terminals: ⌘5 = the claude relay session, ⌘6 = main checkout for reading
   artifacts / git log / the build-beat branch work.
4. At the top of Part 8: deliver the reframe line (Decision 3), kick off `/grilling`,
   and send the room into `/grilling <their feature>` while the agent grills you back —
   you and the room are now literally running the same skill at the same moment; say so.
