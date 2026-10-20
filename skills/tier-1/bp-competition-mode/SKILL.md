---
name: bp-competition-mode
description: Use whenever stack-profile.md has `mode: competition` (competitions, hackathons, time-boxed POCs) — defines which rules of the other skills are relaxed, which stay mandatory, and how to proceed on missing information without stopping to ask.
---

# Competition Mode

Owning skill for time-boxed work. Other skills only point here (`If mode: competition, apply overrides from bp-competition-mode`); the override table lives **only** in this file.

Goal: ship a working, demo-able result fast **without** losing the structure that makes changes cheap (layers, ports, mappers, lint boundaries).

## Activation

`<guideline>/stack-profile.md` → `mode: competition`. If the field is absent, the mode is `standard` and this skill does not apply.

## Overrides

| Area | `standard` | `competition` |
|---|---|---|
| Lifecycle (`bp-feature-lifecycle`) | Phases 0–8 | Phases 0 (max one batch of questions, or go straight to assumptions), 2, 3, 5, 6, 7, 8. Phase 1 (spec files) skipped. Phase 4 only when logic exists |
| Tests (`bp-testing`) | Full table, test-first with red → green evidence | Required only for domain functions and mappers; test-first order and red → green evidence not required. Everything else optional |
| Ports (`bp-architecture`) | Every external call | **Still mandatory** — cheap and keeps structure |
| Mock adapter (`bp-api-integration`) | Required | Only when the backend is not available yet |
| Tier-3 skills | Active | **Not used** |
| Git (`bp-git-workflow`) | Ticket required | No ticket; format `<type>: <subject>`. Small, frequent commits |
| Questions | Stop and ask | See § Assumptions |
| Done criteria (`bp-conventions`) | Full list | `lint` + `typecheck` + `build` + domain/mapper tests. File size limits still apply |
| Security (`bp-security`) | Active | **Active — no override** |
| Lint boundaries | Enforced | **Enforced — never disable to save time** |

## Assumptions

In competition mode, when information is missing, pick the default from `stack-profile.md` or `references/<stack>.md`, record it in `ASSUMPTIONS.md` at the repo root (create it from `<guideline>/scaffold/ASSUMPTIONS.md` if missing), and continue.

One line per assumption:

```markdown
- <what was assumed> — <why (missing info / default from X)> — <how to change it (file or setting)>
```

Still **stop and ask** only for:

1. Destructive or irreversible actions (deleting data or files, force-push, rewriting history).
2. Secrets or credentials (never invent or hardcode them; ask or use env placeholders).
3. Choices that change the stack (framework, state library, UI kit, package manager).

## Priorities when time runs short

1. A working end-to-end path of the main user flow.
2. Correct layer placement (refactoring later is cheap only if this holds).
3. Loading / empty / error states on the main flow.
4. Visual polish.
5. Secondary flows.

Record anything skipped in `ASSUMPTIONS.md` under a `## Not done` heading so it is visible to reviewers/judges.

## Examples

Worked examples (fictional `topup` feature): `examples.md` in this skill folder. Read it when the rules above leave a placement or format decision unclear.
