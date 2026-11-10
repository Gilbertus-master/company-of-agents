# Setup guide – a new company, the fleet account and updates

Step by step: how to set up an agentic organisation from this template, run the fleet in a variant that is permitted under our reading of Anthropic's terms (as of 10 November 2026; this is not legal advice – you are responsible for compliance with the current terms), measure
its cost and keep it on the latest release of the template. Two people work in the organisation: the **owner** and the **operator**.
Details: `ceo/handoff/konto-floty.md`, `flota/konto-start.md`, `procesy/00-nowa-organizacja.md`, `procesy/15-rozwoj-szablonu.md`.

## 0. Three things you must not confuse
| What | What it is for | Where it is set |
|---|---|---|
| **Fleet account** (one) | the operator's seat on the Team plan (or an API key) on which all agent tasks run; scheduled tasks, connectors and plugins belong to it | the Claude app or `claude` → `/login`; `org.yaml → runtime.konto` |
| **GitHub account** | access to the template and the company repo – through `gh` on the machine, independent of the Claude account | `gh auth login` |
| **Company repo** | the organisation's memory; a copy of the template with updates as PRs | a copy of the template |

**No account rotation.** The fleet does not switch to another person's account that has free limit: Pro/Max/Ultra plans are for
non-commercial use and must not be shared, and on the Team plan every seat belongs to one named person and limits must not be
transferred or shared (`ceo/handoff/konto-floty.md`).

## 1. Once per machine
1. `git`, `gh` (`gh auth login`), Python 3 (CI uses 3.12), the Claude app or Claude Code CLI.
2. A GitHub account allowed to create private repositories (the template is public).
3. The company repo lives at the path from `org.yaml → repo.organizacja` – prompts contain this path. Different path → fix `org.yaml`
   and run `python3 flota/generuj.py`.

## 2. Billing – choose one compliant option (`org.yaml → runtime.rozliczenie`)
| Option | What to buy | Settings |
|---|---|---|
| `team` (default) | Team plan: 2 seats (owner, operator – premium seat), usage credits and a spend limit enabled in Organization settings | `konto: "A"`, `cena_planu_usd_miesiac`, `nadwyzka_usd_miesiac` = spend limit at Anthropic |
| `api` | an API key from Claude Console; the fleet runs through `scripts/przebieg-headless.sh` (Agent SDK) with its own schedule | `glowny: agent-sdk`, USD budgets in `flota/zadania.yaml`, `mandat.budzet_floty_usd_miesiac` (HALT) |
| Usage-based Enterprise | a larger company, everything at API rates (terms are set by Anthropic) | as `api` |

Once the seat's weekly limit is used up, the fleet keeps working only on paid overage; without overage only P1 runs until the reset.

## 3. A new company – once
```bash
gh repo create <OWNER>/<firma>-agenci --private --template Gilbertus-master/company-of-agents --clone
cd <firma>-agenci
scripts/init-org.sh --nazwa "Nowa Firma" --domena nowafirma.example --prefiks nf \
  --github <OWNER>/<firma>-agenci --operator operator@nowafirma.example --runtime claude-routines   # or desktop-scheduled / agent-sdk
# org.yaml → runtime: rozliczenie, konto, cena_planu_usd_miesiac, nadwyzka_usd_miesiac; repo: szablon, skrzynka_propozycji
python3 flota/generuj.py        # agent prompts and budgets (flota/budzety.tsv)
git add -A && git commit -m "start organizacji Nowa Firma" && git push
gh secret set STRAZNIK_TOKEN    # fine-grained token: read PRs of the product repo, write to the organisation repo
```
Output of `init-org.sh`: `org.yaml`, the starting decision `D-<today>-01`, core agents (ceo, pm, recenzent), Issue labels, prompts,
tests ✓ and a list of tasks to create. The watchdog starts by itself in GitHub Actions every 15 min.

## 4. The fleet on the fleet account
1. **Sign-in:** the Claude app on the fleet account → Code → the company repo folder; a session started from this machine's keyboard
   (scheduled tasks remember the permissions mode). Bypass is disabled (`.claude/settings.json`), the hook holds the boundaries.
2. **Account:** connectors and plugins the roles need (`flota/konto-start.md`). The class models from `org.yaml → runtime.modele` must be
   available on the account; a model missing → change the class mapping and run `python3 flota/generuj.py`.
3. **First message:** the starting prompt (section 5).
4. **Core tasks** (order: `limit`, `scalanie`, `ceo`, `recenzent`, `pm`):
   - in the app: `python3 flota/generuj.py --konto A --wyjscie ~/<prefiks>-prompty-a --tylko limit,scalanie,ceo,pm,recenzent`
     → one JSON line per task; the CEO of this session creates them;
   - in the cloud: `python3 flota/generuj.py --routines --konto A --wyjscie ~/<prefiks>-routines-a --tylko limit,scalanie,ceo,pm,recenzent`
     → one file per task; you create them at claude.ai/code/routines or with `/schedule`.
5. **"Run now"** each task once – otherwise it will stall on a consent prompt.
6. **Same-day check:** lines in `przebiegi/<agent>/`, files `przebiegi/<agent>/koszty-*.jsonl`, the watchdog without "bez przebiegu"
   (no run), no `bezpieczenstwo/HALT`. Further roles join in stages (`procesy/00`, 3-day gate).

## 5. Starting prompt – a new company
```text
You are the CEO agent of the organisation <NAZWA> on fleet account A. First set the session role: echo ceo > .agent-rola.
Read PRZECZYTAJ-NAJPIERW.md, zasady.md, uprawnienia.yaml, org.yaml, ceo/handoff/konto-floty.md and flota/konto-start.md.
Then: python3 flota/generuj.py --konto A --wyjscie ~/<prefiks>-prompty-a --tylko limit,scalanie,ceo,pm,recenzent
and create the scheduled tasks in the order of the list (limit first, then scalanie). Run each one once.
Commit, push. Finally write briefly to the operator: what works, what is in progress, what needs a human.
```

## 6. Fleet cost – measured from day one
- Every run records its cost: the `SessionStart` hook remembers the session transcript, and at the end of the run the agent (and the
  `SessionEnd` hook) runs `python3 -I scripts/koszt.py zapisz` – tokens × `flota/ceny-modeli.yaml` → `przebiegi/<agent>/koszty-<data>.jsonl`.
- Report: `python3 -I scripts/koszt.py raport --dni 14` – cost per agent and model, monthly projection at API prices, comparison
  with the price of seats and overage. After 14 days enter the measured seat capacity (`runtime.pojemnosc_miejsca_usd_miesiac`) and run
  `python3 flota/generuj.py` – agent budgets become shares of the fleet account's capacity.

## 7. Updates and template development (process 15)
- **Every Monday the architect** runs `scripts/aktualizuj-z-szablonu.sh --pr`: the latest release (`repo.szablon_kanal: stabilny`)
  on a separate branch, prompts, tests, PR → reviewer → merge queue. Changes to `zasady.md` and `.claude/` wait for an owner decision.
- **Every Monday the architect** submits improvements: `python3 -I scripts/propozycje.py zbierz` → `sprawdz` → `wyslij` – only changes
  to mechanisms and proven rules, through the anonymisation gate, to a private inbox (`repo.skrzynka_propozycji`) or – in the free core – to the public template repository,
  with consent `repo.dziel_sie_ulepszeniami: true` (off by default; sending to the shared repository also requires the owner's decision in `repo.dziel_sie_decyzja`).

## 8. Day to day and when something does not work
| Situation | What to do |
|---|---|
| Owner – decisions | `ceo/do-decyzji.md`: options, effect, recommendation; consent = file `decyzje/D-RRRRMMDD-NN.md` |
| Operator – tasks | `operator/kolejka.md`: ready steps, deadline, expected result |
| Talking to the CEO | chat in the repo folder: `echo ceo > .agent-rola`, orders through `zlecenia/ceo.md` |
| Fleet account limit used up | the regulator appends Pilne (urgent); with overage work continues, without it only P1 until the reset; persistently too little – owner decision (overage, seat, `api`) |
| `bezpieczenstwo/HALT` exists | the fleet is stopped; the operator reads the reason and deletes the file after review |
| `przebiegi/dead-letter/<agent>.md` | 3 errors in a row; cause → fix → the operator deletes the file |
| Alert "bez przebiegu" (no run) | the task on the fleet account: enabled? model, mode, connectors? |
| Manifest change (`flota/zadania.yaml`) | `python3 flota/generuj.py`, commit, change the tasks on the fleet account |
| Update PR with a conflict | the architect merges manually on a working branch (`scripts/aktualizuj-z-szablonu.sh` without `--pr`) and opens a PR |
| Moving the fleet to another account (change of operator or plan) | `ceo/handoff/START-nowe-konto.md`; the old account disables its tasks |
