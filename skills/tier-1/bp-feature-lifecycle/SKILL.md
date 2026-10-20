---
name: bp-feature-lifecycle
description: Use when building a feature end-to-end from requirements, a ticket, an API contract, or a design — or when asked to scaffold, dry-run, or "execute phase N" of a feature. Orchestrates spec, domain, ports, application, infrastructure, DI, UI, and verification in order.
---

# Feature Lifecycle (orchestrator)

Drives a feature through fixed phases. Each phase delegates rules to the owning skill. Phases can run all at once or one at a time ("execute phase 4").

## Stack context (read first)

Read `<guideline>/stack-profile.md` and `<guideline>/references/<stack>.md`. If either is missing or incomplete for the phase you are in, stop and ask (competition mode: see `bp-competition-mode` § Assumptions).

If `mode: competition` in the profile, apply overrides from `bp-competition-mode`.

## Phases

| # | Phase | Output | Owning skill |
|---|---|---|---|
| 0 | **Discover** | Open questions answered: scope, screens, endpoints, roles, edge cases, empty/error states | — |
| 1 | **Spec** | Numbered behavior specs (`specs/<feature>/NN-<subject>.md`, Given/When/Then) | `bp-testing` |
| 2 | **Domain** | Types + pure rules, test-first | `bp-architecture`, `bp-testing` |
| 3 | **Ports** | Contracts in `application/` returning `Result` | `bp-architecture` |
| 4 | **Application** | Use cases **only if logic exists**, test-first | `bp-architecture`, `bp-testing` |
| 5 | **Infrastructure** | Endpoints, raw types, mappers (tested), adapters, mock adapter | `bp-api-integration` |
| 6 | **DI** | Register adapters/use cases/store in the composition root | `references/<stack>.md` § 8 |
| 7 | **UI** | Store/hooks (branching logic test-first) → components from shared atoms → page/route | `bp-ui-components`, `bp-figma-to-code` |
| 8 | **Verify** | Done criteria green; summary of files and decisions | `bp-conventions` |

## Rules

1. **Do not skip phases silently.** If a phase is not needed (e.g. no use case), state why in the summary.
2. **Phase 0 first** when requirements are vague. Ask grouped questions once; do not interleave questions with code.
3. **Specs before code.** Each spec names the subject, collaborators to fake, and observable outcomes.
4. **Inside-out order**: domain → ports → application → infrastructure → DI → UI. Never start from the page.
5. **Each phase leaves the code compiling** and its tests passing.
6. **Dry-run mode** ("scaffold", "plan only"): create specs and empty files with correct names/locations, no implementation.
7. If the design shows a surface that does not exist yet, it is a **new feature folder** — do not attach it to the closest existing feature.
8. Update `<guideline>/page-map.md` with the new route, its raw-prompt aliases, and entry files (create it from `<guideline>/scaffold/page-map.md` if missing).

## Phase 0 question checklist

- What user-visible outcome defines "done"?
- Which screens/routes? New or existing surface?
- Which endpoints? Is there a contract (OpenAPI, doc)? Success and error codes?
- Who can see/do what (roles, permissions)?
- Empty, loading, error, and partial states?
- Pagination, sorting, filtering, search?
- Anything cached, persisted locally, or shared with another feature?

## Done criteria

- All phases completed or explicitly skipped with a reason.
- `bp-conventions` done criteria pass.
- Summary lists: files created/changed per layer, decisions taken, open questions.

## Examples

Worked examples (fictional `topup` feature): `examples.md` in this skill folder. Read it when the rules above leave a placement or format decision unclear.
