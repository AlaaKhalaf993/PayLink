# Design pack

Source files for the Phase 1 design pack. The `.puml` files are the source of truth. The images in `img/` are only for the PDF report.

## Files by Abdelrahman Ahmed Ashry (Architecture Lead)

| File | What it is |
|---|---|
| `glossary.md` | Single source of truth for names (modules, entities, fields, enums, payment states), the code-to-database naming rule and the known differences with the ER |
| `architecture.md` | Hexagonal modular monolith: modules and the tables each one owns, interfaces, dependency rules, patterns, hard cases, section 9 = decisions and open questions |
| `stack-decision.md` | Java 21, Spring Boot 4.x, PostgreSQL, Maven (matches `pom.xml`) |
| `payment-state.puml` | State diagram, the 9 transitions of the brief section 2.5 |
| `class-diagram.puml` | Domain class diagram: 22 entities, 8 State classes, 6 pattern interfaces |
| `seq-pay-bill.puml` | Sequence: pay a bill (inquiry, pay, all four biller answers, notification) |
| `seq-refund-failed.puml` | Sequence: refund a failed payment (four ways to FAILED, then the refund) |
| `seq-refund-completed.puml` | Sequence: refund a completed payment (FR-27): request, support rejects or approves, reversing ledger entries, payment stays COMPLETED |
| `seq-unknown-confirmed.puml` | Sequence: an UNKNOWN payment later confirmed COMPLETED (re-query, callback, settlement file or support): `captureHold`, `DEBIT`, `FEE_INCOME` and `COMMISSION` entries |
| `../prompt-log.md` | Prompt log (14 entries; entries 2, 3, 4, 6, 8, 9, 11, 12, 13 and 14 include real AI errors that were fixed) |

## Files by Mariam Mohamed Ali (Data & Integration Lead)

| File | What it is |
|---|---|
| `er.puml` | ER diagram (images: `img/er.png`, `er.svg`, `er.pdf`) |
| `use-case-1-customer-agent.puml` | Use cases 1/3: Customer and Agent |
| `use-case-2-admin-support.puml` | Use cases 2/3: Admin, Support and Management |
| `use-case-3-pay-a-bill.puml` | Use cases 3/3: inside "Pay a Bill" |

They must use the names in `glossary.md`. The differences that are still open are at the end of the glossary and in `architecture.md` section 9.

## How to render

PlantUML needs no install: paste a `.puml` file into https://www.plantuml.com/plantuml and use the PNG or SVG link (the site limits images to 4096 px; the diagrams of Abdelrahman Ahmed Ashry fit). In VS Code, use the PlantUML extension by jebbs.

## Open points for the team (not decided by Abdelrahman Ahmed Ashry)

Details are in `architecture.md` section 9.

Closed in the latest version: `top_ups.status` (`TopUpStatus` = PENDING, SUCCESS, FAILED), one storage place for min/max amount (`biller_services`), `MinMaxRule`, the audit-log rule, `Account` attributes.

Closed earlier: the ledger fields (the ER now has `available_delta` and `held_delta`, same as the class diagram), the enum values (the class diagram now uses `TOPUP`, `CAPTURED` and the three `DifferenceKind` values of the ER), `BillerCategory`, and the ER columns `biller_account_ref` and `failure_reason`. See the glossary, section "Known differences".

1. **Decisions marked "team to confirm".** D-09, D-10, D-13, D-14, D-15, D-17 and D-18. The refund of a completed payment (`seq-refund-completed.puml`) follows D-12, D-13, D-14 and D-18.
2. **Can the `SYSTEM` wallet go below zero?** It pays the refunds of completed payments, and NFR-01 says no negative balances.
3. **Requirements pack.** The conflicts table (section 8, row 4) still says that `RefundRequest` is open; it is in the glossary and the class diagram now.
4. **Assumptions of the design** (not in the requirements): one approved refund per payment, a rejected payment counts in the velocity rule, and the daily limit counts only payments in `FUNDS_HELD`, `SENT_TO_BILLER`, `UNKNOWN` or `COMPLETED` (architecture.md 4.3).
5. **Not checkable in this pack:** `pom.xml` and `ci.yml` are not included, so the Spring Boot version in `stack-decision.md` is not verified here.
