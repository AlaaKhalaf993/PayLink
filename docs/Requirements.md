# PayLink - Requirements Document

**Phase 1 | Requirements Pack | FR-1 to FR-28, NFR-01 to NFR-06**

Team members, their roles and what each wrote in this document.

| Person | Name | ID | Role | Responsibility in this document and Phase 1 |
|---|---|---|---|---|
| 1 | Shaza Mosad Abdelrahman | 20240274 | Requirements & Product Lead | Problem summary, stakeholders, user stories US-01 to US-12 (FR-1 to FR-12: Wallet, Catalog, Basic Payment), questions and decisions Q-01 to Q-10 and D-01 to D-10 |
| 2 | Abdelrahman Ahmed Ashry | 20240290 | Architecture Lead | Stack decision, glossary, architecture style, state diagram, sequence diagrams (pay a bill, refund), class diagram |
| 3 | Alaa Khalaf AbdElmotamed | 20242049 | Quality & DevOps Lead | Repository and branch protection, Dockerfile and docker compose, GitHub Actions (build, lint, test), the 6 non-functional requirements with numbers |
| 4 | Mariam Mohamed Ali | 20240581 | Data & Integration Lead | User stories US-13 to US-28 (FR-13 to FR-28: callbacks, fraud, notifications, reconciliation, admin), questions and decisions Q-11 to Q-18 and D-11 to D-18, use-case diagram, ER diagram, final PDF assembly |

## 1. Introduction

PayLink is a software project that aims to develop a digital wallet and bill payment backend system for customers in Egypt. The project focuses on providing an organized platform for managing customer accounts, wallets, billers, and payment transactions.

This document defines the requirements and scope of the PayLink system. It identifies the main stakeholders, describes the functional requirements through user stories and acceptance criteria, defines measurable non-functional requirements, and specifies the priorities and important decisions that will guide the development process. Names of states, entities and fields follow the project glossary exactly.

## 2. Problem Summary

Currently, customers need a convenient and reliable way to pay different utility bills, such as electricity, water, mobile, and internet bills, without visiting payment offices. Managing these payments through different providers can make the process inconvenient and time-consuming.

PayLink addresses this problem by providing a centralized platform where customers can use a digital wallet to pay their bills. However, handling financial transactions introduces important challenges, such as preventing duplicate payments, maintaining accurate wallet balances, managing failed transactions, and ensuring secure communication with different biller companies.

Therefore, the system must provide accurate payment processing, reliable wallet management, and proper handling of transaction failures to ensure a safe and efficient payment experience.

## 3. Stakeholders

| Stakeholder | Role |
|---|---|
| Customer | Pays bills using the wallet |
| Agent | Pays on behalf of customers |
| AdminUser (Admin) | Manages billers and services |
| SupportUser (Support Employee) | Handles payment issues |
| Biller | Receives payments |
| PayLink Management | Monitors business operations |

## 4. Functional Requirements

### 4.1 Traceability

| User Story | Requirement | Topic | Priority |
|---|---|---|---|
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
| US-13 | FR-13 | Unknown Payment & Re-query | Must |
| US-14 | FR-14 | Asynchronous Callbacks | Must |
| US-15 | FR-15 | Multiple Billers | Must |
| US-16 | FR-16 | Fee Calculation | Must |
| US-17 | FR-17 | Agent Payments & Commission | Should |
| US-18 | FR-18 | Payment & Daily Limits | Must |
| US-19 | FR-19 | Velocity Rule | Should |
| US-20 | FR-20 | Payment Search for Support | Must |
| US-21 | FR-21 | Block & Unblock Account | Should |
| US-22 | FR-22 | Completion & Failure Notifications | Must |
| US-23 | FR-23 | Notification Retry | Should |
| US-24 | FR-24 | Editable Fees & Limits | Should |
| US-25 | FR-25 | Audit Log | Must |
| US-26 | FR-26 | Reconciliation Report | Should |
| US-27 | FR-27 | Refund of a Completed Payment | Could |
| US-28 | FR-28 | Admin Daily Totals | Could |

### 4.2 User Stories, Acceptance Criteria and Priority

#### US-01 (FR-1): Customer Registration

**User Story:** As a new customer, I want to register with my name, mobile number, email and password, so that I can have a PayLink account and wallet.

**Acceptance Criteria:**

1. Given a new customer enters valid and unique registration details, When the customer submits the registration form, Then an Account with role CUSTOMER and verificationLevel BASIC is created, and a Wallet in EGP with zero balance is created for it.
2. Given the mobile number or email is already registered, When the customer submits the registration form, Then the registration is rejected with a message that the identifier is already in use.
3. Given a required field is missing or invalid, When the customer submits the registration form, Then the registration is rejected and the invalid fields are indicated.

**Priority:** Must

#### US-02 (FR-2): Upgrade Account Verification

**User Story:** As a customer, I want to upgrade my account from BASIC to VERIFIED, so that I can benefit from higher daily payment limits.

**Acceptance Criteria:**

1. Given a customer has an Account with verificationLevel BASIC, When the customer completes the required verification process, Then the verificationLevel becomes VERIFIED.
2. Given a customer has a VERIFIED account, When the customer makes a payment, Then the daily payment limit is 30,000 EGP.
3. Given a customer has a BASIC account, When the customer makes payments, Then the daily payment limit remains 5,000 EGP.

**Priority:** Should

#### US-03 (FR-3): Wallet Balance & History

**User Story:** As a customer, I want to see my wallet balance and transaction history, so that I always know how much money I have, how much is held, and where it went.

**Acceptance Criteria:**

1. Given a registered customer is logged in, When the customer opens the wallet, Then both the current availableBalance and the heldBalance are displayed.
2. Given the customer has previous transactions, When the customer opens the history, Then transactions are listed newest first with type, amount, date and status.
3. Given a customer requests wallet data, When the request is processed, Then only the customer's own Wallet and transactions are returned.

**Priority:** Must

#### US-04 (FR-4): Wallet Top-up

**User Story:** As a customer, I want to add money to my wallet, so that I can use it to pay bills.

**Acceptance Criteria:**

1. Given a customer selects a supported funding source (simulated card or agent cash) and enters a positive amount, When the top-up is confirmed, Then a LedgerEntry is recorded, the availableBalance increases by that amount and the top-up appears in the history.
2. Given the amount is zero or negative, or the funding source is not supported, When the customer submits the top-up, Then the top-up is rejected and the balance is unchanged.
3. Given the top-up fails at the funding source, When the failure is reported, Then the balance is unchanged and the customer is notified.

**Priority:** Must

#### US-05 (FR-5): Ledger Entries

**User Story:** As PayLink Management, I want every money movement recorded as a LedgerEntry, so that balances are computed from entries and can be audited and trusted.

**Acceptance Criteria:**

1. Given any credit, debit, hold or release happens on a wallet, When the operation is saved, Then a LedgerEntry is saved in the same transaction as the balance change.
2. Given a wallet has ledger entries, When the balance is checked, Then availableBalance and heldBalance are computed from the LedgerEntry records (active Hold entries for the held part) and are never overwritten.
3. Given a LedgerEntry already exists, When someone tries to edit or delete it, Then the action is rejected; corrections are made only by adding a new reversing entry.
4. Given any money movement or admin action happens, When it is completed, Then an AuditLog record shows who did it, when and why.

**Priority:** Must

#### US-06 (FR-6): Manage Billers and Services

**User Story:** As an AdminUser, I want to add, edit, and disable billers and their services, so that customers can access only supported services.

**Acceptance Criteria:**

1. Given an AdminUser enters valid Biller or BillerService details (name, category, required fields, min/max amounts), When the AdminUser saves it, Then the biller or service is created and appears in the catalog.
2. Given a biller or service is active, When the AdminUser disables it, Then it is hidden from customers and payments already in progress are not affected.
3. Given an AdminUser edits the configuration of an existing biller or service (such as updating required fields or the service name), When the AdminUser saves the changes, Then the updated details are immediately reflected in the catalog for subsequent inquiries and payments.
4. Given a user who is not an AdminUser, When tries to add, edit or disable a biller or service, Then the action is denied.

**Priority:** Must

#### US-07 (FR-7): View Billers & Required Fields

**User Story:** As a customer, I want to browse billers and see which details I must enter, so that I can prepare the right information for my bill.

**Acceptance Criteria:**

1. Given a customer opens the catalog, When the catalog loads, Then active billers are shown with their services (electricity, water, mobile top-up, internet).
2. Given a customer selects a BillerService, When the service details open, Then the required input fields for that service are displayed (for example meter number or phone number).
3. Given a biller or service is disabled, When a customer opens the catalog, Then it is not shown.

**Priority:** Must

#### US-08 (FR-8): Bill Inquiry

**User Story:** As a customer, I want to look up my bill before paying, so that I can see the exact amount due.

**Acceptance Criteria:**

1. Given the customer enters valid required fields for a BillerService, When the customer requests an inquiry, Then the system displays the bill amount, the applicable feeAmount, and the totalAmount before payment.
2. Given the entered account or number does not exist at the biller, When the customer requests an inquiry, Then a not-found message is shown and payment cannot be started.
3. Given the biller does not respond within 10 seconds, When the customer requests an inquiry, Then the inquiry fails with a try-again message and no money is taken.

**Priority:** Must

#### US-09 (FR-9): Pay a Bill

**User Story:** As a customer, I want to pay an inquired bill from my wallet, so that I do not have to pay in cash or visit the biller.

**Acceptance Criteria:**

1. Given the customer has an inquired bill and the availableBalance covers the totalAmount, When the customer confirms the payment, Then a Payment is created as INITIATED, a Hold moves the totalAmount from the availableBalance to the heldBalance (FUNDS_HELD) and the payment request is sent to the biller (SENT_TO_BILLER).
2. Given the payment is SENT_TO_BILLER and the biller confirms success, When the confirmation is received, Then the payment becomes COMPLETED, the Hold becomes the final debit, PayLink credits its fee income, any applicable Commission (40% of the feeAmount) is recorded for the agent, and a confirmation with the payment reference is shown.
3. Given the availableBalance is less than the totalAmount, or a fraud check fails (daily limit, velocity or blocked account), When the customer confirms the payment, Then the payment becomes REJECTED and the balance is unchanged.
4. Given the payment amount is below 5 EGP or above 10,000 EGP, When the customer confirms the payment, Then the payment becomes REJECTED and the balance is unchanged.
5. Given the totalAmount would exceed the customer's daily limit (5,000 EGP for BASIC, 30,000 EGP for VERIFIED), When the customer confirms the payment, Then the payment becomes REJECTED and the balance is unchanged.
6. Given the payment is SENT_TO_BILLER and the biller refuses it, When the refusal is received, Then the payment becomes FAILED, the Hold is released and the payment becomes REFUNDED.

**Priority:** Must

#### US-10 (FR-10): Prevent Duplicate Payments

**User Story:** As a customer, I want to be charged only once even if I tap Pay twice or the app retries, so that I never lose money to a duplicate payment.

**Acceptance Criteria:**

1. Given a payment request has an idempotencyKey that has not been used before, When the request is received, Then the payment is processed once and an IdempotencyRecord stores the result.
2. Given a request arrives with an idempotencyKey that was already used with the same data, When the request is received, Then the stored result is returned and no second deduction is made.
3. Given a request arrives with an already used idempotencyKey but different data, When the request is received, Then the request is rejected as a conflict.

**Priority:** Must

#### US-11 (FR-11): Payment Life Cycle

**User Story:** As a SupportUser, I want every payment to follow a defined set of states, so that I can tell exactly where a payment is and handle issues correctly.

**Acceptance Criteria:**

1. Given a payment request is accepted, When the Payment is created, Then its status is INITIATED.
2. Given a payment is INITIATED, the fraud checks pass and the wallet has enough funds, When the funds are reserved, Then the status becomes FUNDS_HELD.
3. Given a payment is INITIATED and a fraud check fails or the funds are insufficient, When the checks are completed, Then the status becomes REJECTED.
4. Given a payment is FUNDS_HELD, When the request is sent to the biller, Then the status becomes SENT_TO_BILLER.
5. Given a payment is SENT_TO_BILLER, When the biller confirms, Then the status becomes COMPLETED.
6. Given a payment is SENT_TO_BILLER, When the biller refuses, Then the status becomes FAILED.
7. Given a payment is SENT_TO_BILLER, When the biller does not answer within the 10-second timeout, Then the status becomes UNKNOWN, the biller is re-queried by the retry rules (see D-06), and it is then sent for support follow-up.
8. Given a payment is UNKNOWN, When the biller later confirms (callback, retry or settlement file), Then the status becomes COMPLETED.
9. Given a payment is UNKNOWN, When the biller later refuses, or reconciliation finds no payment, Then the status becomes FAILED.
10. Given a payment is FAILED, When the Hold is released to the wallet, Then the status becomes REFUNDED.
11. Given a payment is in a final state (COMPLETED, REJECTED or REFUNDED), or any transition not listed above is attempted, When the change is requested, Then the change is rejected.
12. Given a payment changes state, When the change is saved, Then a PaymentStatusHistory record of the transition is stored.

**Priority:** Must

#### US-12 (FR-12): Hold & Release Funds

**User Story:** As a customer, I want the payment amount to be held while the payment is processed, so that I cannot spend it twice and I get it back if the payment fails.

**Acceptance Criteria:**

1. Given a payment is INITIATED and the checks pass, When the payment starts, Then a Hold moves the totalAmount from the availableBalance to the heldBalance and the status becomes FUNDS_HELD.
2. Given a payment is SENT_TO_BILLER and the biller confirms, When the payment becomes COMPLETED, Then the Hold becomes the final debit.
3. Given a payment is FAILED, When the failure is confirmed, Then the Hold is released to the availableBalance and the payment transitions to REFUNDED.
4. Given a payment is UNKNOWN, When its outcome is not yet confirmed, Then the funds stay held until the outcome is confirmed and are never refunded automatically.
5. Given a payment is REJECTED, When the payment ends, Then no Hold is created and the availableBalance is unchanged.

**Priority:** Must

#### US-13 (FR-13): Unknown Payment & Re-query

**User Story:** As a customer, I want a payment that timed out to be treated as "being processed", so that I am not charged twice or refunded by mistake.

**Acceptance Criteria:**

1. Given a payment is SENT_TO_BILLER, When the biller does not answer within 10 seconds, Then the status becomes UNKNOWN, the Hold stays, and the customer is told the payment is being processed.
2. Given a payment is UNKNOWN, When 1, 5 and 15 minutes have passed, Then the system re-queries the biller at each time, and after the third attempt waits for reconciliation.
3. Given a payment is UNKNOWN, When the biller later confirms, Then the status becomes COMPLETED; When the biller refuses, Then it becomes FAILED.
4. Given a payment is UNKNOWN, When no outcome is confirmed yet, Then the money is never refunded automatically.

**Priority:** Must

#### US-14 (FR-14): Asynchronous Callbacks

**User Story:** As PayLink, I want to accept TopUpNet callbacks and ignore duplicates, so that a payment completes only once.

**Acceptance Criteria:**

1. Given a TopUpNet payment is SENT_TO_BILLER and the biller replied "accepted", When the callback "success" arrives, Then the status becomes COMPLETED.
2. Given a TopUpNet payment is SENT_TO_BILLER, When the callback "failed" arrives, Then the status becomes FAILED.
3. Given a payment is already COMPLETED, When a second "success" callback arrives, Then it is ignored, logged, and no new LedgerEntry is created.

**Priority:** Must

#### US-15 (FR-15): Multiple Billers

**User Story:** As an AdminUser, I want ElectroCo, AquaCo and TopUpNet supported, so that customers can pay electricity, water and mobile top-up bills.

**Acceptance Criteria:**

1. Given a bill inquiry or payment for any supported biller, When the request is sent, Then the biller's own protocol and format is used and the response is converted to PayLink's common model.
2. Given a new biller must be added, When it is added, Then only a new integration and configuration are added and existing modules are unchanged.
3. Given one biller is unavailable, When customers pay other billers, Then those payments continue normally.

**Priority:** Must

#### US-16 (FR-16): Fee Calculation

**User Story:** As a customer, I want to see the correct fee before paying, so that I know the total I will be charged.

**Acceptance Criteria:**

1. Given an electricity or water bill, When the fee is calculated, Then the fee is 2.50 EGP.
2. Given a mobile top-up, When the fee is calculated, Then it is 1.5% of the amount with a minimum of 1 EGP.
3. Given an internet bill, When the amount is up to 200 EGP, Then the fee is 2 EGP; above 200 EGP, it is 4 EGP.
4. Given a promotion is active for one biller, When today is between its start and end dates, Then the fee for that biller is 0.
5. Given the fee is calculated, When the total is displayed, Then totalAmount equals amount plus feeAmount, using exact decimal values.

**Priority:** Must

#### US-17 (FR-17): Agent Payments & Commission

**User Story:** As an Agent, I want to pay on behalf of a customer and earn a commission, so that my shop is rewarded for the service.

**Acceptance Criteria:**

1. Given an Agent starts a payment for a customer, When it is created, Then the payment is paid from the customer's wallet and recorded with the Agent's id (D-10).
2. Given an agent payment becomes COMPLETED, When the completion is recorded, Then a Commission of 40% of the service fee is credited to the Agent's wallet.
3. Given an agent payment becomes FAILED or REJECTED, When the payment ends, Then no Commission is credited.

**Priority:** Should

#### US-18 (FR-18): Payment & Daily Limits

**User Story:** As PayLink, I want payments above the limits to be rejected with a clear reason, so that risk is controlled.

**Acceptance Criteria:**

1. Given the amount is below 5 EGP or above 10,000 EGP, When the customer confirms the payment, Then the payment becomes REJECTED with the reason shown.
2. Given a BASIC account whose payments today plus the new total would exceed 5,000 EGP, When the customer confirms, Then the payment becomes REJECTED.
3. Given a VERIFIED account, When the daily total would exceed 30,000 EGP, Then the payment becomes REJECTED.
4. Given a payment is rejected for a limit, When it ends, Then no funds are held and the balance is unchanged.

**Priority:** Must

#### US-19 (FR-19): Velocity Rule

**User Story:** As PayLink, I want to block accounts that pay too fast, so that fraud is limited.

**Acceptance Criteria:**

1. Given an account made 5 payments in the last 60 seconds, When it attempts a 6th, Then the payment becomes REJECTED and the account is blocked for 10 minutes.
2. Given an account is blocked by the velocity rule, When a payment is attempted during the 10 minutes, Then it becomes REJECTED with the reason shown.
3. Given 10 minutes have passed, When the customer pays again, Then the payment is processed normally.

**Priority:** Should

#### US-20 (FR-20): Payment Search for Support

**User Story:** As a SupportUser, I want to search a payment by its reference and see its full history, so that I can resolve customer problems.

**Acceptance Criteria:**

1. Given a valid reference, When the SupportUser searches, Then the payment details are shown with every PaymentStatusHistory record and its time.
2. Given an unknown reference, When the SupportUser searches, Then a "not found" message is shown.
3. Given a user who is not a SupportUser or AdminUser, When they search by reference, Then the action is denied.

**Priority:** Must

#### US-21 (FR-21): Block & Unblock Account

**User Story:** As a SupportUser, I want to block and unblock an account, so that suspicious accounts cannot pay.

**Acceptance Criteria:**

1. Given an active account, When a SupportUser blocks it, Then new payments from it become REJECTED with the reason "account blocked".
2. Given a blocked account, When a SupportUser unblocks it, Then the customer can pay again.
3. Given an account is blocked or unblocked, When the action is saved, Then it is written to the AuditLog with who, when and why.

**Priority:** Should

#### US-22 (FR-22): Completion & Failure Notifications

**User Story:** As a customer, I want an SMS and a push message when my payment completes or fails, so that I know the result.

**Acceptance Criteria:**

1. Given a payment becomes COMPLETED, When it is recorded, Then an SMS and a push Notification with the amount and reference are sent.
2. Given a payment becomes FAILED, When it is recorded, Then a message with a clear reason is sent.
3. Given a notification cannot be sent, When the payment is processed, Then the payment result is not affected.

**Priority:** Must

#### US-23 (FR-23): Notification Retry

**User Story:** As PayLink, I want failed notifications to be retried, so that temporary provider problems do not lose messages.

**Acceptance Criteria:**

1. Given the SMS provider is down when a payment completes, When the SMS fails, Then the payment still becomes COMPLETED.
2. Given an SMS or email fails, When the system retries, Then it retries up to 3 times.
3. Given all 3 retries fail, When the last attempt fails, Then the Notification is marked undelivered.

**Priority:** Should

#### US-24 (FR-24): Editable Fees & Limits

**User Story:** As an AdminUser, I want to edit fee and limit values without redeploying, so that business rules can change quickly.

**Acceptance Criteria:**

1. Given an AdminUser changes a FeeRule or limit, When the change is saved, Then the next payment uses the new value without redeploying.
2. Given an invalid value (negative or empty), When the AdminUser saves, Then the change is rejected and the old value stays.
3. Given a change is saved, When it is stored, Then it is written to the AuditLog.
4. Given a user who is not an AdminUser, When they try to edit these values, Then the action is denied.

**Priority:** Should

#### US-25 (FR-25): Audit Log

**User Story:** As PayLink Management, I want every money movement and admin action recorded, so that everything can be traced.

**Acceptance Criteria:**

1. Given any LedgerEntry is created or an admin action is done, When it is saved, Then an AuditLog record with who, when and why is created.
2. Given an AuditLog record exists, When someone tries to edit or delete it, Then the action is rejected (D-17).
3. Given a money movement, When the audit records are counted, Then every movement has an audit record.

**Priority:** Must

#### US-26 (FR-26): Reconciliation Report

**User Story:** As a SupportUser, I want a daily report comparing PayLink's payments with the biller's settlement file, so that differences are found.

**Acceptance Criteria:**

1. Given a settlement file is received, When reconciliation runs, Then each SettlementRecord is compared with the matching Payment and a ReconciliationReport is produced.
2. Given a SettlementRecord exists for a Payment that is FAILED in PayLink, When the report is created, Then the difference is flagged for support review.
3. Given a Payment is UNKNOWN and is not in the settlement file, When reconciliation runs, Then the payment becomes FAILED.
4. Given a Payment is UNKNOWN and is in the settlement file, When reconciliation runs, Then the payment becomes COMPLETED.

**Priority:** Should

#### US-27 (FR-27): Refund of a Completed Payment

**User Story:** As a customer, I want to ask for a refund of a completed payment, so that support can review it and return my money if it is justified.

**Acceptance Criteria:**

1. Given a payment is COMPLETED, When the customer submits a refund request with a reason, Then a RefundRequest is created as PENDING and the payment stays COMPLETED.
2. Given a RefundRequest is PENDING, When a SupportUser approves it, Then reversing LedgerEntries return the totalAmount to the customer's wallet, reverse the fee income and any agent Commission, and the actions are written to the AuditLog (D-12, D-13, D-14).
3. Given a RefundRequest is PENDING, When a SupportUser rejects it, Then no money moves and the reason is saved.
4. Given a payment is not COMPLETED, When the customer requests a refund, Then the request is refused.

**Priority:** Could

#### US-28 (FR-28): Admin Daily Totals

**User Story:** As an AdminUser, I want to see daily totals, so that I can monitor the business.

**Acceptance Criteria:**

1. Given a selected date, When the AdminUser opens the totals, Then the number and amount of payments, fees earned, commissions paid, and failures per biller are shown.
2. Given a day with no payments, When the AdminUser opens the totals, Then all values show 0.
3. Given a user who is not an AdminUser, When they open the totals, Then the action is denied.

**Priority:** Could

## 5. Non-Functional Requirements

The six qualities and their targets come from the project brief. The verification methods are the proposed way to measure each target and can be refined by the team.

### 5.1 Summary

| ID | Quality | Measurable Target |
|---|---|---|
| NFR-01 | Correctness | Zero negative balances and zero double charges in the concurrency and duplicate tests. |
| NFR-02 | Performance | 100 payments per second for one minute against mock billers. |
| NFR-03 | Resilience | If one biller fails, payments to the other billers continue to work. |
| NFR-04 | Modifiability | Adding a biller touches only the Biller Integration module (billers) and configuration. |
| NFR-05 | Auditability | 100% of money movements have an audit record. |
| NFR-06 | Testability | At least 70% line coverage on domain logic, measured in CI. |

### 5.2 Detailed Requirements

#### NFR-01: Correctness

- **Requirement:** The system never loses or creates money by mistake, even when many requests arrive together or the same request is sent twice.
- **Measurable target:** Zero negative balances and zero double charges in the concurrency and duplicate tests.
- **How it is verified:** Automated tests that send concurrent payments from one Wallet and repeat the same request with the same idempotencyKey. After the tests, no availableBalance is negative, no payment is charged twice, and each balance equals the sum of its LedgerEntry records.
- **Related stories:** US-05, US-09, US-10, US-12
- **Priority:** Must

#### NFR-02: Performance

- **Requirement:** The payment flow stays fast under load.
- **Measurable target:** 100 payments per second for one minute against mock billers.
- **How it is verified:** A load test that sends 100 payments per second for 60 seconds (about 6,000 payments) to mock billers. The measured throughput and results are saved in the test report.
- **Related stories:** US-09, US-11
- **Priority:** Must

#### NFR-03: Resilience

- **Requirement:** A problem at one biller does not stop payments to the other billers.
- **Measurable target:** If one biller fails, payments to the other billers continue to work.
- **How it is verified:** A test where one mock biller is down or does not answer, while payments to the other billers are sent and reach COMPLETED. Payments to the failing biller follow the normal UNKNOWN or FAILED rules.
- **Related stories:** US-11, US-15
- **Priority:** Must

#### NFR-04: Modifiability

- **Requirement:** A new biller can be added without changing the rest of the system.
- **Measurable target:** Adding a biller touches only the Biller Integration module (billers) and configuration.
- **How it is verified:** Add a new mock biller as an exercise and check the change list: only a new adapter in billers and configuration entries are added, and no other module is edited.
- **Related stories:** US-06, US-15
- **Priority:** Must

#### NFR-05: Auditability

- **Requirement:** Every money movement can be traced.
- **Measurable target:** 100% of money movements have an audit record.
- **How it is verified:** A test or query that counts LedgerEntry records and AuditLog records after a set of payments, refunds and admin actions. Every money movement has an AuditLog record showing who, when and why.
- **Related stories:** US-05, US-25
- **Priority:** Must

#### NFR-06: Testability

- **Requirement:** The business logic is covered by automated tests that run on every change.
- **Measurable target:** At least 70% line coverage on domain logic, measured in CI.
- **How it is verified:** The CI pipeline runs the tests and publishes the line coverage report for the domain logic. The target is 70% or higher.
- **Related stories:** All user stories
- **Priority:** Must

## 6. Questions and Ambiguities

| ID | Question | Related Stories |
|---|---|---|
| Q-01 | What happens if the customer has insufficient balance? | US-09 |
| Q-02 | What happens if the biller does not respond? | US-08, US-11 |
| Q-03 | Can the same payment request be sent twice? | US-10 |
| Q-04 | When should held funds be released? | US-12 |
| Q-05 | What is the difference between a BASIC and a VERIFIED account? | US-01, US-02 |
| Q-06 | What happens to an UNKNOWN payment afterwards? | US-11, US-12 |
| Q-07 | What are the limits on a single payment and on the daily total? | US-02, US-09 |
| Q-08 | Which currency does the wallet use? | US-03, US-04 |
| Q-09 | How long is an idempotency key remembered? | US-10 |
| Q-10 | Can an Agent pay on behalf of a customer, and from which wallet? | US-09 |
| Q-11 | Who confirms an UNKNOWN payment: the system's re-query or the SupportUser? | US-13, US-26 |
| Q-12 | How can a COMPLETED payment be refunded if COMPLETED is final and REFUNDED comes only from FAILED? | US-27 |
| Q-13 | Is the service fee returned in a refund? | US-27 |
| Q-14 | Is the agent Commission reversed in a refund? | US-17, US-27 |
| Q-15 | Where are fee and limit values stored so the AdminUser can edit them? | US-24 |
| Q-16 | Does "more than 5 payments in 60 seconds" block the 5th or the 6th payment? | US-19 |
| Q-17 | Can AuditLog records be changed or deleted? | US-25 |
| Q-18 | Who pays the refund, since the biller already received the money? | US-27 |
| Q-19 | Can PayLink's internal system wallet go below zero when it pays refunds? | US-27, NFR-01 |
| Q-20 | Which payments count toward the daily limit? | US-09 |

## 7. Decisions Made

| ID | Question | Decision |
|---|---|---|
| D-01 | Q-01 | Reject the payment (REJECTED). The balance is not changed and nothing is sent to the biller. |
| D-02 | Q-02 | Mark the payment as UNKNOWN after 10 seconds without a biller response. Its funds stay held. |
| D-03 | Q-03 | Use an idempotencyKey. A repeated key with the same data returns the original result. |
| D-04 | Q-04 | Release held funds only when payment failure is confirmed. The payment then moves from FAILED to REFUNDED. |
| D-05 | Q-05 | A new account is BASIC and can use the wallet with a daily limit of 5,000 EGP. Upgrading to VERIFIED raises the daily limit to 30,000 EGP. |
| D-06 | Q-06 | An UNKNOWN payment keeps its funds held and is never refunded automatically. First the system re-queries the biller 3 times (after 1, 5 and 15 minutes), then waits for reconciliation. A SupportUser can follow up manually. When the real outcome is confirmed (callback, re-query, settlement file or reconciliation), the payment becomes COMPLETED, or FAILED and then REFUNDED. This matches D-11. |
| D-07 | Q-07 | As specified in the assignment: minimum single payment is 5 EGP, maximum single payment is 10,000 EGP, BASIC daily limit is 5,000 EGP and VERIFIED daily limit is 30,000 EGP. These numbers are kept in configuration, not in code. |
| D-08 | Q-08 | EGP only, as defined in the project glossary. |
| D-09 | Q-09 | A key is remembered for 24 hours per customer. This is not an assignment requirement. |
| D-10 | Q-10 | An Agent pays from the customer's wallet, identified by the customer's mobile number, and the payment is recorded with the Agent's id. |
| D-11 | Q-11 | First the system re-queries the biller 3 times (after 1, 5 and 15 minutes), then waits for reconciliation. A SupportUser can follow up manually. |
| D-12 | Q-12 | A refund of a COMPLETED payment is a separate process: RefundRequest plus reversing LedgerEntries. The payment status does not change. |
| D-13 | Q-13 | The service fee is returned: the refund equals totalAmount. |
| D-14 | Q-14 | The agent Commission is reversed. If the agent already spent it, PayLink absorbs the difference. |
| D-15 | Q-15 | Values are stored in database configuration tables that the AdminUser edits through the API. |
| D-16 | Q-16 | The 6th payment inside 60 seconds is rejected and the account is blocked for 10 minutes. |
| D-17 | Q-17 | AuditLog records are append-only. This is not in the brief. |
| D-18 | Q-18 | The refund is paid from PayLink's internal system wallet. |
| D-19 | Q-19 | Yes. The internal system wallet is PayLink's own operational wallet, not a customer wallet, so it may go below zero when it pays refunds of COMPLETED payments; PayLink covers it with its own funds. NFR-01 (no negative balances) applies to customer and agent wallets. |
| D-20 | Q-20 | Payments that reached FUNDS_HELD, SENT_TO_BILLER, UNKNOWN or COMPLETED on that day count toward the daily total (total amount, per US-09.5). Payments that are REJECTED, FAILED or REFUNDED do not count. |

All decisions D-01 to D-20 are confirmed team decisions (as of 25 November 2026). D-05, D-07 and D-08 follow the assignment and the project glossary.

## 8. Conflicts Found Between Documents

| # | Conflict | Where | How it was handled |
|---|---|---|---|
| 1 | An early version of D-06 said a SupportUser confirms an UNKNOWN payment, but FR-13 says the system re-queries the biller 3 times first. | D-06 vs US-13 | Resolved: D-06 now matches D-11. |
| 2 | An early version wrote states as FundsHeld and SentToBiller, but the glossary requires FUNDS_HELD and SENT_TO_BILLER. | US-09, US-11, US-12 vs glossary | Resolved: glossary names are used everywhere. |
| 3 | FR-27 would move a COMPLETED payment to REFUNDED, but COMPLETED is final and REFUNDED comes only from FAILED. | US-27 vs state machine | Resolved by D-12: separate RefundRequest process, payment status unchanged. |
| 4 | RefundRequest is not yet in the glossary. | US-27 vs glossary | Resolved: RefundRequest is now in the glossary and the class diagram; it is the refund_requests table in the ER diagram. |
