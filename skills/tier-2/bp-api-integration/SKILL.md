---
name: bp-api-integration
description: Use when integrating a backend endpoint or API contract (OpenAPI, Postman, contract doc, sample JSON) into a feature — endpoint constants, raw types, mappers, success/error code handling, adapters, and mock adapters.
---

# API Integration

Infrastructure-side workflow. Layer ownership rules come from `bp-architecture`.

## Stack context (read first)

Read `<guideline>/stack-profile.md` → **HTTP** (client, envelope shape, success codes, endpoints file, error normalizer) and `<guideline>/references/<stack>.md` § 6.

If `mode: competition` in the profile, apply overrides from `bp-competition-mode`.

## Inputs to gather

Read the contract first from the contracts folder in the profile (**HTTP → API contracts location**, default `<guideline>/api-contracts/<feature>.md`). If the folder does not exist, copy `<guideline>/scaffold/api-contracts/` there. When the user gives a contract in chat or a sample response, save it there using `_template.md` (personal data removed) before coding.

1. Method + path + query/body params.
2. Response envelope and the success code(s) — HTTP status **and** backend business code if the API has one.
3. Error codes with meaning (e.g. `ABC40901 = slot full`).
4. Which fields are guaranteed vs optional; date formats; enums.

If the contract is missing, ask for it or a real sample response (competition mode: see `bp-competition-mode` § Assumptions — a mock adapter with assumed fields is acceptable there, recorded in `ASSUMPTIONS.md`). Do not guess field names silently.

## Steps

1. **Endpoint**: add to `core/http/api-endpoints.ts` (grouped by feature). No inline URLs in adapters.
2. **Raw type**: `infrastructure/<entity>.api.ts` — exact transport shape, all fields optional unless guaranteed.
3. **Mapper**: `infrastructure/<entity>.mapper.ts` with `toEntity(raw)` (and `toPayload(input)` for writes).
   - Normalize: defaults, enum casing (`'FINISHED' → 'finish'`), dates to one type, drop unused fields.
   - Required business fields missing → return an error from the adapter (`type: 'unknown'`/`'server'`), do not silently fabricate them.
   - Mapper is pure and **tested first** (`bp-testing`).
4. **Business codes**: if the backend uses business codes, add them to the feature's constants and map them to `AppError.type` through the shared normalizer, so the UI can branch on `type`, never on raw codes.
5. **Adapter**: implements the port, calls the http client, checks success (HTTP + business code), maps, returns `Result`. Never throws.
6. **Mock adapter** (`Mock<Entity>Repository`): same port, deterministic raw data passed through the **real mapper**; selectable in the composition root for local dev/tests. Mock code must not ship in production client bundles — select it at build time or on the server (`references/<stack>.md` § 8), not with a runtime flag in client code.
7. **Wire** in the composition root (`references/<stack>.md` § 8).

## Rules

- Pagination meta is mapped to a feature-neutral shape (`{ page, pageSize, total, totalPages }`).
- Never pass `status`, `meta`, or `error` envelopes to presentation; return domain data plus a user-facing `message` only if the screen needs it.
- File downloads (Blob + filename) get a named type in `application/`, not inline `Promise<Blob>`.
- Auth headers, base URL, and retries belong to `core/http`, not to each adapter.

## Done criteria

- Mapper tests cover: full payload, missing optional fields, unknown enum value, invalid date.
- Adapter returns `err` for HTTP failure, business-code failure, and network error.
- No raw type imported outside `infrastructure/`.

## Examples

Worked examples (fictional `topup` feature): `examples.md` in this skill folder. Read it when the rules above leave a placement or format decision unclear.
