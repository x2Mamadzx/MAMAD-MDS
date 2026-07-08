---
name: Orval zod codegen vs installed zod version
description: Why OpenAPI string format:email breaks the generated zod schemas in this workspace
---

Orval's zod client generator emits zod v4 top-level helpers (e.g. `zod.email()`)
for certain string `format` values in OpenAPI schemas (email, uuid, etc.),
regardless of which zod major version is actually installed in the workspace.

**Why:** This workspace's `zod` catalog version is 3.25.x. `zod.email()` does not
exist as a top-level export on zod v3's default import, only under `zod/v4`.
Orval's generated import (`import * as zod from 'zod'`) resolves to the v3
default export, so `tsc --build` fails with
`Property 'email' does not exist on type ...` in `lib/api-zod/src/generated/api.ts`.

**How to apply:** When writing OpenAPI schemas consumed by this workspace's
orval + zod codegen pipeline, avoid `format: email` (and likely other formats
that map to zod v4-only helpers). Use a plain `type: string` with `minLength`
instead, and validate stricter formats (e.g. actual email shape) at the
application layer if needed. Re-run `pnpm --filter @workspace/api-spec run
codegen` after any schema change to catch this early via the typecheck step
it runs.
