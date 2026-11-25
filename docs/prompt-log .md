# Prompt Log

Rules from the brief:
- Log every AI prompt, honestly. Rejected or fixed answers earn marks when the reason is explained.
- Phase 1 needs at least 5 entries, including at least 2 where the first AI answer was wrong and the prompt was improved.
- Use structured prompts: Role, Context, Constraints, Output format, and an example when useful.
- Never paste passwords, real personal data or other teams' code into an AI tool.

| Date | Author | Stage | Tool and model | Prompt | Summary of answer | Verdict (accepted, fixed, rejected) and why |
|---|---|---|---|---|---|---|
| 2026-11-23 | Mariam | Requirements | Claude Sonnet 5.5 | Uploaded the project brief PDF and the team role table. Asked (in Egyptian Arabic): "Explain the project and what I should do as Person 4 in Phase 1?" | Explained the payment flow, split my work (stories FR-13 to FR-28, use-case diagram, ER diagram, final PDF) and gave a 2-day plan. | Accepted. I checked the deadline and my role against the brief and the role table myself. |
| 2026-11-23 | Mariam | Requirements | Claude Sonnet 5.5 | "Let's write the 16 user stories." (no glossary attached, translated from Arabic) | 16 stories in Given/When/Then with priorities from the brief. | Fixed. State and entity names did not match our glossary (`Unknown` instead of `UNKNOWN`, `SentToBiller` instead of `SENT_TO_BILLER`). The AI did not have the glossary. FR-27 also moved a `COMPLETED` payment to `REFUNDED`, which our state machine does not allow. I attached the glossary in the next prompt. |
| 2026-11-23 | Mariam | Requirements | Claude Sonnet 5.5 | Uploaded glossary.md, stack-decision.md and prompt-log.md from Abdelrahman and asked: "Explain these files and how we proceed." (translated from Arabic) | Explained the glossary as the single source of names, listed what must change in my stories and which tables the ER needs. Pointed out the FR-27 conflict with the state machine. | Accepted. I read the glossary myself and confirmed the points (8 states, `COMPLETED` is final, `REFUNDED` only from `FAILED`). |
| 2026-11-23 | Mariam | Requirements | Claude Sonnet 5.5 | "Fix the names using the glossary. Leave the wrong ones and tell me how to fix them, I will fix them myself." (translated from Arabic) | Stories renamed with glossary names. FR-27 left as wrong with an explanation. | Fixed. The renamed stories were correct, FR-27 was the problem. I chose the option where the payment stays `COMPLETED` and the refund is a separate `RefundRequest` with a reverse `LedgerEntry`. Added to the ambiguities list. |
| 2026-11-23 | Mariam | Requirements | Claude Sonnet 5.5 | Uploaded Shaza's requirements.md/PDF (US-01 to US-12) and asked: "Read Person 1's work and tell me what is required from me" (translated from Arabic) | Confirmed my 4 tasks (stories FR-13 to FR-28, use-case diagram, ER diagram, final PDF). Compared his format and decisions with our glossary and found 3 conflicts (listed below). | Accepted after checking each conflict myself against his file, our glossary and the brief. See "Conflicts found". |
| 2026-11-23 | Mariam | Requirements | Claude Sonnet 5.5 | "Write the 16 stories in Shaza's format: US-13 to US-28, numbered Given/When/Then criteria, Priority line, traceability rows, Q-11+ and D-11+." Attached: requirements.md and glossary.md. (translated from Arabic) | 16 stories (US-13 to US-28) with traceability table, 7 new questions (Q-11 to Q-17) and 7 decisions (D-11 to D-17). Several decisions were marked as assumptions. | Fixed. The format and names are correct, but the decisions D-13, D-14, D-15 and D-17 are AI assumptions that are not in the brief. I marked them "needs team approval" instead of treating them as final. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Uploaded a screenshot of my row in the role table. Asked: "this is my software assignment i need you to read it carefully" | Translated the Arabic row: role Requirements & Product Lead, with problem summary, stakeholders, user stories FR-1 to FR-12 and ambiguities and decisions. Said it could not see the full brief. | Accepted. I checked the translation against the role table. The AI correctly said it did not have the brief, so I pasted my instructions next. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Pasted my part of the instructions (problem summary, stakeholders table, stories FR-1 to FR-12 with Given/When/Then and Priority, ambiguities table, final `requirements.md` and PDF). Asked (in Egyptian Arabic): "Apply this for me." | `requirements.md` and PDF with intro, problem summary, 6 stakeholders, US-01 to US-12, Q-01 to Q-10 and D-01 to D-10. | Fixed. The AI only had the topic names of FR-1 to FR-12, so it guessed: OTP verification in US-02, US-03 as Should, payment deducted at once, invented statuses and limits. I corrected these in the next prompts. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Pasted corrections A to E from the brief: US-02 is an upgrade from Basic to Verified (5,000 and 30,000 EGP a day), US-03 is Must, US-06 includes services, US-08 shows fee and total, US-09 holds funds before the biller call. Asked (in Egyptian Arabic): "Fix these in the document." | Applied all five. Also changed US-01 (account starts Basic), Q-05, D-05 and added a daily-limit criterion to US-09. | Fixed. The OTP rule contradicted FR-2 and deducting at once contradicted FR-12. I checked each change against the brief. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Pasted the 8 exact payment statuses. Asked (in Egyptian Arabic): "Use these exact statuses everywhere, with `COMPLETED`, `REJECTED` and `REFUNDED` as final, failed funds going to `REFUNDED`, and `UNKNOWN` keeping funds held." | Rewrote US-11 and US-12 and updated US-09, D-01, D-02, D-04 and D-06. When my first message was cut off, it asked for the missing statuses instead of guessing. | Fixed. The first version used Pending, Successful, Failed and Unknown, which are not the brief's names. I searched the file to confirm no old status name was left. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Told it Q-07 was wrong because the brief gives minimum 5, maximum 10,000, Basic 5,000 and Verified 30,000 EGP, and D-09 (24 hours) must be marked as an assumption. Asked (in Egyptian Arabic): "Update D-07 with these exact values." | Updated D-07 with the exact values, added a single-payment-limit criterion to US-09, and labeled D-09 "Assumption, needs team approval". | Fixed. The AI had written that the limits would be agreed with the team, but the brief defines them. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Asked (in Egyptian Arabic): "Are the last changes correct and consistent with the assignment? What do you think?" | Said the changes matched what I pasted but that it had not seen the brief. Listed 7 doubtful points (see Ambiguities A-01 to A-07). | Accepted. I used the list as a checklist against the brief. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Pasted 4 more brief checks: US-03 shows available and held balance for any registered customer, US-04 names the channels (simulated card, agent cash) and does not need Verified, US-06 needs an edit criterion, US-09 credits fee income and records agent commission. Asked (in Egyptian Arabic): "Apply these fixes." | Applied all four fixes. | Fixed. The earlier version limited the wallet to Verified customers and missed the held balance. I checked FR-3, FR-4, FR-6 and the payment steps. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Uploaded the PDF. Asked (in Egyptian Arabic): "Why did you change the intro and the problem summary?" | Checked and said it had not changed them. The PDF I uploaded had different text. I then chose the PDF text and it copied it into the `.md` and rebuilt the PDF. | Accepted. The difference came from my own copy. The "Document conventions" list was removed because the PDF did not have it. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Uploaded `glossary.md`. Asked (in Egyptian Arabic): "The names in the glossary are the same as yours, take the naming from it." | Renamed states (`FUNDS_HELD`, `SENT_TO_BILLER`), levels (`BASIC`, `VERIFIED`), entities (`LedgerEntry`, `Hold`, `AuditLog`, `IdempotencyRecord`) and fields (`availableBalance`, `totalAmount`). Rewrote US-11 with the 9 allowed transitions. | Fixed. The AI did not have the glossary at first, so the names did not match. I checked the 8 states and 9 transitions against the glossary. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Asked (in Egyptian Arabic): "Update D-06 to match D-11." I first uploaded the wrong file (`prompt-log.md`), then the Part 2 PDF. | With the first file it refused to guess because D-11 was not in it. With Part 2 it rewrote D-06: 3 re-queries (1, 5, 15 minutes), then reconciliation, support can follow up, `UNKNOWN` never refunded automatically. | Accepted. Not guessing was correct. I compared the new D-06 with D-11 myself. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Sent a screenshot of review notes. Asked (in Egyptian Arabic): "Finish these and make the NFRs in a separate file." | Changed US-11 criterion 7 to mention the retry rules, removed "above the allowed limit" from US-04 (no top-up limit in the brief), created `non-functional-requirements.md` and PDF with 6 NFRs. | Accepted after checking. The NFR targets come from the brief table in the screenshot. The verification methods are AI proposals, so the team must confirm them. |
| 2026-11-23 | Shaza | Requirements | Claude Sonnet 5.5 | Asked (in Egyptian Arabic): "Remove the green color, make it all black." Then: "send the requirment file as md" | Rebuilt both PDFs in black with light gray table headers. Sent the `.md` again. | Accepted. The old color was dark blue, not green, but it is black now. |
| 2026-11-23 | Mariam | Requirements | Claude Sonnet 5.5 | "Explain D-13, D-14, D-15 and D-17: what each is about, the options, and what the team must discuss." (translated from Arabic) | Explained each decision with 3 or 4 options and trade-offs, suggested one option per decision, and listed 12 open points for the team. | Accepted after team discussion. We agreed on the suggested options: full refund including the fee (D-13), commission reversed and PayLink absorbs the shortfall (D-14), separate fee and limit tables (D-15), append-only AuditLog (D-17), refund paid from the system wallet (D-18). |
| 2026-11-23 | Mariam | Requirements | Claude Sonnet 5.5 | Attached all three parts (Shaza's requirements, Alaa's NFRs, my stories) and asked: "Check them and merge everything in the order the doctor wants." (translated from Arabic) | Checked that the three parts are consistent (state names, D-06 vs D-11, NFR story links) and produced one merged document. | Accepted after review. I checked the merged text against each source file. Items still open: remove "team to confirm" from D-13, D-14, D-18, and mark the RefundRequest conflict as resolved once Abdelrahman updates the glossary. |
| 2026-11-24 | Mariam | Design | Claude Sonnet 5.5 | "Write structured prompts (Role, Context, Constraints, Output format, Example) for the ER diagram and the use-case diagram in PlantUML, with everything the team agreed." (translated from Arabic) | Two prompts: an ER prompt with 20 tables, columns and relationships, and a use-case prompt with actors, use cases and include/extend rules. | Accepted. Used as the base for the two prompts below. |
| 2026-11-24 | Mariam | Design | [write the AI tool and model you used for the PlantUML code] | ER prompt. Structured prompt (Role, Context, Constraints, Output format, Example) written with Claude's help. Full text in "Full prompts" below. | One er.puml file with 24 tables. | Fixed. The AI did not follow the table list: it added top_ups, biller_callbacks, requery_attempts and settlement_files, and renamed service_input_fields to biller_required_fields. ledger_entries had no direction column, so a balance cannot be summed from it. Four relationships used mandatory cardinality (`\|\|`) for nullable foreign keys (payments to ledger_entries, refund_requests to ledger_entries, payments to notifications, ledger_entries to audit_logs). I corrected these. |
| 2026-11-24 | Mariam | Design | [write the AI tool and model you used for the PlantUML code] | Use-case prompt. Structured prompt (Role, Context, Constraints, Output format, Example) written with Claude's help. Full text in "Full prompts" below. | Three .puml files: Customer and Agent, Admin and Support, and the inside of "Pay a Bill". | Fixed. NetCo was drawn although the prompt said to leave it out (it is optional in the brief), there was no use case for the agent commission (US-17), and the third diagram had no Customer actor. "Monitor Ledger Entries" is not in any user story. I corrected these. |
| 2026-11-24 | Mariam | Design | Claude Sonnet 5.5 | Screenshot of "HTTP Status 400 - Request header is too large" from plantuml.com when rendering the ER diagram. (translated from Arabic) | Explained that the online server puts the whole diagram in the URL, so a large diagram is rejected, and gave steps to render locally in VS Code. | Accepted. I rendered the diagram locally with the VS Code PlantUML extension and it worked. |
| 2026-11-24 | Mariam | Design | Claude Sonnet 5.5 | Uploaded er.puml and the three use-case files and asked: "Is my work finished?" (translated from Arabic) | Reviewed all four files against the stories and decisions, found the cardinality and direction problems, and listed what is still missing for submission. | Accepted. I checked every point in the files myself before changing them. |
| 2026-11-25 | Mariam | Design | Claude Sonnet 5.5 | Uploaded Abdelrahman's design pack (PayLink-design-pack-fixed.zip, with my ER and use-case files) and asked: "Check everything in this folder. Is it all fine, or is something in conflict with something else?" (translated from Arabic) | Read the glossary, architecture, class diagram, state diagram, four sequence diagrams, ER, three use-case diagrams, README and prompt log, and compared them with each other and with the requirements. Findings: the log entries written under my name did not match my work (tool name, rendering, class diagram); the README told readers to render on plantuml.com, which failed for my ER; the requirements still had "team to confirm" and an open RefundRequest conflict; four ER relationships used the mandatory-one symbol (`\|\|`) for nullable foreign keys; architecture.md said three fraud rules but there are four (MinMaxRule); small use-case gaps (no commission use case, no Customer in diagram 3/3, "Monitor Ledger Entries" is not in any story, NetCo is optional); the ER and class images are very wide for a PDF page. | Accepted. I used the findings as a checklist against the files. I replaced the log entries under my name with the entries in this log, and updated the requirements document (next entry). Left open for Phase 2: the ER cardinality notation, the small use-case gaps and the image width in the PDF. The AI could not render the diagrams, so it compared the source files and the SVG text only. |
| 2026-11-25 | Mariam | Requirements | Claude Sonnet 5.5 | "The team agreed on the decisions. What must change in the requirements so it is final?" Then: "Remove them (the assumption labels on D-09 and D-10) and send me everything." (translated from Arabic) | Edited the requirements document: removed "team to confirm" from D-13, D-14 and D-18; rewrote D-15 (fee_rules, limit_rules, min and max amounts in biller_services) and D-17 (append-only in code, UPDATE and DELETE revoked on audit_logs); marked D-01 to D-04 and D-11 to D-18 as agreed; changed the RefundRequest conflict to Resolved; after my instruction, removed the assumption labels from D-09 and D-10. | Accepted after team agreement. The new wording follows architecture.md, er.puml and seq-refund-completed.puml. Removing the D-09 and D-10 labels was my instruction after the team accepted them. The AI cannot know what the team agreed, so these decisions come from us. |
| 2026-11-23 | Abdelrahman Ahmed Ashry | Design - state diagram | Claude (Sonnet 5.5) | Structured prompt: role, context, 8 states, 9 transitions from brief section 2.5, final states, UNKNOWN never refunded, output = one PlantUML file. Full text in "Full prompts" below | A PlantUML state diagram with 8 states, 9 transitions, 3 final states and two notes (UNKNOWN, FUNDS_HELD), plus a script that checks it against the brief | Accepted after checking: I compared every transition with the brief's table, ran the check script (9/9 passed) and rendered it on plantuml.com. I added `skinparam dpi 200` for a sharper image and removed the diagram name from `@startuml` |
| 2026-11-24 | Abdelrahman Ahmed Ashry | Design - architecture | Claude (Sonnet 5.5) | Structured prompt: role (software architect), context (8 modules from the glossary, hexagonal modular monolith, brief hard cases, ElectroCo v2 change), constraints (only `payments` orchestrates, no module reads another module's tables, no cycles, at least 5 patterns, money never float), output = `architecture.md`. Full text in "Full prompts" below | Architecture document: hexagonal style with reasons, 8 modules with the tables each owns, public interfaces, dependency rules, 6 patterns, mapping of the 8 hard cases, package layout, Phase 2 evolution and 4 open assumptions | Fixed. First version was accepted after name checks, but a later review against the brief and the class diagram (2026-11-25) found 3 real problems: (1) it never said what `payments` does when a biller answers PENDING (TopUpNet replies "accepted"), which the pay sequence diagram needs; (2) `CommissionCalculator` used `AgentId` while the class diagram uses `AccountId`; (3) the text pointed to section 5 for events but they are described in section 4. Fix: added a "What payments does with each biller answer" table (4.1), corrected the two references, and added 3 open decisions (agent pays for a customer, TopUpNet never calls back, where limits are stored). The 4 original assumptions in section 9 still have to be confirmed by the Requirements Lead. The improved prompt is the follow-up under "Follow-up prompt" below |
| 2026-11-24 | Abdelrahman Ahmed Ashry | Design - class diagram | Claude (Sonnet 5.5) | Structured prompt: role (architect writing PlantUML), context (glossary entities, architecture modules, FR-5 ledger rule), constraints (Wallet has no stored balance, money is `Money`, 8 State classes, 6 pattern interfaces, Customer/Agent are roles of Account), output = one `.puml` file. Full text in "Full prompts" below | PlantUML class diagram with the 16 glossary entities, 8 payment State classes, the 6 pattern interfaces, enums and multiplicities, plus a script that checks it | Fixed. Real AI errors found on review: (1) the PNG export was silently cut at PlantUML's 4096-pixel limit (real width 10,712 px), so the AI delivered and described as fine a picture showing under 40% of the diagram; found on 2026-11-25 when the image was cut off and exactly 4096 px wide. Fix: first raised `PLANTUML_LIMIT_SIZE` and used `left to right direction` (portrait 4395x6613); then on 2026-11-25 I rebuilt the file more compactly (polyline links, no extra layout hints) so it exports at 1722x3215 px with the default limit, which means it also works on plantuml.com without any setting. A hidden-link grid layout I tried first made it wider (6134 px), so I dropped it. (2) First layout was very wide because of about 15 "uses" arrows between services; I removed them (they are in `architecture.md`). (3) Not consistent with `architecture.md`: `balances()` returned `Money` instead of `Balances`; `AccountService`, `BillerCatalog` and `ReconciliationService` were missing; `AuditLog` was in an `audit` package while the architecture says `shared`. Fixed all three. Improved prompt: the "Follow-up prompt" below (it adds: open every exported image in full before saying it is fine, and names must be identical across files), which is what found these errors. Checks: class checker 15/15, a negative test rejected a stored `balance: double` in Wallet, and a cross-check script confirmed glossary, architecture and class diagram use the same names and enums. Names the class diagram uses for the new value classes and fields: `Money`, `Balances`, `FraudDecision`, `PaymentEvent`, `ReconciliationDifference`, `billerAccountRef`, `availableDelta`/`heldDelta`. Correction made on 2026-11-25: an earlier version of this entry said that `Role.SYSTEM` and `FraudBlock` were added to the glossary; they are not in it (`SYSTEM` is a wallet type, decision D-18, and the fraud classes are `BlockedAccountRule`, `DailyLimitRule`, `VelocityRule` and `MinMaxRule`) |
| 2026-11-25 | Abdelrahman Ahmed Ashry | Design - sequence diagram (pay a bill) | Claude (Sonnet 5.5) | Structured prompt: role (architect writing PlantUML), context (brief 2.2, architecture.md section 4.1, class diagram interface names), constraints (every state change must be one of the 9 allowed transitions, show idempotent repeat, fraud reject, insufficient funds, and all four biller answers CONFIRMED / REFUSED / PENDING / UNKNOWN, notifications must not block the payment), output = one `seq-pay-bill.puml`. Full text in "Full prompts" below | Sequence diagram with 3 phases (inquiry, pay, notification), 10 participants named as in the class diagram, one branch per biller answer, and notes for the retry rule and the duplicate callback | Fixed (small). I walked each branch (fraud reject, no funds, confirmed, refused, unknown) against the 9 transitions of brief 2.5 with a script, checked that every participant and method name exists in the class diagram, and rendered it (2236x2614 px, readable). Fixed one small AI error: the file used `skinparam ParticipantPadding` and `BoxPadding`, which the newer PlantUML on plantuml.com ignores and shows as a yellow warning above the diagram; I found it on the website and removed both lines (no message changed). Improved prompt: I added the constraint "Do not use skinparam settings that newer PlantUML versions ignore" (shown under Entry 4 below and reused in Entry 5). One assumption is still open: when an agent pays, the customer's wallet is charged (architecture.md section 9, waiting for Requirements Lead) |
| 2026-11-25 | Abdelrahman Ahmed Ashry | Design - sequence diagram (refund a failed payment) | Claude (Sonnet 5.5) | Structured prompt: role (architect writing PlantUML), context (brief 2.5 FAILED -> REFUNDED, architecture.md 4.1, class diagram names), constraints (only FAILED can be refunded, UNKNOWN never refunded automatically, hold released with a HOLD_RELEASE ledger entry, releasing twice must not credit twice, notification must not change the result, hard case 7 shown as a flagged difference only), output = one `seq-refund-failed.puml`. Full text in "Full prompts" below | Sequence diagram with 3 phases: three ways a payment becomes FAILED, the refund itself (guard on status, double-release guard, ledger and audit entries), and the notification; plus a note for the settlement-file difference | Accepted after checking: the three transitions used (SENT_TO_BILLER -> FAILED, UNKNOWN -> FAILED, FAILED -> REFUNDED) are all in brief 2.5, every participant, method and enum value exists in the class diagram, and the file rendered on both an older and a newer PlantUML without warnings (the ParticipantPadding mistake of entry 4 was not repeated). I decided to keep the refund of a COMPLETED payment (FR-27) out of this diagram because it is a separate reversal, not a state transition |
| 2026-11-25 | Abdelrahman Ahmed Ashry | Design - review against team requirements | Claude (Sonnet 5.5) | Structured prompt: role (reviewer), context (the repository: `requirements.md` with US-01 to US-12, NFR-1 to NFR-6, D-01 to D-10, `pom.xml`, `ci.yml`, plus my design files), constraints (check every story, NFR and decision against my glossary, class diagram, two sequence diagrams and architecture; report real conflicts only; do not invent fields for entities that are not in the repository). Full text in "Full prompts" below | List of conflicts and the fixes | Fixed. Real AI omissions and wrong assumptions found: (1) the pay sequence did not show the idempotency conflict of US-10.3 (same key, different data), so `IdempotencyRecord` got `requestHash`; (2) the sequence did not show a bill inquiry that is not found or times out (US-08.2 and US-08.3); (3) `AuditLog` had no `refId`, so the SQL check of NFR-5 could not link a ledger entry to its audit row; (4) the AI had assumed the daily limit is counted on the bill amount only, but US-09.5 says total amount; (5) the stack files said Spring Boot 3.x but `pom.xml` uses 4.2.0. Not changed, only listed as open for the team: package name `billers` vs `integration/` in NFR-4, `ledger_entry` vs `ledger_entries`, US-05 `direction` vs `availableDelta`/`heldDelta`, and D-06 (support confirms UNKNOWN). I did not add `LimitRule` or `RefundRequest` because their fields are not in the repository |
| 2026-11-25 | Abdelrahman Ahmed Ashry | Design - update after the ER and use cases of Mariam Mohamed Ali | Claude (Sonnet 5.5) | Structured prompt: role (architect), context (my glossary, class diagram, architecture, two sequences and the ER and use-case files of Mariam Mohamed Ali), constraints (change only my own files; add the ER entities that are missing in the class diagram; SYSTEM is a wallet type; FeeRule belongs to a biller; block status moves into Account; support can confirm an UNKNOWN payment; do not touch the ledger, enum or table decisions that the team has not made yet), output = edited files plus a check script. Full text in "Full prompts" below | Class diagram with 6 new entities and 6 new enums, glossary with a code-to-database naming rule and a table of known differences, architecture with table ownership, `AccountService.block` and `PaymentService.resolveUnknown`, refund sequence with a fourth way to FAILED | Accepted after checking: a script confirmed that every glossary entity and enum matches the class diagram, that every ER table is owned by one module, and that all transitions in the refund sequence are in brief 2.5; I rendered everything on two PlantUML versions (largest image 2805x3524 px). Decisions the AI made that I must confirm with the team: `TopUp.status` is a String because the ER gives no values; `CallbackStatus` is upper case while the ER uses lower case; `LimitRule.value` is a BigDecimal because some limits are counts; the velocity rule blocks an account through `AccountService.block` so that `fraud` never writes the accounts table. No AI error was found in this entry |
| 2026-11-25 | Abdelrahman Ahmed Ashry | Design - update after the final requirements pack | Claude (Sonnet 5.5) | Structured prompt: role (architect keeping a design pack consistent), context (the final Phase 1 requirements PDF with US-01 to US-28, NFR-01 to NFR-06 and D-01 to D-18, plus my design files), constraints (change only my own files; add the refund of a COMPLETED payment with request, approve, reject and a reversal in the wallet module; the payment status must not change; AuditLog is append-only; write the velocity rule of D-16; replace the placeholder names; do not decide the ledger fields, the enum values or whether the SYSTEM wallet may go below zero; do not add rules that no requirement states), output = edited files, one new sequence diagram and a check of names across files. Full text in "Full prompts" below | Added `WalletService.reverseCompleted` and `PaymentService.requestRefund`, `approveRefund` and `rejectRefund`; a new sequence diagram `seq-refund-completed.puml`; `AuditLog` marked append-only; architecture sections 4.2 (refund after COMPLETED), 4.3 (fraud and velocity rule) and 11 (NFRs and the design); section 9 rewritten against the final requirements; real names instead of placeholders in README, architecture and this log | Fixed. Real AI errors found on review: (1) the AI had told me in the chat that no placeholder role name was left in the files, but `README.md` still had one twice and `architecture.md` had another once; I found them by searching all files, and they are replaced now; (2) the AI suggested renaming the package `billers` to `integration` because it had read an older `requirements.md` (NFR-4 with the `integration/` path); the final requirements pack says `billers` (NFR-04), so nothing is renamed and I should have read the latest file first; (3) section 9 of `architecture.md` still listed the package name and the table names as open after the requirements changed, so I rewrote it; (4) the first draft of the new text invented two things that no requirement states, an order of the three fraud rules and database permissions for the audit log, and I removed both before delivery. Improved prompt: the constraints "search all files for placeholder names before saying they are gone", "read the newest requirements first" and "do not add rules that no requirement states" (shown under Entry 8 below). Assumptions I still must confirm with the team: only one APPROVED refund per payment, a rejected payment counts in the velocity rule, and whether the SYSTEM wallet may go below zero when it pays a refund. Decisions D-13, D-14 and D-18 are marked "team to confirm" in the requirements |
| 2026-11-25 | Abdelrahman Ahmed Ashry | Design - align names and enum values with the updated ER | Claude (Sonnet 5.5) | Structured prompt: role (architect keeping a design pack consistent with the ER), context (the newest `er.puml` of Mariam Mohamed Ali, the final requirements pack and my design files), constraints (do not change the ER; change only my files; the class diagram follows the ER names and enum values; add what the ER has and my class diagram lacks; do not guess the values of `top_ups.status`; list every difference that remains), output = edited files, re-rendered images and a script that compares class attributes with ER columns. Full text in "Full prompts" below | `LedgerEntryType.TOPUP` and `CREDIT`, `HoldStatus.CAPTURED` with `captureHold`, `DifferenceKind` with the three values of the ER, `BillerCategory` and `Biller.category`, `IdempotencyRecord.storedResponse`, a glossary with the known differences split into closed, accepted and open, the architecture and README updated, and all images re-rendered | Fixed. Real AI errors found on review: (1) the AI had told me in the chat that the ledger, `biller_account_ref`, `failure_reason` and `settlement_record_id` were still open in the ER; the newest `er.puml` already had them (it has `available_delta` and `held_delta`, and the columns are nullable), so those chat lists were out of date, and I found it by reading the file again; (2) an earlier AI check said that all enums match, but it only compared the enums that exist in my class diagram, so it missed `BillerCategory` (the ER has `billers.category`) and listed 3 enum differences when there were 5; (3) the glossary said that everything else follows the mapping rule, but a script that compares every class attribute with the ER columns found 6 more different names (`LedgerEntry.type`, `FeeRule.type`, `FeeRule.minimumFee`, `AuditLog.actor`, `ReconciliationDifference.kind` and `requiredFields`); they are now listed as accepted differences. Trade-offs I must confirm with the team: `CREDIT` exists in the class diagram only because the ER lists it and no flow writes it; with the three `DifferenceKind` values of the ER, two cases are not covered in Phase 1 (a `COMPLETED` payment missing from the file, and a file line unknown to PayLink); `storedResponse` is a String that holds the first response; `TopUp.status` values are still not defined in the ER. I also reordered the class diagram layout with hidden links only (a script confirmed that no class or relationship changed). Rendered with PlantUML 1.2025.4 only (largest image 2912x3729 px) |

## Full prompts

### Entry 1 - State diagram

```
Role: You are a senior software architect who writes PlantUML.

Context: PayLink is a bill-payment and wallet backend. Every payment is in
exactly one state. I need the payment life-cycle state diagram.

Constraints:
- Exactly these 8 states: INITIATED, FUNDS_HELD, SENT_TO_BILLER, COMPLETED,
  FAILED, UNKNOWN, REJECTED, REFUNDED.
- Exactly these 9 transitions, no others:
  INITIATED -> FUNDS_HELD (fraud checks pass and wallet has enough funds)
  INITIATED -> REJECTED (fraud check fails or funds insufficient)
  FUNDS_HELD -> SENT_TO_BILLER (request sent to biller)
  SENT_TO_BILLER -> COMPLETED (biller confirms)
  SENT_TO_BILLER -> FAILED (biller refuses)
  SENT_TO_BILLER -> UNKNOWN (no answer within timeout)
  UNKNOWN -> COMPLETED (biller later confirms)
  UNKNOWN -> FAILED (biller later refuses, or reconciliation finds no payment)
  FAILED -> REFUNDED (hold released to the wallet)
- Final states: COMPLETED, REJECTED, REFUNDED.
- UNKNOWN is never refunded automatically.

Output format: a single PlantUML file only, no explanation.
```

### Entry 2 - Architecture

The request was given in the chat; this is the same request written as a structured prompt.

```
Role: You are a senior software architect.

Context: PayLink is a bill-payment and wallet backend (Java 21, Spring Boot 3.x,
PostgreSQL). Modules: accounts, catalog, payments, billers, fees, notifications,
fraud, reconciliation. Billers: ElectroCo (REST), AquaCo (SOAP, slow), TopUpNet
(async callback), NetCo (daily CSV). In Phase 2 ElectroCo changes its API (new
field names, amounts in piasters, signed requests) and must be absorbed by changing
only the ElectroCo integration.

Constraints:
- Hexagonal modular monolith: domain, ports, adapters.
- Only `payments` orchestrates the pay flow; no module reads another module's tables.
- No compile-time cycles between modules.
- At least 5 design patterns, mapped to modules.
- Show how the 8 hard cases are supported by the design.
- Money is never float or double.

Output format: one Markdown file `architecture.md` with sections: style and reasons,
modules and owned tables, public interfaces, dependency rules, pattern mapping,
hard-case mapping, package layout, Phase 2 evolution, open decisions.
```

### Entry 3 - Class diagram

The request was given in the chat; this is the same request written as a structured prompt.

```
Role: You are a senior software architect who writes PlantUML.

Context: Domain class diagram for PayLink. Use the entity names from
docs/design/glossary.md and the modules from docs/design/architecture.md.

Constraints:
- Wallet has availableBalance() and heldBalance() computed from LedgerEntry (FR-5);
  no stored balance field.
- Money is a `Money` value object (BigDecimal), never float or double.
- Payment has reference, idempotencyKey, amount, feeAmount, totalAmount, status.
- PaymentStatus has exactly the 8 states; one State class per state.
- Interfaces for the patterns: FeeStrategy, PaymentState, BillerAdapter, FraudRule,
  NotificationObserver, RetryCommand.
- Customer and Agent are roles of Account, not separate classes.
- Show multiplicities.

Output format: a single PlantUML file only, no explanation.
```

### Entry 4 - Sequence diagram: pay a bill

The request was given in the chat; this is the same request written as a structured prompt.

```
Role: You are a senior software architect who writes PlantUML.

Context: PayLink pay-a-bill flow (brief section 2.2). Use the participant and
method names of class-diagram.puml and the biller-answer table in
architecture.md section 4.1.

Constraints:
- Phases: bill inquiry, pay, notification.
- Every payment state change must be one of the 9 allowed transitions.
- Show: repeated idempotency key, fraud rejection, insufficient funds, and the
  four biller answers CONFIRMED, REFUSED, PENDING (TopUpNet callback, duplicate
  ignored) and UNKNOWN (10 s timeout, retries after 1, 5 and 15 minutes, never
  refunded automatically).
- Notifications are sent from an event and never block the payment.
- Only `payments` orchestrates; only `accounts` writes the ledger.

Output format: a single PlantUML file only, no explanation.
```

Improved prompt (after the warning on plantuml.com): the same prompt with one more constraint, in the Constraints list:

```
- Do not use skinparam settings that newer PlantUML versions ignore
  (for example ParticipantPadding and BoxPadding).
```

### Entry 5 - Sequence diagram: refund a failed payment

The request was given in the chat; this is the same request written as a structured prompt.

```
Role: You are a senior software architect who writes PlantUML.

Context: PayLink refund of a FAILED payment (brief section 2.5: FAILED ->
REFUNDED, hold released to the wallet). Use the participant and method names
of class-diagram.puml and architecture.md section 4.1.

Constraints:
- Show the three ways a payment becomes FAILED: biller refuses, UNKNOWN payment
  later refused (retry or callback), UNKNOWN payment not found at reconciliation.
- Only a FAILED payment can be refunded. UNKNOWN is never refunded automatically.
- releaseHold writes a HOLD_RELEASE ledger entry and an audit record; releasing
  twice must not credit twice. The whole total (amount + fee) returns; no
  FEE_INCOME and no COMMISSION is posted.
- Notification must not change the result; a failed SMS is retried 3 times.
- Hard case 7: a settlement file listing a FAILED payment is only flagged for
  support, nothing changes automatically.
- Do not use skinparam settings that newer PlantUML versions ignore.

Output format: a single PlantUML file only, no explanation.
```

### Entry 6 - Review against the team requirements

The request was given in the chat; this is the same request written as a structured prompt.

```
Role: You are a reviewer checking a student design pack against the team's
requirements document.

Context: The repository contains docs/requirements.md (user stories US-01 to
US-12, NFR-1 to NFR-6, questions Q-01 to Q-10 and decisions D-01 to D-10),
pom.xml, Dockerfile, docker-compose.yml and the CI workflow. The design pack is
glossary.md, architecture.md, stack-decision.md, payment-state.puml,
class-diagram.puml, seq-pay-bill.puml and seq-refund-failed.puml.

Constraints:
- For every user story, NFR and decision, say whether the design supports it.
- Compare names, fields, tables, packages and versions across all files.
- Report real conflicts only, with the file and the fix. Do not invent errors.
- Do not invent fields for entities that are not in the repository.
- Separate what I can fix in my files from what the team must decide.

Output format: a list of conflicts, then the edits made, then the open items.
```

### Entry 7 - Update after the ER and use cases

The request was given in the chat; this is the same request written as a structured prompt.

```
Role: You are a software architect keeping a design pack consistent.

Context: Mariam Mohamed Ali added an ER diagram and three use-case diagrams. My files are
glossary.md, class-diagram.puml, architecture.md, seq-pay-bill.puml and
seq-refund-failed.puml.

Constraints:
- Change only my files. Do not edit the ER or the use cases.
- Add the entities that exist in the ER but not in the class diagram:
  RefundRequest, LimitRule, TopUp, BillerCallback, RequeryAttempt, SettlementFile.
- SYSTEM is a wallet type, not a role (decision D-18).
- FeeRule belongs to a biller and has the tiered fields.
- The block status moves into Account; no module may write another module's table.
- The use case "Follow up UNKNOWN payment manually" needs a method on PaymentService.
- Do not decide the open points (ledger fields, enum values, table names). List
  them as known differences.
- Add a code-to-database naming rule to the glossary.

Output format: the edited files, and a script result that checks names across files.
```

### Entry 8 - Update after the final requirements pack

The request was given in the chat; this is the same request written as a structured prompt.

```
Role: You are a software architect keeping a design pack consistent with its
requirements.

Context: The final Phase 1 requirements pack (US-01 to US-28, NFR-01 to NFR-06,
D-01 to D-18) is attached. My files are glossary.md, architecture.md,
class-diagram.puml, seq-pay-bill.puml, seq-refund-failed.puml and README.md.

Constraints:
- Read the newest requirements first. Where two versions disagree, the final pack wins.
- Change only my files. Do not edit the ER, the use cases or the requirements.
- Refund after COMPLETED (US-27, D-12, D-13, D-14, D-18): add requestRefund,
  approveRefund and rejectRefund to PaymentService and reverseCompleted to
  WalletService. The payment status does not change. Draw one sequence diagram.
- AuditLog is append-only (D-17, US-25.2). A LedgerEntry is never edited or deleted.
- Velocity rule (D-16): the 6th payment in 60 seconds is rejected and the account
  is blocked for 10 minutes. The numbers are LimitRule rows.
- Replace the placeholder role names (student numbers) with the real names.
- Search all files for placeholder names before saying they are gone.
- Do not add rules that no requirement states. List every assumption.
- Do not decide the ledger fields, the enum values or whether the SYSTEM wallet
  may go below zero. List them as open.
- Use only PlantUML settings that older and newer versions both support.

Output format: the edited files, the new .puml file, and a script result that
checks names and method signatures across all files.
```

### Entry 9 - Align names and enum values with the updated ER

The request was given in the chat; this is the same request written as a structured prompt.

```
Role: You are a software architect keeping a class diagram, a glossary and an
architecture document consistent with an ER diagram that another person owns.

Context: The newest er.puml, the final Phase 1 requirements pack (US-01 to US-28,
NFR-01 to NFR-06, D-01 to D-18) and my files (glossary.md, architecture.md,
class-diagram.puml, seq-pay-bill.puml, seq-refund-failed.puml, README.md,
prompt-log.md) are attached.

Constraints:
- Do not change the ER, the use cases or the requirements. Change only my files.
- The class diagram follows the ER for names and enum values:
  TOPUP and CREDIT, CAPTURED (and captureHold), the three DifferenceKind values,
  Biller.category with an enum BillerCategory, stored_response.
- Do not guess the values of top_ups.status. List them as open.
- Compare every class attribute with the ER columns with a script, not by eye.
  List each difference that remains and say if it is accepted or open.
- Search all files for the old names before saying they are gone.
- Do not decide whether the SYSTEM wallet may go below zero.
- Use only PlantUML settings that older and newer versions both support.

Output format: the edited files, re-rendered PNG and SVG images, and the result of
the comparison script.
```

### Follow-up prompt (used for entries 2 and 3)

The request was given in the chat; this is the same request written as a structured prompt.

```
Role: You are a reviewer checking a student design pack against its assignment brief.

Context: architecture.md, class-diagram.puml, payment-state.puml, glossary.md and
the assignment brief (PDF) are attached. Phase 1 is due on 25 November.

Constraints:
- Check every file against the brief (sections 2.2, 2.4, 2.5, 2.6, 2.7, 3, 4).
- Check the files against each other: interface signatures, entity names, enums,
  module and package names must be identical.
- Open the exported images in full before saying they are fine.
- Report real problems only. Do not invent errors.

Output format: a list of problems with the file, the fix, and what is still missing.
```

### Prompts of Mariam - ER diagram and use-case diagram (PlantUML)

Written with Claude's help. The two prompts below are the structured versions (Role, Context, Constraints, Output format, Example).

**ER diagram prompt**

```
Role: You are a senior database architect who writes clean PlantUML.

Context:
PayLink is a bill-payment and wallet backend (Java Spring Boot, PostgreSQL, Flyway migrations).
Customers pay utility bills from a wallet. Balances are NEVER stored: they are computed
from LedgerEntry records. The team's glossary is the single source of names. Attached: the
project brief, glossary.md and our requirements document (US-01 to US-28).

Task: Draw the ER diagram of the database as ONE PlantUML file.

Naming rules:
- Table names: snake_case plural (ledger_entries). Column names: snake_case.
- Money columns: NUMERIC(19,2). Never float or double. Currency is EGP only.
- Payment states (exactly 8): INITIATED, FUNDS_HELD, SENT_TO_BILLER, COMPLETED, FAILED,
  UNKNOWN, REJECTED, REFUNDED.
- Every table has an id primary key (marked PK) and foreign keys are marked FK.

Tables and key columns (use ALL of them, 20 tables):
1. accounts: role (CUSTOMER, AGENT, ADMIN_USER, SUPPORT_USER), verification_level (BASIC,
   VERIFIED), name, mobile (unique), email (unique), password_hash, status (ACTIVE, BLOCKED),
   blocked_until (nullable), created_at
2. wallets: account_id FK (nullable, because the PayLink system wallet has no account),
   wallet_type (USER, SYSTEM), currency. NO balance columns.
3. ledger_entries: wallet_id FK, payment_id FK (nullable), refund_request_id FK (nullable),
   entry_type (TOP_UP, HOLD, RELEASE, DEBIT, FEE_INCOME, COMMISSION, REFUND, REVERSAL),
   direction (CREDIT, DEBIT), amount, reference, created_at. Append-only.
4. holds: wallet_id FK, payment_id FK, amount, status (ACTIVE, RELEASED, CAPTURED),
   created_at, released_at
5. billers: name, code, protocol (REST_JSON, SOAP_XML, ASYNC_REST, BATCH_CSV), active
6. biller_services: biller_id FK, name, category (ELECTRICITY, WATER, MOBILE, INTERNET),
   min_amount, max_amount, active
7. service_input_fields: biller_service_id FK, field_name, label, required
8. payments: reference (unique), customer_account_id FK, agent_account_id FK (nullable),
   biller_service_id FK, amount, fee_amount, total_amount, status (the 8 states),
   query_attempts, next_query_at, created_at, updated_at
9. payment_status_history: payment_id FK, from_status, to_status, reason, changed_at
10. idempotency_records: account_id FK, idempotency_key, request_hash, payment_id FK, result,
    created_at, expires_at. Unique on (account_id, idempotency_key).
11. fee_rules: biller_service_id FK, fee_type (FIXED, PERCENTAGE, TIERED), fixed_amount,
    percentage, min_fee, tier_threshold, fee_below_threshold, fee_above_threshold, active
12. promotions: biller_id FK, start_date, end_date, active (fee is 0 for this biller in this period)
13. limit_rules: verification_level, min_payment, max_payment, daily_limit,
    velocity_max_payments, velocity_window_seconds, velocity_block_minutes,
    updated_by FK, updated_at
14. commissions: payment_id FK, agent_account_id FK, amount, status (CREDITED, REVERSED), created_at
15. notifications: account_id FK, payment_id FK (nullable), channel (SMS, EMAIL, PUSH),
    template, status (PENDING, SENT, UNDELIVERED), attempts, last_attempt_at, created_at
16. refund_requests: payment_id FK, requested_by FK, reason, status (PENDING, APPROVED,
    REJECTED), decided_by FK (nullable), decided_at, decision_reason, created_at
17. settlement_records: biller_id FK, settlement_date, biller_reference, amount, file_name
18. reconciliation_reports: biller_id FK, report_date, matched_count, difference_count, created_at
19. reconciliation_items: reconciliation_report_id FK, settlement_record_id FK,
    payment_id FK (nullable), difference_type, reviewed_by FK (nullable), reviewed_at
20. audit_logs: actor_account_id FK, action, entity_type, entity_id, reason, created_at.
    Append-only.

Relationships (draw every one with cardinality):
- accounts 1..0..1 wallets (an account has one wallet; the SYSTEM wallet has none)
- wallets 1..* ledger_entries; wallets 1..* holds
- payments 1..* ledger_entries; payments 1..* holds; payments 1..* payment_status_history
- payments 1..0..1 commissions; payments 1..* notifications; payments 1..* refund_requests
- accounts 1..* payments (as customer); accounts 0..* payments (as agent)
- billers 1..* biller_services; biller_services 1..* service_input_fields
- biller_services 1..* payments; biller_services 1..* fee_rules; billers 1..* promotions
- billers 1..* settlement_records; billers 1..* reconciliation_reports
- reconciliation_reports 1..* reconciliation_items
- settlement_records 1..* reconciliation_items
- accounts 1..* idempotency_records; accounts 1..* audit_logs

Constraints:
- Use ONLY the tables, columns and states above. Do not invent extra tables.
- Do not put availableBalance or heldBalance columns anywhere (they are computed from ledger_entries).
- Group tables with PlantUML packages named after the modules: accounts, catalog, payments,
  fees, notifications, fraud, reconciliation, billers.
- Valid PlantUML only (entity ... { * pk : type, fk : type }), must render without errors.

Output format: ONE plantuml code block with @startuml ... @enduml, nothing else, no explanation.

Example of the expected style:
@startuml
entity wallets {
  * id : bigint <<PK>>
  --
  account_id : bigint <<FK>>
  wallet_type : varchar
}
entity ledger_entries {
  * id : bigint <<PK>>
  --
  wallet_id : bigint <<FK>>
  amount : numeric(19,2)
}
wallets ||--o{ ledger_entries
@enduml
```

**Use-case diagram prompt**

```
Role: You are a senior business analyst who writes clean PlantUML.

Context:
PayLink is a bill-payment and wallet backend. Attached: the brief, glossary.md and our
requirements document with user stories US-01 to US-28. Use names from the glossary.

Task: Draw the use-case diagram as ONE PlantUML file.

Actors:
- Customer, Agent, AdminUser, SupportUser (primary, left side)
- ElectroCo, AquaCo, TopUpNet (external biller systems, right side; NetCo is optional, leave it out)
- Scheduler (system actor for timed jobs, right side)

Use cases (label each with its story id), grouped in packages named after the modules:
- accounts: Register (US-01), Upgrade to VERIFIED (US-02), View Wallet Balance and History
  (US-03), Top Up Wallet (US-04), Block and Unblock Account (US-21), Request Refund (US-27),
  Approve or Reject Refund (US-27)
- catalog: Manage Billers and Services (US-06), Browse Billers and Required Fields (US-07)
- payments: Bill Inquiry (US-08), Pay a Bill (US-09), Prevent Duplicate Payment (US-10),
  Track Payment Life Cycle (US-11), Hold and Release Funds (US-12), Search Payment by
  Reference (US-20), Pay on Behalf of Customer (US-17)
- billers: Handle Biller Callback (US-14), Re-query UNKNOWN Payment (US-13)
- fees: Calculate Fee (US-16), Credit Agent Commission (US-17), Edit Fees and Limits (US-24)
- fraud: Check Limits (US-18), Check Velocity (US-19)
- notifications: Send Notification (US-22), Retry Notification (US-23)
- reconciliation: Reconcile Settlement File (US-26)
- Audit Log (US-25) and View Daily Totals (US-28) as use cases in packages "audit" and "admin".

Relationships:
- Customer: Register, Upgrade to VERIFIED, View Wallet Balance and History, Top Up Wallet,
  Browse Billers, Bill Inquiry, Pay a Bill, Request Refund
- Agent: Pay on Behalf of Customer (it is a specialization of Pay a Bill), Top Up Wallet (agent cash)
- AdminUser: Manage Billers and Services, Edit Fees and Limits, View Daily Totals
- SupportUser: Search Payment by Reference, Block and Unblock Account, Approve or Reject
  Refund, review the reconciliation report
- <<include>> from Pay a Bill: Check Limits, Check Velocity, Calculate Fee, Hold and Release
  Funds, Prevent Duplicate Payment, Send Notification
- <<include>> from Bill Inquiry: Calculate Fee
- <<extend>> Re-query UNKNOWN Payment extends Pay a Bill (when the biller times out)
- <<include>> from every money-moving use case and every admin action: Audit Log
- Billers: ElectroCo, AquaCo, TopUpNet connect to Bill Inquiry and Pay a Bill; TopUpNet also
  connects to Handle Biller Callback
- Scheduler: Re-query UNKNOWN Payment, Retry Notification, Reconcile Settlement File

Constraints:
- Use ONLY the actors and use cases above. Use exact glossary names (AdminUser, SupportUser,
  UNKNOWN in upper case).
- Valid PlantUML (left to right direction, rectangle "PayLink System"), must render without errors.

Output format: ONE plantuml code block with @startuml ... @enduml, nothing else, no explanation.

Example of the expected style:
@startuml
left to right direction
actor Customer
rectangle "PayLink System" {
  usecase "Pay a Bill (US-09)" as UC9
  usecase "Check Limits (US-18)" as UC18
}
Customer --> UC9
UC9 .> UC18 : <<include>>
@enduml
```

## Prompt template

```
Role: You are a senior software architect.
Context: <what PayLink is, the part of the brief that matters, the glossary names>
Constraints: <exact states, transitions, naming, tool (PlantUML/Mermaid), what must not appear>
Output format: <one .puml file, no explanation>
Example: <a small example of the expected style>
```
