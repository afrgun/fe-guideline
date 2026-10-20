---
name: bp-testing
description: Use when adding or changing behavior that needs tests, writing behavior specs, fixing failing tests, or deciding which tests are required for a change and where they go.
---

# Testing Strategy

## Stack context (read first)

Read `<guideline>/stack-profile.md` → **Testing** (runner, component testing, e2e, location) and `<guideline>/references/<stack>.md` § 12.

If `mode: competition` in the profile, apply overrides from `bp-competition-mode`.

## What must be tested

| Target | Required? | Style |
|---|---|---|
| Domain functions | **Yes — test-first** | Pure unit tests, no mocks |
| Mappers | **Yes — test-first** | Full payload, missing fields, unknown enums, bad dates |
| Use cases | **Yes — test-first** | Fake ports (plain objects) |
| Adapters | Yes | Fake HTTP / HTTP testing utilities → assert `Result` ok/err |
| Store / state hooks | **Yes — test-first for branching logic** (status transitions, validation before calling ports, success/error paths) | Fake ports, assert state transitions |
| Components | When they hold interaction logic | Testing Library, user-visible behavior |
| Full flows | Critical paths only | e2e |

Purely mechanical changes (renames, moves) need no new tests — say so explicitly.

## Red → green → refactor (for test-first targets)

Test-first targets: domain functions, mappers, use cases, and store branching logic. Adapters and components must be tested but may be written after the code.

1. **No production code before a failing test.** For a test-first target, write the test (or a stub that only exports the signature) first. Do not write the implementation until that test has been run and failed.
2. Run it and see it **fail for the right reason** — an assertion about the missing behavior, not a typo, import error, or syntax error.
3. Implement the minimum to pass; run again until green.
4. Refactor with tests green.

**Evidence is mandatory.** The summary of the change lists, for each test-first target:

```markdown
TDD evidence
- validateAmount — red: "expected null to equal { type: 'validation', … }" → green: 4 passed
- toBank (lowercase flag) — red: "expected false to be true" → green: 6 passed
```

A test-first target without red → green evidence counts as **not done** (`bp-conventions` done criteria). If code already existed before the test (legacy, generated code), say so explicitly instead of claiming red → green.

## Specs (used by `bp-feature-lifecycle` phase 1)

```markdown
# NN — <subject>
Subject: <function/store/adapter under test>
Fakes: <ports or http to fake>

## When <condition>
- Given <state>
- When <action>
- Then <observable outcome>
```

## Rules

1. Assert observable behavior, not implementation details or call counts unless the call *is* the behavior.
2. Fake at the boundary (ports, HTTP); never mock the unit under test.
3. No snapshot tests for logic; snapshots only for stable rendered markup, sparingly.
4. Each test is independent: fresh store/query client/DI container per test.
5. Test file location and naming follow the profile.
6. A bug fix comes with a regression test that fails before the fix.

## Done criteria

- Required targets above have tests.
- Test-first targets have red → green evidence in the summary.
- Test command from the profile passes for the touched area.

## Examples

Worked examples (fictional `topup` feature): `examples.md` in this skill folder. Read it when the rules above leave a placement or format decision unclear.
