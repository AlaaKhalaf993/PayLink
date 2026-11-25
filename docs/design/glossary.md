# PayLink Glossary

Single source of truth for names. Requirements, diagrams, ER and code must all use these names exactly.

## 1. Modules (packages)

| Module | Java package | Responsibility |
|---|---|---|
| Accounts and Wallet | `accounts` | Customers, agents, verification levels, wallets, ledger entries, holds, top-ups, refunds |
| Biller Catalog | `catalog` | Billers, services, required input fields, min/max amounts, active/disabled |
| Payment Processing | `payments` | Payment life cycle, idempotency, orchestration of the payment steps |
| Biller Integration | `billers` | One adapter per biller; timeout, retry and callback handling |
| Fees and Commissions | `fees` | Fee calculation (fixed, percentage, tiered, promotion) and agent commission |
| Notifications | `notifications` | SMS, email, push with retry and templates |
| Fraud Checks (extra) | `fraud` | Daily limit, velocity and blocked-account rules as a chain |
| Reconciliation (extra) | `reconciliation` | Settlement file comparison and difference report |
| Shared (cross-cutting) | `shared` | `Money`, ids and the `AuditLog` |

## 2. Core entities

| Name | Meaning |
|---|---|
| `Account` | A user of the system. Has a `role`, a `verificationLevel` and a block status (`blocked`, `blockedUntil`, `blockReason`) |
| `Customer` | Account with role CUSTOMER |
| `Agent` | Account with role AGENT (shop with a PayLink terminal) |
| `AdminUser` / `SupportUser` | Staff roles |
| `VerificationLevel` | `BASIC` or `VERIFIED` |
| `Wallet` | Money container in EGP with a `walletType`. Customers and agents have one wallet each. PayLink has one internal `SYSTEM` wallet with no account; it receives the fee income and pays refunds (D-18) |
| `LedgerEntry` | One money movement. Balances are computed from entries, never overwritten. An entry is never edited or deleted; a correction is a new `REVERSAL` entry |
| `Hold` | Money reserved for a payment that is not yet confirmed |
| `Payment` | One bill payment, always in exactly one state |
| `PaymentStatusHistory` | Record of every state transition of a payment |
| `Biller` | A company that issues bills (ElectroCo, AquaCo, TopUpNet, NetCo) |
| `BillerService` | A service of a biller (electricity, water, mobile top-up, internet) |
| `FeeRule` | Configurable fee definition (FIXED, PERCENTAGE, TIERED) for one biller |
| `Promotion` | Zero-fee period for one biller, with start and end dates |
| `Commission` | Agent commission, 40% of the service fee |
| `Notification` | An SMS, email or push message with delivery status |
| `IdempotencyRecord` | Stored result for an idempotency key |
| `SettlementRecord` | One line from a biller settlement file |
| `ReconciliationReport` | Differences between our payments and the settlement file |
| `AuditLog` | Who, when, why for every money movement and admin action. It is append-only: no update and no delete (D-17) |
| `Money` | Value object: `BigDecimal` amount plus currency (EGP) |
| `Balances` | Value object returned by `WalletService.balances`: `available` and `held` |
| `FraudDecision` | Result of the fraud chain: `allowed` and a `reason` |
| `PaymentEvent` | Event published when a payment changes state; consumed by `notifications` |
| `ReconciliationDifference` | One difference line inside a `ReconciliationReport` |
| `TopUp` | One wallet top-up from a funding source (simulated card or agent cash). Books a `LedgerEntry` |
| `RefundRequest` | A customer's request to get a `COMPLETED` payment back (FR-27). Support approves or rejects it (`PaymentService.approveRefund` / `rejectRefund`). The payment stays `COMPLETED`; on approval `WalletService.reverseCompleted` returns the whole `totalAmount`, fee included, through reversing ledger entries, paid from the `SYSTEM` wallet (D-12, D-13, D-18) |
| `LimitRule` | One configurable limit or velocity number (for example `DAILY_BASIC`). Admins can change it without redeploying (FR-24) |
| `BillerCallback` | A callback received from a biller (TopUpNet). Duplicates are stored with `duplicate = true` and ignored |
| `RequeryAttempt` | One re-query of an `UNKNOWN` payment (after 1, 5 and 15 minutes) and its result |
| `SettlementFile` | One daily settlement file of a biller. It contains `SettlementRecord` lines and has one `ReconciliationReport` |

## 3. Important fields

| Field | Notes |
|---|---|
| `availableBalance` | Computed from the ledger |
| `heldBalance` | Computed from the ledger (active holds) |
| `idempotencyKey` | Unique per client request |
| `reference` | Unique payment reference sent to the biller |
| `amount` | Bill amount, `Money` (BigDecimal, EGP) |
| `feeAmount` | Service fee charged by PayLink |
| `totalAmount` | `amount + feeAmount` |
| `status` | One of the 8 payment states |
| `availableDelta` / `heldDelta` | Signed effect of one `LedgerEntry` on the available and held balance. `availableBalance` = sum of `availableDelta`, `heldBalance` = sum of `heldDelta` |
| `billerAccountRef` | The customer's number at the biller (meter number, phone number) |
| `failureReason` | Why a payment became `FAILED` or `REJECTED`; shown to the customer in the notification |
| `requiredFields` | Input fields a `BillerService` needs from the customer (for example meter number, phone number) |
| `attempts` | How many times a `Notification` was sent (retried up to 3 times, then `UNDELIVERED`) |
| `newStatus` | The payment state carried by a `PaymentEvent` |
| `occurredAt` | When the state change of a `PaymentEvent` happened |
| `storedResponse` | The response returned for the first request with an `idempotencyKey` (stored in `IdempotencyRecord`; it contains the payment id). A repeated request gets this response back (US-10.2) |
| `billerReference` | The payment reference as it appears in a biller's settlement file (`SettlementRecord`) |
| `fileDate` | The settlement date of a `SettlementFile` |
| `billerStatus` | The status the biller gives to one `SettlementRecord` line |
| `blockedUntil` / `blockReason` | Until when and why an `Account` is blocked (velocity rule or support). `blocked` says if it is blocked now |
| `category` | The kind of a `Biller`, a `BillerCategory`: ELECTRICITY, WATER, MOBILE or INTERNET. The catalog lists billers by category (US-07) |
| `walletType` | `CUSTOMER`, `AGENT` or `SYSTEM` |
| `fundingSource` | Where a `TopUp` comes from: `SIMULATED_CARD` or `AGENT_CASH` |
| `tierThreshold`, `feeBelowThreshold`, `feeAboveThreshold` | Fields of a TIERED `FeeRule`: fee up to the threshold and above it (internet: 2 EGP up to 200 EGP, 4 EGP above) |
| `ruleName` / `value` | A `LimitRule` is one named number. `value` is a `BigDecimal` because some rules are money (5,000 EGP) and some are counts or seconds |
| `VELOCITY_COUNT` / `VELOCITY_WINDOW_SECONDS` / `VELOCITY_BLOCK_MINUTES` | Velocity rule (D-16): 5, 60 and 10. When the account already created `VELOCITY_COUNT` payments in the last `VELOCITY_WINDOW_SECONDS`, the next (6th) payment is rejected and the account is blocked for `VELOCITY_BLOCK_MINUTES` |
| `attemptNo` | Number of a `RequeryAttempt` (1 to 3) |
| `duplicate` | `true` when a `BillerCallback` repeats one that was already handled |
| `decisionReason` / `decidedAt` | Support's reason and time when a `RefundRequest` is approved or rejected |
| `updatedBy` / `updatedAt` | Who changed a `FeeRule` or `LimitRule` and when (audit of admin edits) |
| `runDate` | The day a `ReconciliationReport` was produced |
| `requestHash` | Fingerprint of the request data stored with an `idempotencyKey`. Same key and same hash returns the first result; same key and a different hash is rejected as a conflict (US-10) |
| `refId` | The id of the ledger entry or admin target that an `AuditLog` row is about. Lets a test check that every money movement has an audit record (NFR-5) |

## 4. Payment states (exactly 8)

`INITIATED`, `FUNDS_HELD`, `SENT_TO_BILLER`, `COMPLETED`, `FAILED`, `UNKNOWN`, `REJECTED`, `REFUNDED`

Final states: `COMPLETED`, `REJECTED`, `REFUNDED`.

### Allowed transitions (brief, section 2.5: nothing else is allowed)

| # | From | Event | To |
|---|---|---|---|
| 1 | INITIATED | Fraud checks pass and wallet has enough funds | FUNDS_HELD |
| 2 | INITIATED | Fraud check fails or funds insufficient | REJECTED |
| 3 | FUNDS_HELD | Request sent to biller | SENT_TO_BILLER |
| 4 | SENT_TO_BILLER | Biller confirms | COMPLETED |
| 5 | SENT_TO_BILLER | Biller refuses | FAILED |
| 6 | SENT_TO_BILLER | No answer within timeout | UNKNOWN |
| 7 | UNKNOWN | Biller later confirms (callback, retry, settlement file) | COMPLETED |
| 8 | UNKNOWN | Biller later refuses, or reconciliation finds no payment | FAILED |
| 9 | FAILED | Hold released to the wallet | REFUNDED |

`UNKNOWN` is never refunded automatically.

## 5. Naming conventions

- Money: always `Money` / `BigDecimal`, never `float` or `double`. In the database use `NUMERIC(19,2)`.
- Currency: EGP only.
- Class names in PascalCase singular (`LedgerEntry`); table names in snake_case plural (`ledger_entries`).
- Code to database mapping: an attribute in camelCase is a column in snake_case (`feeAmount` is `fee_amount`). A boolean `active` or `blocked` is the column `is_active` or `is_blocked`. A foreign key `xId` is the column `x_id`.
- Enum values are UPPER_SNAKE_CASE in code and in the database (for example `SUCCESS`, not `success`). One exception: `biller_callbacks.callback_status` is stored in lowercase (`success`, `failed`) in the ER; the code keeps `CallbackStatus.SUCCESS` and `FAILED`, and a JPA `AttributeConverter` maps the two.
- Design pattern interfaces: `FeeStrategy`, `PaymentState`, `BillerAdapter`, `FraudRule`, `NotificationObserver`, `RetryCommand`.
- Business-rule numbers (limits, fees, timeouts) live in configuration, not in code.
- Diagram tool: PlantUML. Source `.puml` files live in `docs/design/`; images are only for the PDF report.

## 6. Enumerations

| Enum | Values |
|---|---|
| `Role` | CUSTOMER, AGENT, ADMIN, SUPPORT |
| `WalletType` | CUSTOMER, AGENT, SYSTEM (the `SYSTEM` wallet has no account and holds the PayLink fee income) |
| `VerificationLevel` | BASIC, VERIFIED |
| `LedgerEntryType` | TOPUP, CREDIT, HOLD, HOLD_RELEASE, DEBIT, FEE_INCOME, COMMISSION, REVERSAL (undoes an earlier money entry; used by the refund of a `COMPLETED` payment). `CREDIT` is in the ER check list but no flow writes it: money that comes in is `TOPUP`, `HOLD_RELEASE`, `COMMISSION` or `REVERSAL` |
| `HoldStatus` | ACTIVE, CAPTURED, RELEASED (`CAPTURED` = the hold became the final charge; `BillerOutcome.CONFIRMED` is a different thing, the biller's answer) |
| `BillerCategory` | ELECTRICITY, WATER, MOBILE, INTERNET |
| `FeeType` | FIXED, PERCENTAGE, TIERED |
| `BillerProtocol` | REST_JSON, SOAP_XML, REST_ASYNC, CSV_FILE |
| `BillerOutcome` | CONFIRMED, REFUSED, PENDING, UNKNOWN |
| `NotificationChannel` | SMS, EMAIL, PUSH |
| `NotificationStatus` | PENDING, SENT, UNDELIVERED |
| `FundingSource` | SIMULATED_CARD, AGENT_CASH |
| `TopUpStatus` | PENDING, SUCCESS, FAILED (FAILED covers a failed top-up, US-04.3) |
| `RefundRequestStatus` | PENDING, APPROVED, REJECTED |
| `CallbackStatus` | SUCCESS, FAILED |
| `LimitRuleName` | DAILY_BASIC, DAILY_VERIFIED, VELOCITY_COUNT, VELOCITY_WINDOW_SECONDS, VELOCITY_BLOCK_MINUTES |
| `DifferenceKind` | SETTLED_BUT_FAILED (the file lists a payment that PayLink has as `FAILED`, hard case 7), UNKNOWN_NOT_IN_FILE (a payment that is `UNKNOWN` in PayLink is not in the file, US-26.3), AMOUNT_MISMATCH |

Not covered in Phase 1 (no user story asks for them): a `COMPLETED` payment that is missing from the file, and a line in the file for a payment PayLink does not know. If a later story needs them, add new `DifferenceKind` values and update the ER check list.

### Known differences between the class diagram and the ER diagram

The class diagram follows the ER diagram wherever the ER was clearer. Everything else follows the mapping rule in section 5.

**Closed (the class diagram now uses the ER name or value):**

| Was in the class diagram | Now (same as the ER) |
|---|---|
| `LedgerEntry.availableDelta` / `heldDelta` | The ER now has `available_delta` and `held_delta`, so both agree |
| `LedgerEntryType.TOP_UP` | `TOPUP`; `CREDIT` added (kept for the ER check list, no flow writes it) |
| `HoldStatus.CONFIRMED` and `confirmHold` | `CAPTURED` and `captureHold` (also removes the clash with `BillerOutcome.CONFIRMED`) |
| `DifferenceKind` (4 values) | `SETTLED_BUT_FAILED`, `UNKNOWN_NOT_IN_FILE`, `AMOUNT_MISMATCH` |
| `IdempotencyRecord.resultPaymentId` | `storedResponse` (column `stored_response`) |
| `Payment.failureReason`, `Payment.billerAccountRef` | The ER has `failure_reason` and `biller_account_ref` |
| `Biller` without a category | `Biller.category : BillerCategory` (column `billers.category`) |

**Accepted (same meaning, different name):**

| Class diagram | ER diagram | Status |
|---|---|---|
| `Notification.attempts` | `notifications.retry_count` | Same meaning, accept |
| `PaymentStatusHistory.reason` | `payment_status_history.note` | Same meaning, accept |
| `ReconciliationReport.runDate` | `reconciliation_reports.report_date` | Same meaning, accept |
| `ReconciliationDifference.kind` | `reconciliation_differences.difference_type` | Same meaning, accept |
| `LedgerEntry.type` | `ledger_entries.entry_type` | Same meaning, accept |
| `FeeRule.type` and `FeeRule.minimumFee` | `fee_rules.fee_type` and `min_fee` | Same meaning, accept |
| `AuditLog.actor` | `audit_logs.actor_account_id` | Same meaning, accept (an `Account` id) |
| `AuditLog.refId` | `audit_logs.ledger_entry_id` and `entity_id` | Same meaning, accept (NFR-05 counts `ledger_entries` and `audit_logs`) |
| `BillerService.requiredFields` | table `biller_required_fields` (`field_name`, `is_required`) | Same meaning, the ER stores the list as rows |
| `CallbackStatus` `SUCCESS` / `FAILED` | `callback_status` `success` / `failed` | Stored in lowercase, mapped by a converter (section 5) |

Also closed in this version:

| Was | Now |
|---|---|
| `TopUp.status : String`, `top_ups.status` without values | `TopUpStatus` = `PENDING`, `SUCCESS`, `FAILED` in the class diagram and the ER |
| `MIN_PAYMENT` and `MAX_PAYMENT` in `limit_rules` and in `LimitRuleName`, and also `biller_services.min_amount` / `max_amount` (two places for the same numbers) | One source: `biller_services.min_amount` and `max_amount` (`BillerService.minAmount` / `maxAmount`, US-06). `limit_rules` and `LimitRuleName` keep only the daily and velocity numbers. A `MinMaxRule` in the `FraudRule` chain reads them |
| `Account` without `fullName`, `mobileNumber`, `email`, `passwordHash` | Added (ER columns `full_name`, `mobile_number`, `email`, `password_hash`, US-01) |
| `Wallet`, `Hold`, `Promotion`, `Commission` without some ER columns | Added `Wallet.currency`, `Hold.createdAt` / `releasedAt`, `Promotion.feeOverride`, `Commission.createdAt`, `Account.createdAt` |

**Accepted (the ER has it as a link, the class diagram as a relationship):** `Hold.paymentId`, `Commission.paymentId`, `Commission.agentAccountId` and `ledgerEntryId` are drawn as associations in the class diagram, not as attributes.

**Not added (the ER has no such column):** `Hold.currency`. `Hold.amount` is a `Money`, which carries the currency. If the team wants a `currency` column on `holds`, the ER owner adds it first.

**Still open:**

| Class diagram | ER diagram | Status |
|---|---|---|
| (none left in the class/ER naming) | | The remaining open points are decisions for the team: see `architecture.md` section 9 |
