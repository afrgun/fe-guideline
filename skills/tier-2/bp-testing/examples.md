# bp-testing — Examples

Examples use the common `describe/it/expect` test API. Runner and file location come from the stack profile.

## 1. Spec file

```markdown
# 02 — validateAmount
Subject: domain/topup.ts → validateAmount(amount)
Fakes: none (pure function)

## When the amount is below the minimum
- Given MIN_TOPUP = 10,000
- When validateAmount(9,999)
- Then it returns a validation error "Minimum top up is 10,000"

## When the amount is not a multiple of 1,000
- When validateAmount(10,500)
- Then it returns a validation error "Amount must be a multiple of 1,000"

## When the amount is valid
- When validateAmount(50,000)
- Then it returns null
```

## 2. Domain test (required, test-first)

```ts
describe('selectableBanks', () => {
  const bank = (code: string, isActive: boolean, isOtherBank = false) => ({ code, name: code, isActive, isOtherBank });

  it('keeps only active banks', () => {
    expect(selectableBanks([bank('A', true), bank('B', false)]).map((b) => b.code)).toEqual(['A']);
  });

  it('puts the other-bank option last', () => {
    expect(selectableBanks([bank('OTH', true, true), bank('A', true)]).map((b) => b.code)).toEqual(['A', 'OTH']);
  });
});
```

## 3. Mapper edge cases (required)

| Case | Input | Expected |
|---|---|---|
| Full payload | all fields | every domain field mapped |
| Missing optional | `bank_name: null` | `name` falls back to `code` |
| Missing required | no `bank_code` | adapter returns `err`, not a half-built bank |
| Unknown enum | `active: "maybe"` | `isActive: false` (documented default) |
| Bad date (invoice) | `created_at: "not-a-date"` | adapter returns `err` |

## 4. Use case with fake ports

```ts
it('rejects a bank that is no longer active', async () => {
  const banks = { getBanks: async () => ok([{ code: 'A', name: 'A', isActive: false, isOtherBank: false }]) };
  const topups = { create: async () => { throw new Error('must not be called'); } };

  const result = await createTopup({ banks, topups })({ bankCode: 'A', amount: 50_000 });

  expect(result).toEqual({ ok: false, error: expect.objectContaining({ type: 'validation' }) });
});
```

## 5. What not to do

| ❌ | Why | ✅ |
|---|---|---|
| `expect(repo.getBanks).toHaveBeenCalledTimes(1)` as the only assertion | Tests wiring, not behavior | Assert the returned `Result` / rendered state |
| Mocking `selectableBanks` inside the store test | Mocks the unit's own logic | Use real domain functions; fake only ports |
| Snapshot of the whole bank list page | Brittle, hides intent | Assert visible bank names and the empty/error state |
| Shared store/query client across tests | Leaks state | Fresh instance per test |

## 6. Red → green, step by step (validateAmount)

| Step | Action | Output to keep as evidence |
|---|---|---|
| 1 | Write the test only; `topup.ts` exports `validateAmount = (_amount: number) => null` (signature stub) | — |
| 2 | Run the single test file | **red**: `expected null to equal { type: 'validation', message: 'Minimum top up is 10,000' }` |
| 3 | Implement the minimum rule | — |
| 4 | Run again | **green**: `4 passed` |
| 5 | Refactor (extract `MIN_TOPUP`), run again | still green |

Wrong "red": `Cannot find module '../topup'` or a syntax error. That proves nothing about the behavior — fix the setup and run again until the failure is the missing behavior.

Reported in the summary as:

```markdown
TDD evidence
- validateAmount — red: "expected null to equal { type: 'validation', … }" → green: 4 passed
```
