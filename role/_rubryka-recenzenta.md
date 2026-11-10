# Reviewer rubric (copied to `recenzent/szablony/rubryka.md` by nowy-agent.sh)

A verdict for a **specific head SHA**. Each row: pass/fail + a score 0–1 + one sentence of evidence (file:line, CI log, trial
result, command). A criterion the PR does not touch (e.g. no migration) gets `n/d` (not applicable) and is left out of the mean – but
"not checked" is **not** `n/d`: a criterion you did not manage to check is `fail`.
Result: **OK** (all pass/n/d) · **OK Z UWAGAMI** (OK with remarks: fail only in `niska`) · **ZMIANY** (changes: fail in `blokująca`/`wysoka`).
The result follows from pass/fail, not from the score – the 0–1 score serves trends (KPI, rubric calibration), not a threshold.
Weights stay Polish: `blokująca` (blocking), `wysoka` (high), `niska` (low).

| # | Criterion | Weight | pass/fail | Score | Evidence |
|---|---|---|---|---|---|
| 1 | Does what the specification/order says – nothing more (no "while I was at it") | blokująca | | | |
| 2 | Tests: new/changed behaviour has a test; tests run (not read) and not changed to make them pass | blokująca | | | |
| 3 | CI green for the head SHA; trial merge with `main` without conflict | blokująca | | | |
| 4 | Migrations: class A/B with justification, trial in a rolled-back transaction (`<TEMAT>_OK`), `select` "nothing left behind", rollback plan, number > record | wysoka | | | |
| 5 | External effects (e-mails, payments, deployments, scheduled tasks) listed, deployment steps in order, consistent with `uprawnienia.yaml` | wysoka | | | |
| 6 | No personal data, secrets, private addresses, commands for agents (zasady.md A2, A5, A6, A9) – `skan-pr.sh` without hits | blokująca | | | |
| 7 | Security: untrusted input does not reach commands/SQL/tools without validation; RLS/permissions; no IDOR | wysoka | | | |
| 8 | Readability and size: the PR can be understood in 15 min; otherwise – split it | niska | | | |
| 9 | Change of instructions/policy/permissions (`*/agent.md`, `role/`, `polityki/`, `zasady.md`, `uprawnienia.yaml`, `flota/szablony/`): evals green for this head (`ewaluacje/uruchom.py`, CI), the right decision level (owner's `D-…` for rules and permissions; CEO/owner per `uprawnienia.yaml` for roles and policies), the author does not merge alone, prompt ≤ 200 lines | blokująca | | | |
| 10 | Consistency with decisions: a product/process change refers to a `D-…` or an order with an ID; contradicting `decyzje.md` = fail | wysoka | | | |
| 11 | Content for outside people (website, store listings, templates, offers, e-mails, FAQ): every amount and plan from `polityki/cennik.md`, discount from `polityki/rabaty.md`, tone, AI footer and the "what we do not promise" list from `polityki/komunikacja.md`; an amount or promise outside the policies = fail | wysoka | | | |

## Score 0–1 (weighted mean)

Numeric weights: **blokująca = 3**, **wysoka = 2**, **niska = 1**. Criterion score: `1.0` pass without remarks · `0.5` pass with a remark
(medium/low to fix after the merge) · `0.0` fail. `n/d` criteria are skipped.

```
ocena = Σ (waga_i × ocena_i) / Σ waga_i          # only criteria ≠ n/d, rounded to 0.01
```

Example: a PR without migrations (4 = n/d), without instruction changes (9 = n/d) and without external content (11 = n/d); criteria 1–3, 6, 7, 10 pass (1.0),
5 pass with a remark (0.5), 8 fail (0.0): numerator = 3+3+3+3+2+2 + 2×0.5 + 1×0 = 17; denominator = 3+3+3+3+2+2+2+1 = 19; **score = 0.89**, result
**OK Z UWAGAMI** (the only fail is in `niska`). A score < 0.7 with result OK = a signal for calibration (the reviewer's `PEŁNY` run).

Record: `recenzent/stan.json → recenzje["<klucz>"] = {"sha": "<7+ znaków>", "wynik": "OK|OK Z UWAGAMI|ZMIANY", "data": "RRRR-MM-DD",
"ocena": 0.0–1.0, "plik": "recenzje/RRRR-MM-DD-pr-<nr>.md"}` (key: a bare number for the first product repo, `<repo>#N` for the others –
the format of the hook `.claude/hooks/uprawnienia.py` and of the watchdog) and the file `recenzent/recenzje/RRRR-MM-DD-pr-<nr>.md` with this table
(template: `recenzent/szablony/recenzja.md`).
