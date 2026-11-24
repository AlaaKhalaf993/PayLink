# PayLink \- Requirements Document

**Delivery:** First delivery  
**Role:** Requirements & Product Lead  
**Scope:** FR-1 to FR-12 (Wallet, Catalog, Basic Payment)

## 1\. Introduction

PayLink is a software project that aims to develop a digital wallet and bill payment backend system for customers in Egypt. The project focuses on providing an organized platform for managing customer accounts, wallets, billers, and payment transactions.

This document defines the requirements and scope of the PayLink system. It identifies the main stakeholders, describes the functional requirements through user stories and acceptance criteria, and specifies the priorities and important decisions that will guide the development process.

## 2\. Problem Summary

Currently, customers need a convenient and reliable way to pay different utility bills, such as electricity, water, mobile, and internet bills, without visiting payment offices. Managing these payments through different providers can make the process inconvenient and time-consuming.

PayLink addresses this problem by providing a centralized platform where customers can use a digital wallet to pay their bills. However, handling financial transactions introduces important challenges, such as preventing duplicate payments, maintaining accurate wallet balances, managing failed transactions, and ensuring secure communication with different biller companies.

Therefore, the system must provide accurate payment processing, reliable wallet management, and proper handling of transaction failures to ensure a safe and efficient payment experience.

##   

## 3\. Stakeholders

| Stakeholder | Role |
| :---- | :---- |
| Customer | Pays bills using the wallet |
| Agent | Pays on behalf of customers |
| Admin | Manages billers and services |
| Support Employee | Handles payment issues |
| Biller | Receives payments |
| PayLink Management | Monitors business operations |

## 4\. Functional Requirements

### 4.1 Traceability

| User Story | Requirement | Topic | Priority |
| :---- | :---- | :---- | :---- |
| US-01 | FR-1 | Customer Registration | Must |
| US-02 | FR-2 | Upgrade Account Verification | Should |
| US-03 | FR-3 | Wallet Balance & History | Must |
| US-04 | FR-4 | Wallet Top-up | Must |
| US-05 | FR-5 | Ledger Entries | Must |
| US-06 | FR-6 | Manage Billers and Services | Must |
| US-07 | FR-7 | View Billers & Required Fields | Must |
| US-08 | FR-8 | Bill Inquiry | Must |
| US-09 | FR-9 | Pay a Bill | Must |
| US-10 | FR-10 | Prevent Duplicate Payments | Must |
| US-11 | FR-11 | Payment Life Cycle | Must |
| US-12 | FR-12 | Hold & Release Funds | Must |

### 4.2 User Stories, Acceptance Criteria and Priority

#### US-01 (FR-1): Customer Registration

**User Story:** As a new customer, I want to register with my name, mobile number, email and password, so that I can have a PayLink account and wallet.

**Acceptance Criteria:**

1. **Given** a new customer enters valid and unique registration details, **When** the customer submits the registration form, **Then** an account is created with the Basic verification level and a wallet with zero balance is created.  
2. **Given** the mobile number or email is already registered, **When** the customer submits the registration form, **Then** the registration is rejected with a message that the identifier is already in use.  
3. **Given** a required field is missing or invalid, **When** the customer submits the registration form, **Then** the registration is rejected and the invalid fields are indicated.

**Priority:** Must

#### US-02 (FR-2): Upgrade Account Verification

**User Story:** As a customer, I want to upgrade my account from Basic to Verified, so that I can benefit from higher daily payment limits.

**Acceptance Criteria:**

1. **Given** a customer has a Basic account, **When** the customer completes the required verification process, **Then** the account is upgraded to Verified.  
2. **Given** a customer has a Verified account, **When** the customer makes a payment, **Then** the daily payment limit is 30,000 EGP.  
3. **Given** a customer has a Basic account, **When** the customer makes payments, **Then** the daily payment limit remains 5,000 EGP.

**Priority:** Should

#### US-03 (FR-3): Wallet Balance & History

**User Story:** As a customer, I want to see my wallet balance and transaction history, so that I always know how much money I have and where it went.

**Acceptance Criteria:**

1. **Given** a registered customer is logged in, **When** the customer opens the wallet, **Then** both the current available balance and the held balance are displayed.  
2. **Given** the customer has previous transactions, **When** the customer opens the history, **Then** transactions are listed newest first with type, amount, date and status.  
3. **Given** a customer requests wallet data, **When** the request is processed, **Then** only the customer's own wallet and transactions are returned.

**Priority:** Must

#### US-04 (FR-4): Wallet Top-up

**User Story:** As a customer, I want to add money to my wallet, so that I can use it to pay bills.

**Acceptance Criteria:**

1. **Given** a customer selects a supported funding source (simulated card or agent cash) and enters a positive amount within the allowed limit, **When** the top-up is confirmed, **Then** the wallet balance increases by that amount and the top-up appears in the history.  
2. **Given** the amount is zero, negative or above the allowed limit, or the funding source is not supported, **When** the customer submits the top-up, **Then** the top-up is rejected and the balance is unchanged.  
3. **Given** the top-up fails at the funding source, **When** the failure is reported, **Then** the balance is unchanged and the customer is notified.

**Priority:** Must

#### US-05 (FR-5): Ledger Entries

**User Story:** As PayLink Management, I want every balance change recorded as a ledger entry, so that all wallet balances can be audited and trusted.

**Acceptance Criteria:**

1. **Given** any credit, debit, hold or release happens on a wallet, **When** the operation is saved, **Then** a ledger entry (id, wallet, direction, amount, reference, timestamp) is saved in the same transaction as the balance change.  
2. **Given** a wallet has ledger entries, **When** the balance is checked, **Then** the balance equals the sum of the wallet's ledger entries.  
3. **Given** a ledger entry already exists, **When** someone tries to edit or delete it, **Then** the action is rejected; corrections are made only by adding a new reversing entry.

**Priority:** Must

#### US-06 (FR-6): Manage Billers and Services

**User Story:** As an Admin, I want to add, edit, and disable billers and their services, so that customers can access only supported services.

**Acceptance Criteria:**

1. **Given** an Admin enters valid biller or service details (name, category, required fields), **When** the Admin saves the biller, **Then** the biller or service is created and appears in the catalog.  
2. **Given** a biller or service is active, **When** the Admin disables it, **Then** it is hidden from customers and payments already in progress are not affected.  
3. **Given** an Admin edits the configuration of an existing biller or service (such as updating required fields or the service name), **When** the Admin saves the changes, **Then** the updated details are immediately reflected in the catalog for subsequent inquiries and payments.  
4. **Given** a user who is not an Admin, **When** tries to add, edit or disable a biller or service, **Then** the action is denied.

**Priority:** Must

#### US-07 (FR-7): View Billers & Required Fields

**User Story:** As a customer, I want to browse billers and see which details I must enter, so that I can prepare the right information for my bill.

**Acceptance Criteria:**

1. **Given** a customer opens the catalog, **When** the catalog loads, **Then** active billers are shown grouped by category (electricity, water, mobile, internet).  
2. **Given** a customer selects a biller, **When** the biller details open, **Then** the required input fields for that biller are displayed (for example meter number or phone number).  
3. **Given** a biller is inactive, **When** a customer opens the catalog, **Then** that biller is not shown.

**Priority:** Must

#### US-08 (FR-8): Bill Inquiry

**User Story:** As a customer, I want to look up my bill before paying, so that I can see the exact amount due.

**Acceptance Criteria:**

1. **Given** the customer enters valid required fields for a biller, **When** the customer requests an inquiry, **Then** the system displays the bill amount, applicable service fee, and total amount before payment.  
2. **Given** the entered account or number does not exist at the biller, **When** the customer requests an inquiry, **Then** a not-found message is shown and payment cannot be started.  
3. **Given** the biller does not respond within 10 seconds, **When** the customer requests an inquiry, **Then** the inquiry fails with a try-again message and no money is taken.

**Priority:** Must

#### US-09 (FR-9): Pay a Bill

**User Story:** As a customer, I want to pay an inquired bill from my wallet, so that I do not have to pay in cash or visit the biller.

**Acceptance Criteria:**

1. **Given** the customer has an inquired bill and sufficient available balance, **When** the customer confirms the payment, **Then** the system moves the total amount from the available balance to the held balance (FundsHeld) and sends the payment request to the biller (SentToBiller).  
2. **Given** the payment is SentToBiller and the biller confirms success, **When** the confirmation is received, **Then** the payment becomes Completed, the held amount becomes the final debit, PayLink credits its fee income, any applicable agent commission is recorded, and a confirmation with a reference number is shown.  
3. **Given** the available balance is less than the total amount, or the fraud check fails, **When** the customer confirms the payment, **Then** the payment becomes Rejected and the balance is unchanged.  
4. **Given** the payment amount is below 5 EGP or above 10,000 EGP, **When** the customer confirms the payment, **Then** the payment becomes Rejected and the balance is unchanged.  
5. **Given** the total amount would exceed the customer's daily limit (5,000 EGP for Basic, 30,000 EGP for Verified), **When** the customer confirms the payment, **Then** the payment becomes Rejected and the balance is unchanged.  
6. **Given** the payment is SentToBiller and the biller rejects it, **When** the rejection is received, **Then** the payment becomes Failed, the held funds are released and the payment becomes Refunded.

**Priority:** Must

#### US-10 (FR-10): Prevent Duplicate Payments

**User Story:** As a customer, I want to be charged only once even if I tap Pay twice or the app retries, so that I never lose money to a duplicate payment.

**Acceptance Criteria:**

1. **Given** a payment request has an idempotency key that has not been used before, **When** the request is received, **Then** the payment is processed once.  
2. **Given** a request arrives with an idempotency key that was already used with the same data, **When** the request is received, **Then** the original result is returned and no second deduction is made.  
3. **Given** a request arrives with an already used key but different data, **When** the request is received, **Then** the request is rejected as a conflict.

**Priority:** Must

#### US-11 (FR-11): Payment Life Cycle

**User Story:** As a Support Employee, I want every payment to follow a defined set of statuses, so that I can tell exactly where a payment is and handle issues correctly.

**Acceptance Criteria:**

1. **Given** a payment request is accepted, **When** the payment is created, **Then** its status is Initiated.  
2. **Given** a payment is Initiated, the fraud check passes and the available balance is sufficient, **When** the funds are reserved, **Then** the status becomes FundsHeld.  
3. **Given** a payment is Initiated and the fraud check fails or the balance is insufficient, **When** the checks are completed, **Then** the status becomes Rejected.  
4. **Given** a payment is FundsHeld, **When** the request is sent to the biller, **Then** the status becomes SentToBiller.  
5. **Given** a payment is SentToBiller, **When** the biller confirms success, **Then** the status becomes Completed.  
6. **Given** a payment is SentToBiller, **When** the biller rejects the payment, **Then** the status becomes Failed.  
7. **Given** a payment is SentToBiller, **When** the biller does not respond within 10 seconds, **Then** the status becomes Unknown and it is sent for support follow-up.  
8. **Given** a payment is Failed, **When** the held funds are released, **Then** the status becomes Refunded.  
9. **Given** a payment is Unknown, **When** the real outcome is confirmed, **Then** the status becomes Completed or Failed accordingly.  
10. **Given** a payment is Completed, Rejected or Refunded, **When** a change to another status is attempted, **Then** the change is rejected because these are final states.

**Priority:** Must

#### US-12 (FR-12): Hold & Release Funds

**User Story:** As a customer, I want the payment amount to be held while the payment is processed, so that I cannot spend it twice and I get it back if the payment fails.

**Acceptance Criteria:**

1. **Given** a payment is Initiated and the checks pass, **When** the payment starts, **Then** the total amount moves from the available balance to the held balance and the status becomes FundsHeld.  
2. **Given** a payment is SentToBiller and the biller confirms success, **When** the payment is Completed, **Then** the held amount becomes the final debit.  
3. **Given** a payment is Failed, **When** the failure is confirmed, **Then** the held funds are released to the available balance and the payment transitions to Refunded.  
4. **Given** a payment is Unknown, **When** its outcome is not yet confirmed, **Then** the funds stay held until the outcome is confirmed.  
5. **Given** a payment is Rejected, **When** the payment ends, **Then** no funds are held and the available balance is unchanged.

**Priority:** Must

### 4.3 Non-Functional Requirements

Each NFR has an ID, a numeric target, a measurement method, the tool or place where the number is produced, and the brief rule or hard case it comes from. In Phase 1 the targets and methods are fixed and the test plan is written. The measured numbers are reported in Phase 2 by the CI pipeline.

|  ID   |   Quality   | Target (numeric) | How we measure | Evidence / where | Priority |
|-------|-------------|------------------|----------------|------------------|----------|
| NFR-1 | Correctness | **0** double charges and **0** negative balances across the duplicate-request and concurrency tests. Money uses exact decimal types, never float. | Hard case 1 (same payment sent twice with the same idempotency key): 1 payment row, 1 debit ledger entry. Hard case 4 (2 parallel payments, balance covers 1): exactly 1 `Completed`/`FundsHeld`, 1 `Rejected`. Run the concurrency test with 50 parallel requests per wallet, repeated 100 times. After each run assert `SUM(ledger) == wallet balance` and `balance >= 0`. Static check: a lint rule that fails the build if `float` or `double` is used for money. | Test report in CI (JUnit/Surefire XML), assertion counts in the PDF report | Must |
| NFR-2 | Performance | **100 payments/second sustained for 60 seconds** against the mock billers, with **p95 latency <= 500 ms** and **error rate < 1%** (excluding payments rejected on purpose by business rules). Mock ElectroCo answers in under 1 s as the brief says. | Load test with k6: constant arrival rate of 100 req/s for 60 s on the `pay a bill` endpoint, using the ElectroCo mock. Read throughput, p95 and error rate from the k6 summary. Run on `docker compose up` locally. | k6 summary output saved as `/docs/perf/k6-summary.txt` | Must |
| NFR-3 |  Resilience | When one biller is down, payments to the **other billers keep succeeding at >= 99%**, and the failing biller's payments become `Unknown` after the **10 s timeout** (money stays held), then are re-queried at **1, 5 and 15 minutes** (3 retries). | Integration test: mock AquaCo set to fail/hang (and ElectroCo with a 12 s delay, hard case 2). During the outage send 100 payments to ElectroCo and TopUpNet and assert >= 99 succeed. Assert the state is `Unknown`, the hold is not released, and 3 re-queries are scheduled at 1/5/15 min (use a fake clock). Notification down (hard case 5): payment is still `Completed`, SMS retried 3 times, then marked `undelivered`. | Integration test report in CI | Must |
| NFR-4 | Modifiability | Adding a new biller changes **only the Biller Integration module + configuration**: **0 files** changed in Wallet, Payment Processing, Fees, Notifications, Fraud. Business-rule values (limits, fees, timeouts, retries, commission %) live in configuration: **0 hard-coded** occurrences. | Hard case 8: add a dummy biller in a branch and run `git diff --stat main`; a CI script fails if any path outside `integration/` and `config/` changed. Grep check for the rule values (5000, 30000, 2.50, 0.4, ...) in source outside config/tests. Phase 2: same check for ElectroCo API v2. | CI script output + diff stat in the report | Must |
| NFR-5 | Auditability | **100%** of money movements and admin actions have an audit record with **who, when, why**. | Test: after each scenario (pay, fail + refund, top-up, commission, admin edits biller/fee/limit, block/unblock), compare the count of ledger entries + admin actions with the count of audit rows; they must be equal, and every row has non-null `actor`, `timestamp`, `reason`. SQL check: `SELECT count(*) FROM ledger_entry l LEFT JOIN audit_log a ON a.ref_id = l.id WHERE a.id IS NULL` must return **0**. | SQL check run in integration tests, result in the report | Must |
| NFR-6 | Testability  | **>= 70% line coverage on domain logic**, reported by CI on every push. Also **100%** of the 8 hard cases (section 2.7 of the brief) have at least 1 automated test. | JaCoCo. Coverage gate in GitHub Actions: the build fails below 70% on the domain package. Hard-case traceability table: case # -> test name, checked in review. | Coverage report artifact in the GitHub Actions run | Must |

#### NFR notes and decisions

- The brief lists 6 expected qualities (Correctness, Performance, Resilience, Modifiability, Auditability, Testability). We use exactly these 6 so every NFR matches the brief's table; we only made each one more precise.
- The brief says "100 payments per second sustained for 1 minute". Decision: we add p95 <= 500 ms and error rate < 1% so the test can pass or fail. These two numbers are our own decision, not from the brief (to be confirmed by the team).
- The 99% in NFR-3 is our own decision, not from the brief. The 10 s timeout and the 1/5/15 min retries come from the business rules.
- Performance is measured against mock billers on a laptop with docker compose, so the number depends on the machine. We write the CPU/RAM next to the result.
- Phase 1 has no running payment system yet, so NFR numbers are measured in Phase 2. In Phase 1 we deliver the target, the method and the test plan.


## 5\. Questions and Ambiguities

| ID | Question | Related Stories |
| :---- | :---- | :---- |
| Q-01 | What happens if the customer has insufficient balance? | US-09 |
| Q-02 | What happens if the biller does not respond? | US-08, US-11 |
| Q-03 | Can the same payment request be sent twice? | US-10 |
| Q-04 | When should held funds be released? | US-12 |
| Q-05 | What is the difference between a Basic and a Verified account? | US-01, US-02 |
| Q-06 | What happens to an Unknown payment afterwards? | US-11, US-12 |
| Q-07 | What are the limits on a single payment and on the daily total? | US-02, US-09 |
| Q-08 | Which currency does the wallet use? | US-03, US-04 |
| Q-09 | How long is an idempotency key remembered? | US-10 |
| Q-10 | Can an Agent pay on behalf of a customer, and from which wallet? | US-09 |

## 6\. Decisions Made

| ID | Question | Decision |
| :---- | :---- | :---- |
| D-01 | Q-01 | Reject the payment (status Rejected). The balance is not changed and nothing is sent to the biller. |
| D-02 | Q-02 | Mark the payment as Unknown after 10 seconds without a biller response. Its funds stay held. |
| D-03 | Q-03 | Use an idempotency key. A repeated key with the same data returns the original result. |
| D-04 | Q-04 | Release held funds only when payment failure is confirmed. The payment then moves from Failed to Refunded. |
| D-05 | Q-05 | A new account is Basic and can use the wallet with a daily limit of 5,000 EGP. Upgrading to Verified raises the daily limit to 30,000 EGP. |
| D-06 | Q-06 | Funds stay held. A Support Employee (or a later check with the biller) confirms the real outcome, and the payment then becomes Completed, or Failed and then Refunded. |
| D-07 | Q-07 | As specified in the assignment: minimum single payment is 5 EGP, maximum single payment is 10,000 EGP, Basic daily limit is 5,000 EGP and Verified daily limit is 30,000 EGP. |
| D-08 | Q-08 | A single currency (EGP) is used (assumption). |
| D-09 | Q-09 | Assumption, needs team approval: a key is remembered for 24 hours per customer. This is not an assignment requirement. |
| D-10 | Q-10 | An Agent pays from the customer's wallet, identified by the customer's mobile number, and the payment is recorded with the Agent's id (assumption, to be confirmed). |

> 