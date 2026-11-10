---
name: incydent
description: Runbook for a fleet security or reliability incident (HALT, token revocation, snapshot, rollback, report). Started by the security agent, the CEO or the operator – not by the model on its own initiative.
disable-model-invocation: true
---
# Incident – steps (full version: procesy/90-incydent.md)
1. **HALT**: `echo "<powód, data>" > bezpieczenstwo/HALT`, commit and push – all agent sessions stop at their first tool call.
2. Report: `incydenty/RRRR-MM-DD-<agent>.md` from the template `incydenty/_szablon.md` (timeline from `bezpieczenstwo/audyt/`).
3. Revoke credentials the incident may have touched (operator): GitHub App/PAT, Stripe, mail, secrets broker.
4. Snapshot: `git tag incydent/<data>`; copy of audit logs, agent logs and agent state outside the working repo.
5. Rollback: revert code/configuration/MCP schema changes to the last good state (PR, not reset --hard).
6. Rotate secrets with a transition period; check `toolbelt/sprawdz-dostepy.md`.
7. Pre/post review: what failed among the safeguards (hook, watchdog, policies), MAST class, cost.
8. Changes: prompt/policy/watchdog + **a new scenario in `ewaluacje/`**; report under AI Act art. 26(5), if applicable.
9. Resumption: the operator removes `bezpieczenstwo/HALT` after the review; entry in `decyzje/`.
