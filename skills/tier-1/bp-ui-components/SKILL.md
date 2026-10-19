---
name: bp-ui-components
description: Use when creating, refactoring, or reviewing UI — placing components in atomic-design levels, reusing shared atoms instead of raw HTML, composing components, rendering API data safely, and basic accessibility.
---

# UI Components (Atomic Design)

Owning skill for UI composition. Stack-specific template/markup syntax comes from `<guideline>/references/<stack>.md` § 10.

## Stack context (read first)

Read `<guideline>/stack-profile.md` → **Shared UI atoms available**, **Naming**. If the atom list is empty, stop and ask before creating new atoms (competition mode: see `bp-competition-mode` § Assumptions).

## Levels and placement

| Level | Examples | Location | May contain |
|---|---|---|---|
| Atom | Button, Input, Select, Checkbox, Badge, Tooltip | `shared/ui/atoms` | styling, states, a11y — no business logic |
| Molecule | SearchInput, FormField, Pagination | `shared/ui/molecules` | atoms composition |
| Organism | DataTable, FilterBar, Modal shell | `shared/ui/organisms` or feature `presentation/` if feature-specific | molecules/atoms; feature data via props |
| Template | Page layout of a screen | feature `presentation/` | organisms; no data fetching |
| Page | Route entry | per `references/<stack>.md` § 11 | wiring only |

Promote a feature-local component to `shared/ui` only when a second feature needs it.

## Core rules

1. **Never render raw `<input>`, `<button>`, `<select>`, `<textarea>`, `<table>`** when a shared atom exists. Pass props (variant, size, disabled, error) instead of re-styling. An atom may be a component or a styling directive applied to the native element (`references/<stack>.md` § 10); what is forbidden is a native element without the atom.
2. **Missing atom → create it once in `shared/ui`**, then use it. No one-off raw versions inside a page.
3. **When touching a file that uses raw form elements**, migrate them to atoms in the same change.
4. **Composition over boolean props**: use children/slots/compound components instead of `isX`, `showY`, `variantZ` flag piles.
5. **No business rules in UI**. Conditions like "can approve" come from domain functions via the store.
6. **Render API-derived values through `formatOrDash(value, format?)`** from `src/shared/lib/format.ts` — it returns `'-'` for `null`, `undefined`, empty string, or `NaN`, and applies the optional formatter otherwise. Do not hand-write `value || '-'` / `?? '-'` in components. If the helper does not exist yet, create it there (template in `references/<stack>.md` § 3) with tests.
7. **Error titles/messages come from the central message map** keyed by `AppError.type` (in `src/shared/lib`, so presentation may import it); no inline status-code maps in components.
8. **Repeated class strings or markup** (2+ times) → extract a component or a variant map.
9. **Every interactive element is keyboard reachable** and has an accessible name; clickable non-button elements are a smell — use the Button atom.
10. **Loading, empty, error states are designed**, not afterthoughts: use shared Skeleton / EmptyState / ErrorState components.

## Decision guide

- Same markup in two places → extract.
- Component file grows past the size limit in `bp-conventions` → split into organisms/molecules.
- Need a new visual variant → add a variant to the atom, do not fork it.
- Design shows something no atom expresses → create the atom first, then the screen.

## Done criteria

- No raw form elements where atoms exist (search the diff).
- Empty/loading/error states covered.
- Missing values render `'-'`.
- `bp-conventions` done criteria pass.

## Examples

Worked examples (fictional `topup` feature): `examples.md` in this skill folder. Read it when the rules above leave a placement or format decision unclear.
