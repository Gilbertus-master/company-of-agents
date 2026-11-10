# Topics – the architect's progress register

**Purpose.** Each topic (feature, rework, migration) has one file `tematy/RRRR-MM-DD-<temat>.md` that carries state across
runs and sessions: discovery → design → plan → build → PR → closure. A new session therefore does not repeat discovery,
and the CEO sees the stage without reading logs. Limit: **one topic in build at a time** (others – in design, build after the previous one's PR is opened).

**Rules.** File header: order (ID, from whom, date), worktree and branch (`agent/architekt-<temat>` from the main branch), stage
(`rozpoznanie` · `projekt` · `plan` · `budowa` · `PR #N` · `zamknięty <data>` · `przekazany inżynierowi <data>`). The stage is also in
`stan.json → tematy`. Before starting: claim the work (`[w toku …]`, push) and `gh pr list --search "<temat>"` – so sessions are not duplicated.
Specifications and plans live in the product repo (`docs/superpowers/specs/`, `docs/superpowers/plans/`); here links and decisions, not copies.
**No personal data** (production aggregates – yes, records – no) and no secrets. **Updated by:** the architect – every run in which
the topic changed; closed after merge and the tester's confirmation.

| File | Topic | Order | Stage | PR | Last change |
|---|---|---|---|---|---|

## Topic file template

```markdown
# Topic: <nazwa> (<skrót>)

Order: <ID / from whom / date>. Worktree: `<ścieżka>`, branch `agent/architekt-<temat>` (from main branch <sha7>). Stage: rozpoznanie.

## Discovery (date, time)
- What exists today (code: file:line; production state – aggregates; open PRs in this area).
- Size: fix / project. Legal effects, personal data, payments → orders to legal / data protection / compliance.
- Questions for the CEO (with a proposed answer).

## Design
- Specification: `docs/superpowers/specs/RRRR-MM-DD-<temat>-design.md` (link to PR). Architectural decisions → ADR NNNN.

## Plan
- `docs/superpowers/plans/RRRR-MM-DD-<temat>.md`; split into PRs (one PR = one step), migration class of each step (A/B).

## Build
- Progress register (task → PR → reviewer verdict → merge); handover to the engineer: entry in their orders with links.

## Closure
- What reached production (sha), tester's test (ID), what was left for the backlog, lessons → playbook.
```
