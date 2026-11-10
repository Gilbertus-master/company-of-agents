# Review of recurring tasks – Friday RRRR-MM-DD (PEŁNY PM, account X)

Template: `pm/szablony/przeglad-cyklicznych.md`. File: `pm/przeglady/RRRR-MM-DD.md`. Written by the PM every Friday (`role/pm.md`, Weekly;
`backlog.md` C-01). The review question: **does every recurring task deliver more than it costs – and who reads its output?**
Proposals go to the CEO (`zlecenia/ceo.md`); changes to scheduled tasks, crons and rhythm are made by the CEO per `uprawnienia.yaml`
(`nowy_agent_lub_zadanie_cykliczne` – ceo), not by the PM. Every number from a command and with a time.

**Sources:** `backlog.md` (section Cykliczne), `flota/zadania.yaml` (cron, model, budgets), `koordynacja-odstepy.tsv`,
`python3 -I pm/narzedzia/przebieg-raport.py podsumuj 7` (GG:MM), `przebiegi/<agent>/*.jsonl` (field `akcje` – what was actually produced),
`ceo/kpi/kpi.md` (value = the KPI the task feeds), `pm/raporty/RRRR-MM-DD-flota.md` (the same week).

## 1. Table of recurring tasks (7 days)

| Id | Task (agent / cron / loop) | Rhythm | Runs | CISZA | With actions (IDs) | BŁĄD + NIEPEŁNY | Cost USD | Cost / run with an action | Value (KPI) | Who reads the output | Rating |
|---|---|---|---|---|---|---|---|---|---|---|---|
| C-01 | | | | | | | | | | | keep / less often / on event / merge / delete / change model |

Legend: **CISZA** = contract lines `tryb: CISZA`; **With actions** = lines with a non-empty `akcje` (by ID – PR, commit, order, file);
**Who reads the output** = the agent/human who referred to the output within 7 days (order, log, decision) – "nobody" = candidate for deletion.

## 2. Tasks without a backlog row / manifest mismatches

- Tasks from `flota/zadania.yaml` without a C-nn row (`python3 -I pm/narzedzia/backlog.py --sprawdz`): …
- Rhythm in `flota/zadania.yaml` ≠ `koordynacja-odstepy.tsv` (`python3 -m unittest discover flota`): …
- Tasks reported in `zlecenia/pm.md` as new/changed this week: … (no report on a change = discrepancy, who).

## 3. Proposals for the CEO (max. 5)

| # | Task | Proposal (what to change) | Why (numbers from table 1) | Effect (cost / value) | Risk | How we check after a week |
|---|---|---|---|---|---|---|
| 1 | | | | | | |

Examples: an agent with 7/7 CISZA → a less frequent rhythm or "on event" mode (woken up by the CEO); two tasks with the same output → merge;
a task with no recipient of its output → delete; an expensive task with simple actions → a weaker model; an agent with `BŁĄD` ≥ 3 → fix the
instructions first, not the rhythm. Do not propose a change whose effect cannot be measured in the next review.

## 4. What changed after the previous review and what it achieved

| Proposal from RRRR-MM-DD | CEO decision (link) | Implemented (date) | Effect after a week (numbers) |
|---|---|---|---|

## 5. PM KPIs
Recurring tasks slimmed down/deleted in the month: N (target ≥ 1) · fleet cost / week: N USD (previous: N) · CISZA share: N % (previous: N %).
