# JomJimat --- Technical Stack and Architecture

**Document status:** Draft baseline\
**Version:** 1.0\
**Last updated:** 4 November 2026\
**Application:** JomJimat\
**Target platforms:** Android and iOS

------------------------------------------------------------------------

## 1. Purpose

This document defines how JomJimat should be built: application stack,
architecture, data ownership, Firebase services, financial integrity,
security, testing and release workflow.

The product requirements and user-facing behaviour are defined in
`PROJECT_SPEC.md`.

## 2. Technology decisions

  -----------------------------------------------------------------------
  Area                    Selected technology     Purpose
  ----------------------- ----------------------- -----------------------
  Language                TypeScript              Typed application code
                                                  and domain models

  Mobile framework        React Native            Cross-platform native
                                                  mobile UI

  App platform/tooling    Expo                    Development, native
                                                  builds and app
                                                  lifecycle

  Navigation              Expo Router             File-based navigation
                                                  and route organization

  Backend platform        Firebase                Authentication,
                                                  database, file storage
                                                  and server workflows

  Authentication          Firebase Authentication Email/password and
                                                  Google sign-in

  Database                Cloud Firestore         User-owned financial
                                                  records and app data

  File storage            Firebase Storage        Expense/refund receipt
                                                  images and profile
                                                  pictures

  Backend functions       Cloud Functions for     Trusted workflows and
                          Firebase                scheduled recurring
                                                  generation

  Scheduled jobs          Cloud Scheduler via     Daily recurring
                          scheduled Cloud         transaction processing
                          Functions               

  Monitoring              Firebase Crashlytics    Production crash/error
                          and/or an appropriate   visibility
                          error-monitoring        
                          service                 

  Testing                 Jest or                 Unit, component,
                          Vitest-compatible       integration and
                          setup, React Native     security tests
                          Testing Library,        
                          Firebase Emulator Suite 

  Builds                  EAS Build               Android/iOS
                                                  development, preview
                                                  and production builds
  -----------------------------------------------------------------------

### Expo SDK baseline

The project was created with the Expo Go learning path using SDK 57.
Keep the current project SDK as the starting baseline and verify
supported package versions before upgrading. Do not independently
upgrade React Native, Expo or native dependencies without checking Expo
compatibility.

### Firebase SDK approach

Start with the Firebase JavaScript SDK for Expo Go compatibility. Use
Firebase Authentication, Cloud Firestore and Firebase Storage through
supported Expo-compatible APIs.

React Native Firebase is not the default for the initial Expo Go
workflow because it requires native configuration and a development
build. Reconsider it only if a specific native Firebase capability
becomes necessary.

## 3. Architecture overview

Use a layered architecture that keeps financial rules independent of
screens and Firebase implementation details.

``` text
React Native UI (Expo Router)
        |
        v
Presentation / View Models
        |
        v
Application Use Cases
        |
        v
Domain Layer
  - Money
  - Accounts
  - Transactions
  - Transfers
  - Refunds
  - Budgets
  - Recurring rules
        |
        v
Repository Interfaces
        |
        v
Firebase Data Layer
  - Firebase Authentication
  - Cloud Firestore
  - Firebase Storage
  - Cloud Functions
```

### Layer responsibilities

**Presentation** - Screens, components, navigation and user
interactions. - Formatting and displaying values. - Loading, empty,
success and error states. - No authoritative financial calculations
embedded in UI components.

**Application** - Use cases such as CreateExpense,
CompletePendingTransaction, TransferBetweenAccounts, CreateRefund and
DeleteAccount. - Coordinates validation and repository operations. -
Exposes results and errors to the presentation layer.

**Domain** - Financial entities and business rules. - Integer-sen money
representation. - Balance, budget, transfer, refund and recurring
calculations. - Must be testable without Firebase or React Native.

**Data** - Firestore repositories and document mapping. - Firebase
Storage upload/download handling. - Authentication integration. - Cloud
Function calls. - Index/query definitions and persistence concerns.

## 4. Suggested project organization

Use a feature-oriented structure while keeping shared domain logic
centralized.

``` text
JomJimatRN/
├── app/
│   ├── (auth)/
│   ├── (tabs)/
│   ├── transactions/
│   ├── accounts/
│   ├── budgets/
│   └── more/
├── src/
│   ├── app/
│   │   ├── providers/
│   │   └── navigation/
│   ├── features/
│   │   ├── authentication/
│   │   ├── dashboard/
│   │   ├── transactions/
│   │   ├── accounts/
│   │   ├── budgets/
│   │   ├── categories/
│   │   ├── recurring/
│   │   ├── reports/
│   │   ├── notifications/
│   │   └── profile/
│   ├── domain/
│   │   ├── money/
│   │   ├── account/
│   │   ├── transaction/
│   │   ├── budget/
│   │   └── recurring/
│   ├── data/
│   │   ├── firebase/
│   │   ├── repositories/
│   │   └── mappers/
│   ├── shared/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── theme/
│   │   ├── formatting/
│   │   ├── validation/
│   │   └── utilities/
│   └── config/
├── functions/
│   └── src/
├── assets/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── security/
├── app.config.ts
├── package.json
├── tsconfig.json
├── TECH_STACK.md
└── PROJECT_SPEC.md
```

The structure is a guide, not a requirement to create every directory
immediately. Add folders as features are implemented.

## 5. Financial money representation

### 5.1 Integer sen

Use integer sen as the authoritative internal representation of MYR.

Examples:

  User-facing amount     Stored integer
  -------------------- ----------------
  RM0.00                              0
  RM0.01                              1
  RM12.50                          1250
  RM1,250.00                     125000

Users enter and see normal decimal currency. Integer sen is an internal
implementation detail.

### 5.2 Why

Binary floating-point cannot represent every decimal fraction exactly.
Integer arithmetic avoids the common precision drift caused by repeated
decimal operations.

### 5.3 Implementation rules

-   Define a `Money` domain type whose amount is an integer number of
    sen.
-   Parse user-entered decimal strings into integer sen using validated
    decimal parsing; do not use floating-point multiplication as the
    authoritative conversion.
-   Store amounts in Firestore as integer values.
-   Format values at the UI boundary, e.g. `1250` -\> `RM12.50`.
-   Never expose raw sen values in normal UI.
-   Avoid using JavaScript `number` for values outside its safe integer
    range; validate against `Number.MAX_SAFE_INTEGER` or use a suitable
    integer representation if scale requires it.
-   Define and test rounding rules for percentages, ratios and
    divisions.
-   For category splits, require the sum of integer-sen allocations to
    equal the transaction total exactly.
-   CSV export converts integer sen to decimal MYR text with exactly two
    fractional digits.

### 5.4 Rounding

Use an explicit, centralized rounding policy. For operations that
produce fractional sen, round to the nearest sen using a documented
tie-breaking rule. Do not let individual screens or features choose
their own rounding behaviour.

## 6. Financial ledger and derived balances

### 6.1 Source of truth

The canonical financial records are the source of truth. Account
balances, dashboard totals and budget summaries are derived/optimized
data.

Derived summaries are allowed for performance but must be reproducible
from canonical records.

### 6.2 Account balance model

Conceptually:

``` text
Current account balance
  = opening balance
  + completed financial movements
  + completed balance adjustments
```

The sign and effect of each movement depend on the account's accounting
behaviour and transaction direction.

Examples: - Expense from cash/bank asset account: decreases the asset
balance. - Income to cash/bank asset account: increases the asset
balance. - Credit card purchase: increases the liability (represented by
a more negative account balance in the chosen display convention). -
Card repayment: decreases cash/bank and moves the negative card balance
toward zero. - Transfer: decreases source and increases destination
consistently. - Pending transaction: no effect until completion.

Implement a single domain-level calculation policy and test it
thoroughly. Do not duplicate sign logic across screens.

### 6.3 Atomic writes

Use Firestore transactions or batched writes for operations that update
related records. Use a transaction when the operation needs to read
current state before writing.

Examples: - Transfer creation/edit/deletion. - Completing a pending
transaction. - Creating or editing a refund and its derived summaries. -
Updating canonical records and related summary documents.

Firestore transaction limits and retry behaviour must be respected. Keep
transactions bounded and avoid external side effects inside retried
transaction callbacks.

### 6.4 Reconciliation

Provide a trusted way to rebuild or verify derived balances and
summaries from canonical records. This may be an admin/backend job or a
controlled maintenance function; it is not a normal user-facing action
in MVP.

## 7. Firebase data model

The following is a logical starting schema. Final collection names and
document fields should be reviewed during implementation, but ownership
and financial invariants are mandatory.

### 7.1 User namespace

A recommended structure is:

``` text
users/{uid}
  profile fields
  preferences
  createdAt
  schemaVersion

users/{uid}/accounts/{accountId}
users/{uid}/accountTypes/{accountTypeId}
users/{uid}/categories/{categoryId}
users/{uid}/transactions/{transactionId}
users/{uid}/transfers/{transferId}
users/{uid}/refunds/{refundId}
users/{uid}/recurringTemplates/{templateId}
users/{uid}/budgets/{budgetId}
users/{uid}/notifications/{notificationId}
users/{uid}/accountOrder/{orderId}
```

Subcollections provide a clear ownership boundary. Backend functions
must validate the authenticated UID and never trust a UID supplied only
by the client.

### 7.2 User profile and preferences

Possible fields: - `displayName` - `email` - `photoURL` - `currency`
(fixed to MYR in MVP) - `themeMode` (`light` or `dark`) -
`accentPaletteId` - `hideMonetaryValues` - `schemaVersion` -
`createdAt` - `updatedAt`

### 7.3 Account document

Possible fields: - `name` - `accountTypeId` - `accountingBehaviour`
(`asset` or `liability`) - `openingBalanceSen` (integer) -
`cachedBalanceSen` (derived integer, optional) - `icon` - `colour` -
`description` - `sortOrder` - `status` (`active` or `deleted`) -
`createdAt` - `updatedAt` - `deletedAt`

Accounting behaviour is immutable after account creation. A soft-deleted
account remains readable in historical views but cannot be selected for
new records.

### 7.4 Category document

Possible fields: - `name` - `type` (`income` or `expense`) - `icon` -
`colour` - `sortOrder` - `status` (`active` or `deleted`) -
`createdAt` - `updatedAt` - `deletedAt`

Do not overwrite historical category names in existing records when a
category is renamed or deleted. Store a category name snapshot on
financial records where necessary for historical integrity.

### 7.5 Transaction document

Possible fields: - `type` (`income`, `expense`, `adjustment`) - `status`
(`pending`, `completed`) - `amountSen` (integer) - `accountId` -
`accountNameSnapshot` - `categoryAllocations`: array of
`{ categoryId, categoryNameSnapshot, amountSen }` - `date` (calendar
date representation) - `notes` - `receiptStoragePath` -
`recurringTemplateId` (optional) - `createdAt` - `updatedAt` -
`idempotencyKey` (where applicable)

For a non-split transaction, category allocations contain one item. For
split expenses, allocations sum exactly to `amountSen`.

Do not use a transfer as an income/expense transaction. Keep transfer
semantics explicit.

### 7.6 Transfer document

Possible fields: - `sourceAccountId` - `sourceAccountNameSnapshot` -
`destinationAccountId` - `destinationAccountNameSnapshot` -
`amountSen` - `date` - `notes` - `status` - `createdAt` - `updatedAt` -
`idempotencyKey`

A transfer has one logical identity and one timeline representation. Its
effects on both accounts must be applied consistently. If the
implementation uses separate ledger legs internally, both legs must
share a transfer ID and be committed atomically.

### 7.7 Refund document

Possible fields: - `originalExpenseId` - `accountId` -
`accountNameSnapshot` - `amountSen` - `date` - `notes` -
`receiptStoragePath` - `createdAt` - `updatedAt`

Refunds are independent records so they can remain accessible if the
original expense is deleted. While the original expense exists, its
detail view queries and displays linked refunds.

### 7.8 Recurring template document

Possible fields: - `type` (`income` or `expense`) - `amountSen` -
`accountId` - `categoryAllocations` - `notes` - `intervalUnit` (`day`,
`week`, `month`) - `intervalCount` - `anchorDate` -
`nextOccurrenceDate` - `status` (`active`, `paused`, `deleted`) -
`createdAt` - `updatedAt`

Store enough scheduling information to preserve month-end behaviour and
calculate future occurrences consistently.

### 7.9 Recurring execution/idempotency record

Use a unique occurrence identity derived from the recurring template ID
and scheduled calendar date. Record execution state so retries cannot
create a duplicate transaction for the same occurrence.

### 7.10 Budget document

Possible fields: - `period` (calendar month, e.g. `2026-11`) -
`overallLimitSen` - `categoryLimits`: map/list of category IDs to
integer-sen limits and name snapshots - `createdAt` - `updatedAt`

Historical budget documents are immutable snapshots of that month's
limits/progress context, except for intentional edits to that month's
budget.

### 7.11 Notification document

Possible fields: - `type` (`recurringGenerated`, `budgetThreshold`) -
`title` - `message` - `relatedRecordId` - `createdAt`

No read/unread state or notification management actions are required in
MVP.

## 8. Date and recurring schedule handling

-   Store transaction dates as calendar dates, not arbitrary local
    timestamps, where the business meaning is date-only.
-   Store audit timestamps such as `createdAt` and `updatedAt` as server
    timestamps.
-   The scheduled recurring job must use one documented canonical
    timezone for date evaluation. For a Malaysia-first MVP, use the
    Asia/Kuala_Lumpur calendar as the default unless product
    requirements change.
-   Generate one occurrence per template/date.
-   A daily job queries due active templates, determines the scheduled
    occurrence, writes the generated transaction and execution record
    idempotently, then advances the template.
-   Missed occurrences are skipped rather than backfilled.
-   Do not run external side effects inside Firestore transaction
    callbacks that may retry.

## 9. Firebase Authentication

Support: - Email/password. - Google sign-in. - Password reset email. -
No mandatory email verification in MVP.

Use supported Firebase Auth persistence for the Expo/React Native
environment. Handle secure token/session persistence using the
recommended supported Firebase configuration for the chosen Expo SDK. Do
not store passwords or raw tokens in application logs or ordinary
AsyncStorage.

Protect sensitive operations such as permanent account deletion with
recent authentication/reauthentication.

## 10. Firebase Security Rules and ownership

Security is a core requirement, not a later enhancement.

-   Every user-owned record is scoped to an authenticated UID.
-   Firestore rules must verify that the authenticated user can access
    only their own user namespace.
-   Storage rules must enforce the same owner boundary for receipt and
    profile images.
-   Do not rely on hiding UI elements or checking ownership only in
    client code.
-   Validate allowed fields, types and immutable properties where
    practical in rules and trusted backend code.
-   Prevent clients from changing immutable accounting behaviour or
    trusted execution metadata.
-   Cloud Functions must validate authentication/authorization and input
    data.
-   Never expose service account credentials in the client.

Test security rules with at least two separate test users and negative
access cases.

## 11. Receipt and profile image storage

-   Store receipt images in Firebase Storage under user-scoped paths,
    for example `users/{uid}/receipts/{receiptId}`.
-   Store the Storage path and relevant metadata in Firestore, not the
    raw image bytes.
-   Compress images before upload.
-   Enforce reasonable file-size and image-type limits.
-   Restrict read/write access to the owning user.
-   Delete associated files during permanent account deletion.
-   Avoid storing unnecessary image metadata.

## 12. Search and Firestore query design

Firestore is not a general-purpose full-text search engine.

For MVP: - Support bounded keyword search over description/notes using
normalized search tokens or prefix tokens. - Support filters for
account, category, type and date range. - Paginate transaction
history. - Create only the composite indexes required by actual
supported query combinations. - Document limitations such as token
matching and prefix behaviour; do not promise arbitrary fuzzy/full-text
search. - If search requirements outgrow Firestore's capabilities,
evaluate a dedicated search service as a later enhancement.

Avoid downloading the entire transaction history to filter it locally.

## 13. Derived summaries and budgets

-   Keep canonical transactions separate from derived dashboard summary
    documents.
-   Update derived summaries consistently with canonical writes, using
    transactions/batches or trusted backend processing as appropriate.
-   Summaries may include account balances, monthly income/expense
    totals, category totals and budget progress.
-   Pending transactions must not be included in finalized summaries.
-   Transfers and balance adjustments must not be counted as
    income/expenses.
-   Refunds reduce net spending according to the product rules.
-   Provide a reconciliation/rebuild mechanism to recover from summary
    drift.
-   Never treat a cached summary as more authoritative than the ledger.

## 14. Account deletion workflow

Permanent deletion is a trusted backend workflow:

1.  Client requires explicit confirmation.
2.  Client requires recent authentication/reauthentication.
3.  Backend verifies caller identity and starts a deletion job.
4.  Backend deletes user-owned Firestore records and subcollections in
    bounded batches.
5.  Backend deletes user-owned Storage objects.
6.  Backend removes the Firebase Authentication user.
7.  Workflow records progress safely enough to retry after partial
    failure.
8.  Repeated requests are idempotent and do not recreate or expose data.

Do not attempt to delete an unbounded amount of user data in one
oversized Firestore batch. Handle subcollections recursively or through
a controlled deletion process.

## 15. Error handling and idempotency

-   Prevent duplicate submissions from repeated Save taps.
-   Use idempotency keys for operations that may be retried.
-   Use deterministic occurrence IDs for recurring generation.
-   Display user-friendly errors without leaking internal details.
-   Provide retry options for transient network failures.
-   Keep financial writes atomic where multiple records are logically
    one operation.
-   Distinguish validation errors, permission errors, connectivity
    errors and unexpected server errors.
-   Log diagnostic metadata without logging sensitive financial
    contents.

## 16. State management and data fetching

Choose one consistent state-management approach and apply it across
features. A lightweight context/hooks approach may be sufficient
initially; use a dedicated state library only when shared state
complexity warrants it.

Recommended separation: - Authentication/session state. - User
preferences. - Server data/repository state. - Temporary screen/form
state.

Do not make UI state the source of truth for balances. Refresh or
observe server-backed records and derived summaries after successful
writes.

## 17. UI system and accessibility

-   Centralize design tokens for spacing, typography, radius, elevation
    and semantic colours.
-   Support light and dark themes.
-   Provide predefined accent palettes.
-   Preserve semantic financial colour meaning across palettes.
-   Global hide/show monetary values must apply consistently.
-   Use accessible labels, readable typography, adequate contrast and
    touch targets.
-   Use animations sparingly and purposefully.
-   Keep transaction-entry interactions optimized for frequent manual
    use.

## 18. Testing strategy

### 18.1 Unit tests

Test pure domain logic without Firebase:

-   Money parsing and formatting.
-   Integer-sen arithmetic.
-   Rounding.
-   Account balance effects.
-   Net financial position.
-   Transfers.
-   Credit card purchases and repayments.
-   Split allocation validation.
-   Pending/completed behaviour.
-   Refund calculations and edge cases.
-   Monthly budget calculations.
-   Recurring schedule date calculations.

### 18.2 Integration tests

Use Firebase Emulator Suite where possible to test:

-   Authentication flows.
-   Firestore repository operations.
-   Atomic transfers.
-   Completing pending transactions.
-   Refund creation/edit/deletion.
-   Recurring generation and idempotency.
-   Receipt upload and access.
-   Account deletion workflow.

### 18.3 Security tests

Test: - User A cannot read User B's records. - User A cannot write to
User B's records. - User A cannot read User B's Storage objects. -
Unauthenticated clients cannot access protected data. - Clients cannot
alter immutable or backend-controlled fields.

### 18.4 Financial scenario tests

At minimum, test: - Expense reduces the correct account balance. -
Income increases the correct account balance. - Transfer updates both
accounts and is not counted as income/expense. - Credit card purchase
increases expense and card debt. - Card repayment reduces cash/bank and
card debt without double-counting expense. - Pending transactions do not
affect finalized summaries. - Split allocations equal the total and flow
into the correct budgets. - Partial refunds reduce net spending. -
Deleted categories/accounts preserve historical associations. - Original
expense deletion does not orphan inaccessible refunds. - Derived
balances can be reconstructed from the ledger.

### 18.5 Recurring tests

Test: - Every N days/weeks/months. - Monthly occurrence on the 31st
across shorter months. - Leap-year February. - Weekend/public-holiday
scheduled dates. - Missed occurrence skipping. - Retry and duplicate
prevention. - Paused/deleted templates. - Template edits affecting
future occurrences only.

### 18.6 Platform testing

Test on Android and iOS devices/simulators. Validate permission flows
for camera/gallery, keyboard behaviour, safe areas, navigation and theme
rendering.

## 19. Environments and configuration

Maintain separate Firebase projects/environments:

-   Development
-   Production

Use environment-specific configuration and never point development
builds at production data by default.

-   Keep secrets and service account credentials out of source control.
-   Use Expo/EAS environment variables and secure CI configuration where
    appropriate.
-   Add `.env.example` with variable names and placeholder values only.
-   Restrict production access and permissions.
-   Use Firebase Emulator Suite for local development and automated
    tests where practical.

## 20. Development workflow

Recommended implementation order:

1.  Initialize and verify the Expo/TypeScript app.
2.  Establish navigation, theme tokens and shared UI primitives.
3.  Configure development Firebase project and authentication.
4.  Implement domain money types and unit tests.
5.  Implement account/category data models and repositories.
6.  Implement transaction ledger and financial calculations.
7.  Implement transfers, credit cards, pending records and refunds.
8.  Implement dashboard summaries and reconciliation.
9.  Implement budgets and monthly rollover/copy rules.
10. Implement recurring templates and scheduled backend generation.
11. Implement reports, search, notifications and CSV export.
12. Implement receipt storage and account deletion.
13. Complete security rules, integration tests and platform testing.
14. Prepare internal beta builds.
15. Configure production Firebase and release pipeline.

Build in vertical slices: complete one user journey end-to-end before
expanding to the next. Avoid generating the entire app in one opaque
code dump.

## 21. Build and release

-   Use Expo Go for early UI and compatible Firebase JS SDK development.
-   Use EAS development builds when native configuration or native-only
    capabilities are required.
-   Use preview/internal builds for testing.
-   Test on both Android and iOS before release.
-   Keep development and production Firebase environments separate.
-   Add crash/error monitoring before public production release.
-   Review Firebase quotas, security rules, indexes and billing before
    launch.
-   Do not ship with permissive development security rules.

## 22. Technical risks and constraints

1.  **Firestore search:** Native Firestore queries are not full-text
    search. MVP search must have explicitly documented token/prefix
    semantics.
2.  **Financial correctness:** All money must use integer-sen arithmetic
    and centralized rounding rules.
3.  **Derived data drift:** Cached balances/summaries require
    reconciliation from canonical records.
4.  **Atomicity:** Transfers and other linked financial operations must
    not partially apply.
5.  **Recurring jobs:** Scheduled functions may retry; deterministic
    occurrence IDs and idempotency are mandatory.
6.  **Deletion:** Permanent deletion spans Firestore, Storage and
    Firebase Auth and must support safe retries.
7.  **Expo compatibility:** Verify Firebase and native package
    compatibility with the project's Expo SDK before upgrades.
8.  **Shared ledger:** Not in MVP. Keep user ownership clear and avoid
    prematurely implementing multi-user permission workflows.

## 23. Definition of done

A feature is not complete merely because its screen renders. It is
complete when:

-   The user journey works end-to-end.
-   Domain rules are implemented outside presentation components.
-   Financial values use integer sen.
-   Loading, empty, success and error states exist.
-   Data ownership and security rules are enforced.
-   Relevant unit and integration tests pass.
-   Android and iOS behaviour is verified where applicable.
-   Documentation is updated when the schema or business rules change.
