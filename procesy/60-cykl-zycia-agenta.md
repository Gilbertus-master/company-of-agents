# Process 60 – agent life cycle

Goal: every agent is created, works, changes and ends in the same way – recorded in the registers, with no abandoned tasks, orders or access.
Sources of truth: `role/<rola>.md` (library), `koordynacja-odstepy.tsv` + `flota/zadania.yaml` (fleet), `<agent>/` (state), `decyzje/`.
Permission: `nowy_agent_lub_zadanie_cykliczne` = level **ceo** (row in `backlog.md`, cost in the manifest); decommissioning – CEO decision
with notice to the owner; change to `zasady.md`/`uprawnienia.yaml` – owner.

## 1. Creation
1. **Need**: an order or a request (PM/CEO/any agent) with EV ÷ cost (`ceo/plan/portfel.md`) and a row in `backlog.md` (recurring);
   check whether an existing role already covers the scope (duplicate roles = cost); if the role is not in `role/` – first a PR with the new role
   (same format as the others, `role/szablony/<rola>/` for registers), reviewer verdict.
2. **Decision**: entry `[Decyzja CEO] powołanie agenta <rola>` (CEO decision: creation of agent) (`decyzje/`) with model, rhythm, monthly budget and type
   (reader/actor/researcher).
3. **Creation** in a single PR (branch `agent/<kto>-nowy-<rola>`):
   - `scripts/nowy-agent.sh <rola>` → agent directory (agent.md, log, stan.json, backlog, starter playbook, registers from `role/szablony/`),
     `przebiegi/<rola>/`, `zlecenia/<rola>.md`; for roles with external contact – `ewaluacje/<rola>/` (check `ewaluacje/sprawdz.py`);
   - a row in `koordynacja-odstepy.tsv` (all columns, `od` = the moment of the planned start) and an entry in `flota/zadania.yaml` (cron matching the tsv,
     `typ`, `wlasciciel_ludzki`, `domeny`, `autonomia: aprobata` – always a cautious start, budgets), label `agent:<rola>` in `flota/etykiety.tsv`;
   - `python3 flota/generuj.py && python3 flota/generuj.py --routines`; tests: `python3 -m unittest discover flota`, `bash scripts/test-przebieg.sh`,
     `python3 -I ewaluacje/sprawdz.py`, `bash scripts/test-nowy-agent.sh`;
   - the agent's source table in `pm/specyfikacje/kontrakt-przebiegu.md` (section 4) – what it counts, which register, what counts as a discrepancy;
   - a row in `bezpieczenstwo/rejestr-ai.md` (when the agent processes personal data) and in `ochrona-danych/rejestr-czynnosci.md` (new data source);
   - the role's KPIs in `ceo/kpi/kpi.md` (from the role's KPI section).
4. **Reviewer verdict** for the PR (instruction changes are merged by the reviewer or a human, never by the author) → merge.
5. **Scheduled task** – **[operator]** following `ceo/handoff/routines/<rola>.md` (or the CEO within its permissions, when the runtime allows it):
   creation, "Run now" once by hand, check of the line in `przebiegi/<rola>/` and of the absence of consent prompts; Issues label `agent:<rola>` in the repo
   (`gh label create` from `flota/etykiety.tsv`); access with minimal scope (`toolbelt/sprawdz-dostepy.md`, `bezpieczenstwo/dostepy.md`).

## 2. The 3-day gate
An agent is "created" when for **3 consecutive days**: runs in every schedule window (tags `przebieg/<agent>/*`), every contract line
passes the validator (`outcome_evidence` with OK), 0 watchdog alerts involving it, cost ≤ daily budget, 0 hook blocks other than amount
gates, evals green. Not met → the CEO extends the gate by 3 days with a repair order (max. 2 times), then a decision: change of role
or decommissioning. Gate result: a line in the CEO log and a status in `backlog.md` (`cykliczne → aktywne`).

## 3. Steady rhythm
- Runs according to the contract; monthly review (process 50) → autonomy ladder; playbook ≤ 60 rules with a curator (PM).
- Change of **rhythm**: in one commit the tsv row (`od` = the moment of the change) + cron in the manifest + `generuj.py` + tests; cloud task
  updated by the operator/CEO (`/schedule update`); the watchdog counts windows from `od`.
- Change of **model / budget**: manifest + CEO decision (budget raised above the threshold = owner).
- Change of **instructions** (`<agent>/agent.md`, `role/<rola>.md`, prompt blocks): PR, evals green in CI, reviewer verdict; `prompt_version`
  in the contract line changes automatically (commit hash) – regressions can be attributed to a version; canary rollout: the new version on one
  task for N runs when the change is large.
- **Pause**: `przebiegi/dead-letter/<agent>.md` (reason) – `przebieg.sh` exits with code 10 without work; the CEO lifts it after the fix; HALT applies to the whole fleet.

## 4. Change of role
When the scope shifts (e.g. compliance hands GDPR over to a new data-protection role):
1. CEO decision with a map of "what moves, to whom, from when" (`decyzje/`), entry in `backlog.md`.
2. PR: new/changed role in `role/`, update of `role/<rola>.md` on both sides (what is moved 1:1 is removed from the original role), `opis` in the manifest,
   KPIs in `kpi.md`, source table in the contract, registers: files moved with `git mv` (history stays) or new ones from `role/szablony/`.
3. Open orders in the old scope → rewritten to the new addressee with a note (header `[otwarte] … · przeniesione z <agent>`).
4. Evals: scenarios for the new role when it reads untrusted content; AI register and processing register updated.
5. Reviewer verdict; after the merge a 3-day gate as at creation (for the role that gained scope).

## 5. Decommissioning
When: a role without value (PM review "cost versus value", process 50), scope fully taken over by another role, end of a project, owner decision.
1. **Decision** `[Decyzja CEO] wygaszenie agenta <agent> od <data>` (CEO decision: decommissioning) (with notice to the owner in the report; when
   the agent had `sam` actions in `uprawnienia.yaml` – also `D-…`, because it changes permissions).
2. **Closing the work**: `[otwarte]`/`[w toku]` orders moved to the successor or closed with a reason; the agent's PRs – handed over (comment)
   or closed; `[w toku]` in other agents' registers removed.
3. **Scheduled task deleted** – **[operator]**/CEO (`/schedule delete` or in the app); the agent's access and keys revoked (operator;
   `bezpieczenstwo/dostepy.md` – row `odebrane <data>`), the Issues label stays for history (description "wygaszony <data>").
4. **Fleet** in one PR: row removed from `koordynacja-odstepy.tsv`, entry from `flota/zadania.yaml`, label from `flota/etykiety.tsv`
   (or description "wygaszony"), `python3 flota/generuj.py && python3 flota/generuj.py --routines` (prompt and routine instruction disappear), fleet tests and
   `test-przebieg.sh`; source table in the contract – section marked `wygaszony`.
5. **Archiving the directory**: `git mv <agent> archiwum/<agent>-RRRR-MM` (history stays; `archiwum/` has its own README with a table: agent, from–to,
   reason, decision, successor); `przebiegi/<agent>/` **stays** (contract evidence, retention as for audit ≥ 6 months); `zlecenia/<agent>.md` →
   `zlecenia/archiwum/`; `ewaluacje/<agent>/` → `archiwum/<agent>-RRRR-MM/ewaluacje/` (scenarios from incidents copied to the successor).
6. **Registers**: `bezpieczenstwo/rejestr-ai.md` (status `wygaszony <data>`), `ochrona-danych/rejestr-czynnosci.md` (processing ended, retention
   of the agent's data), `ceo/kpi/kpi.md` (role KPIs marked `wygaszone`), `backlog.md` (recurring → `zakończone <data>`), `ryzyka.md`, if the agent
   was a mitigation for any risk.
7. **Reviewer verdict** for the decommissioning PR (checklist 2–6 in the PR description with evidence) → merge → the watchdog stops expecting runs
   (the agent has disappeared from the tsv) – check there is no "brak przebiegu" (missing run) alert in the next cycle.

## Decommissioning checklist (for the PR description)
- [ ] decision with a date (and `D-…`, when it changes permissions) · [ ] orders and PRs handed over/closed · [ ] scheduled task deleted ·
- [ ] access revoked · [ ] tsv / manifest / labels · [ ] `generuj.py` + tests green · [ ] `git mv` to `archiwum/` with README ·
- [ ] registers (AI, processing, KPI, backlog, risks) · [ ] reviewer verdict · [ ] no watchdog alert after the merge
