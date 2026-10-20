---
name: bp-conventions
description: Use when checking naming, file size, folder placement, code style, or whether a change is actually done — including which verification commands must pass before claiming completion.
---

# Conventions & Definition of Done

## Stack context (read first)

Read `<guideline>/stack-profile.md` → **Commands**, **Naming**. Commands are always taken from the profile, never assumed.

If `mode: competition` in the profile, apply overrides from `bp-competition-mode`.

## Naming

| Item | Convention |
|---|---|
| Files & folders | per profile (default `kebab-case`) |
| Components | `PascalCase` |
| Functions, variables, stores, hooks | `camelCase`; hooks start with `use` — and **only** hooks: a plain function named `useX` breaks hook lint rules (name it `isX`/`getX`) |
| Types & interfaces | `PascalCase`, no `I` prefix unless the profile says so |
| Constants | `UPPER_SNAKE_CASE` for true constants |
| Mappers | `toX()` / `fromX()` |
| Ports | `<Entity>Repository` / `<Entity>Gateway` |
| Adapters | `Http<Entity>Repository`, `Mock<Entity>Repository` |

Use full words; avoid uncommon abbreviations. All identifiers, comments, and docs in code are English.

## File size limits

| Location | Max lines |
|---|---|
| `domain/` file | 150 |
| `application/` file | 150 |
| `infrastructure/` adapter | 200 |
| mapper | 150 |
| store / state hook | 200 |
| component | 250 |
| page / route file | 80 |

Over the limit → split by responsibility before finishing. Limits may be tuned in the profile; never ignore them.

## Code rules

1. TypeScript strict; no `any` (use `unknown` + narrowing).
2. One function = one responsibility; prefer early returns over deep nesting.
3. `async/await`; no `.then` chains.
4. No dead code, commented-out code, or `console.log`.
5. Follow existing patterns in the repo before introducing a new variation; if a new variation is needed, explain why.
6. Reusable pure helpers → `shared/` or feature `domain/`; page-only helpers may stay local until a second consumer appears.

## Definition of done

A change is done only when **all** of these hold:

1. `lint` passes (including boundary rules).
2. `typecheck` passes.
3. Tests for touched logic pass (`bp-testing` decides which tests are required), with red → green evidence for test-first targets.
4. `build` passes when routes, config, or shared types changed.
5. New routes are reachable through the real auth/permission flow, and navigation entries exist if the page should appear in menus.
6. No secrets, machine paths, or debug leftovers in the diff.
7. Skills/references updated if a shared pattern changed.

Report the commands you ran and their results. Never claim done on unverified work.

## Examples

Worked examples (fictional `topup` feature): `examples.md` in this skill folder. Read it when the rules above leave a placement or format decision unclear.
