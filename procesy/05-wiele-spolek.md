# Process 05 – several companies on one template

Goal: the owner runs several agentic organisations from one template, and each company is **isolated** (limits, secrets, data,
accountability) while still **receiving template fixes** without manual rewriting. Decision from the report (ch. 6, item 5): separate
accounts and a separate GitHub organisation per company.

## 1. Rule: one company = one identity

| Resource | Per company | Why separate |
|---|---|---|
| Claude account (plan with cloud tasks, DPA) | yes | limits and costs are billed per company; data processing agreement with the right controller (`bezpieczenstwo/rejestr-ai.md`) |
| GitHub organisation + private organisation repo `<spolka>-agenci` (+ product repo) | yes | Read/Triage/Write roles and teams exist only within an organisation; commit accountability |
| Domain and functional mailboxes, secrets manager (separate vault), payment keys, toolbelt service accounts | yes | one company's secret never sits next to another's; an incident does not spread between companies |
| Agents' technical account (GitHub App / service account) | yes | company A's token does not open company B's repo |
| Operator and owner | may be the same people | the roles stay separated in each company (`org.yaml → ludzie`); the "operator minutes" KPI is counted per company |
| Template `agentyczna-organizacja` | shared (upstream) | one place for fixes; companies share nothing apart from it |

Nothing is shared "as a shortcut" (one key, one mailbox for two companies) – that is always an owner decision `D-…` with a justification
and an entry in the company register.

## 2. Starting a company from the template

1. **[właściciel]** (owner) A row in the company register (section 5) with the name, mandate and fleet budget.
2. **[operator]** GitHub organisation, repo `<spolka>-agenci` from the template (`gh repo create … --template` or clone + `git remote rename origin szablon`),
   repo **private**; the company's Claude account; secrets manager (the company's vault).
3. `scripts/init-org.sh …` (process 00, stage 0) – fills in `org.yaml`; in `org.yaml → repo.szablon` enter the upstream (`Gilbertus-master/company-of-agents`
   or a full URL) – this is the source for updates.
4. Then as in process 00 (stages 1–4). Each company has its own `decyzje/`, `polityki/`, `uprawnienia.yaml`, agents and fleet.

## 3. Updating from the template

Script: `scripts/aktualizuj-z-szablonu.sh [--sucho] [--url …] [--galaz main]` in the company repo (test: `scripts/test-aktualizuj.sh`).
1. Adds/updates the `szablon` remote (URL from the argument or `org.yaml → repo.szablon`), fetches the template branch.
2. Shows `git diff --stat` of the template changes **without local paths**; `--sucho` (dry run) stops here.
3. Requires a clean working tree; merges with `git merge --no-commit`; for paths in `.szablon-lokalne` restores the company's version
   (`git checkout HEAD -- <plik>`), and **skips and lists new template files in local paths** (the operator copies them by hand if wanted).
4. Conflict outside local paths → the merge stays in progress, the script lists the files and instructions (resolve → `git add` → `git commit`
   or `git merge --abort`), exit code 3. No conflicts → commit `szablon: aktualizacja do <sha7>`.
5. After the merge: `python3 flota/generuj.py && python3 flota/generuj.py --routines` (prompts and task instructions are local – generated
   from the company's manifest), tests from `README.md`, PR to the company's main branch → verdict of the company's reviewer → merge through the
   company's merge queue (`ceo/scalanie/README.md`): such a PR touches excluded paths (class WYJATEK), so the CEO merges it under the lock, and when
   `zasady.md` / `uprawnienia.yaml` change – only with an owner decision.

**Rhythm:** once a month (the first `PEŁNY` (full) CEO run after the agent review – process 50) or on an event, when the template announces a
security fix (hook, watchdog). Done by the company's CEO in a PR or by the operator; the update is not an external action, merging the PR is
(`pr_scalenie_bez_migracji`).
After every update (and at least once a week) the company's CEO measures how closely the skeleton matches with `python3 -I scripts/zgodnosc.py`
and sends a `zgodnosc` record (procesy/16-wymiana-ceo.md, section 6); a permanent deviation = an entry in `.szablon-lokalne` with the reason in a comment.

### What may be overridden locally and what flows from the template

| Flows from the template (merge; conflict = question to the operator) | Local (`.szablon-lokalne` – the template does not change them) |
|---|---|
| `zasady.md` – the constitution (a change to section A in the template = the owner of **each** company confirms with a decision `D-…` at merge) | `org.yaml`, `uprawnienia.yaml`, `.szablon-lokalne` |
| `.claude/` (permissions hook, skills, settings), `moduly/straznik/`, `scripts/`, `flota/generuj.py`, `flota/szablony/`, tests | `polityki/`, `decyzje/`, `decyzje.md`, `ryzyka.md` (company content), `backlog.md`, `rejestr-sesji.md` |
| `role/*.md`, `role/szablony/`, `procesy/`, `docs/`, `toolbelt/*.md` (catalogue, rules), `ewaluacje/_startowe/`, `ewaluacje/*.py` | `koordynacja-odstepy.tsv`, `flota/zadania.yaml`, `flota/etykiety.tsv`, `ceo/handoff/prompty/`, `ceo/handoff/routines/` (generated) |
| `pm/specyfikacje/`, `pm/narzedzia/`, `ceo/kpi/kpi.md` (F1–F12 skeleton – the company adds its own O/K/A in separate sections; conflict possible) | agent directories: `*/agent.md`, `*/dziennik.md`, `*/stan.json`, `*/backlog.md`, `*/wiedza/`, `*/analizy/`, `*/szkice/`, role registers |
| `README.md`, `PRZECZYTAJ-NAJPIERW.md`, `CLAUDE.md` | `zlecenia/`, `przebiegi/`, `incydenty/`, `bezpieczenstwo/audyt/`, `bezpieczenstwo/blokady.jsonl`, `bezpieczenstwo/rejestr-ai.md`, `toolbelt/mcp-odciski.tsv`, `ewaluacje/<agent>/`, `operator/kolejka.md`, `ceo/do-decyzji.md`, `ceo/plan/`, `ceo/straznik-zdarzenia.md`, `ceo/scalanie/stan.json`, `ceo/scalanie/dziennik.jsonl`, `ceo/kontrole/wyniki.jsonl`, `recenzent/szablony/`, `.github/workflows/straznik.yml` (`init-org.sh` turns on its schedule) |

A company may add a path to `.szablon-lokalne` (e.g. its own process `procesy/45-…` – it does not collide anyway) or remove one (e.g. it wants
to take `uprawnienia.yaml` from the template) – that is a decision of the company's CEO recorded in the log. Changing `role/<rola>.md` locally is allowed,
but it is better to generalise it and send it to the template (section 4), and keep local refinements in `<agent>/agent.md` (local).

## 4. Changes flowing upstream (company → template)

A company's lesson (incident, playbook rule, better register, new watchdog check) goes into the template by PR **after generalisation**:
no company name, project IDs, addresses, people, amounts – "owner", "operator", "product repo". Author: a company agent (order from the CEO) or
the operator; the template reviewer gives a verdict; template tests green. Never the other way round: company data does not go into the template, and the
template receives no secrets and no files from `.szablon-lokalne`.

## 5. Company register kept by the owner

Kept by the owner **outside the company repositories** (each company sees only itself): a private owner repo or a document manager.
No personal data and no secrets – names of entries in the secrets manager, not logins.

| Company | Organisation repo | GitHub organisation | Claude account (vault entry) | Operator (role / SLA) | Template version (sha7 / tag) | Last update | Fleet budget USD/month | Notes |
|---|---|---|---|---|---|---|---|---|

Review: once a month by the owner (30 min, with the companies' CEO reports): companies with a template version more than 2 updates old → update
order; shared lessons → PR to the template; nothing is shared apart from the template.
