# Skills pack — facilitator's guide

My map of all 14 skills in `skills-pack/`: what each one does, the one idea it carries,
how it fires, and how they wire into each other. This is the doc I talk from when I
explain the pack — one section per skill, in the order I'd present them.

Quick orientation before the per-skill sections:

- **Install**: `npx skills@latest add rotembm12/AI-workshop`. Zero setup — every skill
  works on plain markdown files in the repo. No issue tracker, no auth.
- **Two invocation kinds** — this is worth explaining early because it's the first
  question people ask ("when does a skill run?"):
  - **Model-invoked** (11 of the 14): carries a description the agent reads every
    turn, so it can fire on its own and other skills can call it. Costs context load.
  - **User-invoked** (3: `improve-codebase-architecture`, `handoff`,
    `writing-great-skills`): fires only when you type its name. Zero context load,
    but you are the index that has to remember it exists.

## The one-slide version

```
fuzzy sentence
  → grilling            interview until the goal is grounded in the code's reality
  → write-a-prd         six-pillar PRD                 → prds/<feature>.prd.md
  → gaps-report         cold reader finds what's missing — loop until empty
  → prd-to-slices       vertical slices                → slices/<feature>/NN-<slice>.md
  → build               one slice per clean context
  → code-review         Standards + Spec, parallel sub-agents
```

The connective tissue: **each stage produces a file the next stage reads**. That's the
whole design. Files mean every handoff can happen in a fresh context, and a cold reader
can tell you whether the file stands on its own. When someone asks "why files and not
just conversation?" — that's the answer: conversation doesn't survive a context reset;
files do.

The pipeline is six of the skills. The rest split into three supporting groups:
**building** (`tdd`, `diagnosing-bugs`, `prototype`), **design vocabulary**
(`codebase-design`, `domain-modeling`, `improve-codebase-architecture`), and **meta**
(`handoff`, `writing-great-skills`). `agent-ready` sits before everything.

---

## Start here: agent-ready

**What it does.** Audits a repo against five checks, shows a pass/fail scorecard with
evidence, then scaffolds whatever failed. Run it first — every other skill works better
in a repo that passes.

**The one idea.** An agent starts every task cold. Agent-ready means the repo itself
answers what a new senior hire would ask in week one: what is this, how do I check my
work, how do we do things here, where does everything live, what must I never touch.

**The five checks** (each phrased as *what passes* → *how it's verified*):

1. **Onboarding file** — a root `CLAUDE.md`/`AGENTS.md`, about a screen long, every
   claim true. Verified by spot-checking claims against the repo. The line I use:
   *short-and-true beats long-and-stale — one stale line fails the check.*
2. **One-command verification** — a single command that tells the agent whether the
   repo is broken, floor (compiles/lints) to ceiling (tests). Verified by running it.
3. **Conventions + golden path** — "our way" written down, plus one real file to copy.
   The patterns you'd correct in code review are exactly the ones the agent can't guess.
4. **Territory map** — where things live, where a change usually starts. A few lines,
   not a wiki.
5. **Guardrails** — the off-limits list: secrets, generated files, never-do actions.
   The one check allowed to end as "ask the user."

**Details worth knowing before someone asks:**

- It audits *before* scaffolding, because "a wrong file is worse than a missing one —
  the agent believes the repo."
- When scaffolding conventions it doesn't invent any — it derives them from what the
  repo already repeats — with one exception: it seeds a **simplicity bar** house rule
  (stdlib before dependency, one line before fifty, YAGNI), and points at the external
  `ponytail` skill for teams that want that bar enforced per-change.
- Exit check: it runs `gaps-report` on the onboarding file. Done = five passes + an
  empty gaps report.

---

## The pipeline, stage by stage

### grilling

**What it does.** Interviews you relentlessly about a plan — walking the design tree,
resolving dependent decisions one by one, with a recommended answer attached to every
question.

**The one idea.** One question at a time. Multiple questions at once are bewildering,
and — the important part — *if a question can be answered by exploring the codebase,
it explores instead of asking*. So you only get asked things only you can answer.

**Worth pointing out.** It's the shortest skill in the pack (~10 lines) and one of the
most reused: `write-a-prd` and `improve-codebase-architecture` both call it. Good
moment to make the point that skills compose — small skills become subroutines for
bigger ones.

### write-a-prd

**What it does.** Interview + codebase exploration + module design → a six-pillar PRD
saved to `prds/<feature-slug>.prd.md`.

**The one idea.** The PRD is grounded in the code's reality, not in wishes: the skill
explores the repo to *verify the user's assertions* before planning anything.

**The flow** (it calls three other skills — this is the composition showcase):

1. Ask for a long, detailed problem description.
2. Explore the repo to verify the assertions.
3. `grilling` — walk the design tree until shared understanding.
4. Sketch the major modules, using the `codebase-design` vocabulary — actively hunting
   for deep modules that can be tested in isolation. Confirm modules and test targets
   with the user.
5. Write the PRD from the six-pillar template.
6. Exit check: `gaps-report`, loop until empty.

**The six pillars**: Problem Statement · Solution · User Stories (long, numbered,
exhaustive) · Implementation Decisions · Testing Decisions · Out of Scope. Two details
I always call out:

- *No file paths or code snippets in the PRD* — they go stale too fast. Durable
  decisions only.
- The Out of Scope section "carries the weight of the PRD — every non-goal you write
  down is a scope argument nobody has later."

### gaps-report

**What it does.** Spawns a fresh-context, read-only subagent that gets *only the
document path* — nothing from the conversation — reads it cold, tests its claims
against the code, and reports every gap. Loop until a fresh reader says `NO GAPS`.

**The one idea.** *The author of a document can't find its gaps — your context fills
them silently, from memory the document doesn't contain. The only honest reader is a
cold one.* This is the pack's quality gate, and it's the clean-context principle
turned into a tool.

**Mechanics people miss:**

- Every gap gets folded *into the document*, never answered only in conversation —
  because the next cold reader (human or agent) only gets the document.
- Each round spawns a **new** reader — the previous one is warm now.
- If the same gap keeps returning reworded, that's a real ambiguity the document can't
  resolve — surface it to the user instead of looping forever.

It's the exit check for `agent-ready`, `write-a-prd`, and `prd-to-slices`.

### prd-to-slices

**What it does.** Breaks a PRD into independently-grabbable vertical slices — one
markdown file per slice in `slices/<feature>/`, plus a `00-overview.md` — so a builder
(human or agent, in a clean context) grabs one slice plus the overview and starts.

**The one idea.** **Tracer bullets.** Each slice cuts through *all* integration layers
end-to-end (schema, API, UI, tests) — never a horizontal slice of one layer. A finished
slice is demoable on its own.

**The moving parts:**

- **Durable decisions first**: routes, schema shape, key models, auth approach go in
  the overview once, so slices reference instead of duplicating. Same staleness rule
  as the PRD — no file names or function names that earlier slices will invalidate.
- **HITL vs AFK**: HITL slices need a human (a design review, an architectural call);
  AFK slices can be built and merged unattended. Prefer AFK. This vocabulary lands
  well — it names the thing people actually care about: "can I fire and forget this?"
- **Quiz step**: the breakdown is presented as a numbered list (title, type, blocked-by,
  user stories covered) and iterated until the user approves granularity and
  dependencies.
- Exit check: `gaps-report` on the whole folder — "a builder gets one slice file and
  the overview, nothing else — the gaps report tells you whether that's enough."

### build (not a skill — a deliberate gap)

The pipeline says "your coding agent, one slice per clean context." There's no `build`
skill on purpose: the whole point of the slice files is that any coding agent in a
fresh context can pick one up. Worth saying explicitly, because someone will ask where
the build skill is.

### code-review

**What it does.** Reviews the diff since a fixed point along two axes in **parallel
sub-agents** — **Standards** (does the code follow this repo's documented rules?) and
**Spec** (does it do what the slice/PRD asked?) — then reports them side by side,
never merged.

**The one idea.** A change can pass one axis and fail the other: perfectly conventional
code that implements the wrong thing, or exactly-what-was-asked code that breaks house
style. Merging the reports lets one axis mask the other, so the skill refuses to pick
a single winner across axes.

**The moving parts:**

- Pins the fixed point first (`git diff <point>...HEAD`, three-dot) and fails fast on
  a bad ref or empty diff — before spawning anything.
- Finds the spec by walking: user-supplied path → `slices/` matching the branch →
  issue refs in commits → `docs/`/`specs/`. No spec found = Spec agent skips and says
  so.
- The Standards axis always carries a **smell baseline** — twelve Fowler smells
  (Mysterious Name, Duplicated Code, Feature Envy, Data Clumps, Primitive Obsession,
  Repeated Switches, Shotgun Surgery, Divergent Change, Speculative Generality,
  Message Chains, Middle Man, Refused Bequest), each phrased as *what it is → how to
  fix*. Two binding rules: **the repo's documented standard always overrides the
  baseline**, and **baseline smells are always judgement calls, never hard violations**.
  Anything tooling already enforces is skipped.

### handoff (pipeline-adjacent)

**What it does.** User-invoked. Compacts the current conversation into a handoff
document a fresh agent can continue from, saved to the OS temp dir — not the workspace.

**The one idea.** It's the escape hatch for when context runs long mid-task: the same
files-over-conversation principle, applied to the conversation itself.

**Nice details**: it takes an argument describing what the next session will focus on
and tailors the doc to it; it includes a "suggested skills" section for the next agent;
it references PRDs/ADRs/commits by path instead of duplicating them; it redacts secrets.

---

## Building skills

### tdd

**What it does.** Red-green-refactor with tracer bullets: one test → one implementation
→ repeat, each cycle informed by the last.

**The one idea.** Tests verify *behavior through public interfaces*, not implementation.
The named anti-pattern is **horizontal slicing** — writing all tests first, then all
code. Tests written in bulk test *imagined* behavior; they go insensitive — passing
when behavior breaks, failing when it's fine. Same tracer-bullet logic as
`prd-to-slices`, one level down.

**Also in there:**

- **Tautological tests** — assertions that recompute the expected value the way the
  code does (`expect(add(a,b)).toBe(a+b)`), so they can never disagree with the code.
  Expected values must come from an independent source of truth.
- Planning step confirms with the user *which* behaviors matter — "you can't test
  everything" — and looks for deep modules via `codebase-design`.
- Reads `CONTEXT.md` (if present) so test names use the project's domain language.
- Never refactor while RED.
- Reference files: `tests.md` (good/bad test examples), `mocking.md` (mock at system
  boundaries only — external APIs, time, randomness; never your own modules).

### diagnosing-bugs

**What it does.** A six-phase discipline for hard bugs: build a feedback loop →
reproduce + minimise → hypothesise → instrument → fix + regression test → cleanup +
post-mortem.

**The one idea.** Phase 1 *is* the skill: build a **tight, red-capable feedback loop**
before touching any theory. "Build the right feedback loop, and the bug is 90% fixed."
The skill hard-blocks the classic failure: *if you catch yourself reading code to build
a theory before the loop command exists, stop.* No red-capable command, no Phase 2.

**The phases, one line each:**

1. **Feedback loop** — ten ways to build one, in preference order (failing test, curl
   script, CLI + fixture, headless browser, trace replay, throwaway harness, fuzz loop,
   bisection harness, differential loop, and last-resort HITL bash script — there's a
   template at `scripts/hitl-loop.template.sh` that drives *the human* so even manual
   clicking stays structured). Then tighten: faster, sharper, deterministic. "A
   30-second flaky loop is barely better than no loop; a 2-second deterministic one is
   a debugging superpower." Flaky bugs: don't chase a clean repro, raise the
   reproduction rate until it's debuggable.
2. **Reproduce + minimise** — watch it go red on the *user's* symptom (wrong bug =
   wrong fix), then cut everything non-load-bearing one piece at a time.
3. **Hypothesise** — 3–5 *ranked, falsifiable* hypotheses before testing any ("if X is
   the cause, changing Y makes it disappear"). Show the list to the user — they often
   re-rank it instantly — but don't block on them.
4. **Instrument** — one probe per prediction, one variable at a time; debugger over
   logs; every debug log tagged `[DEBUG-xxxx]` so cleanup is one grep. Perf bugs:
   measure and bisect, don't log.
5. **Fix + regression test** — test before fix, *but only at a correct seam*. If no
   correct seam exists, that itself is the finding: the architecture is preventing the
   bug from being locked down.
6. **Cleanup + post-mortem** — grep out the tags, state the winning hypothesis in the
   commit message, then ask "what would have prevented this?" If the answer is
   architectural, hand off to `improve-codebase-architecture` — *after* the fix, when
   you know more.

### prototype

**What it does.** Throwaway code that answers exactly one design question. Two
branches: **logic** (`LOGIC.md` — a tiny interactive terminal app to push a state model
through hard cases) or **UI** (`UI.md` — several radically different variations on one
route, switchable from a floating bottom bar).

**The one idea.** The question decides the shape, and *the answer is the only thing
worth keeping*. When done: capture the verdict somewhere durable (commit message, ADR,
NOTES.md), then delete or absorb the prototype.

**The rules** (both branches): clearly marked throwaway, located next to what it
prototypes; one command to run; no persistence by default (persistence is usually what's
being *checked*); no polish — no tests, no abstractions; surface the full state after
every action so you can see what changed.

---

## Design skills

### codebase-design

**What it does.** Not a process — a **shared vocabulary** for designing deep modules,
which three other skills (`write-a-prd`, `tdd`, `improve-codebase-architecture`) import.
This is the pack's ubiquitous language for architecture.

**The seven terms** (the skill insists on these exactly — no "component", "service",
"API", or "boundary"):

- **Module** — anything with an interface and an implementation; scale-agnostic.
- **Interface** — *everything* a caller must know: types, but also invariants,
  ordering, error modes, performance. Deliberately wider than "signature."
- **Implementation** — what's inside.
- **Depth** — leverage at the interface: behavior exercised per unit of interface
  learned. Deep = lots of behavior behind a small interface.
- **Seam** (Feathers) — where you can alter behavior without editing in that place;
  where the interface lives. Its placement is its own design decision.
- **Adapter** — a concrete thing satisfying an interface at a seam; names a role, not
  substance.
- **Leverage / Locality** — what depth buys: leverage for callers (one implementation
  pays back across N call sites), locality for maintainers (fix once, fixed
  everywhere).

**The principles I quote:**

- **The deletion test** — imagine deleting the module. Complexity vanishes → it was a
  pass-through. Complexity reappears across N callers → it was earning its keep.
- **The interface is the test surface** — if you want to test *past* the interface,
  the module is the wrong shape.
- **One adapter means a hypothetical seam; two adapters means a real one.**
- Testability rules: accept dependencies, don't create them; return results, don't
  produce side effects.

It even documents **rejected framings** (e.g. Ousterhout's lines-ratio definition of
depth — rewards padding the implementation) — useful if someone in the room knows the
book. Reference files: `DEEPENING.md` (how to deepen a cluster safely, classified by
dependency type) and `DESIGN-IT-TWICE.md` (parallel sub-agents design the same
interface radically differently, then compare on depth, locality, seam placement).

### domain-modeling

**What it does.** Actively builds and sharpens the project's domain model *as you
design*: a `CONTEXT.md` glossary at the root (or per-context under a `CONTEXT-MAP.md`
in multi-context repos) and ADRs in `docs/adr/`.

**The one idea.** It's the *active* discipline — challenging terms, inventing edge-case
scenarios, writing things down the moment they crystallise. Merely reading `CONTEXT.md`
for vocabulary is not this skill; that's a one-line habit any skill can do (`tdd` and
`diagnosing-bugs` both do). This skill is for when you're *changing* the model.

**The five behaviors:**

- Challenge terms against the glossary ("your glossary defines 'cancellation' as X,
  but you seem to mean Y — which is it?").
- Sharpen fuzzy language ("'account' — Customer or User? Different things.").
- Stress-test relationships with concrete edge-case scenarios.
- Cross-reference with code ("your code cancels entire Orders, but you just said
  partial cancellation is possible — which is right?").
- Update `CONTEXT.md` inline, the moment a term resolves — never batched. And
  `CONTEXT.md` is a glossary *only* — no implementation details, ever.

**ADRs are offered sparingly** — only when all three hold: hard to reverse, surprising
without context, the result of a real trade-off. Any one missing → skip. Files are
created lazily; formats live in `CONTEXT-FORMAT.md` and `ADR-FORMAT.md`.

### improve-codebase-architecture

**What it does.** User-invoked. Explores the codebase for **deepening opportunities**
(shallow → deep refactors), renders them as a visual HTML report that opens in the
browser, then grills through whichever candidate you pick. This is the pack's showpiece
demo — the report is genuinely impressive on screen.

**The flow:**

1. **Explore** — reads `CONTEXT.md` and ADRs first, then walks the codebase with an
   Explore subagent, hunting friction organically: shallow modules, concept-bouncing,
   pure functions extracted "for testability" while the real bugs hide in the callers,
   untestable interfaces. Applies the deletion test to suspects.
2. **Report** — self-contained HTML in the temp dir (nothing lands in the repo),
   Tailwind + Mermaid via CDN. Each candidate is a card: files, problem, solution,
   benefits phrased as locality and leverage, a before/after diagram, and a
   recommendation badge (`Strong` / `Worth exploring` / `Speculative`). Ends with a top
   recommendation. Deliberately does *not* propose interfaces yet — it stops and asks
   which candidate to explore.
3. **Grilling loop** — `grilling` walks the design tree of your pick, while
   `domain-modeling` runs inline as side effects: new module concept → term added to
   `CONTEXT.md`; candidate rejected for a load-bearing reason → offer an ADR so future
   reviews don't re-suggest it.

**Why it's the composition finale**: it calls `codebase-design` (vocabulary),
`grilling` (interview), and `domain-modeling` (side effects) — three skills working as
one. If a candidate contradicts an existing ADR it only surfaces it when the friction
justifies reopening the decision, clearly flagged.

---

## Meta

### writing-great-skills

**What it does.** User-invoked reference (no steps — "this skill is all reference") for
writing skills of your own. Comes with a 195-line `GLOSSARY.md` — a full domain model
for skill-writing.

**The one idea.** *A skill exists to wrangle determinism out of a stochastic system.*
The root virtue is **predictability** — the agent taking the same *process* every run,
not producing the same output. Every other concept is a lever on it.

**The concepts I'd walk through, in order:**

- **The invocation trade-off** — model-invoked spends *context load* (description in
  the window every turn); user-invoked spends *cognitive load* (you must remember it
  exists). This names exactly why the pack itself splits 11/3. When user-invoked
  skills pile past memory, the cure is a **router skill** that indexes the others.
- **Information hierarchy** — a ladder: in-skill steps → in-skill reference → external
  reference behind a pointer. **Progressive disclosure** is the move down it. The
  branching test: inline what every branch needs, push behind a pointer what only some
  branches reach (exactly how `prototype` splits LOGIC/UI).
- **Completion criteria** — each step ends on a checkable, exhaustive "done" condition;
  vague criteria invite **premature completion**. (`diagnosing-bugs` Phase 1 is the
  worked example: "red-capable, deterministic, fast, agent-runnable.")
- **Leading words** — compact concepts already in the model's pretraining (*tracer
  bullets*, *tight*, *red*, *cold reader*) that anchor behavior in few tokens. The
  examples are drawn from this very pack: "fast, deterministic, low-overhead" collapsed
  to *tight*; "a loop you believe in" collapsed to *red*.
- **Failure modes** — premature completion, duplication, **sediment** (stale layers
  that settle because adding feels safe and removing feels risky — "the default fate
  of any skill without a pruning discipline"), **sprawl**, and **no-ops** (lines the
  model already obeys — "be thorough" — pure cost; the fix for a weak leading word is
  a stronger word, not more words).

**Facilitation gold**: after walking the pack, this skill lets attendees see *how the
sausage was made* — every skill in the pack demonstrably practices these principles,
and you can point back at them as examples.

---

## The wiring diagram

Who calls whom — this is the slide that lands the "skills compose" point:

```
agent-ready ──────────────────────────► gaps-report
write-a-prd ──► grilling
            ──► codebase-design (vocabulary)
            ──────────────────────────► gaps-report
prd-to-slices ────────────────────────► gaps-report
tdd ──────────► codebase-design (vocabulary)
    ──────────► CONTEXT.md (reads — domain-modeling's artifact)
diagnosing-bugs ► CONTEXT.md (reads)
                ► improve-codebase-architecture (post-mortem handoff)
improve-codebase-architecture ──► codebase-design
                              ──► grilling
                              ──► domain-modeling
code-review ──► prds/ + slices/ (reads — the pipeline's artifacts)
```

Three hub patterns worth naming out loud:

1. **`gaps-report` is the universal exit check** — three skills end by running it.
2. **`codebase-design` is the shared vocabulary** — three skills import its terms
   instead of redefining them. Single source of truth, practiced.
3. **`CONTEXT.md` connects design-time to build-time** — `domain-modeling` writes it,
   `tdd` and `diagnosing-bugs` read it, `improve-codebase-architecture` does both.

## Recurring themes (the through-lines I keep hitting)

- **Clean context as a first-class constraint.** Files over conversation; cold readers
  over warm ones; one slice per fresh context; sub-agents that don't pollute each
  other. Every skill assumes the reader/builder starts cold.
- **Tracer bullets at every altitude.** Slices cut through all layers end-to-end;
  TDD does one test → one implementation; a prototype answers one question. Thin,
  complete, verifiable — never broad and horizontal.
- **Verify against reality, not intention.** agent-ready spot-checks every claim;
  write-a-prd explores before planning; domain-modeling cross-references code;
  gaps-report tests documents against the repo. Nothing passes on intention.
- **Exhaustive exit checks.** "Loop until NO GAPS", "every remaining element is
  load-bearing", "all five checks pass" — checkable completion criteria everywhere,
  which is `writing-great-skills` theory made flesh.

## Suggested presentation orders

**Pipeline-first** (for the workshop's arc): agent-ready → grilling → write-a-prd →
gaps-report → prd-to-slices → code-review, then "and around the pipeline:" tdd /
diagnosing-bugs / prototype, then the design trio, closing on writing-great-skills as
the reveal.

**Concept-first** (for a shorter session): open with the two invocation kinds and the
files-between-stages idea, show the pipeline diagram, deep-dive only three skills —
gaps-report (clean context), diagnosing-bugs (completion criteria), and
improve-codebase-architecture (composition) — and hand out this guide for the rest.
