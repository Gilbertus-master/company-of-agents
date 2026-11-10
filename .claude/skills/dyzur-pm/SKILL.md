---
name: dyzur-pm
description: PM blocker duty (who is waiting on whom; > 6 h → order to the CEO) and fleet report from the run contract (failed, NIEPEŁNE, F8 false completion, F10 dead-letter) – procedure with commands. Started by the PM agent in mode DYŻUR / DZIENNY / PEŁNY, not by the model on its own initiative.
disable-model-invocation: true
---
# Blocker duty and fleet report – steps (full version: `role/pm.md`, `pm/dyzur.md`, `pm/szablony/raport-floty.md`)

Every number comes from a command run **in this duty** and the time from `date` – nothing from memory or from the previous `pm/dyzur.md`.
Read-only on sources + entries in your own files. You merge nothing, you do not wake agents yourself (the CEO does), you do not close other agents' orders.

0. Start: `date`; `scripts/przebieg.sh start pm <konto>` with exit code 0 (skill `/przebieg`); `git pull --rebase --quiet`. PR repositories: `org.yaml → repo.produkt` + `repo.github`.
1. **PRs** – for each repo: `gh pr list -R <repo> --state open --json number,title,headRefOid,isDraft,mergeable,statusCheckRollup,updatedAt` and `recenzent/stan.json → recenzje`
   (shortcut: `bash recenzent/skrypty/uzgodnienie.sh` – read-only). Classification: verdict `OK`/`OK Z UWAGAMI` for head + all checks SUCCESS + `MERGEABLE`
   = **ready, not merged** (for how many h: `now − verdict date`); no verdict for head > 1 h = **reviewer queue** (the CEO wakes it); `CONFLICTING` = **author**.
2. **Orders**: `python3 -I pm/narzedzia/backlog.py --sprawdz` (overdue, duplicates, md ≠ Issue mismatches, tasks without a C row);
   `grep -n "^## \[czeka\|^## \[w toku\|Pilne" zlecenia/*.md`; Issues: `gh issue list -R <org> --label stan:czeka --state open --json number,title,updatedAt`
   and `--label typ:blokada`. `[w toku]` older than 6 h → check in `<adresat>/dziennik.md` whether there is an entry since that time; none = the addressee is blocking.
3. **Operator and owner**: `operator/kolejka.md` – items older than `org.yaml → ludzie.operator.sla_godziny`; `ceo/do-decyzji.md` – matters past "Valid until" (expired = default "no").
4. **Fleet (24 h)**: `python3 -I pm/narzedzia/przebieg-raport.py podsumuj 1`; `ls przebiegi/dead-letter/ 2>/dev/null`; agents with no line in their window
   (windows from `koordynacja-odstepy.tsv`, tolerance: window + 30 min); `python3 -I pm/narzedzia/przebieg-raport.py sprawdz przebiegi/*/$(date +%F).jsonl`
   (rejected lines, version 1, missing `prompt_version`/`outcome_evidence`); F8 from the „OK bez dowodu” (OK without evidence) column.
5. **Sessions**: `python3 -I pm/narzedzia/sesje.py` (open > 24 h, no parent, duplicates, closed without a result) → lines for the CEO; you do not close them.
6. **Write-up**: overwrite `pm/dyzur.md` per the template (header with date, times and account; sources with time; numbers; table „co · kto blokuje · kto czeka ·
   od ilu h · co zrobić” (what · who blocks · who waits · for how many h · what to do)).
   Blocker **> 6 h** (or PR ready > 6 h, overdue order, dead-letter) → entry in `zlecenia/ceo.md`:
   `## [otwarte] RRRR-MM-DD GG:MM · od: pm · Dyżur blokad GG:MM · Z-RRRRMMDD-NN`, `Termin:` +3 h, `Zrobione gdy: CEO odpowiedział pod wpisem`, max. 10 lines
   (to merge / to wake / overdue / for the operator). To the blocking agent, under its order: `**Pilne (dyżur PM GG:MM):** czeka na Ciebie <kto> – <co>`
   – first `grep -n "Pilne (dyżur PM" zlecenia/<agent>.md` (once per 6 h, not more often).
7. **No blockers**: the sentence „Dyżur: brak blokad.” (Duty: no blockers.), a contract line in mode `DYŻUR`, no changes to other files.
8. **Friday (PEŁNY)**: `python3 -I pm/narzedzia/przebieg-raport.py podsumuj 7` → `pm/raporty/RRRR-MM-DD-flota.md` (`pm/szablony/raport-floty.md`: window coverage,
   F1, F7–F10, cost vs `budzet_floty_usd_miesiac`, coordination cost); `pm/przeglady/RRRR-MM-DD.md` (`pm/szablony/przeglad-cyklicznych.md`, max. 5 proposals);
   `python3 -I scripts/decyzje-indeks.py --sprawdz`; a row in the counter of `ceo/do-decyzji.md`; Monday: division reports `<lider>/pion/<data>.md` (missing = a row in the duty).
9. **End**: `pm/dziennik.md` (Dyżur · Flota · Rejestr sesji · Do CEO · **KPI:**), `pm/stan.json`, contract line v2 (`tryb: DYŻUR|DZIENNY|PEŁNY`; `zrodla`: `zlecenia/*.md`,
   `PR otwarte`, `przebiegi/`, `rejestr-sesji.md`, `operator/kolejka.md`, `ceo/do-decyzji.md` with numbers from this run; `outcome_evidence` = path of `pm/dyzur.md`
   or of the report), `git add pm/ backlog.md przebiegi/pm/ zlecenia/<only your own entries>` (by name), commit `pm: dyżur RRRR-MM-DD GG:MM – <N blokad>`,
   `git pull --rebase` (up to 3 attempts), push.
