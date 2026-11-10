---
name: new-organisation
description: Sets up a new agentic organisation from this template – a private company repo, init-org.sh, fleet prompts, tests and the list of scheduled tasks to create. Use when the user wants to start an agent fleet for their company.
disable-model-invocation: true
argument-hint: "<company name> <domain> <prefix>"
---
# New agentic organisation – steps

Template: `Gilbertus-master/company-of-agents` (full guide after cloning: `docs/instrukcja-uruchomienia.md`; Polish identifiers are explained in
`GLOSSARY.md`). Required: `git`, `gh` (signed in), Python 3. Before you create anything, ask the user for: company name,
domain, a short prefix (2–4 letters), GitHub owner, the operator's address and the runtime (`claude-routines`,
`desktop-scheduled` or `agent-sdk`).

1. Show the user the commands from step 2 with the values filled in and wait for consent – you are creating a repository on their account.
2. Company repo (always **private**):
   ```bash
   gh repo create <owner>/<company>-agents --private --template Gilbertus-master/company-of-agents --clone
   cd <company>-agents
   scripts/init-org.sh --nazwa "<Name>" --domena <domain> --prefiks <prefix> \
     --github <owner>/<company>-agents --operator <operator address> --runtime <runtime>
   python3 flota/generuj.py
   bash scripts/testy.sh
   ```
3. The tests must be green. Red → show the output and stop; do not change tests to make them pass.
4. Read the README section "Sharing improvements" with the user – it is **off by default**. Do not switch it on yourself:
   the owner enables it by a decision (`repo.dziel_sie_ulepszeniami: true` and `repo.dziel_sie_decyzja: "D-…"` in `org.yaml`).
5. Billing and the fleet account: `org.yaml → runtime` as described in `docs/instrukcja-uruchomienia.md` (section 2). The
   fleet runs on **one** account; never suggest switching between other people's accounts or subscriptions.
6. Commit and push only after the user agrees. You do not enter secrets (`STRAZNIK_TOKEN`) – you give the command
   `gh secret set STRAZNIK_TOKEN`, which the user runs themselves.
7. Finish by listing: what works, which scheduled tasks need creating (order: `limit`, `scalanie`, `ceo`, `recenzent`,
   `pm`) and what needs a human. Do not create scheduled tasks without separate consent.

The company repo already carries the project skills (`/przebieg`, `/dyzur-pm`, `/recenzja`, `/incydent`) and the
permissions hook – this plugin adds no hooks or permissions of its own.
