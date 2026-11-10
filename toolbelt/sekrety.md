# Secrets

Rule: a secret (API key, token, password) **never enters the model's context, a file or the repo.**
An agent uses a secret only inside the command that injects it.

## Pattern
- Secrets manager: 1Password (service account, vault "agenci-<org>") or Infisical (Machine Identity).
- Call: `op run --env-file=toolbelt/env/<usluga>.env -- <polecenie>`. The `.env` file contains **references**
  (`STRIPE_KEY=op://agenci/stripe-agent/credential`), not values.
- On a single Mac as a last resort: the keychain `security find-generic-password -s <nazwa> -w` inside the command
  (pattern from the source organisation).

## 1Password configuration for agents (stage D)
1. **[operator]** Vault `agenci-<org>` and a 1Password **service account** with access to that vault only (Settings →
   Developer → Service accounts). The service account token goes to: the `OP_SERVICE_ACCOUNT_TOKEN` variable on the run
   machine (Mac: keychain, read only inside a command; CI: Actions secret). Never to the repo.
2. Vault items named per role and service: `github-pat-ceo`, `resend-obsluga`, `stripe-agent-finansowy`.
3. Role file `toolbelt/env/<rola>.env` with `op://…` references (example: `toolbelt/env/przyklad.env`) – may be committed.
4. Every command with a secret: `op run --env-file=toolbelt/env/<rola>.env -- <polecenie>`. The value does not reach
   the model's context; `op run` also masks it in the output.
5. Check: `toolbelt/sprawdz-dostepy.md` (`op whoami`, `op read op://agenci/<pozycja>/credential >/dev/null`).

## Scopes
| Secret | Minimal scope | Agent | Rotation |
|---|---|---|---|
| Stripe | restricted key tagged "Agent", only the needed resources | finansowy (read) | 90 days |
| GitHub | token per agent, only the needed repos | by role | 90 days |
| Database (Management API) | token with a read-only role for agents outside Product | architekt | 90 days |

## Prohibitions
- A secret printed in a report, log or PR → incident: rotation within 1 h (operator), entry in
  `bezpieczenstwo/incydenty.md`.
- `pm/narzedzia/przebieg-raport.py` rejects run lines that look like a secret.

## Sandbox and secrets outside the agent's environment (rec. 12, operator tasks)
Managed settings (the organisation's `managed settings` or `~/.claude/settings.json` on the run machine – not in the repo,
because an agent can change the repo):
```json
{
  "sandbox": { "enabled": true, "failIfUnavailable": true, "allowUnsandboxedCommands": false,
               "network": { "allowedDomains": ["api.github.com", "github.com", "api.resend.com", "api.stripe.com"] },
               "credentials": { "mode": "mask" } },
  "disableBypassPermissionsMode": "disable",
  "allowManagedHooksOnly": true
}
```
- `allowedDomains` per agent type from `flota/zadania.yaml → domeny` (reader: repo only; actor: repo + the role's services).
- `credentials.mode: mask` – commands get a sentinel value, the real secret is injected only for allowed hosts.
- By default the sandbox reads `~/.ssh` and `~/.aws` – disable it (`permissions.deny` in `.claude/settings.json` already does this for Read).
- Identities: GitHub App (1 h token) instead of a PAT; Stripe keys tagged Agent per agent (before 30.11.2026); read-only DB roles.
- Check: `toolbelt/sprawdz-dostepy.md` – row "no secret is readable from the session" (`printenv | grep -i key` ⇒ empty; `cat .env` ⇒ hook block).
