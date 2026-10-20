# bp-conventions — Examples

## 1. Naming

| Item | ❌ | ✅ |
|---|---|---|
| File | `TopupStore.ts`, `topup_store.ts` | `topup-store.ts` (per profile) |
| Port | `ITopupRepo`, `TopupService` | `TopupRepository` |
| Adapter | `TopupRepositoryImpl` | `HttpTopupRepository`, `MockTopupRepository` |
| Mapper | `mapData()`, `convert()` | `toTopupInvoice()`, `toCreateTopupPayload()` |
| Boolean | `flag`, `status2` | `isActive`, `hasPendingInvoice` |
| Abbreviation | `amt`, `bnk`, `inv` | `amount`, `bank`, `invoice` |

## 2. File too large

`topup-page` component reaches 420 lines (limit 250):

- Extract `AmountForm` (organism) and `BankPicker` (organism) into feature `presentation/`.
- Move "amount must be multiple of 1,000" from the component into `domain/topup.ts`.
- Page keeps only composition + store wiring.

## 3. Done report (good)

```markdown
Verification (commands from stack-profile.md)
- lint: `pnpm lint` → 0 errors, 0 warnings
- typecheck: `pnpm typecheck` → ok
- test: `pnpm test src/features/topup` → 14 passed
- build: `pnpm build` → ok (route added)

TDD evidence (test-first targets)
- validateAmount — red: "expected null to equal { type: 'validation', … }" → green: 4 passed
- toBank — red: "expected false to be true" (lowercase flag) → green: 6 passed
- createTopup — red: "expected create not to be called" → green: 3 passed
- TopupStore.submit — red: "expected 'Choose a bank', received undefined" → green: 4 passed

Route reachability
- /topup visible after login as role "customer"; permission `topup:create` added; menu entry added.
```

## 4. Done report (not acceptable)

```markdown
Done! Everything should work now.
```

Problems: no commands, no results, "should" instead of evidence, route reachability not checked.
