# Layover API conventions

> Disclosed reference for [`add-endpoint-our-way`](../SKILL.md). The agent reaches
> this from step 4 only when a slice needs it — it is **not** loaded with the main
> skill. Keep team-wide API rules here, in one place, so `SKILL.md` stays small.

## Error envelope

Every error response is JSON with exactly these two fields:

```json
{ "error": "human-readable message", "code": "MACHINE_READABLE_CODE" }
```

`code` is `SCREAMING_SNAKE_CASE` and stable — clients switch on it, so never
rename an existing one.

## Status codes

| Code | Use for |
|------|---------|
| `200` | Successful read or update |
| `201` | Resource created |
| `400` | Invalid input (failed validation) |
| `401` / `403` | Unauthenticated / not allowed |
| `404` | Resource not found |
| `409` | Conflict (e.g. duplicate) |
| `500` | Unexpected server error — never thrown deliberately |

## Pagination — list endpoints only

> This section applies **only** when the slice is a list/collection endpoint.
> Single-resource endpoints ignore it. (This is the branch that earns this file
> its own home — most endpoints never read past here.)

- Accept `limit` (default `20`, max `100`) and `offset` (default `0`) as query params.
- Return `{ items: [...], total: <number> }` — never a bare array, so the client
  always gets the total count.
- Validate `limit`/`offset` like any other input: out-of-range → `400`.

## Naming

- Routes are plural nouns: `/trips`, `/trips/:id/days`.
- Handler files match the resource: `src/api/trips.ts`.
- Repository methods read as sentences: `trips.findById`, `trips.saveForOwner`.
