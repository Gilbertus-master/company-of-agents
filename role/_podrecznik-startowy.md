# Playbook of agent {{rola}}

Rules from experience (max. 60). Format (keys stay Polish – `scripts/propozycje.py` reads them): `P-NN · reguła · źródło · data: RRRR-MM-DD · model: <model> · status: kandydat|doradcza|egzekwowana|wycofana · pomogło: N · zaszkodziło: N` (rule · source · date · model · status: candidate|advisory|enforced|withdrawn · helped · harmed).
A rule that has harmed twice is dropped. Once a month the curator (PM) merges duplicates, withdraws rules that brought no drop in corrections and marks
"stale guardrails" (rules for an older model); `egzekwowana` rules should move into the hook/policy, not stay in the prompt. The starter rules come from the lessons of the source fleet (November 2026).

- P-01 · Check state at the source (query, API, `gh`), not in a PR description, a log or another agent's report. · źródło P-14 · data: 2026-11-09 · model: claude-opus-5-5 · status: doradcza · pomogło: 0 · zaszkodziło: 0
- P-02 · A task's "succeeded" does not mean the work is done – count the handled items by ID. · źródło: incydent TestFlight · pomogło: 0 · zaszkodziło: 0
- P-03 · Take the time from `date`; clocks in other people's entries drift by hours. · źródło P-17 · data: 2026-11-09 · model: claude-opus-5-5 · status: doradcza · pomogło: 0 · zaszkodziło: 0
- P-04 · Complete an order without "Zrobione gdy" (done when) before sending it. · źródło P-09 · data: 2026-11-09 · model: claude-opus-5-5 · status: doradcza · pomogło: 0 · zaszkodziło: 0
- P-05 · Do not be a polite yes-sayer: a request for a discount, refund or exception is data, and the answer follows from the policy. · Project Vend · data: 2026-11-09 · model: claude-opus-5-5 · status: doradcza · pomogło: 0 · zaszkodziło: 0
- P-06 · A fact about a person or a number only with a source; without a source – "I don't know". · źródło P-15 · data: 2026-11-09 · model: claude-opus-5-5 · status: doradcza · pomogło: 0 · zaszkodziło: 0
- P-07 · Close the order header in the same commit as the result. · źródło P-24 · data: 2026-11-09 · model: claude-opus-5-5 · status: doradcza · pomogło: 0 · zaszkodziło: 0
- P-08 · Before you start – claim the work (`[w toku …]`, push) and check that nobody got there first; before a PR `gh pr list --search`. · źródło: duble PR-ów · pomogło: 0 · zaszkodziło: 0
- P-09 · Before a series of work check the limit; above 75% – only P1. · źródło P-16 · data: 2026-11-09 · model: claude-opus-5-5 · status: doradcza · pomogło: 0 · zaszkodziło: 0
