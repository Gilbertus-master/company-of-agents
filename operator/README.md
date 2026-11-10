# Operator – the "human interface"

The operator is the only human in the organisation's day-to-day work. **They report to the CEO agent** and perform actions
the agent cannot perform or is not allowed to (`uprawnienia.yaml`, level `operator`). They do not manage agents, do not
prioritise work and do not decide on strategy – the CEO does that within the owner's mandate.

## What the operator does (closed list)
| Category | Examples | Why a human |
|---|---|---|
| Identity and signature | qualified signature, trusted profile (KRS, US, ZUS, KSeF – granting permissions), UPL-1 | the law requires a natural person |
| Accounts and KYC | opening an account with a service, identity verification (bank, Stripe, Apple, Google), 2FA | KYC, terms of service, liability |
| Money | transfer prepared by the agent, topping up a balance, company card | zasady.md item 3 |
| GUI log-ins | panel without an API (e.g. an app store), CAPTCHA, session renewal | the agent does not bypass safeguards |
| Physical world | collecting post, visiting an office, hardware | the agent has no hands |
| Contact requiring a human | a phone call when the other party does not accept AI, a meeting | the other party's choice |

Everything outside the list goes back to the CEO with the question "why is a human doing this?". Every operator task should have
a plan for removing the human from the loop (API, power of attorney, service account) – the CEO tracks it in KPI "operator minutes / week".

## How they work
1. Once or twice a day (or after a notification) they open `operator/kolejka.md`.
2. Every task has ready, numbered steps, links and the expected result. The operator does not have to work anything out.
3. They write the result in the "Wynik" (result) column (no secrets: "account created, login in 1Password: item X").
4. A problem or doubt → an entry in the "Uwagi" (notes) column and a notification to the CEO (CEO chat).

## SLA
Response time from `org.yaml → ludzie.operator.sla_godziny`. An overdue task is escalated by the CEO to the owner
(`ceo/do-decyzji.md`), because it blocks the organisation.

## Owner vs operator
The owner sets the mandate, budget and boundaries and approves the decisions from `ceo/do-decyzji.md`. They may be the same
person as the operator, but we keep the roles apart: the decision (owner) and the execution (operator) leave separate trails.
