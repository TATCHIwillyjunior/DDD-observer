# Code Review — DDD-Observer (Bank Domain)

**Repo:** `TATCHIwillyjunior/DDD-observer`  
**Reviewer:** Makuochukwu (Evelynval)  
**Date:** March 10, 2026  
**Branch:** `main`

---

## General Feedback

Great job on the domain implementation, Tatchi! The banking domain is a solid choice, and you've clearly followed the DDD folder structure from the course guide. The separation between `domain/`, `infrastructure/`, and `events/` is clean, the Observer Pattern is properly wired, and the project compiles and runs without errors. I tested it with `npm run dev` and both observers fired correctly — the audit log recorded the withdrawal and the SMS mock triggered for the large amount.

Below are some specific checks and suggestions based on the DDD criteria.

---

## Strengths

- **Clear project structure:** You followed the complex DDD folder layout (`domain/bank/`, `domain/events/`, `infrastructure/observers/`) exactly as described in the course spec. Easy to navigate.
- **Observer Pattern works correctly:** `registerObserver()` and `emit()` are cleanly implemented. Both `smsObserver` and `auditObserver` share the same `(event: DomainEvent) => void` signature — no signature mismatch.
- **`readonly` on event properties:** All fields in `WithdrawalMadeEvent` and `LargeWithdrawalEvent` are marked `readonly`, which prevents observers from accidentally mutating event data. Good practice.
- **Domain events as discriminated union:** `DomainEvent = WithdrawalMadeEvent | LargeWithdrawalEvent` enables type-safe `event.type` checks inside observers. Clean.
- **Try-catch in `index.ts`:** The test code wraps the entire flow in `try-catch`, so impossible data (e.g., negative balance) won't crash the app.
- **`withdraw()` is a pure function** returning `{ account, events }` instead of mutating state. This is great functional DDD style.

---

## Issues & Suggestions

### 1. Branded Types: Missing `unique symbol` (Important)

**File:** `src/domain/bank/types.ts`

Currently, the branded types use string literal brands:

```ts
type AccountId = string & { readonly __brand: "AccountId" };
type Balance = number & { readonly __brand: "Balance" };
```

The course material recommends using `unique symbol` for the brand, which provides stronger type isolation at compile time. With string literals, two branded types could theoretically be assigned to each other if they share the same brand string. With `unique symbol`, this is impossible:

```ts
// Recommended:
type Balance = number & { readonly __brand: unique symbol };
```

This is a key DDD criterion — it fully eliminates primitive obsession at the type level.

### 2. `createAccountId` Has No Validation (Important)

**File:** `src/domain/bank/factories.ts`, line 14-16

```ts
export function createAccountId(id: string): AccountId {
    return id as AccountId;
}
```

This factory function performs **no validation at all** — it accepts any string, including empty strings. A smart constructor should reject impossible data. At minimum, add a check for empty/whitespace-only values:

```ts
export function createAccountId(id: string): AccountId {
    if (!id || id.trim() === "") {
        throw new Error("Account ID cannot be empty");
    }
    return id as AccountId;
}
```

### 3. `withdrawToday` Is Not Branded (Minor)

**File:** `src/domain/bank/bank.ts`, line 16

```ts
export type BankAccount = {
    // ...
    withdrawToday: number;  // raw number, not branded
};
```

Since the whole point of DDD is to eliminate primitive obsession, `withdrawToday` should ideally be a branded type (or at least use `PositiveAmount` / a dedicated `WithdrawnToday` type) to stay consistent with the rest of the entity.

### 4. Redundant `as` Casts Inside `createBankAccount` (Minor)

**File:** `src/domain/bank/factories.ts`, lines 64-65

```ts
return {
    id: id as AccountId,    // already AccountId
    name: name as AccountName,  // already AccountName
    ...
};
```

The parameters `id` and `name` are already typed as `AccountId` and `AccountName` respectively, so the `as` casts are redundant. They don't cause bugs, but removing them keeps the code cleaner.

### 5. No Negative-Amount Test Cases in `index.ts` (Suggestion)

The `index.ts` file only tests the "happy path" (valid account, valid withdrawal). To fully demonstrate that the smart constructors work, consider adding explicit test cases with impossible data, for example:

```ts
// Test: negative balance
try {
    const bad = createBalance(-500);
} catch (e) {
    if (e instanceof Error) console.error("Caught:", e.message);
}

// Test: empty account name
try {
    const bad = createAccountName("");
} catch (e) {
    if (e instanceof Error) console.error("Caught:", e.message);
}
```

This would fully demonstrate the "impossible data" handling the assignment asks for.

### 6. `domain.md` Could Be More Detailed (Minor)

**File:** `docs/domain.md`

The domain description is brief. It would be stronger if it explicitly listed the branded types, the business invariants (e.g., "balance can never go negative", "daily limit caps total withdrawals per day"), and the events in human language — so a non-technical reader could understand the rules.

---

## Observer Pattern — Specific Checks

| Check | Status | Notes |
|---|---|---|
| `Observer` type accepts `DomainEvent` |  Pass | `type Observer = (event: DomainEvent) => void` |
| Observers share the same contract |  Pass | Both `smsObserver` and `auditObserver` match the signature |
| Events use `readonly` fields |  Pass | All event properties are `readonly` |
| Discriminated union with `.type` |  Pass | Observers use `event.type` checks correctly |
| No raw strings/multiple args to observers |  Pass | Single `DomainEvent` object passed |

---

## Final Verdict

**Recommendation: Request Changes (minor)**

The implementation is solid and demonstrates a good understanding of DDD patterns, smart constructors, and the Observer Pattern. The two main items to address before merging are:

1. Switch branded types from string literal brands to `unique symbol` for stronger compile-time safety.
2. Add validation to `createAccountId` — a factory function that accepts anything defeats the purpose of a smart constructor.

Once those are updated, this is ready to merge. Nice work overall, Tatchi!😎🤜