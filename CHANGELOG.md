# CHANGELOG - تعديلات الـ design pack (25 نوفمبر 2026)

## اللي اتعمل (بالرقم بتاع الـ review)

1. صور الـ class diagram (png/svg/pdf) اتعملها render من جديد من `class-diagram.puml` الحالي (render محلي، من غير cropping).
2. **MinMaxRule**: اتضافت في الـ fraud chain (class diagram + architecture 4.3)، وفي `seq-pay-bill.puml` فيه فرع REJECTED واضح للمبلغ أقل من min أو أكتر من max. الـ min/max مصدرهم واحد بس: `biller_services.min_amount/max_amount`، واتشالوا MIN_PAYMENT/MAX_PAYMENT من `limit_rules` ومن `LimitRuleName`. متسجل في الـ glossary.
3. `Account` في الـ class diagram بقى فيه fullName, mobileNumber, email, passwordHash (وكمان createdAt).
4. AuditLog: الـ ER note والـ architecture بيقولوا نفس الكلام: append-only في الكود + revoke UPDATE/DELETE في الـ DB كقاعدة deployment.
5. ملفات الـ use case اتسمّت `use-case-1-customer-agent`, `use-case-2-admin-support`, `use-case-3-pay-a-bill` (puml + png + svg). صور WhatsApp اتمسحت. الـ README اتحدث.
6. الـ prompt-log: 5 entries جديدة (10 لـ 14) لشغلك: الـ ERD، مشكلة الـ crop في الـ render، تقسيم الـ use case، تنضيف الـ class diagram، والـ review والتعديلات. فيهم أخطاء حقيقية اتصلحت وprompts منظمة.
7. الـ ER: الرصيد = مجموع available_delta / held_delta من الـ ledger (US-05.2)، واتشال كلام الـ ACTIVE holds.
8. `top_ups.status` = PENDING | SUCCESS | FAILED في الـ ER، و`TopUpStatus` enum في الـ class diagram، واتقفل من الـ open في الـ glossary والـ README والـ architecture.
9. ملف جديد `seq-unknown-confirmed.puml`: UNKNOWN بيتأكد COMPLETED (re-query / callback / settlement file / support) مع captureHold وقيود DEBIT و FEE_INCOME و COMMISSION. اتضاف في الـ README.
10. الـ architecture: اتضافت الـ interfaces الناقصة (register/upgrade، topUp/history، manage billers، findByReference، FeeRuleAdmin/LimitRuleAdmin، dailyTotals) وبرضه في الـ class diagram.
11. الـ architecture: فقرة NFR-02/03 (thread pool منفصل لكل biller adapter + الـ wallet row lock هو النقطة الوحيدة المتسلسلة).
12. `seq-pay-bill.puml`: الترتيب المخترع للـ fraud rules اتشال، والقواعد متكتبة من غير ترتيب.
15. use case 3: بدل مربع "Fraud Checks" بقى فيه 4 includes منفصلة (Daily Limit, Velocity, Blocked Account, Min and Max). و UC16 بقى بيعمل include لـ Write Audit Log و Record Ledger Entry. اتعمل render.
16. prompt-log entry 3: اتصلحت الجملة اللي بتقول إن FraudBlock و Role.SYSTEM اتضافوا للـ glossary.
17. attributes اتضافت في الـ class: Wallet.currency, Hold.createdAt/releasedAt, Promotion.feeOverride, Commission.createdAt. **Hold.currency ماتضافتش** لأن الـ ER مفيهوش العمود ده (اتكتبت في الـ glossary).
18. قاعدة الـ daily limit: بيتحسب `FUNDS_HELD, SENT_TO_BILLER, UNKNOWN, COMPLETED` في نفس اليوم. الـ REJECTED و FAILED و REFUNDED ما بيتحسبوش (افتراض، الفريق يأكده). في architecture 4.3.

## اتحدثت بسبب التعديلات
ملفات الـ PDF بتاعة README / architecture / glossary / prompt-log اتعملت من جديد. الـ ERD اتعمل له render جديد: `img/er.png`, `er.svg`, `er.pdf` (بدل `paylink_erd-663416ed.pdf`).

## لسه مش اتعمل (للفريق)
- قرارات الفريق: D-09, D-10, D-13, D-14, D-15, D-17, D-18، وهل الـ SYSTEM wallet ممكن يبقى بالسالب (NFR-01).
- الـ requirements PDF القسم 8 صف 4 (RefundRequest لسه مكتوب open) - ده بتاع اللي كتب الـ requirements.
- `pom.xml` و `ci.yml` مش في الـ zip، فنسخة Spring Boot متراجعتش.
- الـ class diagram لسه كثيف في النص (روابط كتير على Payment) لأن الـ layout أوتوماتيك. الـ packages والـ ortho lines محتفظ بيهم.
