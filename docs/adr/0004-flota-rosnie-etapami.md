# ADR 0004 · The fleet grows in stages with a 3-day gate · status: przyjęty (accepted) · 2026-11-09

## Context
The source organisation stood up 21 agents in about 4 days. Reliability mechanisms (run contract, window lock, watchdog, work claiming,
registers) were added **after** incidents: duplicate PRs, lost feedback, runs without a trace, costs without a budget, agents without a
playbook. Every new agent is a fixed cost (tokens, account limit, operator attention) and a new risk surface (mailbox, access). MAST: 44 % of
multi-agent system failures are system design errors, not model errors.

## Decision
The fleet starts from the **core** (`limit`, `ceo`, `pm`, `recenzent` – created by `scripts/init-org.sh`) and grows **in stages**
(`procesy/00-nowa-organizacja.md`): the next stage begins only when the previous one has had, for **3 consecutive days**, runs with an OK
result in every window, contract lines with evidence, no watchdog alerts and cost within budget. From day one every new agent has: a run
contract, a row in the tsv and in the manifest (consistency test), a monthly budget, a starter playbook, role registers (`role/szablony/`),
evals (when it reads untrusted content), a reader/actor/researcher type and `aprobata` autonomy. The same scheme applies to every
subsequent appointment (`procesy/60-cykl-zycia-agenta.md`).

## Consequences
- Starting a new organisation takes about 2 weeks instead of 4 days – deliberately; the first weeks provide measurement (F7–F9) to calibrate budgets.
- Reliability is an entry condition, not a patch: an agent without a contract line or without a row in the tsv will not pass the CI tests.
- The operator has time to set up accounts and minimum-scope access before the stage that needs them; the fleet budget grows linearly with stages.
- Cost: slower expansion into new areas; business pressure ("we need sales today") must go through an owner decision
  with a risk entry (`ryzyka.md`), not through skipping the gate.
- The same mechanism works in the transformation of an existing company (`procesy/20`): shadow mode and a takeover gate per area.

## Source
`docs/raport/2026-11-09-raport.md` ch. 0 ("Reliability was added after incidents"), 1.4, 4.2 items 2–3 (reliability from day 1, the fleet
grows in stages), ch. 5 (rollout plan); research "Standards 2026" – MAST (arXiv 2503.13657), "What to leave unchanged" (fleet growing in
stages with a 3-day gate).
