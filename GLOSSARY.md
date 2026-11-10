# Glossary – Polish names used in the code

Both editions run the same code, so file names, directories, YAML keys, commands, statuses and script output are Polish.
This page gives the meaning of every Polish name you meet when setting up and running the core. Do not rename them:
scripts, tests, the permissions hook and the watchdog match these strings exactly.

## Start here
| Polish | Meaning |
|---|---|
| `PRZECZYTAJ-NAJPIERW.md` | "read first" – entry point for every agent and human |
| `zasady.md` | rules – security rules (section A) and working rules |
| `uprawnienia.yaml` | permissions – what each agent may do outside the repo; `czynnosci` = actions |
| `org.yaml` | company data and runtime settings (keys below) |
| `docs/instrukcja-uruchomienia.md` | set-up guide |
| `docs/poza-rdzeniem.md` | what is not in the core |

## Directories and registers
| Polish | Meaning |
|---|---|
| `flota/` | fleet – manifest `zadania.yaml` (tasks), prompt templates `szablony/`, generator `generuj.py`, price list `ceny-modeli.yaml`, budgets `budzety.tsv`, labels `etykiety.tsv` |
| `role/` | role instructions; `role/szablony/` = starter registers copied for a new agent |
| `procesy/` | processes (`00` new organisation, `05` several companies, `15` template development, `60` agent life cycle, `90` incident) |
| `polityki/` | policies; `progi.yaml` = thresholds |
| `decyzje/`, `decyzje.md` | decisions (one file each, `D-RRRRMMDD-NN` = D-YYYYMMDD-NN) and their generated index |
| `zlecenia/` | orders – one file per recipient agent; how agents give each other work |
| `przebiegi/` | runs – one JSON line per run, per agent; `koszty-<data>.jsonl` = costs; `dead-letter/` = agents stopped after 3 errors |
| `backlog.md` | backlog kept by the PM |
| `koordynacja.md`, `koordynacja-odstepy.tsv` | coordination – who works where now (locks), and the minimum gaps between runs |
| `rejestr-sesji.md` | session register – developer sessions opened by agents |
| `ryzyka.md` | risk register |
| `incydenty/` | incidents; `_szablon.md` = template |
| `bezpieczenstwo/` | security – audit log, `HALT` file (stops the fleet), `rejestr-ai.md` = AI system register |
| `ewaluacje/` | evals – role scenarios; `sprawdz.py` = check, `uruchom.py` = run |
| `moduly/straznik/` | modules / watchdog – checks without an LLM, run by GitHub Actions |
| `ceo/do-decyzji.md` | "for decision" – questions waiting for the owner |
| `ceo/scalanie/` | merge queue; `ceo/handoff/` = fleet account hand-over and generated `prompty/` (prompts) |
| `ceo/wiedza/podrecznik.md` | knowledge / playbook – rules an agent has learned |
| `ceo/kontrole/progi.yaml` | control thresholds |
| `ceo/straznik-zdarzenia.md` | watchdog events delivered to the CEO |
| `operator/kolejka.md` | operator queue – tasks only a human may do |
| `pm/dyzur.md`, `pm/specyfikacje/`, `pm/narzedzia/` | PM duty log, specifications (incl. `kontrakt-przebiegu.md` = run contract), tools |
| `recenzent/` | reviewer – `stan.json` (state: verdicts), `skrypty/` (scripts), `szablony/` (templates) |
| `toolbelt/` | tools: `katalog.md` (catalogue), `sekrety.md` (secrets), `sprawdz-dostepy.md` (access check), `wtyczki-claude.md` (plugins), `obserwowalnosc.md` (observability) |
| `.szablon-lokalne` | paths that template updates must not overwrite |
| `.agent-rola` | file holding the role of the current session (`echo ceo > .agent-rola`) |

## Agents, model classes, priorities
| Polish | Meaning |
|---|---|
| `limit` | rate-limit regulator (every 30 min) |
| `scalanie` | merge agent (every 15 min) |
| `ceo`, `pm` | CEO agent, project manager |
| `recenzent` | reviewer |
| `architekt` | architect – weekly template updates and improvement proposals |
| `sedzia` · `mocny` · `lekki` · `mechaniczny` | model classes: judge · strong · light · mechanical (IDs only in `org.yaml → runtime.modele`) |
| `P1` · `P2` · `P3` | priority; when the account limit runs low, P3 stops first, then P2 |
| `wlasciciel` · `operator` | owner (decisions) · operator (accounts, secrets, manual steps) |

## Run modes, results, statuses
| Polish | Meaning |
|---|---|
| `PEŁNY` | full run |
| `ZDARZENIE` | event-only run (reacts to new items, no planned work) |
| `CISZA` | silent run – nothing to do, or budget exhausted; one line and exit |
| `wynik` | result of a run: `OK` · `ROZBIEŻNOŚĆ` (discrepancy) · `NIEPEŁNY` (incomplete – a source was not read) · `BŁĄD` (error) |
| `Pilne` | urgent – section and marker for orders handled first |
| `GOTOWY` | ready – PR class that the merge agent may merge |
| `HALT` | file `bezpieczenstwo/HALT`: the whole fleet stops until the operator removes it |
| `BUDŻET-AGENTA` | agent's daily budget exceeded – the agent works in `ZDARZENIE` mode only |
| `bez przebiegu` | "no run" – watchdog alert: a scheduled task did not report in its window |
| `blokada` | lock (coordination lock, merge lock) |
| `wstrzymaj` | hold – pauses the merge queue |
| `werdykt` | reviewer's verdict stored in `recenzent/stan.json` |

## `org.yaml`
| Key | Meaning |
|---|---|
| `organizacja.nazwa` · `podmiot` · `domena` · `jezyk` · `strefa` · `prefiks` | name · legal entity · domain · working language · time zone · task prefix |
| `repo.organizacja` · `github` · `produkt` | local path of this repo · its GitHub name · product code repositories |
| `repo.szablon` · `szablon_kanal` | upstream template · update channel (`stabilny` = latest release, `glowny` = main branch) |
| `repo.dziel_sie_ulepszeniami` · `skrzynka_propozycji` | share improvements (true/false) · private proposal inbox |
| `ludzie.wlasciciel` · `ludzie.operator` | the two humans |
| `mandat.cel` · `budzet_miesieczny_pln` · `prog_zgody_pln` · `budzet_floty_usd_miesiac` | goal · monthly spending limit · single-expense approval threshold · fleet token-cost ceiling (`api` billing) |
| `runtime.glowny` | how tasks are run: `claude-routines`, `desktop-scheduled`, `agent-sdk` |
| `runtime.rozliczenie` · `konto` | billing (`team` or `api`) · letter of the fleet account |
| `runtime.pojemnosc_miejsca_usd_miesiac` · `nadwyzka_usd_miesiac` · `wykorzystanie_cel_proc` · `cena_planu_usd_miesiac` | measured seat capacity · paid overage · share of capacity given to agents · plan price |
| `runtime.modele` · `modele_sprawdzono` · `okno_celu_h` | class → model ID · date of last review · pacing window in hours |
| `straznik.kontrole` · `uruchamianie` | watchdog checks (`przebiegi` runs, `pr`, `zlecenia` orders, `konfiguracja` configuration, `budzety` budgets, `martwe` dead agents) · how it is started |

## `flota/zadania.yaml` (fleet manifest)
| Key | Meaning |
|---|---|
| `zadania` | tasks – one entry per agent |
| `katalog` · `priorytet` · `cron` · `konto_w_prompcie` | agent directory · priority · schedule · account letter written into the prompt |
| `prompt` · `tresc` · `opis` | generated prompt file · content template · description |
| `budzet_usd_przebieg` · `_dzien` · `_miesiac` · `max_tur` | budget per run · per day · per month · maximum turns |
| `typ` · `wlasciciel_ludzki` · `narzedzia` · `domeny` · `autonomia` | agent type (`reader`/`actor`/`researcher`) · human owner · tools · allowed domains · autonomy (`auto`, `aprobata` approval, `eskalacja` escalation) |
| `efekt` · `konektory` | effort · connectors |

## Commands
| Command | Meaning |
|---|---|
| `bash scripts/testy.sh [words]` · `--lista` | run the test groups (optionally those whose name contains a word) · list them |
| `scripts/init-org.sh --nazwa --domena --prefiks --github --operator --wlasciciel --strefa --runtime --dziel-sie tak\|nie --bez-etykiet --tak` | initialise a company: name, domain, prefix, repo, operator, owner, time zone, runtime, share improvements with the template yes\|no (mandatory, the owner's choice), skip labels, no questions |
| `python3 flota/generuj.py` · `--sprawdz` · `--konto` · `--wyjscie` · `--tylko` · `--lista` · `--routines` | generate prompts and budgets · check they match the manifest · account · output directory · only these agents · list task calls · cloud routine files |
| `scripts/nowy-agent.sh` | add a new agent (role, registers, manifest entry) |
| `scripts/przebieg.sh` · `przebieg-headless.sh` | run wrapper (lock, windows, mode) · headless run through the Agent SDK |
| `python3 -I scripts/scalanie.py` | merge queue facts and PR classes (never merges) |
| `python3 -I scripts/koszt.py sesja` · `zapisz` · `raport --dni N` | cost: remember session · record · report for N days |
| `python3 -I scripts/regulator.py --konto --procent --reset` | pacing regulator: account, per cent of limit used, reset time |
| `python3 -I scripts/propozycje.py zbierz` · `sprawdz` · `wyslij` | improvement proposals: collect · check (anonymisation gate) · send |
| `scripts/aktualizuj-z-szablonu.sh --pr` · `--sucho` · `--kanal` · `--galaz` | update from the template as a PR · dry run · channel · branch |
| `python3 -I scripts/decyzje-indeks.py` · `--sprawdz` | rebuild the decision index · check it |
| `/przebieg` · `/dyzur-pm` · `/recenzja` · `/incydent` | skills: run · PM duty · review · incident |

## Words that recur in script output
`brak` none/missing · `błąd` error · `ostrzeżenie` warning · `pominięte` skipped · `czerwone` failing (red) · `zielone` passing ·
`gałąź` branch · `scalono` merged · `otwarte` open · `nowe` new · `obsłużone` handled · `przeczytane` read · `rozbieżności`
discrepancies · `akcje` actions · `raport` report · `dowód` evidence · `termin` deadline · `stan` state · `wpis` entry ·
`dzień` day · `tydzień` week · `miesiąc` month · `konto` account · `sesja` session.
