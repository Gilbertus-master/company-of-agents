# Agentic Organization – free core

A git repository that runs a small fleet of Claude Code agents as an organisation: a CEO agent, a project manager, a
reviewer, an architect, a rate-limit regulator and a merge agent – with the guard rails that keep an unattended fleet safe.
Two humans stay in the loop: the **owner** (decisions) and the **operator** (accounts, secrets, anything agents may not do).

> **Language.** This is the **English edition**: documents, role instructions, skills and agent prompts are in English.
> File names, YAML keys, commands and script output stay in Polish in both editions, so both run the same code and the
> same tests – `GLOSSARY.md` explains every Polish name you will meet. Polish edition: [Gilbertus-master/firma-z-agentow](https://github.com/Gilbertus-master/firma-z-agentow).

> **Status.** Extracted from a private template that runs a real fleet. The core passes its own test suite
> (`bash scripts/testy.sh`, no network, no LLM). It has not yet been run end-to-end by anyone outside the authors –
> expect rough edges. The English text is a translation of the Polish edition, which is the source of truth when the two differ;
> the licence terms are set solely by the English text of `LICENSE`.
> Documents also mention files that exist only in the full template: `docs/poza-rdzeniem.md` lists them.

> **Responsibility.** This template is provided "as is" (Apache-2.0, sections 7–8). It is not legal, tax, financial or
> data-protection advice. AI agents make mistakes. By running a fleet you become the provider and/or deployer of an AI system
> and you alone are responsible for human oversight, for what the agents do (payments, refunds, customer messages,
> publications) and for complying with the law (including the EU AI Act – Art. 50: people must know they are interacting with
> AI – and the GDPR) and with your providers' terms (Anthropic, GitHub and others). Legal, tax, credit, employment and other
> high-risk decisions listed in Anthropic's Usage Policy require review by a qualified person before use. Have a lawyer review
> your configuration before you enable external actions. Descriptions of Anthropic's terms in this repository are our reading
> as of 10 November 2026 – check the current wording.

## What is in the core
| Mechanism | Where |
|---|---|
| Fail-closed permissions hook – every external action is checked against `uprawnienia.yaml`, not against the agent's judgement | `.claude/hooks/uprawnienia.py` |
| Tamper-evident audit log (SHA-256 chain) and a HALT file that stops the fleet | `bezpieczenstwo/` |
| Watchdog without an LLM – missed runs, stuck work, unanswered orders; runs in GitHub Actions every 15 minutes | `moduly/straznik/` |
| Merge queue – a script computes the facts, a merge agent merges, the hook enforces the reviewer's verdict | `scripts/scalanie.py`, `ceo/scalanie/` |
| Run contract – every run ends with one JSON line: result, handled items, what is left | `pm/specyfikacje/kontrakt-przebiegu.md` |
| Cost measurement – tokens × price list per run, with a monthly projection | `scripts/koszt.py` |
| Role evals, including adversarial scenarios | `ewaluacje/` |
| Template updates as a weekly pull request, and improvement proposals back to this repository | `scripts/aktualizuj-z-szablonu.sh`, `scripts/propozycje.py` |
| A minimal start: five scheduled tasks (`limit`, `scalanie`, `ceo`, `recenzent`, `pm`); `architekt` – weekly updates and proposals – is added next | `flota/zadania.yaml` |

Not in the core (part of the full template): the library of further roles, managerial controls and KPIs, the profitability
model, company policies and the business processes – see `docs/poza-rdzeniem.md`.

## Quick start
```bash
gh repo create <you>/<company>-agents --private --template Gilbertus-master/company-of-agents --clone
cd <company>-agents
scripts/init-org.sh --nazwa "New Company" --domena newcompany.example --prefiks nc \
  --github <you>/<company>-agents --operator operator@newcompany.example --runtime claude-routines
python3 flota/generuj.py
bash scripts/testy.sh
```
Then follow `docs/instrukcja-uruchomienia.md` (set-up guide): the fleet account, billing (`team` or `api`), scheduled tasks, cost
measurement. The fleet runs on **one** account; do not rotate it across personal subscriptions.

## Claude Code plugin
The core skills also ship as a plugin (`plugins/`, described in `plugins/<name>/README.md`) – this repository is its marketplace:
```
/plugin marketplace add Gilbertus-master/company-of-agents
/plugin install <name>@<name>
```
where `<name>` is the name of this repository. The `new-organisation` skill creates the company repo and walks through the first start.
The plugin ships no hooks or permissions – the fleet's safeguards work inside the company repo.

## Sharing improvements – OFF by default
This project can learn from the fleets that use it – but only when the company's owner deliberately enables it.
Until you do, nothing leaves your repository.

| | |
|---|---|
| **Turning it on** | An owner decision recorded in `decyzje/`, then in `org.yaml`: `repo.dziel_sie_ulepszeniami: true` and `repo.dziel_sie_decyzja: "D-…"` (the id of that decision). Without a decision id the script sends nothing to this repository. |
| **What may be sent** | Changes you made to *mechanism* files relative to the template version: `scripts/`, `.claude/skills/`, `.claude/hooks/`, `procesy/`, `role/`, `ewaluacje/`, `moduly/`, `flota/szablony/`, `flota/generuj.py`, `pm/narzedzia/`, `pm/specyfikacje/`, `recenzent/skrypty/`, `toolbelt/katalog.md`, `.github/workflows/` – and rules from agent playbooks that helped at least 3 times and never hurt (the rule text only). |
| **What the mechanism does not collect** | Files outside that list: `org.yaml`, policies, decisions, orders, run logs, customer or financial data. If such content ends up inside a listed file (a script or a role description, for example), it will be collected – which is why there is a gate and a preview. |
| **Gate** | Before sending, `scripts/propozycje.py sprawdz` rejects a proposal in which it finds your company name, domain, GitHub owner, contact address, e-mail addresses, phone numbers, Polish personal (PESEL), tax or bank numbers, or secrets. The gate is pattern-based and cannot recognise every form of confidential text. |
| **When and where** | Mondays, by the architect agent (so nothing is sent until you add that agent), as an **Issue in this repository – which is public**: anyone can see the proposal and the GitHub account it came from. To send to a private repository of your own instead, set `repo.skrzynka_propozycji`. |
| **Licence** | Sent content is a Contribution within the meaning of section 5 of the Apache-2.0 licence: you license it under the Apache-2.0 terms, including the patent licence in section 3. For content already sent this licence cannot be withdrawn. Every proposal carries a statement of the owner's consent and of compliance with the Developer Certificate of Origin 1.1. Send only what you have the rights to. |
| **What the publisher does** | Proposals are assessed by the publisher's AI fleet under human oversight. Accepted improvements land in this repository under Apache-2.0; the publisher may also use them in paid editions of the template. Data-processing notice: `PRIVACY.md`. |
| **Preview** | `python3 -I scripts/propozycje.py zbierz` shows what would be sent; `propozycje/wyslane.jsonl` records what was. |
| **Turning it off** | `org.yaml` → `repo.dziel_sie_ulepszeniami: false`. Nothing is sent after that; what has already been published stays public. |

There is no other telemetry.

## Updates
Every Monday the architect agent runs `scripts/aktualizuj-z-szablonu.sh --pr`: the latest release arrives as a pull request
that goes through the reviewer and the merge queue. Changes to `zasady.md` and `.claude/` wait for the owner's decision.

## Licence and contributing
Apache-2.0 (`LICENSE`, `NOTICE`). Contributions: `CONTRIBUTING.md` (DCO sign-off, no CLA). Vulnerabilities: `SECURITY.md`.
Dependencies: Python 3 with the standard library and PyYAML (MIT licence), `git`, `gh` – they are not part of this repository.

> **Trademarks and no affiliation.** This is an independent project. It is not made, sponsored, endorsed or supported by, or
> affiliated with, Anthropic PBC. Claude, Claude Code and Anthropic are trademarks of Anthropic PBC. GitHub, Stripe and other
> names mentioned belong to their respective owners and are used only to identify the services this template works with.
