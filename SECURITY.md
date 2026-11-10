# Security policy

**Do not open a public issue for a vulnerability.** Use GitHub's private vulnerability reporting for this repository
(Security → Report a vulnerability). Include: the affected file, how to reproduce, and the impact.

In scope: the permissions hook (`.claude/hooks/uprawnienia.py`), the audit log, HALT, the merge queue (`scripts/scalanie.py`),
the anonymisation gate (`scripts/propozycje.py`, `recenzent/skrypty/skan-pr.sh`) and anything that lets an agent act outside
`uprawnienia.yaml`.

Out of scope: behaviour of the model itself, and misconfiguration of your own `org.yaml` / `uprawnienia.yaml`.

We aim to acknowledge a report within 7 days. There is no bug bounty.
