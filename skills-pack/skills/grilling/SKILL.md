---
name: grilling
description: Interview the user relentlessly about a plan or design. Use when the user wants to stress-test a plan before building, uses any 'grill' trigger phrases, or when another skill needs an interview loop.
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time, waiting for feedback on each question before continuing. Asking multiple questions at once is bewildering.

If a question can be answered by exploring the codebase, explore the codebase instead.

## Output document

The deliverable of the interview is a handoff document at `./grill-<topic-slug>.md` in the current working directory (not a scratchpad or temp dir). Create it after the first few decisions land and update it as the interview progresses, so nothing is lost if the session stops early.

Write it for a fresh agent with zero conversation context — it should be able to start building from the document alone. Spell everything out; no references to "as discussed above" or the conversation. Structure:

- **Goal** — what is being built and why, in a few sentences.
- **Context** — relevant codebase facts discovered during the interview (files, existing patterns, constraints), with `path:line` references.
- **Decisions** — every decision made, one entry each: the question, the chosen answer, the rationale, and alternatives that were rejected and why.
- **Open questions** — anything left unresolved, with the current leaning if there is one.
- **Out of scope** — things explicitly deferred or cut.
- **Suggested first steps** — where a new agent should start.

When the interview ends, do a final pass over the document to make sure it is complete and self-contained, then tell the user the file path so they can hand it to a new session.
