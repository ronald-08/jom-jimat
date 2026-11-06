# JomJimat --- Product Specification

**Document status:** Draft baseline\
**Version:** 1.0\
**Last updated:** 4 November 2026\
**Product:** JomJimat\
**Platforms:** Android and iOS (mobile-first)\
**Primary currency:** Malaysian Ringgit (MYR)

------------------------------------------------------------------------

## 1. Purpose of this document

This document defines what JomJimat should do in its MVP: product goals,
scope, user journeys, screens, business rules and acceptance criteria.

It is the product-level reference for design, implementation and
testing. Technical implementation choices belong in `TECH_STACK.md`.

### Decision status

-   **Confirmed:** Decisions explicitly selected during product
    discovery.
-   **Recommended baseline:** Implementation and product defaults
    adopted to close the discovery process. These can be revisited if
    implementation reveals a material issue.
-   **Deferred:** Intentionally outside the MVP.

## 2. Product vision

JomJimat helps an individual understand and manage their overall
financial wellness by bringing their accounts, income, expenses,
transfers, budgets and basic financial summaries together.

The experience should feel approachable and clear rather than
intimidating. It should encourage consistent financial habits without
requiring users to configure a complicated accounting system.

### Primary user outcome

> Get a complete picture of my finances in one place.

### Product principles

1.  **Clarity first:** Make balances, spending and budget progress easy
    to understand.
2.  **Manual-first tracking:** Users intentionally record and categorize
    financial activity.
3.  **Accurate financial records:** Transfers, refunds, pending
    transactions and credit card debt must be handled consistently.
4.  **Low-friction workflows:** Use sensible defaults while keeping
    important financial choices explicit.
5.  **Friendly, not childish:** Modern minimal design with playful
    touches; no mascot in the MVP.
6.  **Private by default:** Each user's financial data is private to
    that authenticated user.
7.  **Historical integrity:** Deleting or changing an account/category
    must not silently rewrite historical records.

## 3. Target user and primary use cases

### Target user

An individual who wants to track day-to-day money across cash, bank
accounts, e-wallets and credit cards, understand monthly spending, and
stay within a budget.

### Core use cases

-   Set up opening balances for multiple accounts.
-   Record income, expenses and transfers.
-   Track credit card debt and repayments.
-   Categorize and split expenses.
-   Record partial refunds against original expenses.
-   Create monthly category budgets and an overall spending limit.
-   Review balances, spending and budget progress on a dashboard.
-   Find historical transactions and export records.
-   Set up recurring transactions.
-   Keep data synchronized across Android and iOS.

## 4. MVP scope

### 4.1 Included

-   Email/password registration and sign-in.
-   Google sign-in.
-   Password reset.
-   Multi-account management.
-   Cash, bank account, e-wallet and credit card account types.
-   Custom account types with Asset or Liability behaviour.
-   Opening balances and manual balance adjustments.
-   Income, expense, transfer and refund records.
-   Pending and completed transaction statuses.
-   Category management and category splits.
-   Receipt attachments for expenses and refunds.
-   Monthly category budgets and an overall monthly spending cap.
-   Recurring transaction templates, automatic generation and upcoming
    previews.
-   Home dashboard, transaction history, reports and in-app
    notifications.
-   Search and transaction filters.
-   CSV export.
-   Light/dark themes, accent colour palettes and hide/show monetary
    values.
-   Cloud synchronization between Android and iOS.
-   Self-service permanent account deletion.

### 4.2 Explicitly deferred

-   Shared ledgers, household accounts, shared wallets, invitations and
    collaborative editing.
-   Shared expenses and settlement workflows.
-   Savings goals.
-   AI-generated or advanced rule-based financial insights.
-   Gamification, streaks, rewards and mascot.
-   Web application.
-   Offline access to financial data.
-   Push notifications and email notifications.
-   Investment portfolio, loan and full net-worth asset tracking.
-   Credit limits and available-credit calculations.
-   Biometric app lock.
-   Transaction templates and smart suggestions.
-   Advanced audit-trail UI.
-   Multiple currencies and currency conversion.

Shared ledger is a post-MVP possibility. The MVP remains single-user and
must enforce per-user data ownership.

## 5. Navigation and information architecture

### 5.1 Bottom navigation

The app has five primary tabs:

1.  **Home**
2.  **Transactions**
3.  **Budgets**
4.  **Accounts**
5.  **More**

### 5.2 More section

More contains:

-   Reports
-   Recurring Transactions
-   Categories
-   Notifications
-   Profile and Settings

### 5.3 Global action

A prominent floating action button (+) is available from the main app
experience. It opens a quick menu with:

-   Expense
-   Income
-   Transfer

The selected action opens the corresponding transaction-entry flow.

## 6. Onboarding and authentication

### 6.1 Registration and sign-in

-   Support email/password registration and sign-in.
-   Support Google sign-in.
-   Email/password users can request a password reset email.
-   Email verification is not required for MVP access.
-   Google sign-in follows Google's authentication flow.
-   Users enter the Home screen after successful registration/sign-in.

### 6.2 Initial account setup

-   Account setup is not mandatory before entering Home.
-   Home shows a persistent setup card prompting the user to create
    their first account.
-   The setup card disappears after at least one active account exists.
-   The user must have an active account before saving their first
    income or expense transaction.
-   Do not populate demo financial data.

### 6.3 Empty Home state

Show the normal dashboard structure with zero/empty values, a friendly
welcome message and a prominent **Create first account** action. Avoid
making the empty state look like an error.

## 7. Accounts

### 7.1 Built-in account types

-   Cash
-   Bank Account
-   E-wallet
-   Credit Card

Users can also create custom account types such as Savings Account,
Investment Account or Digital Wallet.

### 7.2 Accounting behaviour

Every account type has one accounting behaviour:

-   **Asset**
-   **Liability**

Built-in credit card accounts use Liability behaviour. Cash, bank and
e-wallet accounts use Asset behaviour.

A custom account type's accounting behaviour is selected at creation and
cannot later be changed. Account type changes are allowed only between
types with the same accounting behaviour.

Changing account type affects classification and presentation only. It
must not rewrite opening balances, transaction history or financial
calculations.

### 7.3 Account fields and customization

Users can create and edit:

-   Account name
-   Account type
-   Opening balance
-   Icon
-   Colour
-   Description

Account names are independent of account type and can be fully
customized. Custom account type names, icons and colours can be edited.
A custom account type can only be deleted when it is not in use.

### 7.4 Account balances

-   Users enter an opening balance when creating an account.
-   Credit card opening balances are entered as negative amounts.
-   Credit card debt is displayed as a negative balance.
-   Asset and liability classifications are used in the net financial
    position calculation.
-   Users can manually adjust an account balance by entering the actual
    balance. The adjustment must create a distinct adjustment record
    rather than silently overwriting the balance.

### 7.5 Account ordering

-   Display accounts in one unified list.
-   Users can drag and drop to reorder accounts.
-   Account ordering synchronizes across devices.

### 7.6 Account deletion

Account deletion is a soft-delete operation:

-   Remove the account from active account lists and new transaction
    selectors.
-   Retain the account and its historical records internally.
-   Preserve the original account name and show a subtle **Deleted**
    label in historical views.
-   Do not offer user restore in the MVP.
-   Do not cascade-delete transactions, refunds or transfers.
-   A deleted account cannot be selected for new transactions.
-   Deletion is blocked while recurring templates reference the account.
    The user must reassign or delete those templates first.

Existing records linked to a deleted account remain accessible:

-   Pending transactions remain linked, but must be reassigned to an
    active account before completion.
-   Existing transfers remain unchanged.
-   Transfers can be edited while retaining the deleted account or
    optionally reassigning it.
-   Deleting a transfer deletes the linked transfer operation
    consistently.
-   Ordinary income/expense records can be edited while retaining or
    optionally reassigning the deleted account.
-   Refunds can remain linked to the deleted account or be reassigned to
    an active account.

## 8. Transactions

### 8.1 Supported transaction types

-   Expense
-   Income
-   Transfer
-   Refund (linked to an original expense)
-   Balance adjustment (system-generated record from a manual
    adjustment)

Transfers and balance adjustments are not ordinary income or expenses.

### 8.2 Transaction fields

A normal income or expense contains:

-   Amount
-   Type
-   Category
-   Account
-   Date
-   Optional notes
-   Optional receipt attachment for expenses

Transactions also retain creation and update timestamps. Detail views
show relevant linked transfer, recurring or refund information.

### 8.3 Entry experience

The transaction-entry screen follows the user-provided reference layout:

-   Transaction type selector centered at the top (Expense / Income /
    Transfer).
-   Full category icon grid immediately below for income/expense entry.
-   Calculator-style keypad at the bottom.
-   Remark/notes field and camera action near the amount.
-   Account and date details accessible from the entry screen.

Additional behaviour:

-   Amount input supports MYR with two decimal places, thousand
    separators and arithmetic expressions.
-   Users enter familiar decimal values such as `12.50`.
-   No category is preselected; users explicitly choose one.
-   Date defaults to today.
-   Users may backdate transactions but cannot manually select a future
    date.
-   Income/expense defaults to the last-used account; users may change
    it.
-   Transfers require separately selected source and destination
    accounts.
-   The user can split one transaction across multiple categories.
-   Split allocations must equal the transaction total before saving.
-   Saving shows success confirmation, returns to the previous screen
    and refreshes relevant summaries.

### 8.4 Pending and completed transactions

-   Users can create pending or completed transactions.
-   Pending transactions are excluded from finalized balances, budget
    consumption and financial summaries.
-   Users manually mark pending transactions as completed.
-   A pending transaction linked to a deleted account cannot be
    completed until reassigned to an active account.

### 8.5 Transaction history and management

-   Show a combined chronological timeline.
-   Transfers appear as one combined timeline item, not two unrelated
    entries.
-   Support filters for account, category, type and date range.
-   Support keyword search over descriptions/notes.
-   Users can view, edit and delete transactions.
-   Confirm destructive actions.
-   No trash/recovery feature in MVP.

### 8.6 Deleted categories in transaction history

-   Soft-deleted categories are excluded from new transaction choices.
-   Historical transactions retain their original category association
    and original category name.
-   When editing a transaction with a deleted category, retain that
    association unless the user explicitly reassigns it to an active
    category.
-   Reports continue to display the original category name for
    historical records.

## 9. Transfers and credit cards

### 9.1 Transfers

-   Transfers move value between accounts and are not income or
    expenses.
-   Both sides of a transfer must be applied consistently or neither
    side should be applied.
-   Transfers between compatible account types are supported.
-   A transfer to a credit card account is treated as a card repayment.
-   Credit-card-originating transfers are allowed, including cash
    advances and card-to-card transfers.
-   A transfer is displayed as one combined history entry with source
    and destination details.

### 9.2 Credit card purchases

-   Record a purchase directly as an expense against the credit card
    account.
-   The expense contributes to spending and category budgets.
-   The credit card liability balance becomes more negative.

### 9.3 Credit card repayments

-   Record repayments as transfers from a cash/bank/e-wallet account to
    the credit card account.
-   Reduce the source account balance.
-   Move the credit card's negative balance toward zero.
-   Do not count repayment as a second expense.

### 9.4 Interest and fees

Cash advance fees, interest and similar charges are manually recorded as
ordinary expenses under an appropriate category such as Financial
Charges.

## 10. Refunds

-   Refunds are dedicated records linked to original expenses.
-   Multiple partial refunds are supported.
-   Refunds reduce net spending and restore the corresponding category
    budget capacity.
-   Refunds can have optional receipt attachments.
-   Refunds can be edited independently (amount, date and notes).
-   Deleting a refund requires confirmation and recalculates affected
    summaries.
-   Deleting the original expense does not delete its refunds.
-   Refunds appear within the original expense detail while the original
    expense exists.
-   If the original expense is deleted, its refunds remain independently
    discoverable as historical records and in relevant reports.

### Refund amount policy --- recommended baseline

-   When creating a new refund, the combined refunds should not exceed
    the original expense's current amount.
-   Existing refunds are preserved if the original expense is later
    edited to a lower amount.
-   Editing an existing refund is allowed even if the combined refunds
    exceed the edited expense amount.
-   No automatic deletion, reduction or reassignment of existing refunds
    occurs when the original expense changes.

## 11. Categories

-   Provide predefined default categories.
-   Users can create custom categories.
-   Users can rename, delete and reorder categories.
-   Users can customize category icons and colours.
-   Users can delete predefined categories as well as custom categories.
-   Category deletion is soft deletion.
-   Deleted categories no longer appear in new transaction choices.
-   Historical transactions and reports retain the deleted category's
    original identity and name.
-   Do not silently reassign historical transactions.

## 12. Budgets

### 12.1 Budget types

Support:

-   Monthly category budgets.
-   One overall monthly spending cap.

### 12.2 Calendar-month behaviour

-   Each budget month begins on the 1st and ends on the month's final
    calendar day.
-   A new budget period is created for each calendar month.
-   Copy the previous month's overall and category budget limits into
    the new month.
-   The new month's limits are independently editable.
-   Preserve historical budgets and their spending/progress.
-   Unused budget amounts do not roll over.

### 12.3 Budget calculations

-   Category spending is based on completed expenses allocated to that
    category, minus linked refunds.
-   Split expenses allocate their respective amounts to each category
    budget.
-   Pending transactions are excluded.
-   Transfers, credit card repayments and balance adjustments are
    excluded.
-   The overall spending cap uses all completed expenses minus refunds,
    regardless of category.
-   Overspending is clearly highlighted.
-   Budget limits are manually configured; no automatic allocation
    suggestions.

### 12.4 Alerts

-   Default alert thresholds: 80% and 100%.
-   Users can configure thresholds.
-   Alerts are delivered in-app only.

## 13. Recurring transactions

### 13.1 Templates

Recurring templates support:

-   Income and expense schedules.
-   Custom intervals of every N days, weeks or months.
-   Account, category, amount and optional notes.
-   Pause, resume and delete actions.

Template edits affect future generated transactions only. Existing
generated transactions remain unchanged.

### 13.2 Schedule rules

-   Use calendar-date scheduling, not user-selected execution times.
-   A monthly schedule on the 31st uses the last day of a shorter month,
    then returns to the 31st when possible.
-   Execute on the scheduled date even if it falls on a weekend or
    public holiday.
-   Missed occurrences are skipped, not backfilled.
-   The backend checks schedules daily.
-   Generation must be idempotent to prevent duplicates.

### 13.3 Upcoming previews

-   Show upcoming scheduled transactions in the Recurring Transactions
    experience.
-   A preview is not a real transaction.
-   A preview does not affect balances, budgets, reports or transaction
    history.
-   Show active, paused and upcoming recurring information in the
    dedicated screen under More.

### 13.4 Notifications

Generate an in-app notification for each generated recurring
transaction, including its description, amount and account. Do not send
push or email notifications in MVP.

## 14. Home dashboard

The Home screen has a fixed layout with these sections:

1.  Net financial position.
2.  Assets and liabilities.
3.  Monthly income and expenses.
4.  Overall monthly budget progress.
5.  Individual account balances.
6.  Recent transactions.

The dashboard is not user-customizable in MVP. It should show the same
overall structure when the user has no financial data, using zero/empty
values and helpful setup guidance.

### Financial position

Net financial position is calculated as total assets minus total
liabilities. In MVP, account balances are the tracked financial
positions; the app does not attempt to track all investments, property
or loans.

## 15. Reports and export

### Reports

Support:

-   Current-month income and expenses.
-   Spending by category.
-   Monthly budget progress.
-   Account balances.
-   Flexible date ranges using predefined periods and a custom date
    range.

Do not include advanced historical analytics or AI insights in MVP.

### CSV export

Export transaction data with:

-   Date
-   Description/notes
-   Category
-   Account
-   Amount
-   Type

The file should be compatible with common spreadsheet applications such
as Excel and Google Sheets. CSV export is not a restorable backup.

## 16. Notifications

-   Provide a simple chronological in-app notification list.
-   No read/unread state is required in MVP.
-   No notification filters or management actions are required.
-   Include recurring-generation notifications and configured
    budget-threshold alerts.
-   No push notifications or email notifications.

## 17. Profile and settings

Profile supports:

-   View/edit display name.
-   View email.
-   Change profile picture.
-   Sign out.
-   Permanently delete account.

Settings support:

-   Light/dark theme selection.
-   Predefined accent colour palettes.
-   Global hide/show monetary values preference.

The hide/show preference applies throughout the app and is remembered
across sessions. Financial semantic colours must remain distinguishable
in both themes and across accent palettes.

## 18. Localization and visual design

-   MVP language: English.
-   MVP currency: MYR only.
-   Always display money with exactly two decimal places,
    e.g. `RM12.50`, `RM1,250.00`, `RM0.00`.
-   Modern minimal visual direction with playful touches.
-   Light and dark themes.
-   Customizable accent colour from predefined palettes.
-   No mascot in MVP.
-   Use readable typography, adequate contrast, accessible touch targets
    and screen-reader labels.

## 19. Money and calculation rules

### Storage and presentation

-   Store money internally as integer sen: RM12.50 is stored as `1250`.
-   Users enter and see normal decimal MYR amounts.
-   Never display integer sen as the primary currency representation.
-   Do not use binary floating-point arithmetic as the authoritative
    representation for money.
-   Define explicit rounding rules for percentages, division and
    allocations.
-   Validate numeric ranges and safe integer limits.

### Canonical financial record

The transaction ledger is the source of truth. Account balances and
dashboard summaries may be maintained as derived, optimized values, but
they must be reproducible from canonical records.

### Core calculation definitions

-   **Net financial position:** total asset balances minus total
    liability balances.
-   **Overall monthly spending:** completed expenses minus refunds.
-   **Category net spending:** completed expense allocations to the
    category minus applicable refunds.
-   **Pending records:** excluded from finalized financial calculations.
-   **Transfers:** excluded from income and expense totals.
-   **Card repayment:** transfer, not expense.
-   **Balance adjustment:** changes account balance through a distinct
    adjustment record, not income/expense.

## 20. Privacy, ownership and account deletion

-   Each user's records are private to that authenticated user.
-   Users cannot access another user's financial records or receipt
    attachments.
-   Enforce ownership on the backend/database security layer, not only
    in UI logic.
-   Permanent account deletion requires confirmation and recent
    authentication.
-   The deletion workflow must remove the user's authentication account
    and associated financial records, receipt files, recurring
    configurations and user-owned data.
-   The deletion workflow must handle retries and partial failures
    safely.

## 21. Non-functional requirements

-   Online connection is required to view and manage financial data.
-   Show clear connection errors and retry options.
-   Prevent duplicate writes from repeated user actions.
-   Show loading feedback for operations that take time.
-   Paginate transaction history.
-   Use database indexes appropriate to supported filters.
-   Compress receipt images and enforce size limits.
-   Maintain data consistency between canonical records and derived
    summaries.
-   Test both Android and iOS.
-   Add crash/error monitoring before production release.

## 22. Acceptance criteria by major journey

### A. First-time user

-   User can register with email/password or Google.
-   User reaches Home without being forced to create an account first.
-   Home clearly prompts the user to create the first account.
-   No sample financial data is displayed.
-   User cannot save an income/expense until an active account exists.

### B. Account and opening balance

-   User can create cash, bank, e-wallet or credit card accounts.
-   User can create custom account types with Asset or Liability
    behaviour.
-   Credit card opening balance is negative.
-   Account balance is reflected in the financial position.
-   User can reorder accounts.
-   Soft-deleted accounts disappear from new-entry selectors but remain
    visible in historical records.

### C. Expense and income

-   User can enter amount, category, account, date and optional notes.
-   Amount is displayed in MYR with two decimal places.
-   User must explicitly choose a category.
-   User can attach an expense receipt.
-   User can backdate but not future-date a manual transaction.
-   Saving updates the relevant account balance and dashboard summaries.

### D. Split expense

-   User can split one expense across multiple categories.
-   The sum of allocations must equal the total.
-   Each allocation contributes to its corresponding category budget.
-   The transaction is represented as one expense in the main timeline.

### E. Transfer and card repayment

-   User selects source and destination accounts.
-   Both account balances update consistently.
-   Transfer appears once in the combined timeline.
-   Transfer does not increase income or expenses.
-   A card repayment reduces cash/bank and moves the card liability
    toward zero without creating another expense.

### F. Pending transaction

-   Pending transaction is visible as pending.
-   Pending transaction does not affect finalized balances, budget
    progress or financial totals.
-   Completing it applies its financial effect once.
-   Completion against a deleted account requires reassignment.

### G. Refund

-   User can create a partial refund linked to an expense.
-   Refund reduces net spending and restores budget capacity.
-   Multiple partial refunds are supported.
-   Refund can have a receipt attachment.
-   Editing/deleting a refund recalculates affected summaries.
-   Deleting the original expense does not make its refunds
    inaccessible.

### H. Budgets

-   User can set category limits and an overall monthly limit.
-   Spending progress reflects completed expenses minus refunds.
-   Pending transactions and transfers are excluded.
-   Overspending is highlighted.
-   New month copies prior limits but has independent values and no
    spending rollover.

### I. Recurring transaction

-   User can create, pause, resume and delete a recurring template.
-   Scheduled generation runs without the app being open.
-   The same occurrence cannot be generated twice.
-   Missed occurrences are skipped.
-   Month-end schedule rules behave as specified.
-   Upcoming previews do not affect balances or budgets.

### J. Search, reports and export

-   User can search descriptions/notes and filter transactions.
-   User can select predefined or custom report date ranges.
-   CSV includes the agreed fields and opens in spreadsheet software.
-   Deleted account/category labels remain meaningful in historical
    records.

### K. Privacy and deletion

-   User A cannot read or modify User B's financial records.
-   User A cannot access User B's receipt files.
-   Permanent account deletion requires confirmation and recent
    authentication.
-   The deletion workflow removes associated user data and can recover
    safely from transient failures.

## 23. Open implementation notes and risks

These are implementation constraints, not reasons to expand MVP scope:

1.  Firestore does not provide general-purpose full-text search. The MVP
    search implementation must define a bounded token/prefix strategy or
    explicitly constrain search behaviour. A dedicated search service
    can be considered later if needed.
2.  Recurring schedules use calendar dates. The backend must document
    the chosen schedule evaluation timezone and ensure a single
    occurrence key per template/date.
3.  Derived balance/summary documents must be reconcilable against the
    canonical ledger.
4.  Account deletion and recurring generation are backend workflows and
    require idempotency and retry handling.
5.  When an original expense is deleted, standalone refunds need a clear
    UI location so they remain discoverable.
6.  Integer money storage still requires explicit rounding rules for
    division and percentage calculations.
7.  Shared ledger is deferred. Do not build multi-user membership or
    permission workflows in MVP.

## 24. Decision register reference

This specification consolidates product decisions Q1--Q132 and adopts
the recommended technical/product baseline Q133--Q178 from discovery.
Where a later clarification supersedes an earlier idea, this document
reflects the latest agreed direction.
