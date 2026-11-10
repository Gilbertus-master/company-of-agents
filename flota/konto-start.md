# Starting the fleet on a Claude account (a new organisation or moving the fleet – no account rotation)

Checklist. Task source: `flota/zadania.yaml` → `python3 flota/generuj.py --konto <K> --wyjscie ~/<prefiks>-prompty-<k>`
(tasks in the app) or `--routines` (cloud tasks). CEO starting message: `ceo/handoff/START-nowe-konto.md`.
Fleet account, compliant billing (team/api), regulator and cost measurement: `ceo/handoff/konto-floty.md`. The fleet works on ONE account.
Guide for humans (step by step): `docs/instrukcja-uruchomienia.md`.

## 1. Operator – at the app (about 10 min, once per account)
0. Team plan (owner and operator; the fleet on the operator's premium seat) with usage credits enabled and a spend limit equal to
   `org.yaml → runtime.nadwyzka_usd_miesiac` (Organization settings) – or an API key with `api` billing.
1. The Claude app signed in to the fleet account → Code → the organisation repo folder; the model from the manifest; a permissions mode
   consistent with `.claude/settings.json` (Bypass disabled – the hook holds the boundaries). Scheduled tasks store the mode of the session
   they were created in – create them from a session started from the machine's keyboard.
2. **Connectors** (Settings → Connectors; tied to the account, not the machine) – only those the roles need
   (`flota/zadania.yaml → konektory`, `toolbelt/katalog.md`):

   | Connector | What for | Scope used by agents |
   |---|---|---|
   | Agent mail (Resend / AgentMail) | functional mailboxes | read; sending per `uprawnienia.yaml` (hook) |
   | Repo/tasks (GitHub) | PRs, Issues | through `gh` on the machine – independent of the account |
   | Payments (Stripe) | KPIs, accounting | read only; key with the Agent tag |
   | Database (Supabase etc.) | KPIs, product | aggregates; migrations per permissions |
   | Owner's Gmail | only with a decision D-… | read |
3. **Plugins** the roles need (`toolbelt/wtyczki-claude.md`) – also per account.
4. Unchanged on the machine: `gh`, the secrets manager, the repo.

## 2. CEO of the fleet account – tasks
```bash
cd <repo> && git pull --rebase --quiet
python3 flota/generuj.py --sprawdz                                   # prompts = manifest
python3 flota/generuj.py --konto B --wyjscie ~/<prefiks>-prompty-b    # prompts with "Konto: B" + list of calls
```
- Each JSON line = one task (`taskId` `<prefiks>-<agent>-<konto>`, cron, description, prompt from a file). **Create them in list order**
  (first `<prefiks>-limit-<konto>` – the pace regulator).
- After creating: "Run now" for the reviewer, customer service and the PM; check the mode of each task.
- When moving the fleet, the old account **disables** its tasks (does not delete them); the fleet account goes in `org.yaml → runtime.konto`.
- `scripts/przebieg.sh start <agent> <konto>` accepts any letter A–Z; limit file `~/.<prefiks>-limit-<K>.json`.

## 3. Check after the start (CEO, the same day)
1. The limit file exists after the regulator's first run and has `procent`, `cel_teraz`, `odchylenie`, `tryb`.
2. Watchdog (`moduly/straznik`): no "bez przebiegu" (no run) after the first window of each created agent.
3. Contract: no "brak narzędzi konektora" (connector tools missing) errors in `przebiegi/<agent>/` (connectors!).
4. `toolbelt/sprawdz-dostepy.md` filled in; `bezpieczenstwo/HALT` does not exist.
5. An entry in `ceo/handoff/konto-floty.md` (table) and the CEO log; after a week the first `python3 -I scripts/koszt.py raport`.

## Changing a task
Manifest → `python3 flota/generuj.py` → `python3 -m unittest discover flota` → commit the manifest and the prompts together → change
the tasks on **all** accounts (level `nowy_agent_lub_zadanie_cykliczne` in `uprawnienia.yaml`).
