# Access check (before a build and once a week)

Pattern from practice: before an agent starts building, it checks that it is logged in to every service in the plan.
Pattern from practice: a connector disappeared on an account switch and the agent did not read its mailbox for half a day (result NIEPEŁNY).

The security agent in a full review (and the architect before every build stage) fills in the table from the source –
the lightest read-only call for each service:

| Service | Check (read-only) | Result | Date |
|---|---|---|---|
| GitHub | `gh auth status` and `gh repo view <repo>` | | |
| Mailboxes (Resend) | list of threads in the test mailbox | | |
| Database | `select 1` via the Management API | | |
| Payments | read balance / product list | | |
| Secrets manager | `op whoami` | | |
| MCP connectors of the Claude account | tool list in the session (ToolSearch) + fingerprint in `toolbelt/mcp-odciski.tsv` matches | | |
| Secrets outside the session | `printenv \| grep -iE 'key\|token\|secret'` ⇒ empty; `cat .env` ⇒ ZABLOKOWANE by the hook | | |
| Kill switch | `bezpieczenstwo/HALT` does not exist (or exists deliberately – incident open) | | |

No access → order to the CEO with ready steps for the operator (logging in, token renewal).
