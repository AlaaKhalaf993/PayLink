# Architecture

Names follow `docs/design/glossary.md`. Stack: Java 21, Spring Boot 4.x, PostgreSQL (see `stack-decision.md`).

## 1. Style: hexagonal modular monolith

PayLink is one deployable application split into modules. Each module is one Java package with its own tests and a small public interface. Inside a module the code is arranged as a hexagon:

- **Domain:** entities and business rules. It has no dependency on Spring, JPA, HTTP or any biller.
- **Ports:** interfaces owned by the domain. Inbound ports are the use cases a module offers. Outbound ports are what it needs from outside (database, biller, SMS provider).
- **Adapters:** implementations of ports (REST controllers, JPA repositories, ElectroCo/AquaCo/TopUpNet clients, SMS gateway).

### Why this style

| Requirement in the brief | How the architecture answers it |
|---|---|
| Adding a biller touches only Biller Integration and configuration (hard case 8, Modifiability NFR) | A biller is one new `BillerAdapter` class plus a configuration entry. `payments` only knows the `BillerAdapter` port. |
| ElectroCo v2 (new field names, amounts in piasters, signed requests) changes only the ElectroCo integration | Field mapping, unit conversion (EGP to piasters) and request signing live inside `ElectroCoAdapter`. The port keeps using `Money` in EGP. |
| Money must stay exact and traceable (FR-5, Auditability NFR) | Only the `accounts` module writes ledger entries. Other modules call it through its public interface. |
| Notifications and Reconciliation become event-driven in Phase 2 | They already react to payment events (section 4), so moving the events to a broker later does not change their logic. |

## 2. Modules

| Module (package) | Responsibility | Owns data (tables) |
|---|---|---|
| `accounts` | Accounts, verification level, block status, wallets, ledger entries, holds, top-ups | accounts, wallets, ledger_entries, holds, top_ups |
| `catalog` | Billers, services, required fields, min/max amount, active flag | billers, biller_services, biller_required_fields |
| `payments` | Payment life cycle, idempotency, orchestration, refund requests | payments, payment_status_history, idempotency_records, refund_requests |
| `billers` | One adapter per biller, timeout, retry, callbacks, settlement parsing | biller_callbacks, requery_attempts (it records what the billers answered) |
| `fees` | Fee rules, promotion, agent commission | fee_rules, promotions, commissions |
| `notifications` | SMS, email, push, retry, undelivered status | notifications |
| `fraud` (extra) | Rule chain: daily limit, velocity, blocked account, min/max amount. Limits are read from `limit_rules` | limit_rules |
| `reconciliation` (extra) | Settlement file comparison and difference report | settlement_files, settlement_records, reconciliation_reports, reconciliation_differences |
| (cross-cutting) | Audit log written for every money movement and admin action | audit_logs |

Rule: a module reads and writes only its own tables. Other modules use its public interface.

## 3. Public interfaces

Signatures are design-level; exact types are finalised in Phase 2.

```java
// accounts
interface WalletService {
    Balances balances(AccountId id);                       // computed from ledger
    TopUpId  topUp(AccountId id, FundingSource source, Money amount);  // FR-4: books a TOPUP ledger entry; TopUp.status PENDING, SUCCESS or FAILED
    List<LedgerEntry> history(AccountId id);               // FR-3: wallet history, newest first
    HoldId   placeHold(AccountId id, PaymentId p, Money total);  // available -> held
    void     captureHold(HoldId h);                        // held -> spent
    void     releaseHold(HoldId h);                        // held -> available
    void     reverseCompleted(PaymentId p, UUID requestId, AccountId actor); // refund after COMPLETED (FR-27): one REVERSAL entry per money entry of the payment
}
interface AccountService {
    AccountId register(RegistrationInfo info);                  // FR-1: new account starts BASIC
    void    upgrade(AccountId id, VerificationLevel level);     // FR-2: BASIC -> VERIFIED
    VerificationLevel levelOf(AccountId id);
    boolean isBlocked(AccountId id);
    void    block(AccountId id, Instant until, String reason);   // used by the velocity rule and by support
    void    unblock(AccountId id);                               // support (FR-21)
}

// catalog
interface BillerCatalog {
    BillerServiceInfo service(ServiceId id);               // limits, fields, active
    void manage(BillerChange change, AccountId admin);     // FR-6: add, edit, activate or deactivate a biller or service (audited)
}

// fees
interface FeeCalculator { Money feeFor(ServiceId s, Money amount, LocalDate day); }  // Strategy
interface CommissionCalculator { Money commission(Money fee, Optional<AccountId> agent); }
interface FeeRuleAdmin { void updateRule(FeeRule rule, AccountId admin); }  // FR-24: edit fee rules (audited, sets updatedBy and updatedAt)

// fraud
interface FraudChecker { FraudDecision check(FraudContext ctx); }        // Chain of Responsibility
interface LimitRuleAdmin { void updateRule(LimitRule rule, AccountId admin); }  // FR-24: edit limits (audited)

// billers  (port used by payments; adapters are internal to this module)
interface BillerAdapter {
    BillInfo      inquire(BillInquiry q);
    BillerOutcome pay(BillerPayment p);   // CONFIRMED | REFUSED | PENDING | UNKNOWN (timeout)
}
interface BillerAdapters { BillerAdapter forBiller(BillerId id); }        // Factory
interface BillerCallbackListener { void onBillerResult(Reference r, BillerOutcome o); }

// payments
interface PaymentService {
    PaymentResult pay(PayCommand cmd);     // cmd carries idempotencyKey
    Payment       find(PaymentId id);
    Payment       findByReference(String reference);   // FR-20: support search by reference, with status history
    DailyTotals   dailyTotals(LocalDate day);          // FR-28: admin daily totals
    void          refund(PaymentId id);    // only from FAILED
    void          resolveUnknown(PaymentId id, BillerOutcome outcome, AccountId actor); // support confirms an UNKNOWN payment (UC16)
    UUID          requestRefund(PaymentId id, AccountId customer, String reason);       // refund of a COMPLETED payment (FR-27, US-27.1)
    void          approveRefund(UUID requestId, AccountId actor, String reason);        // support approves: calls WalletService.reverseCompleted
    void          rejectRefund(UUID requestId, AccountId actor, String reason);         // support rejects: no money moves
}

// notifications
interface NotificationObserver { void onPaymentEvent(PaymentEvent e); } // Observer

// reconciliation
interface ReconciliationService { ReconciliationReport reconcile(BillerId b, SettlementFile f); }
```

In `class-diagram.puml`, `PaymentService`, `FeeCalculator`, `CommissionCalculator`, `FraudChecker` and `BillerAdapters` are drawn as the classes that implement these entry points. Interfaces with several implementations (`BillerAdapter`, `FeeStrategy`, `FraudRule`, `NotificationObserver`) are drawn as interfaces.

## 4. Dependency rules

An arrow means "may call".

```
            +---------+
 REST --->  |payments | ---> accounts
            +---------+ ---> catalog
              |   |    ---> fees
              |   |    ---> fraud
              |   +--------> billers
              |
              +-- publishes PaymentEvent --> notifications
                                         --> reconciliation (later)

 reconciliation ---> payments (via PaymentService), billers (settlement parsing)
 billers        ---> BillerCallbackListener (port), implemented by payments
```

- `payments` is the only orchestrator. It calls `catalog`, `fees`, `fraud`, `accounts`, `billers` in the order of the pay flow (brief 2.2).
- `accounts`, `catalog`, `fees`, `fraud`, `billers` do **not** call `payments`.
- `billers` reports callbacks and retry results through `BillerCallbackListener`. `payments` implements it, so there is no compile-time cycle.
- `notifications` never blocks a payment: it receives events and retries on its own (hard case 5).
- No module reads another module's tables.
- Only `accounts` writes ledger entries and holds.
- The block status lives in the `accounts` table. `fraud` (velocity rule) and support change it only through `AccountService.block` and `unblock`.
- The `AuditLog` is append-only (D-17, US-25.2). Two layers, the same as the note in `er.puml`: (1) in code, there is only an insert and a read operation, no update or delete method or endpoint; (2) as a deployment rule, the database user of the application has `UPDATE` and `DELETE` revoked on `audit_logs`. A hash chain is possible later, it is not part of this design.
- A `LedgerEntry` is never edited or deleted either (US-05.3). A correction is a new `REVERSAL` entry.

### 4.1 What `payments` does with each biller answer

| Biller answer (`BillerOutcome`) | Payment state | Money |
|---|---|---|
| `CONFIRMED` (ElectroCo, AquaCo, later callback or settlement file) | `COMPLETED` | `captureHold` turns the hold into a final charge (`HoldStatus.CAPTURED`), fee income is posted, commission is posted if an agent was involved |
| `REFUSED` | `FAILED`, then `REFUNDED` | `releaseHold` returns the money to the available balance |
| `PENDING` (TopUpNet replies "accepted") | stays `SENT_TO_BILLER` | Hold stays. The callback later gives `CONFIRMED` or `REFUSED`. A second callback is ignored and logged (hard case 6) |
| `UNKNOWN` (no answer within 10 s) | `UNKNOWN` | Hold stays. Re-query after 1, 5 and 15 minutes, then wait for reconciliation. Never refunded automatically. A Support Employee can also confirm the real outcome by hand with `resolveUnknown` (UC16; the later `COMPLETED` is drawn in `seq-unknown-confirmed.puml`): `CONFIRMED` gives `COMPLETED`, `REFUSED` gives `FAILED` and then the normal refund. It is written to the audit log |

### 4.2 Refund after `COMPLETED` (FR-27, decisions D-12, D-13, D-14, D-18)

`COMPLETED` is final and `REFUNDED` comes only from `FAILED`, so a refund of a completed payment is a separate process and the payment status does not change (D-12). The sequence is in `seq-refund-completed.puml`.

| Step | Module | What happens |
|---|---|---|
| 1 | `payments` | `requestRefund`: only for a `COMPLETED` payment of that customer, and only if the payment has no `APPROVED` request yet. Creates a `RefundRequest` with status `PENDING`. No money moves |
| 2a | `payments` | `rejectRefund`: only from `PENDING`. Status `REJECTED`, reason saved, `AuditLog` entry. No money moves |
| 2b | `payments` then `accounts` | `approveRefund`: only from `PENDING`. Calls `WalletService.reverseCompleted`, then sets the request to `APPROVED` and writes an `AuditLog` entry |

What `reverseCompleted` does, in one database transaction:

- It reads the `DEBIT`, `FEE_INCOME` and (if an agent was involved) `COMMISSION` entries of the payment and posts one `REVERSAL` entry for each. Nothing is edited or deleted.
- The customer gets back the whole `totalAmount`, fee included (D-13).
- The `SYSTEM` wallet pays it: it returns the fee income and pays the bill amount that the biller already kept (D-18).
- The agent commission is reversed only up to what the agent still has available. If the agent already spent part of it, PayLink absorbs the difference (D-14). The agent wallet never goes negative.
- Every `REVERSAL` has an `AuditLog` row with `refId` = the ledger entry id (NFR-05).
- Approving the same request twice is refused, because only a `PENDING` request can be approved. So the customer is never credited twice.

### 4.3 Fraud rules and the velocity rule (D-16, US-19)

The four rules are independent. The first rule that says no stops the chain and the payment becomes `REJECTED` with that reason (US-09.3, US-18).

- **Velocity rule (D-16).** It counts the payments that the account created in the last `VELOCITY_WINDOW_SECONDS` (60). If the count is already `VELOCITY_COUNT` (5), the new payment is the 6th: it is rejected and the account is blocked for `VELOCITY_BLOCK_MINUTES` (10) through `AccountService.block`. During the block, `BlockedAccountRule` rejects every payment. After the 10 minutes the account pays normally. The three numbers are `LimitRule` rows, not code.
- **Daily limit.** The sum of today's payments plus the new `totalAmount` must not exceed `DAILY_BASIC` (5,000 EGP) or `DAILY_VERIFIED` (30,000 EGP).
  - *Which payments count (assumption, team to confirm):* the payments of the same customer account created today (calendar day) that are in `FUNDS_HELD`, `SENT_TO_BILLER`, `UNKNOWN` or `COMPLETED`. `REJECTED`, `FAILED` and `REFUNDED` payments do not count, because the money did not stay spent. A `COMPLETED` payment that later gets a refund request still counts, because it stays `COMPLETED` (D-12).
- **Min and max amount (`MinMaxRule`, US-09.4, US-18.1, D-07).** The bill `amount` must be at least `minAmount` and at most `maxAmount` of its `BillerService` (5 and 10,000 EGP for the four billers). The two numbers are stored once, in `biller_services.min_amount` and `max_amount` (US-06); they are not in `limit_rules`. A payment outside the range becomes `REJECTED` with a clear reason (see `seq-pay-bill.puml`).

## 5. Pattern mapping (at least 5 are required)

| Pattern | Where | Why |
|---|---|---|
| State | `payments`: `PaymentState` | Only the 9 allowed transitions (`payment-state.puml`) |
| Strategy | `fees`: `FeeStrategy` (fixed, percentage with minimum, tiered) | New fee types without changing callers |
| Adapter + Factory | `billers`: `BillerAdapter`, `BillerAdapters` | One interface for REST, SOAP, async and CSV billers |
| Chain of Responsibility | `fraud`: `FraudRule` | Daily limit, velocity, blocked-account and min/max rules are independent |
| Observer | `notifications`: `NotificationObserver` | SMS/push failure never fails the payment |
| Command (optional) | `payments`: `RetryCommand` | Scheduled inquiry retries after 1, 5 and 15 minutes |

## 6. How the design supports the hard cases

| # | Hard case | Design answer |
|---|---|---|
| 1 | Same payment sent twice | `payments` stores `IdempotencyRecord` and returns the first result |
| 2 | ElectroCo takes 12 s | Adapter times out at 10 s and returns `UNKNOWN`. `payments` moves to `UNKNOWN` and the hold stays |
| 3 | AquaCo refuses | `FAILED`, then `accounts.releaseHold`, then `REFUNDED`, then notification with the reason |
| 4 | Two payments, funds for one | `accounts.placeHold` runs in one database transaction with a row lock on the wallet |
| 5 | SMS provider down | Notification is an event handled outside the payment transaction, retried 3 times |
| 6 | TopUpNet callback twice | `onBillerResult` ignores a result for a payment already `COMPLETED` |
| 7 | Settlement file lists a `FAILED` payment | `reconciliation` flags it in the report, no automatic money movement |
| 8 | New biller | One new adapter and one configuration entry |

## 7. Package layout (proposal for the app of Alaa Khalaf AbdElmotamed)

```
src/main/java/com/paylink/
  accounts/        domain/  port/  adapter/
  catalog/         ...
  payments/        ...
  billers/         electroco/  aquaco/  topupnet/  netco/
  fees/            ...
  notifications/   ...
  fraud/           ...
  reconciliation/  ...
  shared/          Money, ids, audit
src/main/resources/db/migration/   (Flyway)
```

## 8. Phase 2 evolution (for the ADRs)

Notifications and Reconciliation move from in-process events to a broker (RabbitMQ in docker compose). The `PaymentEvent` contract stays the same, only the transport changes. Architecture decision records will compare: modular monolith vs services, in-process events vs broker, and shared database vs one schema per module.

## 9. Decisions and open questions

Checked against the Phase 1 requirements pack (FR-1 to FR-28, NFR-01 to NFR-06, D-01 to D-18) on 2026-11-25.

### Settled by the requirements pack

| Question | Answer | Source |
|---|---|---|
| Is the fee deducted together with the amount? | Yes. The held amount is `totalAmount` (amount + fee) | US-09.1 |
| Is the daily limit counted on the amount only? | No. It is counted on the **total amount** | US-09.5 |
| When an agent pays, whose wallet is charged? | The customer's wallet. The payment stores the agent id and the agent earns the commission | D-10 (marked "to be confirmed") |
| Same idempotency key, different data? | Rejected as a conflict. `IdempotencyRecord` therefore stores `requestHash` | US-10.3 |
| How long is an idempotency key remembered? | 24 hours per customer | D-09 (marked "needs team approval") |
| Who confirms an `UNKNOWN` payment? | Automatic rules first (re-query after 1, 5 and 15 minutes, callback, settlement file). A Support Employee can also confirm it by hand: `PaymentService.resolveUnknown` | D-06, D-11, use case UC16 |
| Where are limits and fee values stored? | `fee_rules` and `limit_rules` tables, edited by an `AdminUser` through the API | D-15 (assumption), US-24 |
| Where does PayLink keep its fee income? | In one internal `SYSTEM` wallet that has no account (`WalletType.SYSTEM`) | D-18 |
| Where is the block status of an account? | In the `accounts` table (`blocked`, `blockedUntil`, `blockReason`) | ER diagram |
| What is the package of the Biller Integration module? | `billers` | NFR-04 |
| What are the table names of the ledger and the audit log? | `ledger_entries` and `audit_logs` | NFR-05, glossary |
| How is a `COMPLETED` payment refunded? | `RefundRequest` plus reversing ledger entries. The payment status does not change | D-12, US-27, section 4.2 |
| Is the fee returned in that refund? Is the commission reversed? | Yes and yes. PayLink absorbs what an agent already spent | D-13, D-14 (both "team to confirm") |
| Who pays that refund? | The `SYSTEM` wallet | D-18 ("team to confirm") |
| Does the 5th or the 6th payment in 60 seconds get blocked? | The 6th. The account is blocked for 10 minutes | D-16, section 4.3 |
| Can the `AuditLog` be changed or deleted? | No, it is append-only: enforced in code, plus `UPDATE`/`DELETE` revoked in the database as a deployment rule | D-17 (assumption), US-25.2, section 4, ER note |
| Which values has `top_ups.status`? | `PENDING`, `SUCCESS`, `FAILED` (`TopUpStatus`) | US-04.3, ER |
| Where are the min and max payment amounts stored? | Only in `biller_services.min_amount` / `max_amount`; `MinMaxRule` reads them | US-06, US-09.4, D-07 |
| Which payments count in the daily limit? | `FUNDS_HELD`, `SENT_TO_BILLER`, `UNKNOWN`, `COMPLETED` of the same day; not `REJECTED`, `FAILED`, `REFUNDED` | assumption, section 4.3 |
| Which ledger fields? | `type` (column `entry_type`), `availableDelta` and `heldDelta` (columns `available_delta`, `held_delta`). Balance = sum of the deltas | ER diagram, US-05.2, NFR-01 |
| Enum values that differ between the first class diagram and the ER? | The class diagram follows the ER: `LedgerEntryType.TOPUP` (and `CREDIT`, never written by a flow), `HoldStatus.CAPTURED` with `captureHold`, `DifferenceKind` = `SETTLED_BUT_FAILED`, `UNKNOWN_NOT_IN_FILE`, `AMOUNT_MISMATCH` | glossary section 6 |
| Does a `Biller` have a category? | Yes: `BillerCategory` (ELECTRICITY, WATER, MOBILE, INTERNET), so the catalog can list billers by category | US-07, US-06.1, ER `billers.category` |
| What does an `IdempotencyRecord` store? | `requestHash` and `storedResponse` (the response of the first request, which holds the payment id) | US-10, ER `idempotency_records` |

### Still open (needs a team decision)

- **Can the `SYSTEM` wallet go below zero?** A refund of a completed payment is paid from it (D-18), and NFR-01 says no negative balances. Not decided. Until then `reverseCompleted` is described as posting the entries, and the check on the `SYSTEM` balance is not specified.
- **One refund per payment.** The design refuses a new `RefundRequest` when the payment already has an `APPROVED` one. This is my assumption, it is not in US-27.
- **Does a rejected payment count in the velocity rule?** Assumed: yes, every payment that the account creates counts.
- Is agent commission 0 when the promotion makes the fee 0? (assumed: yes)
- What if TopUpNet answers "accepted" and never calls back? (assumed: after a configured wait the payment is treated like a timeout, becomes `UNKNOWN`, and is re-queried and reconciled)

## 10. Related diagrams (by Mariam Mohamed Ali)

- ER diagram and the three use-case diagrams are in this folder. They must use the names in `glossary.md`. The differences that are still open are listed at the end of the glossary and in section 9.
- Use cases and design: UC16 (support follows up an `UNKNOWN` payment) is `PaymentService.resolveUnknown`; UC15 (approve or reject a refund request) is `PaymentService.approveRefund` and `rejectRefund` on the entity `RefundRequest`; UC20 to UC22 are the three `FraudRule` classes.

## 11. Non-functional requirements and the design

| NFR | How the design supports it | Where |
|---|---|---|
| NFR-01 Correctness | Money is `Money` (BigDecimal), never float. Balances are computed from the ledger. `placeHold` runs in one transaction with a row lock. `IdempotencyRecord` stops a double charge | glossary section 5, section 6 (hard cases 1 and 4) |
| NFR-02 Performance | Notifications and reconciliation run outside the payment transaction. Only `payments` orchestrates, so there is one short path per payment. A bounded thread pool per biller adapter (see below) keeps slow billers from using all threads | sections 1, 4, 11 |
| NFR-03 Resilience | One `BillerAdapter` per biller with its own 10 s timeout and its own thread pool (bulkhead). A failing biller gives `UNKNOWN` for its own payments only | sections 4.1, 6, 11 |
| NFR-04 Modifiability | A new biller is one new class in `billers` plus configuration. `payments` knows only the `BillerAdapter` port | sections 1, 5, 6 (hard case 8) |
| NFR-05 Auditability | Every `LedgerEntry` and every admin action has an `AuditLog` row with `refId`. The `AuditLog` is append-only. Tables: `ledger_entries` and `audit_logs` | sections 2, 4, 4.2 |
| NFR-06 Testability | Domain code has no Spring, JPA or HTTP dependency, so it is tested with plain unit tests | section 1 |

**NFR-02 and NFR-03: slow billers must not block the others.** The target is 100 payments per second (NFR-02), and a biller call can last up to 10 seconds (the timeout). If every call used one shared pool, a slow biller could hold up to 1,000 waiting calls and starve the other billers. So each `BillerAdapter` runs its calls in its own bounded thread pool and queue (a bulkhead). When the pool of one biller is full, new payments for that biller wait or are rejected with a clear "try again later" message, while the other three billers keep working. The only serialized point in a payment is the row lock on the wallet in `placeHold` (hard case 4); it is held for the hold only and never during the biller call. The pool sizes are configuration values, to be chosen with a load test in Phase 2.
