# Run-of-Show: Building a Great Skill, Live

> **What this is.** The presenter run-of-show for **M4 — Skills** of the TripWhale AI-First Engineering workshop. It is the script Rotem walks the room through to demonstrate, live, how to build a *great* skill — not just a skill. Same convention as the syllabus: written in English, delivered in Hebrew; keep the jargon (`SKILL.md`, *model-invoked*, *leading word*) in English even when you speak Hebrew — that shared vocabulary is itself a teaching tool.
>
> **Sources this is built from:**
> - Syllabus **M4** — *Skills: Teaching the Agent Your Team's Moves* (anatomy, "the description is the product," good-vs-bad descriptions, authoring with `skill-creator`, the rule of three).
> - **Matt Pocock — "Building Great Agent Skills: The Missing Manual"** (AI Engineer, 2026). The 4-part checklist that is the spine of this walkthrough: **Trigger → Structure → Steering → Pruning**.
> - **`writing-great-skills`** SKILL.md + GLOSSARY.md (the precise vocabulary — see the [glossary card](#appendix-d--vocabulary-card-the-leading-words) at the end).
>
> **The shape of the demo.** You author one small skill live with `skill-creator` — **`add-endpoint-our-way`** for the *Layover* repo — get a rough first pass on screen fast, then run the 4-part checklist *back over it* to make it great. The rough draft is the "before"; the checklist is the craft; the pruned result is the "after." Everything goes through the agent — you never touch the editor, same as the rest of the week.
>
> **Time:** ≈50 min (the M4 slot). Beat timings below sum to ~50. Sections marked **⏱️ CUT IF TIGHT** are the first to drop; **➕ EXPAND** notes are for when you have a longer dedicated session.

---

## The one-card spine (put this on a slide and leave it up all module)

> **A skill exists to wrangle determinism out of a stochastic system.** The thing you are buying is **predictability** — the agent taking the *same process* every run. Everything below is a lever on that.
>
> **The checklist for a great skill:**
> 1. **Trigger** — how it's invoked. *(user-invoked vs model-invoked; the description)*
> 2. **Structure** — how it's laid out. *(steps + reference; keep `SKILL.md` small; hide branches behind pointers)*
> 3. **Steering** — how it makes the agent actually do the thing. *(leading words; legwork)*
> 4. **Pruning** — how you make it as small as possible. *(duplication, sediment, no-ops)*

---

## Beat 0 — Cold open: "skill hell" (≈4 min)

**Goal:** make the room *want* a rubric before you hand them one. Do not open a terminal yet.

**[SAY]** *"Quick history of developer suffering. A few years ago we had **tutorial hell** — you watch twenty tutorials and still can't build the thing. Then **framework hell** — a new JavaScript framework every Tuesday, learn it or fall behind. This week you're going to leave here with skills. So let me warn you about the next one: **skill hell**. That's where everyone has skills — download them, share them, generate them — and nobody can tell a *good* skill from a *bad* one. So people pile up skills that don't fire, contradict each other, and quietly don't deliver what they promised. The meme is: **'just one more skill, bro, I promise this one fixes it.'**"*

**[SAY]** *"Here's the actual missing piece: there is no shared rubric. You can't look at a skill and say 'this part is good, this part is rotting.' By the end of this module you'll have one — a four-item checklist — and you'll have built a real skill for our repo with it."*

**Facilitator note.** This is "build workflows, not faster code" (M0's third shift) made concrete. Say that out loud: *"This is the module where the word 'workflow' stops being a slogan and becomes a file in your repo your whole team reuses."*

**[WATCH FOR]** the advanced engineers who already write skills — they perk up at "you can't tell good from bad." That's your hook for them: *"Some of you write these already. Today's question isn't 'how do I make one' — it's 'how do I make one that's small, fires reliably, and doesn't rot.'"*

**Room leaves this beat with:** a reason to care — skills are easy to make and hard to make *well*, and they're about to learn the difference.

---

## Beat 1 — Anatomy in 90 seconds (≈3 min)

**Goal:** just enough structure to follow the build. Resist going deep here — the syllabus warns you: anatomy is not where the value is.

**[SHOW]** one slide, this and no more:

```
.claude/skills/add-endpoint-our-way/
├── SKILL.md            ← frontmatter (name, description) + the body
└── reference/          ← optional bundled files, loaded only when needed
    └── conventions.md
```

**[SAY]** *"A skill is a folder with a `SKILL.md`. The file has two parts: **frontmatter** — a `name` and the all-important **`description`** — and a **body** of plain markdown instructions. It can bundle extra files: templates, reference docs, scripts. Personal skills live in `~/.claude/skills/`; team skills live in `.claude/skills/` inside the repo, so they ship with the code and everyone gets them."*

**[SAY] — the one anatomy fact that matters:** *"Here's the part to remember. By default the agent only sees each skill's **name and description**. The body — all the instructions — loads **only when the description matches what you're doing.** That's called **progressive disclosure**, and it's why the next thing we talk about, the description, is the entire ballgame."*

**Facilitator note.** Do **not** mention hooks, MCP, or subagents here. If asked: *"Same idea — teach the agent a reusable move — different mechanism. It's in the Rosetta; today is skills."*

---

## Beat 2 — Draft it live with `skill-creator` (≈6 min)

**Goal:** get a *first-pass, deliberately-rough* skill on screen fast, so there's something concrete to apply the checklist to. The roughness is on purpose — it's your "before."

**[DO]** Invoke the skill-creator skill and describe the capability in plain prose. Stay in prose — no editor.

**[SAY]** *"We've all done this by hand three times: someone adds an API endpoint to Layover and it comes out shaped differently every time — different error handling, different file layout, no test. That repetition is exactly the signal that it should be a skill. Watch — I'm going to describe it and let `skill-creator` draft it."*

**[DO]** Prompt roughly:

> *Create a skill that adds an HTTP API endpoint to our Layover app the way our team does it: a route, a validated handler, a call into the repository layer, and a test that proves the round-trip. It should fire whenever someone is adding a new endpoint.*

**[SHOW]** the draft `SKILL.md` it produces. It will be *fine* — and that's the teaching moment.

**[SAY]** *"Okay. This is **a** skill. It is not yet a **great** skill. It'll have a vague description, some lines the agent would already do without being told, maybe a template buried in the middle. Perfect. Now I'm going to show you the four-item checklist by running it over this draft and watching it get sharper, smaller, and more reliable."*

**Facilitator note.** If `skill-creator` produces something *too* clean, don't fight it — narrate the flaws it *would* commonly have and introduce them as "the usual first-draft problems," then fix those. The checklist is the lesson; the draft is just the raw material. Keep [Appendix B](#appendix-b--the-before-skillmd-rough-first-pass) open as a guaranteed "before" if the live draft is uncooperative.

---

## Beat 3 — Checklist ① TRIGGER: the description is the product (≈10 min) ⭐ CENTERPIECE

> This is the single most valuable stretch of the module. The syllabus calls the good-vs-bad description contrast "the single most useful 5 minutes." Give it the time. Slow down here.

### 3a — The choice nobody knows they're making (≈4 min)

**[SAY]** *"Every skill is invoked one of two ways, and choosing wrong is the first way skills go bad."*

**[SHOW]** two-column slide:

| | **User-invoked** | **Model-invoked** |
|---|---|---|
| Who can fire it | Only you, by name | The agent (on its own) **and** you |
| Has a `description` in context? | No | Yes — **every turn** |
| Cost it pays | **Cognitive load** — *you* must remember it exists | **Context load** — tokens + attention on every request |
| Risk | You forget to use it | The agent *doesn't* fire it when it should |

**[SAY]** *"Two famous skill libraries make opposite bets. **Superpowers** is mostly **model-invoked** — the agent reaches for skills itself; very flexible. Matt Pocock's skills are mostly **user-invoked** — he types the name himself. Why would you give up the flexibility? Because a model-invoked skill is a bet that the agent will *notice* the description and fire it — and sometimes it just... won't, even when it's the perfect skill for the job. That unpredictability is a class of bug you then have to **eval** away. User-invoked removes that class of bug entirely — at the cost of you having to remember the skill exists."*

**[SAY] — the rule:** *"So: make it model-invoked **only when the agent genuinely needs to reach it on its own**, or another skill needs to. Otherwise make it user-invoked and pay zero context load. And here's the discipline — every model-invoked skill you add is a tax on **every** request your whole team makes, forever. A hundred skills is a hundred descriptions in the window. 'Just one more skill, bro' has a bill."*

**[DECISION — say it out loud]** *"For `add-endpoint-our-way`: we **do** want the agent to reach for this automatically the moment anyone starts adding an endpoint — that's the whole point of encoding 'our way.' So this one is **model-invoked.** Which means the description has to earn its place. Let's make it great."*

**Mechanics aside (10 sec):** *"To make a skill user-invoked instead, you put `disable-model-invocation: true` in the frontmatter — the description disappears from the agent's view and only you can call it by name."*

### 3b — Bad description vs good description, side by side (≈6 min)

**[SHOW]** the draft's likely description first:

```yaml
# ❌ BEFORE — what the first draft gives you
description: Helps you add API endpoints to the codebase.
```

**[SAY]** *"Read it like the agent does. 'Helps you add API endpoints.' When does this fire? It doesn't tell the agent the situations or the words that should trigger it. It restates its own name. It's vague enough that the agent will skip it half the time — and a skill that doesn't fire is a skill that rots, unused. Remember: the description is **the only part of this skill the agent sees until it fires.** The description *is* the product."*

**[DO]** Now sharpen it live, *with the agent*, talking through each fix:

```yaml
# ✅ AFTER — what a great description looks like
description: >-
  Add an HTTP API endpoint to Layover the way our team builds them — a thin
  vertical slice from route to a round-trip test. Use when adding or scaffolding
  a new endpoint or API route, exposing a resource over HTTP, or when someone
  says "add an endpoint", "new route", "wire up an endpoint", or "expose X over
  the API".
```

**[SAY] — name the three moves as you make them:**
1. *"**Front-load the leading concept.** It opens with 'Add an HTTP API endpoint... the way our team builds them' — the exact thing it does, first."*
2. *"**List the triggers — the real situations and the real words.** 'adding a new endpoint,' 'new route,' 'expose X over the API,' and literal phrases people say. The agent matches on these. One trigger per genuinely different case — don't pad it with five synonyms for the same case; that's just noise the agent reads every turn."*
3. *"**Cut anything already in the body.** The description is triggers, not a summary. Every word here costs context load on every request — it earns even harder pruning than the body does."*

**[SAY] — the payoff:** *"And notice 'thin vertical slice' in there — that's a phrase from this morning's slicing session (M3). When the same words live in your prompts, your `CLAUDE.md`, and your skill descriptions, the agent links them and fires the skill more reliably. Your shared vocabulary becomes the trigger."*

**[WATCH FOR]** the lightbulb. This is where the room understands that a skill is mostly a *good description*. If you see it land, you can move faster through the rest.

**➕ EXPAND (if you have a longer session):** pull a random community skill off the internet and read its description cold — ask the room "would this fire? when?" Live-critique two or three. Nothing teaches description craft faster than judging bad ones.

**Room leaves this beat with:** the single most important idea in the module — the description is what makes a skill fire, so the description is the skill.

---

## Beat 4 — Checklist ② STRUCTURE: steps + reference, kept small (≈7 min)

**Goal:** the two units a skill is made of, and the discipline of keeping `SKILL.md` small by pushing branch-specific material out behind a pointer.

**[SAY]** *"Inside the body, think in two units. **Steps** — the ordered procedure. **Reference** — the supporting facts the steps lean on. A skill can be all steps, all reference, or both. Our endpoint skill is both: the steps are find-the-pattern → route → validate → repository → test; the reference is our team's conventions."*

**[SHOW]** the body's step list (from the [finished skill](#appendix-c--the-after-the-finished-skill)):

```markdown
## Steps
1. Find the pattern — match the two most recent endpoints under src/api/.
2. Add the route, registered in src/api/index.ts.
3. Validate input with a schema; reject bad input with 400 before any work.
4. Call the repository, never the database directly (see reference/conventions.md).
5. Write the round-trip test. Not done until it's green.
```

**[SAY] — the small-`SKILL.md` rule:** *"Now the discipline: **make `SKILL.md` as small as you can.** Smaller skills are easier to audit, easier to maintain, and every word you cut is a token cut on every use. The lever for shrinking it is **branches** — the different ways a skill gets used."*

**[SAY] — the contrast that makes it click:** *"Matt's `to-prd` skill always does one thing — write a PRD — so its PRD template is needed on every run; it stays inline. But his `domain-modeling` skill has **branches**: sometimes it updates a glossary, sometimes it writes an architecture-decision record, sometimes neither. So those templates are needed only *sometimes* — and material needed only sometimes does **not** belong in the main file. You push it into a separate file and leave a **context pointer**: 'if you need the conventions, see `reference/conventions.md`.' The agent pulls it in only on the branch that needs it."*

**[DO]** Show the move live on the endpoint skill: the long-form conventions (status-code table, error-envelope shape, pagination rules) get lifted out of `SKILL.md` into `reference/conventions.md`, and step 4 keeps a one-line pointer to it.

**[SAY]** *"Watch what just happened to the main file — it went from a wall of text to five clean steps. The conventions still exist, fully, one file away. The agent reads them exactly when it needs them and not a token sooner. That's **progressive disclosure**."*

**Facilitator note.** Keep this brisk — it's a *show*, not a deep-dive. The point lands in the visible before/after shrink, not in a lecture on hierarchy.

**Room leaves this beat with:** steps + reference as the two building blocks, and "push branch-only reference behind a pointer" as the move that keeps a skill small.

---

## Beat 5 — Checklist ③ STEERING: leading words (≈7 min)

**Goal:** the technique that fixes "I told it clearly and it still didn't do the thing." This is the most *transferable* idea in the module — it works in every prompt, not just skills.

**[SAY]** *"Most-asked question about agents: 'I wrote it clearly and it ignored me.' The fix is usually a technique called **leading words.** A leading word is a compact phrase that already carries a ton of meaning in the model's training — and when you put it in the skill, the agent **repeats it back to itself** in its reasoning and changes its behavior to match."*

**[SAY] — the concrete example (use the one from M3, it's already in the room's head):** *"Agents love to build layer by layer — whole database, then whole API, then whole UI — and nothing works until the end. You can write a paragraph begging it not to. Or you can use a leading word: **'thin vertical slice.'** Two words. The model already knows the concept cold, so it unpacks the whole behavior — build one thin path end-to-end first. We put 'thin vertical slice' at the top of the endpoint skill and repeat it."*

**[SAY] — the part that makes it real:** *"And here's the kicker — **you can check if it worked.** After you add the leading word, watch the agent's reasoning trace. If you see it say 'okay, I'll do this as a thin vertical slice' — it took the steering. If you don't, your leading word is too weak and you need a stronger one. **English is a wide API** — there are lots of these words, and the agent itself is good at helping you find them. Most of you already do this instinctively with little phrases. All I'm asking is that you do it **consistently**, inside your skills, and watch the traces."*

**[DO]** Point at "thin vertical slice" in the live skill's first line, then (if a model is running) show it echoing the phrase back in a reasoning trace.

**➕ EXPAND — legwork & the split (⏱️ CUT IF TIGHT):**

**[SAY]** *"One more steering lever: **legwork** — the digging the agent does inside a step. Sometimes a step gets rushed. Classic case: plan mode has two steps, 'ask clarifying questions' then 'make a plan' — and the agent always skimps on the questions because it can *see* the goal is the plan, so it races there. Matt's fix: he split 'ask clarifying questions' into its own separate skill (`grill-with-docs`), so while the agent is doing it, it can't see the next step at all — and it does far more legwork. **Hiding the next step buys you depth on the current one.** Use it when a step keeps getting rushed."*

**Facilitator note.** Tie legwork back to M2's loop: more legwork per step = a richer observation to steer the next turn on. Don't over-explain the split — name it, give the plan-mode example, move on.

**Room leaves this beat with:** a portable technique — pack intent into a leading word, repeat it, verify in the reasoning trace — usable in every prompt they write this week.

---

## Beat 6 — Checklist ④ PRUNING: make it as small as possible (≈7 min)

**Goal:** the failure modes that bloat skills, and the deletion test that kills them. This is "how are your skills so small?" answered.

**[SAY]** *"Last item. A working skill isn't a finished skill — now you make it as small as it can be. Three things bloat a skill, and one test cuts through all of them."*

**[SHOW]** the three bloat sources:
- **Duplication** — the same meaning in two places. Keep every fact in **one** authoritative spot (a *single source of truth*). Repeating it costs tokens, costs maintenance, and falsely inflates its importance.
- **Sediment** — the shared-doc rot. Everyone adds, nobody dares delete, and the skill silts up with stale, half-relevant lines. The default fate of any skill without a pruning habit. Cure: re-sort it through structure — move branch-only stuff out, kill the stale stuff dead.
- **No-ops** — lines the agent would already obey without being told.

**[DO] — the deletion test, live and visceral:**

**[SAY]** *"Here's the test that makes my skills small. Take any line and ask: **what happens if I just delete it?**"*

**[DO]** Point at a line the draft almost certainly has, like *"Write a clear, descriptive commit message."*

**[SAY]** *"If I delete 'write a clear commit message' — does the agent suddenly write a *bad* one? No. It writes a perfectly good commit message because that's its default. So that line is a **no-op** — I'm paying tokens and attention to tell the agent to do what it already does. Delete it. Run the deletion test on every line. Most prose that fails it should be *deleted*, not reworded."*

**[DO]** Delete two or three no-op lines from the live skill. Show the file get visibly shorter.

**[SAY]** *"That's the whole secret to small skills. Not cleverness — **deletion**. Leading words instead of paragraphs, one source of truth, no sediment, and the deletion test on every line."*

**[WATCH FOR]** skeptics nodding — this is concrete, testable, and un-hypeable. It's a credibility deposit. Let it land.

**Room leaves this beat with:** the deletion test as a reflex, and three named bloat modes to hunt.

---

## Beat 7 — When is a skill worth writing at all? The rule of three (≈3 min)

**Goal:** scoping judgment, said honestly — so they don't go to the clinic and skill-ify everything.

**[SAY]** *"Counter-message, because I just made skills look great: **most things should not be a skill.** The test isn't 'was this cool to automate' — it's **repetition**. The rule of three: you've done the same thing by prompt **three times**, and you'll do it **thirty more**. *That's* a skill. A one-off dressed up as a skill is just ceremony — and if it's model-invoked, it's ceremony you're taxing the whole team's context for. Remember 'just one more skill, bro'? The discipline is the opposite: earn each one."*

**[SAY]** *"`add-endpoint-our-way` passes easily — we add endpoints constantly and they should all look the same. That's the bar. In the clinic, that's your filter: what has your team typed into the agent three times already?"*

**Room leaves this beat with:** permission to *not* write skills, and a sharp test for when to.

---

## Beat 8 — Land it + handoff to the clinic (≈3 min)

**[DO]** Put the finished `SKILL.md` on screen next to the rough first draft. Let the contrast speak.

**[SAY]** *"Recap the checklist, because this is the rubric you keep. **Trigger** — we chose model-invoked on purpose and wrote a description that actually fires. **Structure** — steps plus reference, conventions pushed behind a pointer to keep the file small. **Steering** — a leading word, 'thin vertical slice,' that the agent echoes back. **Pruning** — deletion test, no no-ops, no sediment. That's a great skill. Same four questions work on *any* skill — including the dodgy ones you download."*

**[SAY] — the mic-drop:** *"One last thing. This whole rubric — the four-item checklist — Matt Pocock shipped it *as a skill* called `writing-great-skills`. It practices everything it preaches: it's user-invoked so it costs the agent nothing, it's pure reference, and it discloses its full vocabulary to a separate glossary file. The rubric is itself a great skill. Go read it — it's the best worked example there is."*

**[SAY] — handoff:** *"In the clinic in a few minutes, every team writes **one** real skill for your own project. Use `skill-creator` to draft it, then run these four over it. Pick by the rule of three — the move you've already prompted three times. That skill ships in your repo and your whole team gets it. Let's go build."*

**Room leaves M4 with:** a real skill they watched get built well, a four-item rubric they can apply to any skill, and the judgment to know which of their team's moves earns one.

---

## Appendix A — Demo-gods survival kit

The syllabus rule for any live build: *never let the recovery be the unrehearsed part.* For this demo specifically:

- **Known-good fallback skill.** The finished `add-endpoint-our-way` is checked in at [`guides/examples/add-endpoint-our-way/`](examples/add-endpoint-our-way/). If `skill-creator` wedges or the live authoring stalls, open the finished files and walk the checklist over *them* as a read-through instead of a build. You lose the "watch it improve" arc but keep 100% of the teaching.
- **Pre-written "before."** [Appendix B](#appendix-b--the-before-skillmd-rough-first-pass) is a guaranteed rough draft. If the live `skill-creator` output is too polished to teach from, switch to this one — *"here's a more typical first pass"* — and improve it.
- **If a model isn't echoing your leading word** during Beat 5: don't stall waiting for it. Say *"sometimes you have to strengthen the word — that's the point, you check the trace and iterate"* and move on. The lesson is the verification habit, not a guaranteed echo.
- **Rehearse the description rewrite (Beat 3b) to muscle memory.** It's the centerpiece; it must flow. Everything else can be a little rough.
- **Keep the *Layover* repo open** at a clean commit so file paths in the skill (`src/api/...`) are real on screen.

---

## Appendix B — The "before": `SKILL.md` rough first pass

> Use this as the guaranteed "before" if the live `skill-creator` draft comes out too clean to teach from. The flaws are labelled; do **not** show the labels to the room — let them spot the problems as you apply the checklist.

```yaml
---
name: add-endpoint-our-way
description: Helps you add API endpoints to the codebase.   # ❌ vague, no triggers, restates the name
---
```
```markdown
# Add Endpoint

This skill helps you add a new API endpoint to the application. Adding API
endpoints is a common task and this skill makes it easier and more consistent.   # ❌ no-op preamble; pure identity

## Instructions

When adding an endpoint, you should first think carefully about what the
endpoint needs to do and plan your approach before writing code.   # ❌ no-op — the agent plans by default

1. Create a new route for the endpoint.
2. Add a handler function for the route. Make sure the handler is well-written
   and follows good coding practices and is clean and readable.   # ❌ no-op — "good practices / clean / readable" is default behaviour
3. Validate the incoming request body so that bad input is handled.
4. Save the data to the database.
5. Write a test for the endpoint.
6. Make sure to write a clear and descriptive commit message when you commit.   # ❌ no-op — agent does this anyway

## Conventions

Our API uses a standard error envelope. Errors are returned as JSON with an
`error` field containing a message and a `code` field with a machine-readable
code. Status codes follow REST conventions: 200 for success, 201 for created,
400 for bad input, 404 for not found, 500 for server errors. List endpoints
should support pagination via `limit` and `offset` query parameters, defaulting
to limit 20...   # ⚠️ branch-only reference (pagination only matters for list endpoints) buried in SKILL.md — should be disclosed

Remember to validate the request body and reject bad input with a 400.   # ❌ duplication — already said in step 3
```

**What the checklist does to this, live:**
- **Trigger** → rewrite the description (front-load, add triggers, cut identity). *Beat 3b.*
- **Structure** → lift the whole `## Conventions` block into `reference/conventions.md`, leave a pointer. *Beat 4.*
- **Steering** → introduce the leading word **"thin vertical slice"** and reorder the steps around it. *Beat 5.*
- **Pruning** → deletion-test the preamble, the "plan carefully," the "clean and readable," the commit-message line, and the duplicated validation line. All go. *Beat 6.*

---

## Appendix C — The "after": the finished skill

The pruned result lives in the repo so it's real on screen and copy-pasteable in the clinic:

- [`guides/examples/add-endpoint-our-way/SKILL.md`](examples/add-endpoint-our-way/SKILL.md) — the body, small and sharp.
- [`guides/examples/add-endpoint-our-way/reference/conventions.md`](examples/add-endpoint-our-way/reference/conventions.md) — the disclosed reference, reached only via the step-4 pointer.

> **Stack note.** The example assumes Layover is a TypeScript app with an Express-style API under `src/api/`, a repository data layer, schema validation, and a test runner. Adjust the paths and tool names to *Layover*'s actual stack before you present — the *structure* of the skill is the lesson, not the specific stack.

---

## Appendix D — Vocabulary card (the leading words)

> Keep these terms in English even in Hebrew delivery. Used consistently, they become the room's shared language — which, per Beat 5, is exactly what makes skills fire reliably. Distilled from `writing-great-skills`/GLOSSARY.md.

| Term | Say it on stage as… |
|---|---|
| **Predictability** | "The whole point — same *process* every run, not same output. Every other idea serves this." |
| **Model-invoked** | "The agent can fire it itself. Costs context load on every turn; risks not firing when it should." |
| **User-invoked** | "Only you fire it, by name. Zero context load; costs *you* — you have to remember it exists." |
| **Context load** | "What a model-invoked skill costs the *agent* — its description sits in the window every turn." |
| **Cognitive load** | "What a user-invoked skill costs *you* — one more thing to remember to reach for." |
| **Description** | "The one line the agent sees before firing. The trigger. The product." |
| **Context pointer** | "A line that says 'for X, go to that file.' Its *wording* decides whether the agent actually goes." |
| **Steps / Reference** | "The two units of a body: the ordered procedure, and the facts it leans on." |
| **Progressive disclosure** | "Pushing reference out of `SKILL.md` behind a pointer, so the top stays small." |
| **Branch** | "A different way the skill gets used. Branch-only reference doesn't belong in the main file." |
| **Leading word** | "A compact phrase the model already understands ('thin vertical slice'). Pack intent into it, repeat it, watch it echo in the trace." |
| **Legwork** | "The digging the agent does inside a step. Hide the next step to buy more of it." |
| **Completion criterion** | "How the agent knows a step is done. Vague criteria → it quits early (premature completion)." |
| **Single source of truth** | "Each fact in exactly one place. Two places = duplication." |
| **Sediment** | "Stale build-up from everyone adding and no one deleting. Prune it." |
| **No-op** | "A line the agent already obeys by default. The deletion test finds these. Delete, don't reword." |

---

## Appendix E — Anticipated questions & crisp answers

- **"Isn't model-invoked always better — more flexible?"** → "Flexibility you pay for twice: context load on every request, and the risk it silently doesn't fire, which forces you to eval it. Model-invoke only when the agent genuinely must reach it alone."
- **"How is a skill different from `CLAUDE.md`?"** → "`CLAUDE.md` is always-on context for the whole repo — stack, conventions, how the team works. A skill is a *named, reusable move* that loads only when its description fires. Use `CLAUDE.md` for what's always true; a skill for a procedure you invoke."
- **"How is it different from a slash command?"** → "In Claude Code, custom commands are now expressed as skills — same mechanism. A user-invoked skill *is* effectively your slash command."
- **"On Cursor / Codex?"** → "Same open standard — `SKILL.md`. Cursor and Codex both support skills now; the description-and-body model ports directly. See the Rosetta in the syllabus."
- **"Can a skill call another skill?"** → "A model-invoked one, yes — because it has a description other skills can reach. A user-invoked one, no — no description, so nothing but you can fire it. That's why a 'router' skill that lists your user-invoked skills can only *hint* at them."
- **"Who maintains these?"** → "They live in `.claude/skills/` in the repo, so they're code — reviewed, versioned, and pruned like code. The pruning discipline (Beat 6) is the maintenance."
