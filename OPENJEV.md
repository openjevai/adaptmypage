# OpenJEV support

This fork adds **optional** support for [OpenJEV](https://openjev.sh) — a free
community gateway to the same Jev model — alongside the existing TypeSafe
integration. **TypeSafe stays the default**; anyone with a TypeSafe key sees
zero behaviour change. Jev is built by [TypeSafe](https://typesafe.ai); OpenJEV
is a community gateway to it, not a replacement.

## What was added

All changes are additive — nothing TypeSafe was renamed, removed or
re-defaulted.

- `packages/adaptmypage/src/server/jev.ts` — new `openjev` provider `kind` in the
  `JevProvider` union (endpoint `https://api.openjev.sh`, model `openjev`, key
  env `OPENJEV_API_KEY`). `resolveProvider` now falls back to OpenJEV after
  TypeSafe / Vercel Gateway / OpenRouter, and honours `JEV_PROVIDER=openjev`.
  `providerName` reports `openjev:openjev`. The retry logic already covered HTTP
  503 (OpenJEV's unavailable status) and 429 via `>= 500` / `429`; a comment
  documents this.
- `packages/adaptmypage/src/server/evaluate.ts` — docstring updated to list the
  full resolution order.
- `packages/adaptmypage/test/evaluate.test.ts` — `resolveProvider` test extended
  to cover `OPENJEV_API_KEY` (lowest-priority network provider) and
  `JEV_PROVIDER=openjev` (explicit env wins over keys).
- `apps/web/src/app/api/intent/route.ts` — provider comment updated.
- `README.md`, `packages/adaptmypage/README.md` — `OPENJEV_API_KEY` added to the
  env-var examples; the package README `provider` option now lists `openjev`.
- `SECURITY.md` — `OPENJEV_API_KEY` added to the list of server-only keys.

## Provider selection rule

1. Explicit `provider: { kind: "openjev" }` (or `JEV_PROVIDER=openjev` env) wins.
2. Otherwise `TYPESAFE_API_KEY` → TypeSafe (unchanged default).
3. Otherwise `AI_GATEWAY_API_KEY` → Vercel AI Gateway.
4. Otherwise `OPENROUTER_API_KEY` → OpenRouter.
5. Otherwise `OPENJEV_API_KEY` → OpenJEV.
6. Otherwise the built-in heuristic evaluator (no network).

## How to configure

Set one credential:

```bash
OPENJEV_API_KEY=oj_…   # from https://openjev.sh/dashboard
```

Or select OpenJEV explicitly even when other keys are present:

```bash
JEV_PROVIDER=openjev
```

Or programmatically:

```ts
createIntentHandler({ provider: { kind: "openjev" } });
```

## How it was verified

- A live `POST https://api.openjev.sh/v1/systemone` request with model
  `openjev`, state `ping`, and one `noul` question returned **HTTP 200**.
- `grep` confirms no new hardcoded `api.typesafe.ai` default was introduced;
  the TypeSafe default remains in place and OpenJEV is purely additive.

## Upstream

Original project: https://github.com/bvicsay/adaptmypage by @bvicsay.
