# Product

<!-- impeccable:product-schema 1 -->

## Platform

adaptive

Android and iOS from one React Native codebase.

## Users

An individual managing their own day-to-day money — cash, bank accounts, e-wallets and credit cards —
who wants one complete picture instead of four apps and a spreadsheet. Malaysian market (confirmed);
thinks and spends in Malaysian Ringgit.

They record spending themselves, at the point of spend or in a later batch, on a phone, usually
one-handed. They are not accountants and do not want to configure an accounting system: they want
balances, monthly spending and budget progress to be obvious at a glance, and to trust that the
numbers are right.

Single owner per account. Household/shared ledgers are out of scope for the MVP.

## Product Purpose

JomJimat brings accounts, income, expenses, transfers, refunds, budgets and basic summaries together
so the user can see and manage their overall financial position.

Success is the user knowing, without digging, what they have, what they spent this month, and whether
they are inside their budget — and keeping the habit of recording because it stays cheap to do.

The experience must feel approachable and clear rather than intimidating, and encourage consistent
habits without demanding configuration.

## Positioning

JomJimat treats the awkward parts of personal finance as first-class, correct semantics instead of
squeezing them into "income" and "expense":

- a transfer is **one** timeline entry and never counted as income or expense;
- a credit card repayment is a transfer, not a second expense; a card purchase is one expense that
  grows the liability;
- a refund is a record linked to its original expense, so partial refunds reduce net spending and
  restore the corresponding category's budget capacity;
- pending records exist but are excluded from finalized balances, budget consumption and summaries
  until completed;
- deleting an account or category is soft: historical records keep the original name and association,
  and nothing is silently reassigned;
- money is stored as integer sen, and every balance and summary is reproducible from the canonical
  ledger.

MYR is a first-class citizen, not a formatting afterthought: `RM12.50`, `RM1,250.00`, `RM0.00`, with
thousand separators and exactly two decimals everywhere.

## Operating Context

- **Manual-first tracking.** The user enters transactions themselves; the transaction-entry screen is
  the most-used surface in the app and must be fast for frequent, one-handed, keypad-driven entry.
- **Transaction entry layout is prescribed** (see Evidence on Hand): type selector centred at the top,
  full category icon grid directly below, amount with a calculator-style keypad at the bottom,
  remark/notes and camera action near the amount, account and date as compact chips. The owner supplied
  a reference screenshot and the layout follows it.
- **Reference-only understanding of the categories.** Default categories exist and are user-editable
  (icon, colour, name, order, delete). A category may be preselected from the most recent transaction;
  otherwise none is selected. Pending records may carry a future date (they plan forward); completed
  records may not.
- **Currency:** MYR only. **Language:** English only, for the MVP.
- **Books.** Accounts belong to a book — `Personal`, `Business`, or others the user names — and every
  record is entered against an account inside the active book. A book is a real grouping with its own
  account membership, not a label carried on each account.
- **Connectivity:** online is required to view or manage data; clear connection errors and retry paths.
- **In-app only messaging.** Budget-threshold and recurring-generation notices live in a chronological
  in-app notification list. No push, no email.
- **Display preferences are global and persistent:** light/dark theme, predefined accent palettes, and
  a hide/show monetary-values switch that applies everywhere and survives restarts.
- **Manual calendar-month budgets.** New months copy the previous month's limits, are independently
  editable, and unused budget does not roll over.
- **Fixed home dashboard.** Six sections in a set order, not user-customisable, and structurally present
  (with zero/empty values plus setup guidance) before the user has any data.

## Capabilities and Constraints

**In the MVP:** email/password and Google auth, password reset; books that group accounts; multi-account
management with cash,
bank, e-wallet, credit-card and custom asset/liability account types; opening balances and manual
balance adjustments; income, expense, transfer and refund records; pending and completed states;
category management and split expenses; receipt attachments; monthly category budgets plus an overall
monthly cap; recurring templates with automatic generation and upcoming previews; home dashboard,
transaction history with search and filters, reports and CSV export; profile and settings; self-service
permanent account deletion.

**Deliberately deferred:** shared/household ledgers and settlement flows; savings goals; AI or advanced
rule-based insights; gamification, streaks, rewards and mascot; a web application; offline access;
push and email notifications; investment/loan/full net-worth tracking; credit limits and
available-credit calculations.

**Technical constraints that shape design:** Expo + React Native + Expo Router; Firebase
(Auth, Firestore, Storage, Cloud Functions); money as integer sen with one centralised rounding policy;
transfers and other linked financial writes are atomic; Firestore has no general-purpose full-text
search, so search is bounded token/prefix matching and must not promise fuzzy matching; transaction
history is paginated.

**Open implementation notes** (from the spec, not yet decided in code): how standalone refunds stay
discoverable once their original expense is deleted; the documented timezone for recurring schedule
evaluation; reconciliation of derived balances.

## Brand Commitments

- **Name: JomJimat — final** (confirmed by the owner).
- **Tone:** friendly, approachable and encouraging, never childish. Modern minimal with playful
  touches. No mascot in the MVP.
- **Shared, not per-OS:** one custom visual language across Android and iOS (confirmed by the owner);
  platform conventions only where the OS enforces them — safe areas, back gesture, keyboard, permissions.
- **Binding layout commitment:** the transaction-entry structure in the supplied reference screenshot.
- **Visual direction: settled convention, executed at full fidelity** (owner's choice, taken over the
  rolled directions). A conventional money-entry screen with no irony and no smuggled quirk; craft is
  the differentiator. Its reference set — the two or three products it should sit alongside — is
  recorded with the surface brief once the owner names them.
- No logo, wordmark, illustration system, typeface or palette has been committed to yet.

## Evidence on Hand

- `PROJECT_SPEC.md` — product specification v1.0: scope, rules, acceptance criteria, decision register
  (Q1–Q178).
- `TECH_STACK.md` — technical stack and architecture baseline: layers, Firestore model, money
  representation, security, testing, release.
- `assets/references/mumu-jizhang-transaction-entry.jpg` — the owner-supplied reference for the
  transaction-entry screen layout (a third-party app screenshot, used for structure only).
- **Absences future work must not fabricate:** no real JomJimat screens exist yet — `src/app` is still
  the create-expo-app demo; no logo or brand asset; no testimonials, press, user counts, benchmarks,
  pricing or deployment claims.

## Product Principles

1. **Clarity first.** Balances, spending and budget progress are legible at a glance; if a number needs
   explanation to be understood, it is not finished.
2. **Accurate records outrank convenience.** Transfers, refunds, pending items and card debt are handled
   with correct semantics, and the ledger can always be replayed into every derived figure.
3. **Low friction, explicit money choices.** Defaults carry the routine cases (today's date, last-used
   account); anything that changes the financial meaning — category, split, source/destination — is
   chosen on purpose.
4. **History is immutable in spirit.** Renaming or deleting an account or category never rewrites what
   already happened.
5. **Private by default.** One owner per set of records, enforced in the data layer rather than the UI.

## Accessibility & Inclusion

Readable typography, adequate contrast, accessible touch targets and screen-reader labels are required
across the app. Financial semantic colours (positive, negative, warning) must stay distinguishable in
both light and dark themes and across every accent palette. The global hide/show monetary-values
preference exists partly for privacy in public and must apply consistently.

No formal accessibility standard or specific assistive-technology requirement has been confirmed yet.
