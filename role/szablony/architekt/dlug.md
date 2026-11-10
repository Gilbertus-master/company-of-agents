# Technical debt register (agent {{rola}})

**Purpose.** One place for everything "we will fix later": shortcuts in code, flags, skipped tests, deferred reviewer remarks,
promises in PR descriptions. Recorded debt has an owner and a decision; unrecorded debt comes back as an incident. Source for controls D-01
and D-03 in `ceo/kontrole/kontrole.md`.

**Columns.** ID `DL-NN` (fixed in the order and in the repayment PR) · Where (repo / module / file, flag or test name) · Impact on
customer / pace / risk (for each: high, medium, low or `–`, plus one sentence on what exactly hurts) · Repayment cost
(S – one change in one PR · M – several PRs within a week · L – an item of the month's plan or a decision in `{{rola}}/adr/`) ·
Owner (role) · Review date (`RRRR-MM-DD`) · Decision: `spłacić do RRRR-MM-DD (zlecenie <ID>)` (repay by) or
`akceptujemy do RRRR-MM-DD – powód` (accepted until – reason).

**Rules.**
1. **Where entries come from:** reviewer remarks "not blocking – fix after merge" not closed within 72 h (D-02, `uwagi_recenzenta_h`) ·
   features behind a flag (D-03) · skipped tests with a reason and a deadline (D-04) · the oldest TODO and FIXME markers (D-05) ·
   promises "in a separate PR" without an order (D-06) · lessons from incidents (`incydenty/`).
2. **Review every 14 days** (`ceo/kontrole/progi.yaml → dlug_przeglad_dni`): the architect updates the "Review date" column
   in every open row. A row with an older date = D-01 failed.
3. **High impact = decision.** Every item with "high" impact on any of the three axes has a decision: repayment with a deadline
   or a deliberate deferral. An empty decision field with high impact = D-01 failed.
4. **One debt item in the weekly plan** – chosen by the CEO with the architect in `/przeglad-tygodnia` (impact ÷ repayment cost).
5. **"Accepted until" always has a date and a reason.** After the date the item goes back to decision. Acceptance of an item with high impact on
   the customer or risk is approved by the CEO, not by the author of the debt.
6. **Flags:** each has a date for being switched on or removed within 30 days (`flaga_max_dni`). After the deadline a decision: switch on, remove or
   a new date with a reason (D-03 – architect, PM).
7. **Counters** (table below) are filled in by the architect on Monday. The number of TODO and FIXME does not grow week on week (D-05);
   there are at most 15 open reviewer remarks (`uwagi_recenzenta_max`, starting value) – above that: a clean-up day instead of new features.
8. Repaid items move to "Repaid" with the PR and date; rows are not deleted. No personal data and no key values.

**Updated by:** {{rola}} – with every new debt, after the repayment PR is merged and in the 14-day review. Read by: CEO (D-01, D-03),
PM (weekly plan), reviewer (whether a deferred remark has a row).

Row example: `DL-01 | repo / moduł / plik | klient: niski · tempo: wysoki – każda zmiana wymaga ręcznej poprawki · ryzyko: średni | M | <rola> | RRRR-MM-DD | spłacić do RRRR-MM-DD (zlecenie <ID>)`

## Open

| ID | Where | Impact (on customer / pace / risk) | Repayment cost | Owner | Review date | Decision (repay by / accepted until) |
|---|---|---|---|---|---|---|

## Weekly counters

| Week | TODO + FIXME | Flags without a date | Skipped tests | Open reviewer remarks (older than 72 h) |
|---|---|---|---|---|

## Repaid

| ID | Where | PR | Repayment date | What improved (measure or one sentence) |
|---|---|---|---|---|
