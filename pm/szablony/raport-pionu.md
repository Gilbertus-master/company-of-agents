# Division report <Produkt | Wzrost | Ryzyko | Finanse> – <RRRR-MM-DD> (Mon)

Template: `pm/szablony/raport-pionu.md` (maintained by the PM). Written by the division lead in their first run on Monday, before the CEO's
review (`role/ceo.md`, Full review). File: `<lider>/pion/RRRR-MM-DD.md`. Max. 1 page. No personal data and no secrets.
Every number with a source and a time. No report by the first PM duty after 7:30 = a row in `pm/dyzur.md` (blocks: lead, waits: CEO).
Divisions and leads: `README.md → Agenci`.

**Lead:** <agent> · **Members:** <agents> · **Period:** <previous Mon>–<Sun> · **Sources:** `ceo/kpi/pomiary/<data>.json`,
`python3 -I pm/narzedzia/przebieg-raport.py podsumuj 7`, `zlecenia/<członek>.md` / Issues `agent:<członek>`, `backlog.md`,
`ceo/plan/portfel.md`, `przebiegi/dead-letter/`.

## 1. Division KPIs

| KPI | Today | A week ago | Target | Status (green/yellow/red) | Owner |
|---|---|---|---|---|---|
| <id – name from ceo/kpi/kpi.md> | | | | | |

Members (run contract, 7 days, from `podsumuj 7` and `backlog.py --sprawdz`):

| Agent | Runs with a report / expected | NIEPEŁNY + BŁĄD | Open discrepancies | F8 OK without evidence | F10 dead-letter | Orders open / overdue | Cost USD (7 days) |
|---|---|---|---|---|---|---|---|
| <agent> | N/M | N | N | N % (N/N) | yes/no (since when) | N / N | N |

**Division cost (7 days):** N USD = N % of fleet cost (`podsumuj 7`, sum of the Koszt column for members) · cost / successful run (F7) of the division: N USD ·
the division's external services (invoices, `toolbelt/katalog.md`): N PLN. Budget from `flota/zadania.yaml` (sum of members' `budzet_usd_miesiac`): N USD/month → pace N %.

## 2. Top 3 division matters

1. <matter> – status, next step, owner, deadline (link: order/PR/file).
2. …
3. …

## 3. Blockers

| What | Who blocks | Who waits | For how many h | What to do (who) |
|---|---|---|---|---|

Only blockers inside the division that the lead has not resolved alone, and blockers from other divisions. Cross-division matters → also an entry in
`zlecenia/ceo.md`. A division agent in `przebiegi/dead-letter/` = a blocker with owner "operator" (review) – always in this table.

## 4. Boundaries and sequence (the lead's rulings during the week)

- Boundaries: <dispute> → <who does it>, why (link to the entry under the order).
- Sequence for this week: <member>: 1. … 2. … 3. … (by portfolio EV and deadlines; P1 deadlines are not moved).

## 5. Proposals for the CEO (max. 3, with EV)

| Proposal | EV (revenue/EBITDA × p) | Cost (fleet capacity, money) | Time to effect | Risk | Decision needed from |
|---|---|---|---|---|---|

Decision rule as in the portfolio: EV ÷ (cost + use of fleet capacity). EV figures are hypotheses with a stated source.

## 6. Coordination cost

Lead's runs in the week: N, of which on division coordination (report, settling boundaries, reminders – no domain work): N (N %).
Members' runs without a result (CISZA + BŁĄD + NIEPEŁNY): N of M (N %). Threshold: > 30 % coordination for 2 weeks, > 50 % without a result for
3 days or a report > 1 page → signal for the CEO (the PM collects it in `pm/raporty/RRRR-MM-DD-flota.md` item 4).

## 7. Lesson of the week

Rules added to members' playbooks (Id, one sentence) and mistakes of the week (what failed, whether it became a rule) – max. 5 lines.
