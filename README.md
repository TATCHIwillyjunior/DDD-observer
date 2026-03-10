# Strong Typing in TypeScript for robust domain-driven design

## PREVIOUS KNOWLEDGE

1. Running code in TypeScript with Node.js
2. Basic TypeScript types (number, string, boolean, etc.)
3. Functions and type annotations
4. Type safety and compile-time checks
5. Factory functions
6. Smart constructors
7. Value objects
8. Entities
9. Observer Pattern

## Exercise Domain-Driven Design & Strong Typing in TypeScript

## A Practical Bank Domain with Smart Constructors, Branded Types & Observers
This project is a hands‑on exploration of Domain‑Driven Design (DDD) and strong typing in TypeScript.
It demonstrates how to build a safe, expressive, and bug‑resistant domain using:

* Branded types
* Smart constructors
* Value objects
* Entities
* Pure domain logic
* Observer pattern for side effects

The domain used here is a Bank Account system — simple enough to understand, but rich enough to show real DDD value.


## What You Will Learn

| ---- Concept --------------     | -------- What it Matters ---------------------------------                                             |
| ------------------------------- | ----------------------------------------------------------------------------------------------------- |
| -- **Branded Types** --         | Prevent mixing values that share the same primitive type (e.g., AccountId vs Balance).                    |
| -- **Smart Constructors** --    | Validate business rules at creation time so invalid values never exist                                |
| -- **Value Objects** --         | Represent domain concepts (Balance, DailyLimit, PositiveAmount) as immutable, validated types.        |
| -- **Entities** --              | Model things with identity and lifecycle (BankAccount).             |
| -- **Pure Domain Logic** --     | Keep business rules deterministic and side‑effect‑free. |
| -- **Observer Pattern** --      | Trigger side effects (SMS alerts, audit logs) without polluting domain logic |
| -- **Parse, Don't Validate** -- | Convert raw input into safe domain types at the boundary. |

## Project Structure


```
src/
  ├── domain/
  │   ├── bank/
  │   │   ├── types.ts        # AccountId, Balance, DailyLimit, PositiveAmount, BankAccount
  │   │   ├── bank.ts         # withdraw() domain logic (pure function)
  │   │   └── factories.ts    # createAccountId, createBalance, createDailyLimit,
  │   │                       # createPositiveAmount, createBankAccount
  │   └── events/
  │       └── events.ts       # all domain event types + DomainEvent union
  ├── infrastructure/
  │   └── observers/
  │       ├── observer.ts     # Observer type + observers array + emit() + registerObserver()
  │       ├── sms.ts          # sendSmsMock (Fraud Detection Observer)
  │       └── audit.ts        # saveToAuditLogMock (Audit Log Observer)
  └── index.ts                # wiring + test runs only

```

## 🧠 Core Domain Concepts

### 1. Branded Types
Make illegal states unrepresentable.

```ts
type Balance = number & { readonly __brand: unique symbol }
```

A raw number cannot be used as a Balance unless it comes from a smart constructor.

### 2. Smart Constructors
Validate once → trust everywhere.

```ts
function createBalance(amount: number): Balance {
  if (amount < 0) throw new Error("Balance cannot be negative")
  return amount as Balance
}
```

### 3. Value Objects
Immutable, validated domain concepts:

* Balance
* DailyLimit
* PositiveAmount

### 4. Entities
A BankAccount has identity and enforces its own invariants.

### 5. Pure Domain Logic
withdraw() is a pure function:

* No side effects
* No I/O
* No logs
* Just domain rules

### 6. Observer Pattern
Side effects live outside the domain:

- SMS alerts
- Audit logs
- Fraud detection
- Domain emits events → observers react.

### Getting Started

```bash
# Install dependencies
npm install

# Run the exercises
npm run dev
```


## Key Takeaways

- **Make illegal states unrepresentable.** If a value should never be negative (Balance, DailyLimit, PositiveAmount), enforce this at the type level. Branded types ensure invalid primitives can’t even enter the domain.
- **Push validation to the boundary.** Raw input (API, CLI, database, user forms) should be converted into safe domain types using smart constructors. Inside the domain, you trust the types completely.
- **Use Value Objects to model meaning, not primitives.** A Balance is not just a number. A DailyLimit is not just a number. Value Objects give clarity, safety, and domain expressiveness.
- **Entities enforce their own invariants.** A BankAccount knows how to maintain its own rules (e.g., cannot withdraw more than balance, cannot exceed daily limit). This keeps business logic consistent and centralized.
- **Domain logic must be pure.** Functions like withdraw() should contain no side effects, no logs, no I/O. They should only compute the next valid state of the domain.
- **Side effects belong in the infrastructure layer.** Observers (SMS alerts, audit logs) react to domain events without polluting the domain logic. This keeps the domain clean and testable.
- **Events make the system expressive and extensible.** When something meaningful happens (withdrawal, limit exceeded, suspicious activity), the domain emits an event. Observers can be added or removed without changing domain code.
- **Types are living documentation.** When a function takes Balance instead of number, its intent is immediately clear. Strong typing reduces ambiguity and improves maintainability.
- **Strong typing + DDD = fewer bugs, clearer intent, safer systems.**  The combination of branded types, smart constructors, and pure domain logic creates a system where many bugs become impossible by design.
