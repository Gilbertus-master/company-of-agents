# Fleet watchdog

A deterministic script (no LLM) that every 15 minutes compares what the agent fleet **was supposed to do** with what
is **actually visible** in the organisation repo and on GitHub, and raises an alert on every new discrepancy: an agent
did not start in its window, a PR is waiting for a verdict, an order passed its deadline without a reply, a budget ran
out, runs keep ending in an error.

The watchdog does not judge the quality of work and fixes nothing. It reports – to the operator by e-mail and
notification, to the CEO agent with a `Pilne` (urgent) entry in `zlecenia/ceo.md` – and in two cases it pulls the brake:
it pauses an agent after a series of errors and stops the fleet when the organisation budget is exhausted. Why a script
and not another agent: `docs/adr/0003-straznik-bez-llm.md`.

Paths in this document are relative to the root directory of the organisation repo.

## In brief

| | |
|---|---|
| Checks | 6 in the core: `przebiegi`, `pr`, `zlecenia`, `konfiguracja`, `budzety`, `martwe`; 2 plugins disabled by default: `testflight`, `automaty`; plus a check of its own clone (`klon`) |
| Cadence | every 15 min – GitHub Actions (recommended) or launchd on a Mac |
| Configuration | `org.yaml` in the organisation repo; `STRAZNIK_…` environment variables take precedence |
| Requirements | Python 3.9+ (standard library only, no pip), `git`, `gh` |
| Alert channels | e-mail (Resend), macOS notification (launchd only), a `Pilne` entry in `zlecenia/ceo.md` |
| Secrets | environment variables (CI) or the macOS keychain; never in files, logs or error messages |

### Module files

| File | Role |
|---|---|
| `straznik.py` | one run: clone update, configuration, checks, alerts, state write, commit of its own files |
| `zrodla.py` | data fetching (repo files, `git`, `gh`, plugin sources), reading `org.yaml`, secrets |
| `kontrole.py` | checks as pure functions `(dane, teraz) -> wynik` – no network and no writes |
| `alerty.py` | alert rules, mutes, e-mail body, notification, entry in `zlecenia/ceo.md` |
| `instaluj.sh`, `straznik.plist.szablon` | installing and uninstalling the launchd agent (macOS) |
| `test_straznik.py`, `testdane/` | tests and fictitious sample data (no network) |

### Requirements

- Python 3.9 or newer, standard library only (on a Mac the system `/usr/bin/python3` is enough).
- `git` – run tags in origin, clone update, commits of the watchdog's own files.
- `gh` (GitHub CLI), logged in or with a token in the `GH_TOKEN` variable – open PRs and commit dates.
- macOS only, optional: `security` (secrets from the keychain) and `osascript` (notification). On other systems secrets
  are supplied as environment variables and the notification is skipped.

## Quick start

In the root directory of the organisation repo:

```bash
# 1. Sample data from the module (fictitious): no network and no writes, result as JSON on stdout
python3 moduly/straznik/straznik.py --dane moduly/straznik/testdane --sucho

# 2. Trial on real sources: JSON on stdout, no file writes, e-mails, notifications or commits
python3 moduly/straznik/straznik.py --sucho --repo .

# 3. Module tests (no network)
python3 -m unittest discover moduly/straznik
```

Enabling it permanently is described in the "Running" section.

## How a single run works

1. **Clone update** – `git pull --no-rebase` in the working repo (real run only; details in the section
   "Klon wymaga uwagi").
2. **Configuration** – reading `org.yaml` from the repo; `STRAZNIK_…` variables override values from the file.
3. **Sources** – repo files are read locally, network sources (`git ls-remote`, `gh`, plugin sources) are fetched
   in parallel. A source that did not respond does not abort the run – its check gets a source-error status.
4. **Checks** – each active check is computed separately; an exception in one does not stop the others.
5. **Alerts** – comparison with the previous state (`ceo/straznik.json`) and with mutes from `zlecenia/ceo.md`; an alert
   only when something has changed or a reminder is due.
6. **Effects** – e-mail, notification, `Pilne` entry, circuit breakers (dead-letter, `HALT`), lines in the event log.
7. **Write** – commit and push of the watchdog's own files only, new state in `ceo/straznik.json`, one summary
   line on stdout: `<czas> <kontrola>=<STATUS>(<liczba ID>) … · alerty: N · X s`.

## Configuration (`org.yaml`)

The watchdog reads `org.yaml` from the repo it checks. Keys it uses:

| Key | Meaning for the watchdog | When the key is missing |
|---|---|---|
| `organizacja.prefiks` | prefix of scheduled task names `<prefiks>-<agent>-<konto>` (check `konfiguracja`) | `org` |
| `organizacja.strefa` | time zone: schedule windows, order deadlines, night-time quiet hours, dates in events and entries | `Europe/Warsaw` (also for an unknown zone name) |
| `organizacja.domena` | sender of alert e-mails: `Strażnik floty <straznik@<domena z org.yaml>>` | `alerty@example.com` – set the domain or `STRAZNIK_MAIL_OD` |
| `ludzie.operator.kontakt_firmowy` | recipient of alert e-mails | `ceo@example.com` – set the address or `STRAZNIK_MAIL_DO` |
| `repo.github` | `<właściciel>/<repo organizacji>`: owner of all repos on GitHub and the organisation repo in the `pr` check | placeholder names – the `pr` check will report a source error |
| `repo.produkt` | product repo – a list `[właściciel/nazwa, …]` or a single name. The `pr` check covers all of them and the organisation repo; the first in the list is the repo whose verdicts have a key without a prefix | only the organisation repo is checked |
| `runtime.glowny` | `desktop-scheduled` = the fleet runs as scheduled tasks of the desktop app; with any other value the `konfiguracja` check is skipped | `desktop-scheduled` |
| `runtime.konta` | accounts for which the `konfiguracja` check expects a separate task for each agent | `[A]` |
| `straznik.kontrole` | list of active checks; unknown names are ignored | 6 core checks |
| `straznik.pomin_tytul` | regular expression for PR titles skipped in the `pr` check (match anywhere in the title) | only drafts are skipped |
| `mandat.budzet_floty_usd_miesiac` | monthly spending limit of the whole fleet in USD (check `budzety`) | no organisation limit |

Example:

```yaml
organizacja:
  domena: "example.com"
  strefa: "Europe/Warsaw"
  prefiks: "firma"
ludzie:
  operator:
    kontakt_firmowy: "operator@example.com"
repo:
  github: "wlasciciel/firma-agenci"
  produkt: ["wlasciciel/produkt", "wlasciciel/strona"]
runtime:
  glowny: "desktop-scheduled"
  konta: ["A"]
mandat:
  budzet_floty_usd_miesiac: 300
straznik:
  kontrole: [przebiegi, pr, zlecenia, konfiguracja, budzety, martwe]
  pomin_tytul: '^\[wstrzymane'
```

Notes:

- **Simplified YAML reading.** The watchdog does not use a YAML library. It understands `klucz: wartość`, nesting by
  indentation, `#` comments and lists written on one line (`[a, b]`). A block list (`- …` items on consecutive lines) is
  not read – the key then behaves as if unset. A quoted value is taken literally, without escape sequences; write a
  regular expression in single quotes – any other YAML parser will read it the same way.
- **A repo without `org.yaml`** (or without the `organizacja` and `straznik` sections): default values and all 8 checks,
  plugins included.
- **`straznik.uruchamianie`** is information for humans and agents – the watchdog code does not read it. Where the
  watchdog runs is decided by an enabled schedule in GitHub Actions or an installed launchd agent.
- **An invalid expression in `pomin_tytul`** does not stop the check: the filter is disabled and the `pr` check gets a note.
- The remaining settings (working repo, task directory, plugins) are supplied as environment variables – table in the
  section "Environment variables".

## What the watchdog expects in the repo

| Path | Check | Notes |
|---|---|---|
| `koordynacja-odstepy.tsv` | `przebiegi`, `konfiguracja` | fleet schedule; an empty or unreadable file = source error |
| tags `przebieg/<agent>/<nr okienka>` in origin | `przebiegi` | pushed by `scripts/przebieg.sh` at the start of every agent run |
| `<agent>/agent.md` | `przebiegi` | sign that the role is live; an agent from the schedule without this file is not checked |
| `<agent>/stan.json` → `ostatni_przebieg` | `przebiegi` | missing file = note, not a discrepancy |
| `recenzent/stan.json` → `recenzje` | `pr` | the file must exist (`{}` is enough); without it the check reports a source error – create the file or remove `pr` from `straznik.kontrole` |
| `zlecenia/*.md` | `zlecenia` | entry format: `zlecenia/README.md`; the watchdog reads alert mutes from `zlecenia/ceo.md` |
| `przebiegi/<agent>/*.jsonl` | `budzety`, `martwe` | run contract lines (`pm/specyfikacje/kontrakt-przebiegu.md`) and cost files |
| `flota/zadania.yaml` | `budzety` | monthly agent limits (`budzet_usd_miesiac`) |
| scheduled task directory (outside the repo) | `konfiguracja` | by default `~/.claude/scheduled-tasks`; only with `runtime.glowny: desktop-scheduled` |

## Checks

### Check result

Every check returns:

- `status`: `OK`, `ROZBIEZNOSC` (discrepancy) or `BLAD_ZRODLA` (source error: the source did not respond, so nothing
  can be established);
- `liczby` – counters for a quick sense of scale;
- `id` – identifiers of the discrepancies; their set decides whether an alert is "new";
- `opis` and `uwagi` – text for the alert and observations that do not trigger an alert by themselves;
- `zrodla_bledy` – sources that did not respond. A check can have discrepancies and an error in some sources at the same
  time: the status is then `ROZBIEZNOSC`, and the source error is counted separately.

### Core

| Check | What it checks | Discrepancy | ID |
|---|---|---|---|
| `przebiegi` | whether the agent started in the last completed schedule window and whether its `stan.json` is fresh | no run tag; `ostatni_przebieg` older than 2 × interval | `<agent>`, `<agent>:stan` |
| `pr` | whether every open PR has a reviewer verdict for the current commit | no verdict for longer than 60 min | `<repo>#<nr>` |
| `zlecenia` | whether open orders past their deadline have a reply | deadline passed and the reply is empty | file, entry date and title |
| `konfiguracja` | whether the agents' scheduled tasks match the schedule | missing task, wrong account, different times | `<agent>:brak-zadania`, `:konto`, `:godzina`, `:godziny` |
| `budzety` | the month's spending against agent and organisation limits | spending ≥ 100 % of the limit | `<agent>:100`, `org:100` |
| `martwe` | whether an agent keeps ending its runs in an error | the 3 latest results are `BŁĄD` (error) | `<agent>` |

#### `przebiegi`

Sources: `koordynacja-odstepy.tsv`, tags in origin (`git ls-remote --tags origin przebieg/*`), `<agent>/stan.json`.

- **Schedule.** One tsv row per agent, tab-separated columns: `agent`, `odstep_min` (window length),
  `poczatek` (`GG:MM` – window boundary) and optionally `dni` (`*` or `pn,wt,sr,cz,pt,so,nd`), `godziny` (`*` –
  a run in every window – or a list of `G:MM`) and `od` (`RRRR-MM-DD GG:MM` – from when the schedule applies; `-` or
  empty = no restriction). Lines starting with `#` are comments.
- **Window.** The window number is computed with the same formula as in `scripts/przebieg.sh`, in the organisation's
  time zone. The last completed window in which the schedule expects a run is checked; a window counts as completed
  10 min after its end. With windows shorter than 60 min the tag may also be in the following windows – up to 60 min in
  total (late start).
- **Since when.** Only windows from the later of two moments are expected: the `od` column or the agent's first tag;
  the watchdog looks back at most 8 days. When there is no window with a run in that range, the check adds a note, not
  a discrepancy.
- **Roles not live.** An agent from the schedule without a `<agent>/agent.md` file is skipped (counter `nieutworzeni`).
- **State.** `ostatni_przebieg` older than 2 × `odstep_min` (when the schedule has selected days or times: 2 × the
  longest gap between runs) gives the ID `<agent>:stan`. A missing file or an unreadable date is a note.

#### `pr`

Sources: `gh pr list` for every repo in `repo.produkt` and for the organisation repo, the head commit date from the
GitHub API, `recenzent/stan.json`.

- Drafts and PRs whose title matches `straznik.pomin_tytul` / `STRAZNIK_POMIN_TYTUL` are skipped.
- A verdict is an entry in `recenzje` with a `sha` field. Entry key: the bare PR number for the first product repo (when
  there is no product repo – for the organisation repo); for the others `<repo>#<nr>` (`<repo>/<nr>` and `<repo>-<nr>`
  are also accepted). `sha` may be abbreviated – at least 7 characters, compared as the prefix of the head commit.
- The age of a PR without a verdict counts from the later of two dates: PR creation or the head commit. Over 60 min –
  discrepancy.
- A repo that `gh` did not return is a source error `gh:pr:<repo>`; the remaining repos are checked normally.

#### `zlecenia`

Source: all `zlecenia/*.md` files.

- Entries with the heading `## [otwarte…]` or `## [w toku…]` and a `Termin:` (deadline) line containing a date are
  checked. The earliest date on that line counts; a bare date means end of day (23:59). After an arrow `→` only the part
  after the arrow counts (deadline moved), and dates after the word "wcześniej:" (earlier:) are skipped.
- Discrepancy: the deadline has passed and there is nothing after `Odpowiedź:` (reply) – neither on the same line nor
  further down in the block. An entry without an `Odpowiedź:` line counts as having no reply.
- The ID is three fields separated by the `|` character: file name, the date from the entry heading and the title
  (up to 60 characters).
- The watchdog's own entries (`od: straznik`) are skipped, so that an alert does not trigger another alert.

#### `konfiguracja`

Sources: `koordynacja-odstepy.tsv` and the desktop app's scheduled task directory (`STRAZNIK_ZADANIA`, by default
`~/.claude/scheduled-tasks`). For every agent in the schedule and every account in `runtime.konta` the watchdog checks
the task `<prefiks>-<agent>-<konto małymi literami>/SKILL.md`:

| Condition | ID when not met |
|---|---|
| the task file exists | `<agent>:brak-zadania` |
| it contains the line `Konto: <KONTO>` | `<agent>:konto` |
| the first `GG:MM` time in the `description` field (frontmatter) = the first time of the schedule: the earliest in the `godziny` column, and with `*` – `poczatek` + 30 min | `<agent>:godzina` |
| every `GG:MM` time in the description belongs to the schedule | `<agent>:godziny` |

- With more than one account an account suffix is appended to the ID, e.g. `<agent>:konto:b`.
- A description without times or a task without a `description` field is a note, not a discrepancy. Tasks
  `<prefiks>-…-<konto>` outside the schedule are also notes.
- The watchdog does not compare the interval from the description: `odstep_min` in the tsv is the length of the run
  lock window, not the task's cadence.
- The check is **skipped** (status `OK` with the description "pominięta" (skipped)) when `runtime.glowny` has a value
  other than `desktop-scheduled`, and also when the task directory does not exist and `STRAZNIK_BEZ_ZADAN` is set. With
  `desktop-scheduled`, a missing task directory without that variable is a source error `zadania`.

#### `budzety`

Sources: `org.yaml` (`mandat.budzet_floty_usd_miesiac`), `flota/zadania.yaml`, `przebiegi/<agent>/`.

- Agent limit: `budzet_usd_miesiac` in its `- agent:` block in `flota/zadania.yaml` (the agent is recognised by the
  `katalog:` field), and when it is not there – the `budzet_usd_miesiac` value from the top level of the file. No limit =
  agent not checked.
- Spending in the current month (in the organisation's time zone): the sum of the `koszt_usd` fields from the files
  `przebiegi/<agent>/koszty-<RRRR-MM>-*.jsonl` and `przebiegi/<agent>/<RRRR-MM>-*.jsonl`.
- Spending ≥ 100 % of the agent limit → `<agent>:100`; total spending of all agents ≥ the organisation limit → `org:100`.
  A level ≥ 80 % is a note.
- **Circuit breaker:** on `org:100` the watchdog creates the file `bezpieczenstwo/HALT` (the fleet kill switch described
  in `zasady.md`). Resuming: a budget decision and removal of the file by the operator.

#### `martwe`

Source: the `wynik` field from contract lines in `przebiegi/<agent>/RRRR-MM-DD.jsonl` (the three newest files).

- The agent's three latest results equal to `BŁĄD` → ID `<agent>`.
- **Circuit breaker:** the watchdog then creates `przebiegi/dead-letter/<agent>.md`. As long as the file exists,
  `scripts/przebieg.sh` ends that agent's run with exit code 10, without doing any work. Resuming: a review of the cause
  and removal of the file by the operator. An agent that already has a dead-letter file is only a note.

### Plugins (disabled by default)

A plugin works only once its name is added to `straznik.kontrole`. When disabled it makes no requests to its sources.
Plugins require environment variables and – partly – secrets (section "Secrets").

#### `testflight` – tester feedback from App Store Connect

Compares TestFlight feedback identifiers (screenshots with comments and crash reports) with the register
`aplikacje/opinie-testflight.md`. A piece of feedback is "in the register" when the first 8 characters of its
identifier occur in the file's text. Only the identifier and date are fetched – no comment content and no tester data.

| Variable | Meaning |
|---|---|
| `STRAZNIK_ASC` | directory with the `asc.py` module – the organisation's App Store Connect API client, kept outside the repo |
| `STRAZNIK_ASC_BUNDLE` | the app's bundle identifier (bundle ID) |

- Module contract: the function `asc.call(metoda, sciezka)` returns the API response as a dictionary (`data`,
  `links.next`) or `{"blad": "<komunikat>"}`. Authentication is that module's concern – the watchdog does not know the
  App Store Connect keys.
- Without either variable the check reports a source error with a message saying what to set.
- ID: the first 8 characters of the feedback identifier; crashes take the form `awaria:<8 znaków>`.

#### `automaty` – product automations

Five independent parts. What happens when a part is not configured is described in the points below the table.

| Part | Source | Discrepancy | ID | Configuration |
|---|---|---|---|---|
| database cron jobs | Supabase, `cron.job_run_details` | a job that ended in an error in the last hour | `cron:<nazwa zadania>` | `STRAZNIK_SUPABASE_PROJEKT` + token |
| HTTP calls from the database | Supabase, `net._http_response` | a 5xx response in the last hour | `net:5xx` | as above |
| build error counter | Supabase, custom query | value of the `bledy` column ≥ 3 | `build_stan` | additionally `STRAZNIK_SQL_BUILD` |
| deployment workflow | GitHub Actions, latest run | result `failure`, `timed_out` or `startup_failure` | `fly:<data przebiegu>` | `STRAZNIK_FLY_REPO`, `STRAZNIK_FLY_WORKFLOW` |
| GitHub Pages | latest Pages build | status `errored` or `failure` | `pages:<repo>:<data>` | `STRAZNIK_PAGES` |

- **Supabase.** Queries go through the Management API in read-only mode (`read_only`). A project ref
  (`STRAZNIK_SUPABASE_PROJEKT`) and an access token are needed – the `supabase-pat-straznik` service in the keychain or
  the variable `STRAZNIK_SEKRET_SUPABASE_PAT_STRAZNIK`. Without the project or the token the Supabase parts report a
  source error.
- **Build error counter.** `STRAZNIK_SQL_BUILD` is an SQL query that depends on the organisation's database schema; the
  watchdog takes the first row of the result and the `bledy` column. Without this variable the part is skipped.
- **Deployment workflow.** Repo: `STRAZNIK_FLY_REPO` (name or `właściciel/nazwa`), by default the first repo
  in `repo.produkt`; without both the part is skipped. Workflow name: `STRAZNIK_FLY_WORKFLOW`, by default `Fly Deploy`.
- **Pages.** `STRAZNIK_PAGES` is comma-separated repo names (without the owner); none by default. A repo without GitHub
  Pages is skipped with a note.

### The `klon` check

In a real run the watchdog also adds a check of its own working repo: `OK` when `git pull` succeeded; source error
`git:pull` when neither pull nor fetch works (network); discrepancy `klon:wymaga-uwagi` when fetch works but pull does
not. This check is not listed in `straznik.kontrole`. What to do then – section "Klon wymaga uwagi".

### Timeouts

- `git` and `gh` commands have a 10–20 s timeout, Supabase queries 15 s; after a timeout there is one retry
  in the same run. Timeouts for App Store Connect requests are set by the `asc.py` module.
- All network sources together have 45 s. A source that did not finish in time gets the error "przekroczony czas"
  (timed out).

## Alerts

### When an alert is raised

- **New discrepancy** – immediately on first detection and on every change in the check's ID set.
- **Persistent discrepancy** – reminders: 2 h after the alert, the next after 6 h, then every 24 h.
- **Source error** – only when the same source fails to respond in 2 consecutive runs; the first error is just a line
  in the event log. Immediately when at least 2 sources fail to respond in one run or when it concerns the build error
  counter (`build_stan`). Then reminders as for a discrepancy.
- **Return to `OK`** sends no alert – a line remains in the event log.

The alert key is the check name and the sorted IDs, so a new ID means a new alert, not a reminder. Keys
not seen for 3 days are removed from the state.

### Muting

A known discrepancy is muted in the **reply** to the watchdog's entry in `zlecenia/ceo.md`, with the line:

```
Wycisz do: RRRR-MM-DD[ GG:MM][ (ID, ID)]
```

- Until the given moment the watchdog does not alert about those IDs of that check. A bare date means the end of that day.
- Without the parentheses all IDs from the entry's "Liczby: … ID: …" (counts) line are muted.
- The remaining IDs keep alerting. Muted IDs are listed in the alert notes; when all of them are muted, no alert is
  raised, and the event log gets the line "bez alertu – wszystkie ID wyciszone" (no alert – all IDs muted).
- The line works regardless of the entry's status (also after it is closed), until the date passes.

### Channels

1. **E-mail** via Resend. Sender: `straznik@<domena z org.yaml>` (or `STRAZNIK_MAIL_OD`), recipient:
   `ludzie.operator.kontakt_firmowy` (or `STRAZNIK_MAIL_DO`). Subject: `[STRAŻNIK] <kontrola>: <krótko>`. Body: status,
   counts, IDs (at most 30, then "… i N więcej" (and N more)), description and notes – no personal data and no secrets.
   Without a Resend key the run carries on, and the state and events record "mail pominięty – brak klucza Resend"
   (e-mail skipped – no Resend key).
2. **macOS notification** "Strażnik floty" – only where `osascript` is available, i.e. when run via launchd.
   During **night-time quiet hours 23:00–7:00** (organisation time zone) the notification is skipped; the e-mail and the
   `Pilne` entry are created as usual.
3. **`Pilne` entry** in `zlecenia/ceo.md`:

   ```
   ## [otwarte] RRRR-MM-DD · od: straznik · Pilne (strażnik) · <kontrola>: <krótko>
   Strażnik floty (`moduly/straznik`, przebieg GG:MM): <opis>
   Liczby: <liczniki>. ID: <identyfikatory>.
   Stan: `ceo/straznik.json` (klon strażnika), historia: `ceo/straznik-zdarzenia.md`. Klucz strażnika: `<skrót>`.
   Termin: <chwila alertu + 2 h>
   Odpowiedź:
   ```

   The entry goes at the top of the order list (before the first `## ` heading), away from the agents' additions at the
   end of the file – fewer merge conflicts. When the file does not exist, the watchdog creates it. When an open entry
   with the same key already exists, the watchdog does not duplicate it; the e-mail and notification continue according
   to the reminders.

## Output files

| File | Contents | In the repo? |
|---|---|---|
| `ceo/straznik.json` | full state of the last run: `czas`, `czas_przebiegu_s`, `kontrole`, `alerty`, `bledy_zrodel`, `alert_zrodel`, `aktywne_id`, `wyciszenia`, `ostatni_commit_zdarzen`, `mail`, `alerty_przebiegu`, `uwagi` | no – in `.gitignore`; in GitHub Actions carried between runs in the cache |
| `ceo/straznik-zdarzenia.md` | log: a line only on a change of a check's status or ID set, on an alert and when a circuit breaker trips; newest at the bottom | yes – committed at most once an hour, and immediately on an alert |
| `zlecenia/ceo.md` | `Pilne` entries for the CEO agent | yes – committed immediately |
| `przebiegi/dead-letter/<agent>.md` | pausing an agent after a series of errors (check `martwe`) | yes – committed immediately |
| `bezpieczenstwo/HALT` | stopping the fleet after the organisation budget is exhausted (check `budzety`) | yes – committed immediately |

The watchdog commits only these files (`git commit -- <ścieżki>`) and pushes them with a plain `git push`; when the push
fails, it does `git pull --no-rebase` and tries once more. The event log is merged with the `merge=union` rule
(`.gitattributes`), so additions by other writers and the watchdog's lines are both kept. Even so, it is better to record
alert handling in the reply to the entry in `zlecenia/ceo.md` than in the event log.

## Running

### GitHub Actions (recommended)

The workflow `.github/workflows/straznik.yml` runs the watchdog every 15 minutes on a checkout of the organisation repo –
no laptop needed.

1. **Schedule.** In the template it is disabled (two lines marked `# STRAZNIK-CRON`). It is enabled by
   `scripts/init-org.sh` when the organisation is set up. Manually – the same command, which removes the marker from both
   lines while keeping the indentation:

   ```bash
   sed -i.bak 's/^  # STRAZNIK-CRON //' .github/workflows/straznik.yml && rm .github/workflows/straznik.yml.bak
   ```

   A single run can be triggered from the Actions tab (`workflow_dispatch`).
2. **Repo secrets** (Settings → Secrets and variables → Actions):

   | Secret | What for | Without it |
   |---|---|---|
   | `STRAZNIK_TOKEN` | GitHub token with read access to product repo PRs and write access to the organisation repo | the Actions run token is used – only the organisation repo works, product repo PRs will be a source error |
   | `STRAZNIK_RESEND_KEY` | Resend key for alert e-mails | alert without e-mail: entry in `zlecenia/ceo.md` and the event log |

3. **State** (`ceo/straznik.json`) is carried between runs in the Actions cache – it does not go into the repo.
4. **Plugins.** Add their variables (`STRAZNIK_ASC…`, `STRAZNIK_SUPABASE_PROJEKT`, `STRAZNIK_SEKRET_…` etc.) to the
   `env:` section of the step "Przebieg strażnika" (watchdog run), and pass the secret values from repo secrets.
5. **The `konfiguracja` check in the cloud.** The runner has no desktop app task directory. With `runtime.glowny`
   other than `desktop-scheduled` the check is skipped automatically. If the fleet runs as desktop tasks and the watchdog
   in Actions – add the variable `STRAZNIK_BEZ_ZADAN: "1"` to `env:` or remove `konfiguracja` from `straznik.kontrole`;
   otherwise the check will keep reporting a source error.

### launchd on a Mac (alternative)

For an organisation whose fleet runs on a single Mac: only there does the watchdog have access to the scheduled task
directory (check `konfiguracja`) and can it send a macOS notification.
The watchdog works in its **own clone** of the repo (by default `~/projekty/organizacja-straznik`), never in the agents'
shared checkout.

```bash
moduly/straznik/instaluj.sh                # clone (if missing), launchd agent org.agenci.straznik, starts immediately
launchctl print gui/$(id -u)/org.agenci.straznik | grep -E 'state|last exit'
tail -f ~/Library/Logs/straznik-floty.log
moduly/straznik/instaluj.sh --odinstaluj   # stops and removes the agent; the clone and log remain
```

- `STRAZNIK_REPO` – a different clone location; `STRAZNIK_URL` – address of the repo to clone (by default the `origin` of
  the checkout from which you run the installer).
- The installer creates `~/Library/LaunchAgents/org.agenci.straznik.plist` from the template `straznik.plist.szablon`: a run
  every 900 s and immediately after loading, interpreter `/usr/bin/python3`, log in `~/Library/Logs/straznik-floty.log`.
- Keep secrets in the keychain (section "Secrets"). Add plugins' non-secret variables to the `EnvironmentVariables`
  dictionary in the plist template and run the installer again – the template is taken from the watchdog's clone, so the
  change must be on the repo's main branch.
- Do not run launchd and the Actions schedule at the same time: alerts and entries would be duplicated.

### Manually and dry run

| Option | Effect |
|---|---|
| (no option) | full run with effects: pull, checks, alerts, commit, state write |
| `--sucho` | real sources, result as JSON on stdout (`czas`, `czas_przebiegu_s`, `repo`, `kontrole`); no clone update, file writes, e-mails, notifications or commits |
| `--dane <katalog>` | instead of the network and the real repo: `<katalog>/zewnetrzne.json`, `<katalog>/repo/`, `<katalog>/zadania/`, `<katalog>/teraz.txt` |
| `--repo <ścieżka>` | working repo (by default `STRAZNIK_REPO` or `~/projekty/organizacja-straznik`) |
| `--zadania <ścieżka>` | scheduled task directory (by default `STRAZNIK_ZADANIA` or `~/.claude/scheduled-tasks`) |
| `--teraz <ISO>` | the run moment instead of the clock, e.g. `RRRR-MM-DDTGG:MM` (no zone = organisation time); for tests and replaying situations |

Use `--dane` together with `--sucho`. Without `--sucho` the watchdog sends no e-mails or notifications and commits
nothing, but it writes state, events and `Pilne` entries in the `repo/` directory of the data.

## Klon wymaga uwagi (clone needs attention)

Every real run starts with `git pull --no-rebase` in the working repo. The watchdog never uses `git stash`,
`git reset` or a forced push – it does not revert anyone else's changes.

**Ordinary collision.** When pull refuses because the watchdog's own files (`ceo/straznik-zdarzenia.md`, `zlecenia/ceo.md`)
have uncommitted changes, the watchdog makes a local commit of them, retries the pull and pushes the result. This needs
nobody's attention.

**When pull still fails** (conflict, diverged history), the watchdog:

1. aborts the ongoing merge (`git merge --abort`) – the clone returns to its state from before the pull;
2. runs `git fetch`; if that does not work either, it is an ordinary source error `git:pull` (alert after 2 runs);
3. if fetch works – the `klon` check reports the discrepancy `klon:wymaga-uwagi`, and the watchdog works in fallback mode:
   - it computes the remaining checks on a snapshot of origin's main branch (a read-only copy in a temporary directory);
   - it writes events locally and commits nothing in the clone;
   - it does not create dead-letter or `HALT` files;
   - it adds `Pilne` entries – including the alert about the clone itself – to `zlecenia/ceo.md` through a temporary
     worktree from origin's main branch (commit of that file only, plain push, worktree removed), so the alert is not lost.

**Repair (operator, manually):**

```bash
git -C ~/projekty/organizacja-straznik status
git -C ~/projekty/organizacja-straznik log --oneline origin/main..HEAD
```

The watchdog's local commits concern only its own files. The safest option is to move the clone aside and create a new one:

```bash
mv ~/projekty/organizacja-straznik ~/projekty/organizacja-straznik.stary
moduly/straznik/instaluj.sh    # new clone and agent restart
```

Move unpushed entries and event lines manually from the old clone, then delete it. In GitHub Actions every run
starts from a fresh checkout, so in practice this situation only concerns launchd.

## Secrets

| Secret | What for | Environment variable | Service in the macOS keychain |
|---|---|---|---|
| Resend key | alert e-mail | `STRAZNIK_RESEND_KEY` | `resend-api-key-straznik` |
| Supabase token | plugin `automaty` | `STRAZNIK_SEKRET_SUPABASE_PAT_STRAZNIK` | `supabase-pat-straznik` |
| GitHub token | `gh` (PRs, workflow, Pages) | `GH_TOKEN` | – (or log in with `gh auth login`) |

- The environment variable takes precedence over the keychain. General naming rule:
  `STRAZNIK_SEKRET_<SERVICE NAME in upper case, characters other than letters and digits replaced with _>`.
- Saving to the keychain (the command will prompt for the value – do not type it on the command line):

  ```bash
  security add-generic-password -s resend-api-key-straznik -a straznik -w
  ```

- The Resend key should have send permission only and be restricted to the organisation's domain. The watchdog uses the
  Supabase token only for read-only queries.
- Secrets exist only in process memory. In error messages they are masked, and any string that looks like a JWT
  (`eyJ…`) is replaced with `***`. They never reach `ceo/straznik.json`, events, e-mails or logs.

## Environment variables

| Variable | Meaning | Default |
|---|---|---|
| `STRAZNIK_REPO` | the watchdog's working repo (its own clone) | `~/projekty/organizacja-straznik` |
| `STRAZNIK_ZADANIA` | the desktop app's scheduled task directory | `~/.claude/scheduled-tasks` |
| `STRAZNIK_BEZ_ZADAN` | any non-empty value: when the task directory does not exist, the `konfiguracja` check is skipped instead of reporting a source error | unset |
| `STRAZNIK_GH_ORG` | owner of the repos on GitHub – overrides the owner from `repo.github` | from `org.yaml` |
| `GH_TOKEN` | token for `gh` | `gh` login |
| `STRAZNIK_MAIL_OD` | sender of alert e-mails | `Strażnik floty <straznik@<domena z org.yaml>>` |
| `STRAZNIK_MAIL_DO` | recipient of alert e-mails | `ludzie.operator.kontakt_firmowy` |
| `STRAZNIK_RESEND_KEY` | Resend key | keychain; without a key the e-mail is skipped |
| `STRAZNIK_SEKRET_<USLUGA>` | a service secret instead of the keychain | keychain |
| `STRAZNIK_POMIN_TYTUL` | regular expression for PR titles skipped in the `pr` check – overrides `straznik.pomin_tytul` | none (nothing is skipped) |
| `STRAZNIK_ASC` | plugin `testflight`: directory with the `asc.py` module | none – source error "ustaw STRAZNIK_ASC" (set STRAZNIK_ASC) |
| `STRAZNIK_ASC_BUNDLE` | plugin `testflight`: the app's bundle identifier | none – source error |
| `STRAZNIK_SUPABASE_PROJEKT` | plugin `automaty`: Supabase project ref | none – source error |
| `STRAZNIK_SQL_BUILD` | plugin `automaty`: SQL query for the build error counter (column `bledy`) | none – part skipped |
| `STRAZNIK_FLY_REPO` | plugin `automaty`: repo with the deployment workflow (name or `właściciel/nazwa`) | first repo in `repo.produkt`; without it the part is skipped |
| `STRAZNIK_FLY_WORKFLOW` | plugin `automaty`: name of the deployment workflow | `Fly Deploy` |
| `STRAZNIK_PAGES` | plugin `automaty`: repos with GitHub Pages, comma-separated names | none – part inactive |
| `STRAZNIK_URL` | `instaluj.sh` only: address of the repo to clone | `origin` of the checkout containing the installer |

## Tests

```bash
python3 -m unittest discover moduly/straznik   # module tests
bash scripts/testy.sh                          # all test groups of the skeleton, the watchdog included
```

The tests do not use the network, the keychain or the real configuration: `STRAZNIK_…` variables from the environment
are removed for the duration of the tests, and external sources are replaced with data from `testdane/`. The data is
fictitious – an organisation with the prefix `firma`, repos `firma-agenci`, `produkt` and `strona`.

## Your own check

1. **Function** in `kontrole.py`: `kontrola_<nazwa>(dane, teraz)` returning `wynik(status, liczby, ids, opis, uwagi,
   zrodla)`; when the data is unreadable – `zrodlo_niedostepne(komunikat, zrodlo)`. No network and no writes: everything
   the check needs it receives in `dane`. IDs must be stable between runs – they decide whether an alert is new.
2. **Registration**: a row `("<nazwa>", "<etykieta>", kontrola_<nazwa>)` in the `KONTROLE` list in `kontrole.py` and the
   name in `KONTROLE_WSZYSTKIE` in `zrodla.py` (and in `KONTROLE_RDZEN` if it should run by default).
3. **Data**: a fragment under the key `<nazwa>` in `zrodla.zloz` – read repo files through `bezpiecznie(...)` so that a
   read error becomes `{"blad": "…"}` rather than an exception. Add a network source in `pobierz_zewnetrzne` (only when
   the check is active) and its counterpart in `zewnetrzne_z_pliku` for `--dane` mode.
4. **Enabling**: the name in `straznik.kontrole` in `org.yaml`.
5. **Alert text** (optional): a short description for the e-mail subject and entry heading in the dictionary in
   `alerty.krotko`.
6. **Tests**: `OK`, discrepancy and source-error cases in `test_straznik.py`; sample data in `testdane/`.

Supply configuration for a new check the same way as for the existing ones: a key in the `straznik:` section of
`org.yaml` or a `STRAZNIK_…` variable, always with a safe default value and a clear message when something is missing.

## Limitations

- The watchdog sees only what has been written to the repo and what `git` and `gh` return. It does not know whether an
  agent's work was good – it knows whether it happened and whether it left a trace.
- The 15-minute cadence and the thresholds (60 min for a PR, 10 min margin after a window, 2 runs for a source error)
  are constants in the code (`kontrole.py`, `alerty.py`), not settings in `org.yaml`.
- The `konfiguracja` check reads task descriptions, not their actual cron expressions – those are not available outside
  the app.
- macOS notifications and reading the keychain work only on a Mac.
