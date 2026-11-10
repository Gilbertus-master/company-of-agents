# ADR 0003 · Fleet watchdog without an LLM · status: przyjęty (accepted) · 2026-11-09

## Context
In the source organisation a fleet of 21 agents lost work in ways no agent noticed on its own: 6 tester reviews outside the register for 12 hours
(the task read them but did not record them), runs that did not happen after an account switch, PRs without a verdict, overdue orders
without an answer, duplicate PRs. Agent reports said "succeeded". A supervisor was needed that (a) does not share the model's blind spots,
(b) cannot be manipulated through content, (c) costs zero tokens and works when the account limit is exhausted, (d) checks state **at the source**
(tags in origin, PR API, store API, register files), not in reports.

## Decision
The fleet watchdog (`moduly/straznik/`) is a **deterministic Python script using the standard library, without an LLM**, run every 15 minutes
(GitHub Actions – no laptop needed; alternatively launchd). Core checks: runs (tag `przebieg/<agent>/<okienko>` 1:1 with `scripts/przebieg.sh`
and `koordynacja-odstepy.tsv`), PRs without a reviewer verdict > 60 min, overdue orders without an answer, task configuration, budgets
(80 % warning, 100 % HALT), dead agents. Per-organisation plugins (e.g. tester reviews from the store vs the register). Alert = e-mail to the operator,
a PILNE entry for the CEO, without duplicates and with reminders; its own clone, never stash/reset/force.

## Consequences
- Control independent of the model: the watchdog sees only IDs, numbers and dates – it does not read content, so there is nothing to inject.
- It enforces design: everything that is to be watched **must have an ID and a register** (run contract with reconciliations `przeczytane`/`w_rejestrze`,
  starter registers for roles, slot tags). What cannot be counted, the watchdog does not watch – and that is a signal to improve the register.
- Zero tokens, works with an exhausted limit and the laptop switched off; it may also set HALT (fleet budget) – the only automation with that right.
- Cost: maintaining the checks when the schedule changes (test `test_straznik.py`, consistency with `przebieg.sh`), secrets in CI, false alarms
  on delayed starts (slots < 60 min accept the tag in the next slot).

## Source
Source organisation: fleet reliability analysis (November 2026), `moduly/straznik/README.md`;
`docs/raport/2026-11-09-raport.md` sections 1.2 and 4.2 item 4 ("deterministic control: scripts, tests and hooks watch the agents, not another agent");
research "Standardy 2026" – section "Niezawodność: git + cron wystarcza…", recommendation 5 (dead-letter), "Co zostawić bez zmian" (watchdog without an LLM).
