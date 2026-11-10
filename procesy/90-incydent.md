# Process 90 – incident (security, reliability, costs)

Triggered by: the security agent, the CEO, the watchdog (automatic HALT at ceilings) or the operator. Skill `/incydent`.
Rule: **stop first, understand later.** Legal liability falls on a human (the management board), so every step leaves a trace.

| Step | What | Who | Evidence |
|---|---|---|---|
| 0 | HALT (`bezpieczenstwo/HALT`, reason, commit+push) | whoever detected it | commit |
| 1 | Report `incydenty/RRRR-MM-DD-<agent>.md` (template `incydenty/_szablon.md`) | whoever detected it | file |
| 2 | Revocation of credentials within the incident's reach (GitHub App/PAT, Stripe, mail, broker) | operator | entry in the queue |
| 3 | Snapshot: `git tag incydent/<data>`, copy of audit/logs/states outside the repo | security | tag |
| 4 | Rollback of code/configuration/MCP schemas through a PR (not reset --hard) | architect/engineer, reviewer | PR |
| 5 | Secret rotation with a transition period; `toolbelt/sprawdz-dostepy.md` | operator | table |
| 6 | Pre/post review from `bezpieczenstwo/audyt/`: which safeguards failed; MAST class; cost | security, PM | section in the report |
| 7 | Changes: prompt/policy/watchdog/hook + **new eval scenario** | the relevant agent, reviewer | PR + file in `ewaluacje/` |
| 8 | Report to the owner; if applicable – Art. 26(5) AI Act / GDPR (72 h) through compliance | CEO, compliance | `ceo/do-decyzji.md` |
| 9 | Resumption: the operator removes HALT; decision D-… with conclusions | operator, owner | decision |

Kill switch test: once a quarter the operator sets HALT for 10 minutes during the fleet's working hours and checks in the audit that every
session stopped; result in `incydenty/testy-halt.md`.
