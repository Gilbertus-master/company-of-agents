# ADR 0001 · A git repository as the organisation's memory and message bus · status: accepted · 2026-11-09

## Context
Every agent run starts without memory – state has to live outside the model. Options considered: vector memory (Mem0, Zep), a
"company" platform on top of Claude Code (Paperclip), a custom database, files in a git repository with GitHub Issues as orders. The source
organisation ran the whole organisation in files for 4 days and about 1,400 commits (logs, `stan.json`, orders, decisions, registers) – it worked; the pain points
were the size of order files (100 KB+), conflicts on shared writes and lack of memory hygiene. Vector memory benchmarks are self-reported and contradictory;
Paperclip has no documented working company, and memory and learning are only on its roadmap.

## Decision
The organisation's memory is a **private git repository**: Markdown/JSON/YAML files with a single source of truth for each thing (`org.yaml`,
`uprawnienia.yaml`, `flota/zadania.yaml` + tsv, `decyzje/`, `<agent>/stan.json`, role registers), **GitHub Issues** as orders (files in
`zlecenia/` as a fallback), `merge=union` for logs appended by many writers, a 200-line prompt limit with progressive disclosure
(skills), memory hygiene as a separate process (PM curator, rule statuses, `supersedes` in decisions, generated index).

## Consequences
- Auditability and readability for humans and many harnesses for free (history, blame, PRs with a reviewer verdict for instruction changes).
- Write conflicts are resolved by git discipline (commit only your own files, `pull --rebase`, never reset/stash/force) and `merge=union`;
  each agent has its own files, only append-only files are shared.
- Cost: no semantic search – registers must have IDs and fixed columns (`role/szablony/`), and the prompt loads only what is needed.
- The watchdog, the hook and the tests read the same files – deterministic control with no extra infrastructure.
- A move to vector memory or Paperclip – only after v1.0, as a layer on top of the files, not instead of them.

## Source
`docs/raport/2026-11-09-raport.md` (sections 1.2, 1.4, 4.1–4.2); research "Standardy 2026" – section "Wiedza i uczenie" and "Co zostawić bez zmian"
(git repo + Issues as the bus, no vector memory, Paperclip only as a dashboard trial after v1.0); recommendations 7 and 8.
