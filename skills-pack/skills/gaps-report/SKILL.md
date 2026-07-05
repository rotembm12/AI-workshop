---
name: gaps-report
description: Clean-context gaps check of a document (PRD, plan, slice set) — a fresh subagent reads it cold and reports what's missing, ambiguous, or contradicted by the code. Use when a document needs checking before work builds on it, or when another skill needs an exit check on a document it produced.
---

The author of a document can't find its gaps — your context fills them silently, from memory the document doesn't contain. The only honest reader is a cold one.

## Process

### 1. Spawn a cold reader

Spawn a fresh-context subagent (read-only). Give it ONLY the document path(s) — nothing from this conversation. Its brief:

> You are the engineer who must build from this document, with no access to its author. Read it cold, then explore the repo to test its claims against the code. Report every gap: missing decisions, ambiguous terms, unstated assumptions, claims the code contradicts, acceptance criteria that can't be checked. Gaps only — no praise, no rewrites. If there are none, say exactly: NO GAPS.

### 2. Fold the gaps in

For each reported gap: if it's a decision, take it to the user; if it's factual, fix the document directly. Every gap gets folded into the document — none answered only in conversation, because the next cold reader (human or agent) only gets the document.

### 3. Loop until empty

Spawn a NEW cold reader on the revised document — never reuse the previous one, it's warm now. Done when a fresh reader comes back with NO GAPS. If the same gap keeps returning reworded across rounds, it's a real ambiguity the document can't resolve alone — surface it to the user instead of looping again.
