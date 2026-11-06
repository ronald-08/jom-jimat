# Transaction entry

Route: create, edit and transfer share this screen. Pushed (back chevron), full height.

## Job and audience

The most-used screen in the app. A Malaysian individual recording a spend at the point it happens,
one-handed, thumb already near the keypad, in daylight, in a queue, for the hundredth time this month.
They are not configuring anything; they are filing one fact and getting out.

Mode: **Operate**. Precision and speed outrank expression.

## Outcome and proof

Success is a saved record with a correct category, book, account and date in under ten seconds, without
reading a label twice. Proof: the amount is legible at arm's length, the grid answers "where does this
go" at a glance, and the screen is quiet enough that the amount is the loudest thing on it.

## Structure (confirmed against mocks, v1 → v2)

- **Top row:** back chevron · Expense / Income / Transfer segmented control · gear.
  The gear owns all category and subcategory management. No management card in the grid.
- **Category grid:** 3 per row, 42pt rows, icon + name, top-aligned, scrolls inside its own region.
  Personally ordered. A chevron appears **only** on categories that have subcategories.
  No category is preselected unless a previous transaction supplies one.
- **Middle:** deliberately empty. It is where the screen breathes; do not fill it.
- **Bottom block, right-aligned:** a 9pt label row (`PENDING · RECEIPT · SPLIT · RM x LEFT`), the amount
  with `RM`, the remark, then the keypad.
- **Keypad:** four flat rows, numerals only, no per-key cards. Operator column `+ − ⌫`. Bottom-right
  carries two commit actions side by side: `✓` saves and closes; the double-tick saves and opens a
  fresh entry. `✓` disables while a split is unbalanced.
- **Rail beside the keypad,** four chips aligned to the keypad rows: date · book · account · `⋯`.
  A book is a real grouping (Personal, Business); accounts belong to one.
- **The `⋯` sheet** holds pending, split and receipt. The control is a sheet; the *state* is the label
  row above the amount.
- **Transfer** swaps the grid for source/destination accounts; the rest of the screen is unchanged.
- **From the transaction list,** tapping a record opens a summary bottom sheet carrying **edit and
  delete only**; delete stays there, edit opens this screen.

## Scope and boundaries

In scope: create/edit/transfer, its states, the `⋯` sheet contents, the label row.
Out of scope: category management screens, the transaction list, and the list's summary sheet.

Anti-goals: dense chrome, boxed keypad cards, a visible camera button, a persistent options strip,
management actions inside the grid, decoration in the empty middle.

## States and ranges

At rest · category selected · subcategory chosen · pending set (future date) · receipt attached ·
split unbalanced (amber remaining, commit disabled) · split balanced · editing a record · transfer
mode. Realistic amount range RM0.50–RM5,000; a split of 2–4 categories is typical.

## Constraints and open decisions

- Expo/React Native, one shared visual language across iOS and Android; platform conventions only where
  the OS enforces them.
- The keypad owns the lower third on every supported device; a 667pt screen must still show four-plus
  rows of categories.
- The hide-values preference must not blank the amount being typed.
- **Open:** which of duplicate / save-as-recurring belong in `⋯` on create versus edit.
- **Open:** whether switching book mid-entry changes the account chip or resets it.

## Direction contract

**THESIS.** The screen is conventional on purpose: every control is one a Malaysian already uses in
their banking app, and the differentiator is finish, not novelty. It refuses the themed skeleton — a
default segmented control, a default keypad and default cards with no spacing system between them.

**OWN-WORLD.** White ground, `#F4F5F7` wells, near-black `#111827` ink, `#9CA3AF` secondary. One blue
`#2563EB`, and it is spent only on selection and commit; amber `#B45309` means pending, red only
means destructive. System UI face throughout, tabular figures for every amount. 12px radii, an 8pt
spacing rhythm, 1px `#E5E7EB` hairlines, and a shadow used once — on the selected card, never as decor.

**STORY.** The user arrives mid-task, recognises every control without reading, files the record, and
leaves with the number right. They should never be able to say what the app looks like — only that it
was quick.

**FIRST VIEWPORT.** 390×844. Back chevron, three-part segmented control centred, gear right, all on one
48pt row. Below it fifteen 44pt category plates, three per row, 8pt gaps, top-aligned, the selected one
tinted `#EFF4FF` with a blue hairline. At the foot: 10pt status labels flush right, the amount at
38/700 right-aligned, the remark under it, then four keypad rows with the date · book · account · ⋯
rail beside them, commit bottom-right.

**FORM.** The category standard — the standing exit, taken deliberately: no irony, no smuggled quirk.
Not on the ordered list; it is the door offered to every round. `seed: 7e3fd7e29e27d0d8`.

**FINISH.** unreviewed and undocumented is unfinished; this build ends with the finish review, the
verdict, DESIGN.md, and every shipping raster carrying its provenance.

### Reference set

To be named by the owner (two or three products this should sit alongside); their craft level is the
bar. Until named, the bar is the category's best: aligned grid, consistent spacing, real type scale,
clear hierarchy.
