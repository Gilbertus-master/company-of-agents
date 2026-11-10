# Fleet report – week <RRRR-MM-DD>–<RRRR-MM-DD> (Fri, PEŁNY PM)

Template: `pm/szablony/raport-floty.md`. File: `pm/raporty/RRRR-MM-DD-flota.md`. Written by the PM in the Friday PEŁNY run
(`role/pm.md`, Weekly). Recipients: CEO (link in `zlecenia/ceo.md`), finansowy (cost). Max. 1 page. Every number from a command
and with a date; no personal data. KPIs: `ceo/kpi/kpi.md` F1, F7–F10.

**Sources:** `python3 -I pm/narzedzia/przebieg-raport.py podsumuj 7` (GG:MM), `przebiegi/dead-letter/`, `koordynacja-odstepy.tsv`
(expected windows), `flota/zadania.yaml` (budgets), `org.yaml → mandat.budzet_floty_usd_miesiac`, `ceo/straznik-zdarzenia.md`,
`bezpieczenstwo/blokady.jsonl`, `python3 -I pm/narzedzia/przebieg-raport.py sprawdz przebiegi/*/*.jsonl`.

## 1. Fleet table (paste the output of `podsumuj 7` unchanged)

| Agent | Przebiegi (7 dni) | Tryby | Wyniki | Rozbieżności | Błędy | Koszt USD (n) | F7 koszt/OK | F8 OK bez dowodu | F9 p95 s | Ostatni |
|---|---|---|---|---|---|---|---|---|---|---|
| … | | | | | | | | | | |

Total fleet cost (7 days): N USD · monthly pace: N USD against the budget `budzet_floty_usd_miesiac` = N % (80 % = watchdog warning, 100 % = HALT).

## 2. Schedule coverage (failed runs)

| Agent | Expected windows (tsv) | With a contract line | Without a line (failed) | BŁĄD | NIEPEŁNY ×2 in a row | ROZBIEŻNOŚĆ without action |
|---|---|---|---|---|---|---|
| … | | | | | | |

Fleet coverage: N / M windows (N %). Target F1 ≥ 95 % of runs with result OK.

## 3. Reliability signals

- **F8 false completion** (OK without `outcome_evidence`, outside CISZA): N % of the fleet; agents > 0 %: … → orders to fix instructions (who, link).
- **F10 dead-letter**: agents in `przebiegi/dead-letter/`: … (since when, whether the operator has it queued, item number).
- **Contract**: lines rejected by the validator N (agents: …), version 1 lines / without `prompt_version` N.
- **Watchdog**: discrepancies from `ceo/straznik-zdarzenia.md` open > 24 h: … · hook blocks (`bezpieczenstwo/blokady.jsonl`) 7 days: N (roles: …).
- **F9 p95**: agents above their own threshold (reliability budget) or p95 > 2× the median: …

## 4. Coordination cost

- Runs without a result (CISZA + BŁĄD + NIEPEŁNY) / all: N % of the fleet; agents > 50 %: … (3 days in a row → rhythm proposal).
- Share of division leads' runs spent on coordination (from `<lider>/pion/<pn>.md` item 6): … (> 30 % for 2 weeks → signal for the CEO).
- Cost of CISZA runs: N USD (N % of fleet cost) – candidates for "on event" mode: … (details in `pm/przeglady/RRRR-MM-DD.md`).

## 5. Conclusions and proposals for the CEO (max. 5, each: what · why · effect · risk · cost)

| # | Proposal | Why (numbers from sections 1–4) | Effect | Risk | Change requires (uprawnienia.yaml) |
|---|---|---|---|---|---|
| 1 | | | | | |

## 6. Comparison with the previous week

| Measure | Previous week | This week | Trend |
|---|---|---|---|
| F1 runs OK | | | |
| window coverage | | | |
| F8 | | | |
| F10 | | | |
| fleet cost USD | | | |
| runs without a result % | | | |
