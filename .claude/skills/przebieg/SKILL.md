---
name: przebieg
description: Run contract of an organisation agent – start (lock, budget), modes, finish (log, stan.json, JSON line v2, commit), report. Use at the start of every scheduled run.
---
# Agent run – procedure (full version: zasady.md section B, pm/specyfikacje/kontrakt-przebiegu.md)

1. `cd <repo> && git pull --rebase --quiet`; `scripts/przebieg.sh start <agent> <konto>` – code 10 = finish with one sentence,
   no work and no commit (window taken, budget, dead-letter); any other ≠ 0 = finish with the error description. Time from `date`.
2. Read: `uprawnienia.yaml` (your rows), new `decyzje/` since `stan.json → ostatni_przebieg`, your orders,
   `stan.json`, the last 3 log entries, `wiedza/podrecznik.md`.
3. Mode: PEŁNY (full; once a week, on your day, at most once a day) · ZDARZENIE (event; new order/decision/deadline ≤ 3 h) · DZIENNY (daily;
   first run of the day) · CISZA (silent; nothing new → one sentence, a JSON line and a separate commit of only the `przebiegi/<agent>/` file –
   run contract; you commit no other files).
4. Work within your scope; other agents' areas = an order (zlecenia/README.md); matters for a human → the CEO.
   Check state at the source. Every result has a path in the repo or a URL.
5. Finish: log (header `## RRRR-MM-DD – TRYB (GG:MM, konto X)`, sections Co się zmieniło (what changed) · Co zrobiłem (what I did) ·
   Zlecenia (orders) · **KPI:** · Do decyzji (for decision) · Dalej (next)), `stan.json`, JSON line **version 2** in `przebiegi/<agent>/<data>.jsonl`
   (`prompt_version` = `git log -1 --format=%h -- <agent>/agent.md role/<agent>.md flota/szablony/`, `outcome_evidence` = URL),
   `python3 -I pm/narzedzia/przebieg-raport.py sprawdz …`, run cost `python3 -I scripts/koszt.py zapisz --agent <agent>`
   (`koszt_usd` in the contract line stays `null`), `git add` only your own files (+ `przebiegi/<agent>/koszty-*.jsonl`, `bezpieczenstwo/audyt/`),
   commit, `git pull --rebase` (up to 3 attempts), push. Report ≤ 12 lines for whoever gave the order.
6. Learning: first run after 18:00 – after-action review → a rule in the playbook (format from `role/_podrecznik-startowy.md`).
