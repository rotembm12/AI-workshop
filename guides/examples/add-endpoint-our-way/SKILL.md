---
name: add-endpoint-our-way
description: >-
  Add an HTTP API endpoint to Layover the way our team builds them — a thin
  vertical slice from route to a round-trip test. Use when adding or scaffolding
  a new endpoint or API route, exposing a resource over HTTP, or when someone
  says "add an endpoint", "new route", "wire up an endpoint", or "expose X over
  the API".
---

Build the endpoint as one **thin vertical slice**: route → validated handler →
repository call → a test that proves the round-trip. Depth first, width later —
get one path working end-to-end before adding fields, filters, or edge cases.

## Steps

1. **Find the pattern.** Read the two most recently added endpoints under
   `src/api/`. Match their file layout, error handling, and naming. Do not
   invent a new shape.
2. **Add the route** in `src/api/<resource>.ts` and register it in
   `src/api/index.ts`. One route per slice.
3. **Validate input** with a schema at the top of the handler. Reject invalid
   input with a `400` before doing any work.
4. **Call the repository, never the database directly** — the handler stays
   thin. Our error envelope, status codes, and pagination rules are in
   [`reference/conventions.md`](reference/conventions.md); read it when the
   slice needs them.
5. **Write the round-trip test** in `<resource>.test.ts`: call the endpoint,
   assert the response shape, assert the persisted row. The slice is not done
   until this test is green.

## Done when

Every item is true — not "mostly":

- The route is registered and reachable.
- Invalid input returns `400`; valid input returns the documented shape.
- A test proves the full round-trip and passes.
