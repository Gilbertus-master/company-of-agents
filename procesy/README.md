# Processes of the agentic organisation – index

Numbering in steps of 10 leaves room for the company's own processes (e.g. `45-…`). Each process has steps with an assignee, a gate and evidence; human steps
are marked **[operator]** / **[właściciel]** (owner). Procedures with side effects also have a skill in `.claude/skills/` (`disable-model-invocation`).

| No. | Process | One sentence |
|---|---|---|
| [00](00-nowa-organizacja.md) | New organisation from scratch | About 2 weeks after the owner's decision the CEO, the management core and the first domain agents are running; the fleet grows in stages with a 3-day gate. |
| [05](05-wiele-spolek.md) | Several companies on one template | One company = one Claude account, one GitHub organisation and one repo with the template as upstream; updates through `scripts/aktualizuj-z-szablonu.sh` keeping local files. |
| [10](10-budowa-produktu.md) | Building the product | From idea to working application in 17 stages with CEO gates, a check for contradicting decisions and the cost of each stage. |
| [15](15-rozwoj-szablonu.md) | Template development through client fleets | Companies submit generalised improvements every week (without company data) and receive the latest template release as a PR – the template learns from the work of all fleets. |
| [16](16-wymiana-ceo.md) | Exchange of discoveries and releases between project CEOs and the publisher | The CEO of every project on the framework sends discoveries (what did not work and how it was fixed) – one issue per discovery, in the same run; the publisher generalises them into the template and sends back a harness release – each CEO chooses what to adopt; nobody reads anyone else's repositories. |
| [20](20-transformacja-istniejacej.md) | Transformation of an existing organisation | Audit of processes and people's time → shadow mode → takeover gates per area, with integrations into existing systems, communication with people and HR legal risks. |
| [30](30-zamkniecie-miesiaca.md) | Month-end close | By the 10th: invoices from KSeF, costs, reconciliation bank ↔ payments ↔ invoices, JPK through an authorised person, fleet cost, report (skill `/zamkniecie-miesiaca`). |
| [40](40-wydanie.md) | Product release | Scope → tests → compliance gate G1–G12 → reviewer verdict → CEO decision → shipping by the role named in the permissions → 72 h monitoring and rollback (skill `/wydanie`). |
| [50](50-przeglad-agenta.md) | Monthly agent review | KPIs, cost, incidents, hook blocks and contract-line quality of each agent → decision on the autonomy level per task type (ladder escalation → approval → auto). |
| [55](55-doskonalenie-zarzadzania.md) | Improving management | The way the CEO oversees the fleet improves at a steady rhythm: weekly the three most frequently failed checks and escapes, monthly measures M1–M6, experiments and threshold calibration (skill `/przeglad-zarzadzania`), quarterly a maturity self-assessment and a reviewer audit reported straight to the human. |
| [60](60-cykl-zycia-agenta.md) | Agent life cycle | Creation (`nowy-agent.sh`) → 3-day gate → steady rhythm → change of role → decommissioning with archiving to `archiwum/` and a decision D-…. |
| [90](90-incydent.md) | Incident | Stop first (HALT), understand later: revocation of credentials, snapshot, rollback, review of safeguards, new eval scenario, report (skill `/incydent`). |

Related registers: `ryzyka.md` (organisational risks, quarterly review), `decyzje/`, `incydenty/`, `ewaluacje/`. Architectural decisions
of the template: `docs/adr/`.
