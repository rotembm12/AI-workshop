# Demo run-of-show — the full day, nothing from memory

> Locked in the grilling session of 2026-07-06. One substrate (Daybook), five pinned demo
> states as worktrees, defaults everywhere: API on :8787, web on :5173, one dev server at a
> time via the launcher. The room works on **their own hackathon projects** in Part 8 — Daybook
> is mine alone.

---

## 0 · The map

| Part | Live? | State / tab | The demo | Fallback |
|---|---|---|---|---|
| 1 · Vibe Coding | claude.ai in browser | `part2` files for the paste | greenfield one-shot, then the collapse | slide beats |
| 2 · AI Engineering | terminal | `part2` | save-record slice, planted registration miss | slide trace |
| 3 · Agent-Ready | terminal | `part3-bare` + `part3-ours` | write CLAUDE.md live, then the count proof | slide before/after |
| 4 · Skills | terminal | `part4` | draft skill with skill-creator, run it on pinning | canned skill from `part3-ours` |
| 5 · Harness | — | — | in-deck widgets only | — |
| 6 · Reach (MCP) | — | — | **everything on slides, nothing live** | — |
| 7 · Skills vs MCP | — | — | concept only | — |
| 8 · Workflow | terminal ×2 | `main` (⌘5 relay + ⌘6 artifacts) | **superseded 2026-07-06 → `guides/grill-part8-team-spaces.md`**: full pipeline live on Team Spaces via the skills-pack relay (Archon deleted); room runs the same relay on their own repos | share-records checkpoint files |
| 9 · Autonomy | — | — | deliberately none — slow television | — |

Every live prompt below is verbatim — read it off this page.

---

## 1 · Machine prep (once, before rehearsal week)

All of this is already done; this list exists so I can re-create it from nothing.

```sh
# the five demo worktrees (pinned by demo-*-start tags)
~/.superset/projects/AI-workshop/daybook-demos/
  part2/        demo/part2       03a6ba5  records in memory, no records API, no tests
  part3-bare/   demo/part3-bare  b384219  full CRUD + tests, NO CLAUDE.md
  part3-ours/   demo/part3-ours  868bc28  CLAUDE.md + conventions + the skill
  part4/        demo/part4       6aa288d  agent-ready, .claude/skills DELETED
  part8-plan/   demo/part8-plan  e8192ba  pre-plan state (Archon removed 2026-07-06; unused in the new Part 8 — live run happens in main)
# the main checkout doubles as the Part 8 build state:
~/.superset/projects/AI-workshop/daybook/    main  (tag demo-part8-build-start)
```

Helpers (committed in `daybook/scripts/`):

```sh
daybook/scripts/demo <name>       # kills any daybook dev server, starts this state's
                                  # (names: part2 · part3-bare · part3-ours · part4 · part8-plan · main)
daybook/scripts/reset-demos.sh          # all five worktrees → their tags, DBs wiped
daybook/scripts/reset-demos.sh --main   # + undo rehearsal merges on main, force-sync the local
                                        #   bare mirror (its archon/* branch-drop is a no-op now)
```

Terminal layout (iTerm, one tab per state, in show order):
**⌘1** `part2` · **⌘2** `part3-bare` · **⌘3** `part3-ours` · **⌘4** `part4` · **⌘5** `daybook` (main — the Part 8 relay session) · **⌘6** `daybook` (main — artifacts / git / build branch). Each tab split in two panes: left `claude`, right shell (curl / verify / the launcher). *(2026-07-06: ⌘5 was `part8-plan`; the new Part 8 runs entirely in the main checkout.)*

---

## 2 · Morning-of checklist (venue, ~20 min)

1. `daybook/scripts/reset-demos.sh --main`
2. Presentation font in terminal (⌘+ until ~18pt), Do Not Disturb on, close Slack/mail.
3. `claude --version` runs; in `part4` type `claude` and check **skill-creator** is available
   (`/skill` shows it); `/exit`.
4. `claude mcp list` → should be empty/irrelevant; Part 6 is canned, nothing to install.
5. Browser tabs in order: the deck · claude.ai logged in, **empty new chat** · `localhost:5173`.
6. Seed Part 3's records (the count proof needs data — DB starts empty after reset):
   ```sh
   cd ~/.superset/projects/AI-workshop/daybook-demos/part3-bare && npm run dev &
   sleep 4
   curl -s -X POST localhost:8787/api/records -H 'content-type: application/json' \
     -H 'x-user-id: alex' -d '{"title":"Weekend in the mountains","days":[{"title":"Drive up","notes":""}]}'
   curl -s -X POST localhost:8787/api/records -H 'content-type: application/json' \
     -H 'x-user-id: alex' -d '{"title":"Desert trip","days":[]}'
   curl -s -X POST localhost:8787/api/records -H 'content-type: application/json' \
     -H 'x-user-id: sam'  -d '{"title":"City break","days":[]}'
   # then kill it — the launcher restarts servers per part:
   pkill -f "tsx watch"; pkill -f vite
   ```
7. Open this file on the laptop screen (deck on the projector).

**Between-parts rule:** before each terminal part, run `daybook/scripts/demo <state>` in that
tab's right pane. It kills the previous part's servers itself — nothing to remember.

---

## Part 1 · Vibe Coding (claude.ai, browser — no terminal)

**Beat A — the wow (greenfield one-shot).** New chat on claude.ai, type:

> Build me a daily journal app — records with a title and day-by-day notes, a list view and an
> editor, a nice warm UI. One self-contained file, make it beautiful.

Let the artifact render, click around it for ~30 seconds. Line: *"Five minutes, and it's
gorgeous. This is the high everyone's chasing — and look, I actually have a journal app in
production, so let's use this power on it."*

**Beat B — the collapse (same chat).** Open `daybook-demos/part2/src/web/App.tsx`, copy the
whole file, paste it into the same chat with:

> This is a component from my real app. Records only live in React state — everything dies on
> refresh. Add real saving: a POST /api/records endpoint and wire the Save button to it, so
> records survive a refresh. Give me the code.

What comes back **is the slide's checklist** — walk it against the answer on screen:

- **Invented structure** — it writes an Express (or bare-Node) server; the real app is Hono,
  assembled in `src/api/index.ts`. It can't know that; it never saw the repo.
- **Someone else's conventions** — its error shapes, its file layout, not
  `docs/conventions.md`. (Don't over-explain; Part 3 owns this.)
- **The silent security hole** — no `x-user-id`, no ownership. Everyone saves into one pile.
  *"A regression it can't even know it made."*
- **"This works" — against what?** It never ran `npm run verify`. Nothing here is verified;
  I'm the one who gets to find out in production.

Close: *"The demo got 80% of the way there in five minutes — nobody counts the week the last
20% eats."* Leave the chat tab open; you point back at it from Part 2.

**Fallback:** the Part 1 slide carries the collapse checklist; if claude.ai is down, narrate
over the slide and lean on Part 2's live run.

---

## Part 2 · AI Engineering (terminal ⌘1 — the biggest live demo before Part 8)

**Prep in the tab (before the part starts):** right pane `daybook/scripts/demo part2`, browser
on `localhost:5173` — show a record created, hit refresh, watch it vanish. Left pane: `claude`
running, empty.

**The slice contract slide stays up. Switch to terminal. Narrate intent before each turn, then
shut up and let the room read the diff.**

**Turn 1 — the under-specified ask** (this is the same ask chat got in Part 1 — say so):

> Records only live in React state — add real saving. A POST /api/records endpoint that
> persists to sqlite, and wire the editor's Save button to it so records survive a refresh.
> Hold off on tests — I'll check it myself first.

("Hold off on tests" is what keeps the red observable — *I* run the check, the room sees it.)

**Turn 2 — the observation (right pane), while the diff is on screen:**

```sh
curl -si -X POST localhost:8787/api/records -H 'content-type: application/json' \
  -H 'x-user-id: alex' -d '{"title":"Live from the workshop","days":[]}' | head -1
```

Two branches — **both are wins, know which one you're in**:

- **404 (the planted miss):** the route file exists but was never registered in
  `src/api/index.ts`. Paste the curl output into the agent **verbatim, no editorializing**:
  > I ran: POST /api/records → HTTP/1.1 404 Not Found
  It registers the route, you re-curl → 201. Line: *"The red did the steering, not me."*
- **201 (it got it right):** flip it — *"Chat shipped this exact mistake in Part 1 and called
  it done. This one read `index.ts` before writing — that's what repo access buys."* Show the
  registration line in the diff.

**Turn 3 — close the loop with proof:**

> Now the round-trip test: prove a saved record comes back after a fresh request. Then run
> npm run verify and make it green.

While verify runs: refresh `localhost:5173`, the record is still there. Round-trip on screen.

**Turn 4 — the deferral (someone will ask about sharing; if nobody does, prompt it):**
*"No sharing yet — on purpose. That debt comes due in Part 8."*

**Fallback:** the slide's agent trace (`expected 201, route returned 404 → registering in
src/api/index.ts → green`) is the exact story; narrate it if the live run misbehaves twice.
**/clear after two failed corrections — practice what the do's slide preaches.**

**Exit state:** demo branch has live commits — fine, `reset-demos.sh` erases them.

---

## Part 3 · Agent-Ready (terminals ⌘2 + ⌘3)

**Prep:** ⌘3 (`part3-ours`) left pane: `cat CLAUDE.md` ready to show. ⌘2 (`part3-bare`):
right pane `daybook/scripts/demo part3-bare`, left pane `claude` fresh.

**Move ① — ours first.** In ⌘3: scroll `CLAUDE.md` slowly — the five blocks (what it is /
run / verify / layout / mock identity / golden path). *"This is why Part 2 went smoothly —
you're seeing the setup now."*

**Move ② — add one to a bare repo, live.** In ⌘2, create `CLAUDE.md` and type exactly this
(the shape on the slide, not a novel):

```markdown
# Daybook
Day-by-day plan records — Hono API + React, sqlite via node:sqlite.

Run:      npm run dev        (API :8787, web :5173)
Verify:   npm run verify     — if it is green, the change is good.

Golden path: src/api/records.ts — pattern-match it for any new endpoint.
Every route is registered in src/api/index.ts, nowhere else.

Guardrails: no ORM, prepared statements only. Never commit data/.
```

**Move ③ — the ninety-second proof.** The slide shows the canned **before** (cold agent on
this task counts everyone's records — captured at rehearsal). Then add the two lines live to
the CLAUDE.md above:

```markdown
Identity: the current user is the x-user-id header — 401 if missing or unknown.
Errors:   JSON { error: string }; missing-or-not-yours is 404, not 403.
```

Then in the left pane:

> Add GET /api/records/count — returns how many records the current user has. Follow CLAUDE.md.

Expected: it pattern-matches `records.ts`, mounts inside the existing records group, respects
`x-user-id`. Prove it in the right pane:

```sh
curl -s localhost:8787/api/records/count -H 'x-user-id: alex'   # → 2
curl -s localhost:8787/api/records/count -H 'x-user-id: sam'    # → 1
```

Different counts per user = the ownership convention landed from two lines of context. *"That
contrast is this whole part in miniature."*

**Fallback:** the slide's before/after pair. **Timebox:** if the live run passes 3 minutes,
show the diff, skip the curl.

---

## Part 4 · Skills (terminal ⌘4 — flagship, 50-min slot)

**Prep:** ⌘4 right pane `daybook/scripts/demo part4`, left pane `claude` fresh.
This state is agent-ready but `.claude/skills/` does not exist — verify with `ls .claude` → no such dir.

**Live moment 1 — draft it (during the Trigger/description beat):**

> Use skill-creator to draft a skill for this repo called add-endpoint-our-way: adding an HTTP
> endpoint the house way — thin vertical slice, route registered in src/api/index.ts, handler
> follows docs/conventions.md, data access with prepared statements, a round-trip test via
> app.request(), done means npm run verify is green. Keep SKILL.md small.

While it drafts, talk the description beat: the description is the trigger — "add an
endpoint", "new route", "expose X over the API" must all fire. When it lands, show the
generated `SKILL.md` — **don't polish it live**; that's what the deck's widgets do next.

**In-deck (no terminal):** Structure tabs → steering "thin vertical slice" echo → pruning
clicks. The deck carries these; stay out of the terminal.

**Live moment 2 — run it (start it, then keep talking):**

> Add pinning: a user can pin one of their records and pinned records sort first in their
> list. Endpoint, UI, and a round-trip test — use the add-endpoint-our-way skill.

**Start the run, then deliver the rule-of-three beat over it** (the run needs ~3–4 min; the
beat needs ~3 — they fit). Cut back when it finishes: the diff shows route registered in
`index.ts`, ownership respected, test green — *"Part 2 learned these conventions by red test.
The skill made them free."* Pin a record in the UI.

**Fallback:** the polished skill is one command away:
```sh
git show demo/part3-ours:.claude/skills/add-endpoint-our-way/SKILL.md
```
*"Here's the one this exact process produced"* — then run live moment 2 against it.

---

## Parts 5 · 6 · 7 · 9 — no terminal, ever

- **5 · Harness:** in-deck simulations only ("Run the agent" click beats).
- **6 · MCP:** **decided 2026-07-06 — everything on slides, nothing live.** The add-command,
  the before/after trace and the Cost chips are all in-deck. Nothing to install, no wifi
  dependency. Beat to protect: ④ Cost.
- **7 · Skills vs MCP:** ten minutes, concept, punchline "you can't skill your way to a browser."
- **9 · Autonomy:** deliberately no demo — "autonomy is slow television." Beat to protect: ④ the catch.

---

## Part 8 · The Workflow — ⚠ SUPERSEDED 2026-07-06, hackathon day

> **The runbook for today is `guides/grill-part8-team-spaces.md` — run Part 8 off that page,
> not this one.** What changed: Archon is deleted from daybook (commits `a205f70` main /
> `e8192ba` demo/part8-plan, tags re-pointed). The facilitator now runs the FULL pipeline
> live, end to end, on a genuinely new feature — **Team Spaces** — using the SAME skills-pack
> relay the room runs, installed into daybook:
> ① `/grilling` → ② `/write-a-prd` → ③ its `/gaps-report` exit check → ④
> `/improve-codebase-architecture` → ⑤ `/prd-to-slices` → ⑥ fresh-session build +
> `/code-review`, with `/clear` at every stage boundary. Both terminals live in the `main`
> checkout (⌘5 relay session, ⌘6 artifacts/git/build branch); `part8-plan` is unused.

**The reframe (still true, sharpened):** *"I'll run each stage live on my reference app —
and you run that same stage on your own project, at the same moment, with the same skill."*

**The share-records checkpoint files** (⌘6, `docs/features/share-records/`) are no longer the
spine — they are the **fallback spine** and the "this pipeline already ran once" evidence
(slice 1 merged, `72d3370`). The old stage-by-stage story table still works verbatim for any
stage where the live run hangs or lands flat:

| Stage | File to open | The story to tell |
|---|---|---|
| ① grill | `grounded-context.md` | 4 rounds; **§3.5**: round 4, the agent tried to quietly revert 410→404 — the owner caught it. "The grill runs both ways." |
| ② spec | `prd.md` | six pillars; dwell on Out-of-scope; the canView/canEdit seam chosen before any code |
| ③ verify | `gaps-report.md` | 2 iterations to an empty report — "a second window is enough" |
| ④ architect | `architecture.md` | four dead states → one neutral 410; the access seam `resolveLiveShare`; keep the rejected candidates visible |
| ⑤ slice | `slices/` | slice-1 the ugly column that round-trips; 2–5 blocked-by declared |
| ⑥ build | git log of `72d3370` | slice 1: built, reviewed by a fresh context, merged — 6 minutes gate-to-gate |

(Note the layout split: share-records lives in the Archon-era `docs/features/`; today's
team-spaces artifacts land in the skills-pack layout — `grill-team-spaces.md`, `prds/`,
`slices/team-spaces/`. Both are real trails.)

**The room's relay (their own repos, skills-pack pre-installed) — unchanged:**

| Slot | They run | Timebox |
|---|---|---|
| after ① | `/grilling <their fuzzy feature sentence>` in their repo | 10 min |
| after ② | `/write-a-prd` from the grill | 7 min |
| **peer gate** | swap laptops with a neighbor: find ONE gap or contradiction in theirs | 3 min |
| after ⑤ | `/prd-to-slices` | 5 min |
| **peer gate** | neighbor check: "is slice 1 really a column?" point at the dashed row | 2 min |

- Greenfield/messy repos are fine — planning stages need no tests, no clean git.
- Anyone with a broken setup **pairs** — navigator/driver is maker/checker anyway.
- **If the slot is under ~60 min:** cut `/prd-to-slices` from the relay, move it to the handoff.

**The handoff (closes the part):** *"Your slice 1 is your next hour of hackathon. Build it
with your agent, then run /code-review before you merge. I'm here until the end of the day."*

---

## Rehearsal must-tests (before the day — check each off)

1. **Part 2 both branches:** run the Turn-1 ask 3×. Note how often the registration miss
   happens; screenshot a red-curl run as the canned fallback. Practice the 201 pivot line.
2. **Part 1 dry-run:** paste App.tsx into claude.ai with the Beat-B ask; confirm it invents a
   server and drops identity. Screenshot for backup.
3. **Part 3 before-trace:** in a scratch reset of `part3-bare` (no CLAUDE.md), run the count
   ask cold; capture the trace where it ignores `x-user-id`. This becomes the canned "before."
4. **Part 4 timing:** time skill-creator's draft (target ≤ 4 min) and the pinning run
   (target ≤ 5 min). If the draft runs long, pre-trim the ask.
5. **Part 8 pre-flight (replaces the old archon rehearsal — Archon is deleted):** run the
   pre-flight block in `guides/grill-part8-team-spaces.md` — install the skills-pack INTO
   daybook (`npx skills@latest add rotembm12/AI-workshop`, needs wifi, do it first),
   smoke-test `/grilling` autocompletes in a daybook `claude` session, verify green, no
   team-spaces leftovers.
6. **Full cycle:** `reset-demos.sh --main`, then walk Parts 2→3→4 back-to-back with the
   launcher only. If anything needs a command not on this page, add it to this page.

---

## Attendees & organizers

- **Setup doc:** `skills-pack/SETUP.md` (public, in the repo) — install Claude Code, log in,
  `npx skills@latest add rotembm12/AI-workshop`, smoke-test `/grilling`. Send the **link**, not
  a PDF, through the organizers with the hackathon logistics mail.
- **The auth question to ask organizers THIS WEEK:** are attendees on their own Claude
  accounts, an org-provided key, or nothing? If "many will have nothing" → announce pairing as
  the plan, not the fallback.
- **On the day:** at the break before Part 8 — *"open a terminal, type claude, then /grilling —
  hands up if it autocompletes."* Broken setups pair. Never debug one-on-one from the stage.

Email draft (Hebrew delivery on the day; the mail can be Hebrew too — this is the skeleton):

> Subject: Setup for the AI-First Engineering day — 5 minutes, before you arrive
> Ahead of the workshop day you'll want two things working: Claude Code, and the workshop
> skills pack. Everything is here: https://github.com/rotembm12/AI-workshop/blob/main/skills-pack/SETUP.md
> It takes about five minutes. If `claude` opens and `/grilling` autocompletes, you're done.
> One question back to you: will participants have their own Claude accounts, or should we
> plan for shared/paired machines? The afternoon hands-on depends on it.

---

## Recovery moves (when the demo gods say no)

| Symptom | Move |
|---|---|
| agent flails twice on a correction | `/clear`, restart with a sharper ask — say the do's slide line out loud while doing it |
| dev server port zombie | `daybook/scripts/demo <state>` again (it pkills first) |
| worktree dirty from a wrong turn | `git -C <worktree> reset --hard demo-<name>-start && git clean -fd` (or full `reset-demos.sh` at a break) |
| claude.ai down in Part 1 | narrate over the collapse slide; the terminal parts don't depend on it |
| skill-creator misfires in Part 4 | `git show demo/part3-ours:.claude/skills/add-endpoint-our-way/SKILL.md` — "here's the one it produced" |
| a live stage flails in Part 8 | the share-records checkpoint files ARE the demo for that stage (table above); `git log` of `72d3370` shows the merged relay — skip or shorten the live leg, the room's relay keeps running |
| anything else | the slide is the fallback, not the plan — every live beat has its trace in the deck |
