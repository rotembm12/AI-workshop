# Enpitech Agent Skills

Our AI-first engineering workflow, as installable Claude Code skills: take a fuzzy feature sentence to merged code through grounded context, a PRD, vertical slices, and a two-axis review — with a clean-context check at every altitude.

## Install

```sh
npx skills@latest add rotembm12/AI-workshop
```

Starting from zero? [SETUP.md](SETUP.md) is the five-minute checklist — Claude Code install through smoke test.

Pick the skills you want when prompted. There is no setup step — every skill works with plain markdown files in your repo. No issue tracker, no auth, nothing to configure.

## Start here

Run `agent-ready` in your repo first. It audits against five checks — onboarding file, one-command verification, conventions + a golden-path example, territory map, guardrails — and scaffolds what's missing. Every other skill in this pack works better in a repo that passes.

## The pipeline

```
fuzzy sentence
  → grilling          interview until the goal is grounded in the code's reality
  → write-a-prd       six-pillar PRD                → prds/<feature>.prd.md
  → gaps-report       a cold reader finds what's
                      missing — loop until empty
  → prd-to-slices     vertical slices               → slices/<feature>/NN-<slice>.md
  → build             your coding agent, one slice per clean context
  → code-review       Standards + Spec, in parallel sub-agents
```

Each stage produces a file the next stage reads — so every handoff can happen in a fresh context, and a cold reader (`gaps-report`) can tell you whether the file stands on its own.

## The skills

Two kinds, deliberately. A **user-invoked** skill fires only when you type its name — it costs you remembering it exists, and costs the agent nothing. A **model-invoked** skill carries a description the agent reads every turn — it can fire on its own, and other skills can call it.

### You fire these, by name

| Skill | What it does |
|---|---|
| `improve-codebase-architecture` | Scans for deepening opportunities, presents them as a visual HTML report, then grills through your pick |
| `handoff` | Compacts the conversation into a handoff document a fresh agent can continue from |
| `writing-great-skills` | Reference for writing skills of your own — invocation trade-offs, information hierarchy, pruning |

### The agent fires these itself (you can too)

| Skill | What it does |
|---|---|
| `agent-ready` | Audits the repo against the five agent-ready checks, then scaffolds what's missing |
| `grilling` | Relentless interview to sharpen a plan — one question at a time, recommended answer included; other skills call it too |
| `write-a-prd` | Interview + codebase exploration + module design → a six-pillar PRD in `prds/` |
| `gaps-report` | Spawns a cold-reading subagent on a document; reports gaps; loop until empty |
| `prd-to-slices` | Breaks a PRD into independently-grabbable vertical slices in `slices/` |
| `code-review` | Two-axis review — Standards and Spec — in parallel sub-agents |
| `tdd` | Red-green-refactor with tracer bullets; behavior-level integration tests |
| `diagnosing-bugs` | Diagnosis loop for hard bugs: build a tight feedback loop first, hypothesize second |
| `prototype` | Throwaway prototype to answer one design question — logic or UI |
| `codebase-design` | The deep-module vocabulary: interface, depth, seam, leverage, locality |
| `domain-modeling` | Keeps `CONTEXT.md` and ADRs sharp as design decisions crystallize |

## Local-first, by design

PRDs live in `prds/`, slices in `slices/` — markdown in your repo, versioned with your code. If your team lives in GitHub issues instead: in `write-a-prd`, replace the save-to-`prds/` step with `gh issue create`; in `prd-to-slices`, create one issue per slice (blockers first, so `Blocked by` lines can reference real issue numbers) instead of writing files. Everything else works unchanged.

## License

MIT — see [LICENSE](LICENSE) for the full notice and attribution.
