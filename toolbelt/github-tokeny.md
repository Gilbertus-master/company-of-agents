# GitHub identity and tokens per agent

Lesson from practice: one `gh` account for the whole fleet = no accountability (nobody knows which agent merged, who pushed).
In the template every agent has its own GitHub identity with a minimal scope, and the permissions hook limits what it does with it.

## Recommended variant: the organisation's GitHub App
1. **[operator]** In the GitHub organisation: Settings → Developer settings → GitHub Apps → New. Repo permissions:
   Contents (read/write), Pull requests (read/write), Issues (read/write), Metadata (read). No permissions for
   Administration, Secrets, Actions. Install it in the organisation repo and the product repo.
2. **[operator]** The App's private key → secrets manager (`op://agenci/github-app/private-key`), App ID and Installation ID
   in `toolbelt/env/wspolne.env`.
3. At the start of a run the agent gets a short-lived installation token (1 h) from `scripts/gh-token.sh <rola>`
   (to be written in stage E: JWT from the App key → `POST /app/installations/<id>/access_tokens`), set as `GH_TOKEN`.
   Commits: `git -c user.name="<rola> (agent)" -c user.email="<rola>@<domena>"` – the author is visible in history.
4. Accountability: `gh api` and commits signed with the role name; hook blocks in `bezpieczenstwo/blokady.jsonl`.

## Simple variant (first company): fine-grained PAT per role
- **[operator]** One PAT per role from the company's technical account (not a private one!): `github-pat-ceo`,
  `github-pat-inzynier`, `github-pat-recenzent`… Scope: only the organisation and product repos; reviewer and PM – Contents
  read, Pull requests read/write (verdicts), Issues read/write; engineer and architect – Contents read/write,
  Pull requests read/write; CEO – as engineer + merge (through the hook only with a verdict).
- Validity 90 days, rotation in the security agent's calendar.
- In a run: `op run --env-file=toolbelt/env/<rola>.env -- <polecenie>` sets `GH_TOKEN`.

## What not to do
- Do not use the owner's private account for agents' work.
- Do not give agents Administration or Secrets permissions (repo settings changes = level `zmiana_ustawien_uslugi`).
- Do not write a token into a repo file, log or report (the report hook and the watchdog mask `ghp_…`, `github_pat_…`).
