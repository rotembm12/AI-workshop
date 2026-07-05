---
name: prd-to-slices
description: Break a PRD into independently-grabbable vertical slices (tracer bullets), one markdown file per slice in ./slices/. Use when user wants to slice a PRD, break it into work items, or mentions "tracer bullets".
---

# PRD to Slices

Break a PRD into independently-grabbable vertical slices (tracer bullets). Output is one markdown file per slice in `./slices/<feature>/`, so a builder — human or agent, in a clean context — can grab one slice file plus the overview and start.

## Process

### 1. Locate the PRD

The PRD should be in the conversation or in `./prds/`. If it's neither, ask the user to point you to it.

### 2. Explore the codebase

If you have not already explored the codebase, do so to understand the current architecture, existing patterns, and integration layers.

### 3. Identify durable architectural decisions

Before slicing, identify high-level decisions that are unlikely to change throughout implementation:

- Route structures / URL patterns
- Database schema shape
- Key data models
- Authentication / authorization approach
- Third-party service boundaries

These go in the overview file so every slice can reference them without duplicating them.

### 4. Draft vertical slices

Break the PRD into **tracer bullet** slices. Each slice is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices may be 'HITL' or 'AFK'. HITL slices require human interaction, such as an architectural decision or a design review. AFK slices can be implemented and merged without human interaction. Prefer AFK over HITL where possible.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Do NOT include specific file names, function names, or implementation details that are likely to change as earlier slices are built
- DO include durable decisions: route paths, schema shapes, data model names
</vertical-slice-rules>

### 5. Quiz the user

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **User stories covered**: which user stories from the PRD this addresses

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

Iterate until the user approves the breakdown.

### 6. Write the slice files

Create `./slices/<feature-slug>/`. Write:

**`00-overview.md`** — the shared context every slice points back to:

<overview-template>
# <Feature Name> — slices

> Source PRD: ../../prds/<feature-slug>.prd.md

## Durable architectural decisions

- **Routes**: ...
- **Schema**: ...
- **Key models**: ...
- (add/remove sections as appropriate)

## Slice index

| # | Slice | Type | Blocked by |
|---|-------|------|------------|
| 1 | <title> | AFK | — |
| 2 | <title> | AFK | 1 |
</overview-template>

**`NN-<slice-slug>.md`** — one per slice, numbered in dependency order:

<slice-template>
# Slice NN: <Title>

> Overview: ./00-overview.md · Parent PRD: ../../prds/<feature-slug>.prd.md

**Type**: HITL / AFK
**Blocked by**: Slice NN (or "None — can start immediately")

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation. Reference specific sections of the parent PRD rather than duplicating content.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## User stories addressed

Reference by number from the parent PRD:

- User story 3
- User story 7
</slice-template>

### 7. Exit check

Run the `/gaps-report` skill on the slice folder (overview + every slice). A builder gets one slice file and the overview, nothing else — the gaps report tells you whether that's enough. Loop until it comes back empty.
