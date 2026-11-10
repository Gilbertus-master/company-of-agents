> **In the template:** specification carried over from the source organisation (8.11.2026). The JSON line fields (sections 1–3),
> the validator `pm/narzedzia/przebieg-raport.py` and the rules are universal. The per-agent source table (section 4) is **an example from
> the source organisation** (e.g. the product repo, an app store, mobile build CI) – each new organisation fills it in for its own agents and services from `toolbelt/katalog.md`.
> The rows for roles added in the template (inzynier, tester, lokalizacja, compliance, wywiad) are already universal – they follow from the
> "Sources checked in every run" section in `role/<rola>.md`; new role = new row here in the same PR (`procesy/60-cykl-zycia-agenta.md`).
> **Fields added in the template (optional):** `model` (text); `koszt_usd` in the contract line stays empty (`null`) – the cost of a run
> is measured by `python3 -I scripts/koszt.py zapisz --agent <agent>` (tokens from the session transcript × `flota/ceny-modeli.yaml`) and stored separately
> in `przebiegi/<agent>/koszty-<data>.jsonl` (headless wrapper – from the CLI result); entering the cost here as well would count it twice.
> `konto` is the fleet account letter (org.yaml → runtime.konto); the list of agents = `koordynacja-odstepy.tsv` + `limit`.
> **Contract v2 (`"wersja": 2`, recommended from 10.11.2026 – research "Standards 2026", rec. 5):** additional fields
> `prompt_version` (commit hash of the instructions: `git log -1 --format=%h -- <agent>/agent.md role/<agent>.md flota/szablony/`; required),
> `outcome_evidence` (URL of a PR/commit/file as evidence from outside the report; required for an OK result outside CISZA), `retries` (number of
> step retries), `tool_calls` (number of tool calls), `escalation` (`operator|wlasciciel|ceo|brak`), `next_action` (one sentence).
> Without `prompt_version` regressions cannot be tied to an instruction version and canary rollouts are impossible; without `outcome_evidence` "OK" is
> a claim, not a fact (false completion – MAST). Watchdog: K=3 consecutive runs with result BŁĄD → `przebiegi/dead-letter/<agent>.md`.

# Run contract – specification of the report after every run (stage 1 of fleet reliability)

Author: PM agent, 8.11.2026. Order: `zlecenia/pm.md` (owner via CEO, "start right away"). Basis: `ceo/analizy/2026-11-08-niezawodnosc-floty.md`
section 3 items 1–2 (causes A and E: "run finished" ≠ work done; CISZA without a trace). Rollout into agent instructions – CEO (block in section 7).
Consistent with the parallel orders of 8.11: fleet watchdog (architekt – reads these files, check 1), numeric reconciliations of the reviewer, aplikacje, recepcja and obsluga
(their "N / M / K" lines are the source fields `nowe / obsluzone / otwarte`).

## 1. Rule

**Every run of every agent ends with one JSON line** in `przebiegi/<agent>/<RRRR-MM-DD>.jsonl` – also in **CISZA** mode (silent; a sign of life) and also
when the run failed (then `wynik: "BŁĄD"`). The line says **what the agent read (with numbers), what it did with it (with IDs) and what does not add up**.
A run without a line, without numbers, or with numbers not backed by the source = **failed run**, regardless of the session status in the Claude app.
The report does not replace the log (`<agent>/dziennik.md` – for humans and the CEO) or `stan.json` (the agent's memory); it is for machines: the fleet watchdog, the PM and KPIs.

Does not apply to: a run ended with **code 10** of the `koordynacja.md` protocol (second account; trace = tag `przebieg/<agent>/<okienko>` on origin – the watchdog reads tags).
Chats with the owner and dev sessions write a line only when they do the work of an agent with a mailbox (`tryb: "CZAT"`); the PM on duty – `tryb: "DYŻUR"`.

## 2. File

- Path: `przebiegi/<agent>/<RRRR-MM-DD>.jsonl` – `<agent>` = the agent's directory name (`ceo`, `ochrona-danych`, `marketing` …); date = **start day** of the run
  in Warsaw time. The `org-sklepy-maile` task writes to `przebiegi/aplikacje/` with `zadanie: "org-sklepy-maile-a"`.
- One line = one run, appended at the end (`>>`), UTF-8, no line break in the middle, max. 8,000 characters (shorten long ID lists: first 20 + `"…+N"`).
- Append and validate with one command (section 6): `python3 -I pm/narzedzia/przebieg-raport.py dopisz <plik.json>` – rejects a line that breaks the contract
  (code 1) and says why; the agent fixes it and appends again. Without the tool (no Python): `printf '%s\n' '<json>' >> przebiegi/<agent>/<data>.jsonl`.
- Commit: together with the agent's other files; in CISZA **a separate commit of this file only**: `git add przebiegi/<agent>/ && git commit -m "<agent>: przebieg CISZA <data> <GG:MM>"`,
  then `git pull --rebase` and push as before. A conflict in `.jsonl` (two accounts appended to the same file) is resolved by **keeping both lines**; so that git does
  it by itself, the CEO adds the line `przebiegi/**/*.jsonl merge=union` to `.gitattributes` in the root directory (PM proposal – the file is outside my scope).
- No personal data (no e-mail addresses, names, phone numbers – only thread IDs, numbers, roles), no secrets, no e-mail content. The validator rejects a line with an e-mail address.
- Retention: daily files stay in the repo; on the first day of the month the PM merges the previous month into `przebiegi/<agent>/<RRRR-MM>.jsonl` (one operation, line content unchanged).

## 3. Line fields

| Field | Type | Required | Meaning |
|---|---|---|---|
| `wersja` | number | yes | Contract version – today `1` (the tool fills it in itself). |
| `agent` | text | yes | The agent's directory name (in the template 21 roles from `role/`: ceo, pm, architekt, inzynier, recenzent, tester, aplikacje, lokalizacja, wywiad, marketing, sprzedaz, obsluga, recepcja, inwestorzy, finansowy, ksiegowy, podatkowy, prawny, compliance, ochrona-danych, bezpieczenstwo; an organisation has as many as it has in `koordynacja-odstepy.tsv`). |
| `zadanie` | text | no | Scheduled task id (`org-prawny-a`, `org-sklepy-maile-a`) or `czat` / `sesja:<local_…>` / `dyzur`. |
| `konto` | `"A"`/`"B"` | yes | From the task text ("Konto: B"; missing = A) – as in `koordynacja.md`. |
| `okienko` | number | no | Output of `scripts/przebieg.sh okienko <agent>` – ties the line to the lock tag `przebieg/<agent>/<okienko>`. |
| `start`, `koniec` | ISO 8601 text with time zone | yes | `2026-11-08T13:04:00+01:00` (Warsaw time; `date -Iseconds`). The tool fills in `koniec` when missing. |
| `tryb` | text | yes | As in the agent's instructions: `CISZA`, `ZDARZENIE`, `DZIENNY`, `TYGODNIOWY`, `PEŁNY`, `PRACA` (architekt), `WYDANIE` (marketing), `CZAT`, `DYŻUR`, `BŁĄD` (run aborted before a mode was chosen). |
| `zrodla` | list of objects | yes, non-empty (also CISZA) | One per **each mandatory work source of the agent** from section 4 – even when everything = 0. Fields below. |
| `akcje` | list of objects | yes (may be `[]`) | What the agent did, **by ID**: `{"typ": "zlecenie|pr|commit|plik|wpis|zlecenie_dla|sesja|przypomnienie|decyzja", "id": "…", "opis": "≤ 80 characters"}`. |
| `rozbieznosci` | list of objects | yes (may be `[]`) | Every source in which the numbers do not add up (section 5): `{"zrodlo", "opis", "liczby": {…}, "ids": […], "dzialanie"}`. |
| `bledy` | list of objects | yes (may be `[]`) | `{"gdzie": "Resend|gh|ASC|Supabase|git|skrypt <nazwa>|…", "komunikat": "≤ 200 characters, no secrets", "skutek": "przebieg przerwany|źródło pominięte|akcja nieudana", "zrodlo": "<source name, if applicable>"}`. |
| `wynik` | text | yes | `OK` · `ROZBIEŻNOŚĆ` (discrepancy; `rozbieznosci` list non-empty) · `NIEPEŁNY` (incomplete; some source has `przeczytane: null`) · `BŁĄD` (error; run aborted). The validator computes it and enforces consistency. |
| `raport` | text | no | One sentence for a human (≤ 160 characters) – the same as the closing sentence of the run. |
| `kpi` | text | no | Content of the `**KPI:**` line from the log (KPI procedure item 1), so that measurement does not have to read the log. |

### Source object (`zrodla[]`)

| Field | Meaning |
|---|---|
| `nazwa` | Fixed name from section 4 (e.g. `kontakt@`, `zlecenia/prawny.md`, `PR otwarte produkt`, `ASC uwagi`). |
| `przeczytane` | How many items the agent **actually fetched** from the source in this run (the whole list, not "since last time", when section 4 says "all"). `null` = the source could not be read → mandatory entry in `bledy` with this `zrodlo`; **never `0` instead of `null`**. |
| `nowe` | How many of them are new since the last run or **are not in the register** (compared by ID, not by date/status). |
| `obsluzone` | How many of `nowe` the agent handled in this run (register entry + action). `obsluzone ≤ nowe`. |
| `w_rejestrze` | How many items of this source the agent's register holds after the run (for reconciliation: `przeczytane` ↔ `w_rejestrze`). |
| `otwarte` (optional) | Items pending after the run (e.g. tickets not closed, PRs without a verdict, orders without a reply). |
| `ids_bez_sladu` (optional) | IDs of new, unhandled items (≤ 20) – what the watchdog and the PM need to check. |
| `okres` (optional) | Since when `nowe` is counted, when the source is read incrementally (`"od": "2026-11-08T10:15:00+01:00"`). |

### Example – CISZA (recepcja)

```json
{"wersja":1,"agent":"recepcja","zadanie":"org-recepcja-a","konto":"B","okienko":4712,"start":"2026-11-08T13:15:10+01:00","koniec":"2026-11-08T13:17:02+01:00","tryb":"CISZA",
 "zrodla":[{"nazwa":"kontakt@","przeczytane":1,"nowe":0,"obsluzone":0,"w_rejestrze":1,"otwarte":0},
           {"nazwa":"zlecenia/recepcja.md","przeczytane":0,"nowe":0,"obsluzone":0,"w_rejestrze":0,"otwarte":0}],
 "akcje":[],"rozbieznosci":[],"bledy":[],"wynik":"OK","raport":"kontakt@: wątki nowe 0, obsłużone 0, otwarte 0 – brak nowych zleceń."}
```

### Example – ROZBIEŻNOŚĆ (recenzent)

```json
{"wersja":1,"agent":"recenzent","zadanie":"org-recenzent-a","konto":"B","start":"2026-11-08T10:45:20+01:00","koniec":"2026-11-08T11:40:00+01:00","tryb":"ZDARZENIE",
 "zrodla":[{"nazwa":"PR otwarte produkt","przeczytane":6,"nowe":2,"obsluzone":1,"w_rejestrze":5,"otwarte":1,"ids_bez_sladu":["produkt#214"]},
           {"nazwa":"zlecenia/recenzent.md","przeczytane":3,"nowe":1,"obsluzone":1,"w_rejestrze":3,"otwarte":2}],
 "akcje":[{"typ":"pr","id":"produkt#213","opis":"werdykt OK, klasa A, próba wycofana"},{"typ":"plik","id":"recenzent/recenzje/2026-11-08-pr-213.md","opis":"recenzja"},
          {"typ":"zlecenie_dla","id":"zlecenia/architekt.md › 2026-11-08 · produkt#213 uwaga 2","opis":"poprawka search_path"}],
 "rozbieznosci":[{"zrodlo":"PR otwarte produkt","opis":"PR bez werdyktu dla head SHA po przebiegu","liczby":{"otwarte":6,"z_werdyktem_dla_head":5},"ids":["produkt#214"],
                  "dzialanie":"nie zmieścił się w przebiegu – zapisany w stan.json → kolejka[0], wpis Pilne w zlecenia/ceo.md"}],
 "bledy":[],"wynik":"ROZBIEŻNOŚĆ","raport":"PR otwarte: 6, z werdyktem dla head: 5, bez: 1 (#214 – następny przebieg)."}
```

### Example – source BŁĄD (obsluga, Resend not responding)

```json
{"wersja":1,"agent":"obsluga","konto":"B","start":"2026-11-08T14:35:00+01:00","koniec":"2026-11-08T14:37:00+01:00","tryb":"ZDARZENIE",
 "zrodla":[{"nazwa":"support@","przeczytane":null,"nowe":null,"obsluzone":null,"w_rejestrze":3,"otwarte":1},
           {"nazwa":"zlecenia/obsluga.md","przeczytane":2,"nowe":0,"obsluzone":0,"w_rejestrze":2,"otwarte":2}],
 "akcje":[{"typ":"zlecenie_dla","id":"zlecenia/ceo.md › 2026-11-08 · Resend support@ – brak dostępu","opis":"zgłoszenie awarii narzędzia"}],
 "rozbieznosci":[],"bledy":[{"gdzie":"Resend list-inbox-threads","komunikat":"HTTP 503 po 3 próbach","skutek":"źródło pominięte","zrodlo":"support@"}],
 "wynik":"NIEPEŁNY","raport":"uzgodnienie skrzynki niewykonane: Resend 503 – zgłoszone CEO."}
```

## 4. Agents' work sources – list for reconciliation by ID

Every agent has **mandatory sources** (always in `zrodla`, also in CISZA) and a register against which it reconciles them. Comparison is always **by identifier** (Resend thread ID,
PR number + head SHA, ASC feedback ID, order header = file + date + subject), never by date, status or "from memory" (`stan.json` is a hint, not a source).
Common to all: (a) `zlecenia/<agent>.md` – `przeczytane` = headers `[otwarte]`/`[w toku]`/`[czeka…]`, `nowe` = without a reply since the last run,
`obsluzone` = replied to / status changed in this run, `w_rejestrze` = headers with a status and a `Termin:` line, `otwarte` = after the run; (b) `decyzje.md` –
new entries since `stan.json → ostatni_przebieg` concerning the agent (`nowe`) and taken into account (`obsluzone`). Additional sources below.

| Agent | Source (`nazwa`) | What to count (`przeczytane` / `nowe` / `obsluzone` / `w_rejestrze`) | Register (by ID) | Discrepancy = |
|---|---|---|---|---|
| **ceo** | `PR otwarte produkt` | `gh pr list --state open` in the product repo (no drafts, no "[20.11") / without a verdict for head SHA / reported to the reviewer or task started / PRs with a verdict | `recenzent/stan.json → recenzje` (number + sha) | PR without a verdict > 60 min after opening/update |
| | `zlecenia/ceo.md` | as common; additionally `otwarte` = "Pilne" (urgent) entries without a reply | – | Pilne without a reply after the run |
| | `dzienniki agentów` | 16 logs / agents with a new entry since the last run / read / – ; `ids_bez_sladu` = agents with no entry and no line in `przebiegi/` for > 1 window | `przebiegi/<agent>/` | agent without a run in its window |
| | `rejestr-sesji.md` | sessions `otwarta` / new since the last run / closed or abandoned in this run / rows | `list_sessions` (id) | session > 24 h without progress, without a parent, in `list_sessions` but not in the register |
| | `straznik` (once deployed) | discrepancies in `ceo/straznik.json` / new since the last run / handled (order, decision) / – | `ceo/straznik-zdarzenia.md` | watchdog discrepancy without action > 1 run |
| | `decyzje właściciela` | new entries in `decyzje.md` / – / passed on to agents (order with a reference) / matters in `ceo/do-decyzji.md` | `ceo/do-decyzji.md` (no.) | decision without an order; matter > 48 h |
| **architekt** | `zlecenia/architekt.md` | common; `otwarte` = without a PR/package after the deadline | `stan.json → w_budowie` | order in progress without a stage in the progress register > 1 window |
| | `PR własne otwarte` | PRs of the architect and of dev sessions (`gh pr list --author`, label/branch `claude/architekt-*`, packages `architekt/rownolegle/`) / new / with a reply to reviewer comments / in `stan.json → w_budowie` | `stan.json`, `architekt/rownolegle/*.md` | PR with reviewer comments without a reply > 1 window; package without a session > 24 h |
| | `sesje dev (rodzic: architekt)` | sessions in `rejestr-sesji.md` with parent architekt / new reports / collected (result checked, line closed) / rows | `rejestr-sesji.md` | session report not collected > 1 window |
| | `opinie TestFlight → pakiety` | orders from aplikacje with an ASC feedback ID / new / assessed (valid → package/PR; doubtful → `zlecenia/ceo.md`) / `**TestFlight:**` entries in the log | `aplikacje/opinie-testflight.md` (feedback ID) | feedback at the architect without a PR/package/decision > 24 h |
| **recenzent** | `PR otwarte produkt` | `gh pr list --state open` (no drafts, no "[20.11") **by number and head SHA** / without a verdict for the current SHA / reviewed in the run / `stan.json → recenzje` entries | `recenzent/stan.json → recenzje` | PR without a verdict for head after the run (`ids_bez_sladu`) |
| | `PR scalone 7 dni` (PEŁNY) | merged / remedial (naprawa/fix/revert) after PRs with verdict OK / – / – | `recenzent/stan.json` | remedial after OK → entry in the playbook |
| **aplikacje** | `ASC uwagi` | `betaFeedbackScreenshotSubmissions` + `betaFeedbackCrashSubmissions` **all, by ID** / ID not in the register / registered + order in the run / register rows | `aplikacje/opinie-testflight.md` | `przeczytane ≠ w_rejestrze` or new > handled |
| | `ASC buildy` | builds with `externalBuildState`, groups / new since `stan.json → ostatni_build` / recorded / – | `aplikacje/stan.json` | build in an error state; no new build > 24 h after a merge into `main` (when Codemagic works) |
| | `Codemagic` | latest builds (status) / failed since the last run / reported / – | `aplikacje/stan.json` | `402`/`failed` without a report (incident 2 from the analysis) |
| | `apps@` | Resend threads (all pages) by ID / new or with a new message / summarised + action / mailbox register rows | mailbox register in `aplikacje/` (per `agent.md`) | thread without a trace |
| | `recenzja sklepów` | App Store Connect and Google Play version status / change since `stan.json` / recorded + reported to the CEO / – | `aplikacje/stan.json` | status change without an entry |
| **aplikacje (task `org-sklepy-maile-a`)** | `ASC uwagi`, `ASC buildy`, `recenzja sklepów` as above; `opinie/widziane.json` ↔ register (same ID) | | `aplikacje/opinie-testflight.md`, `opinie/widziane.json` | ID seen but not in the register |
| **recepcja** | `kontakt@` | **all** threads (inbox, archive, spam; pagination) / ID not in the register or a new message in an old thread / row with a hand-over or a spam/test mark / register rows | `recepcja/rejestr.md` (thread Id column) | `otwarte` > 0 after the run (deployed 8.11 – `agent.md` item 2) |
| **obsluga** | `support@` | threads with the last message after `ostatni_przebieg_at` (all pages) / without a trace in the register / row + draft/order / rows; `otwarte` = tickets without "zamknięte"/"wysłane" (closed/sent) | `obsluga/zgloszenia.md` | thread without a trace (`agent.md` item 2a) |
| | `zgłoszenia otwarte` | open rows / new / closed in the run / – ; `otwarte` = waiting for the owner's consent (draft ready) | `obsluga/zgloszenia.md` | draft waiting > 48 h without a reminder |
| **prawny** | `legal@` | Resend threads by ID / new / summarised + action / mailbox register rows | mailbox register in `prawny/` | thread without a trace |
| | `PR-y dokumentów` | `gh pr list --state all --limit 30` with terms/policy/`app/sklepy/teksty`/`lang` files / new since `stan.json → ostatni_pr` / assessed (entry in the risk register or "no comments" in the log) / – | `prawny/stan.json`, `prawny/rejestr-ryzyk.md` | PR changing a document without an assessment > 24 h |
| | `rejestr ryzyk` | open risks / new / updated / rows; `otwarte` = high | `prawny/rejestr-ryzyk.md` | high risk without a dated action |
| | `kalendarz terminów prawnych` (once created – KPI A-prawny-1) | deadlines ≤ 30 days / new / with the action prepared / rows | `prawny/kalendarz.md` | deadline < 14 days without preparation |
| **ochrona-danych** | `privacy@` | threads by ID / new / registered (request → `rejestr-zadan.md` immediately) / rows | `ochrona-danych/rejestr-zadan.md` + mailbox register | data subject request without an entry; Art. 12 deadline (30 days) < 7 days |
| | `naruszenia` | breach reports (mailbox, orders, security logs) / new / assessed (72 h) / rows | breach register (`stan.json → naruszenia_otwarte`) | breach without an assessment > 24 h |
| | `PR-y z danymi osobowymi` | PRs in `dom` changing the `dom`/`anon` schema/export/deletion (`files` from `supabase/migrations`, `dom-konto-usun`) / new / with an opinion / – | `ochrona-danych/analizy/` | migration touching personal data without an opinion before merge |
| **bezpieczenstwo** | `security@` | threads by ID / new / registered + CVSS assessment / rows | `bezpieczenstwo/rejestr.md` | report without an assessment > 24 h |
| | `alerty GitHub` | `gh api repos/OWNER/produkt/dependabot/alerts?state=open` (+ code scanning, secret scanning) / new / assessed / register rows | `bezpieczenstwo/rejestr.md` | alert open > 7 days without an entry |
| | `Supabase advisors` | ERROR / WARN / INFO counts (`get_advisors`) / new ERROR / reported to the architect / previous measurement in `stan.json` | `bezpieczenstwo/stan.json` | ERROR > 0; WARN increased without an order |
| | `PR-y wrażliwe` | PRs with auth/RLS/secrets/Edge Functions files / new / reviewed / – | `bezpieczenstwo/stan.json → ostatni_pr` | sensitive PR merged without review |
| **podatkowy** | `tax@` | threads by ID / new / summarised + action / rows | mailbox register in `podatkowy/` | thread without a trace |
| | `kalendarz terminów podatkowych` | deadlines ≤ 30 days / new / with a return or decision prepared / rows | `podatkowy/kalendarz*.md` | deadline < 7 days without the status "przygotowane" (prepared) |
| **ksiegowy** | `accounting@` | threads by ID / new / registered (attachment → `~/org-spolka-dokumenty/`, only the name in the repo) / rows | mailbox register in `ksiegowy/` | invoice in an e-mail without an entry |
| | `Stripe ↔ księgi` | Stripe transactions of the month (aggregate select from `ksiegowosc`) / unreconciled / reconciled / – | `ksiegowy/miesiace/RRRR-MM.md` | unreconciled > 0 after the 5th day of the following month |
| | `paczka miesiąca` | items of the closing list / new documents / ticked off / – ; `otwarte` = missing | `ksiegowy/miesiace/RRRR-MM.md` | missing item 3 days before the package deadline |
| **finansowy** | `finance@` | threads by ID / new / summarised + action / rows | mailbox register in `finansowy/` | thread without a trace |
| | `alerty finansowe` | alerts from `system-raportowania.md` ch. 7 checked / active / reported / – | `finansowy/kpi.md` | active alert without a report |
| | `KPI tygodnia` (Thursday) | 1 row for the week / missing = 1 new / appended / rows | `finansowy/kpi.md` | Thursday without a row (KPI A-finansowy-1) |
| **sprzedaz** | `partnerships@` + `sales@` | threads by ID (both mailboxes) / new / company in the funnel updated / funnel rows with a thread ID | `sprzedaz/lejek.md` | thread without a company in the funnel |
| | `lejek` | companies / new / with a status update / rows; `otwarte` = "do kontaktu" (to contact) with a past date or waiting for the owner's "yes" | `sprzedaz/lejek.md` | company "do kontaktu" past the date without action |
| **inwestorzy** | `investors@` | threads by ID / new / summarised + passed to the CEO (sends nothing) / rows | mailbox register in `inwestorzy/` | thread without a trace > 1 window |
| | `investor update miesiąca` | 1 / missing in the month / draft ready / – | `inwestorzy/` (KPI A-inwestorzy-1) | no draft after the 5th day of the month |
| **marketing** | `marketing@` | threads by ID / new / summarised + action / rows | mailbox register in `dom/docs/marketing/` or `marketing/` | thread without a trace |
| | `wydania i PR-y` | `app/package.json` on `origin/main`, App Store/Google Play version, PRs > `stan.json → ostatni_pr` / new / described in communications / – | `docs/marketing/stan.json` | release without a communications update > 24 h |
| | `KPI lejka` (week) | row for the week / missing / appended / rows | `marketing/kpi.md` | week without a row (KPI A-marketing-3) |
| | `zadania planu` | marketing W46 items (T46.x) / new / with progress in this run / orders with `Plan:` | `ceo/plan/tydzien-*.md`, `zlecenia/marketing.md` | plan item without an order (PM reports) |
| **inzynier** | `plany architekta` | plans in `architekt/plany/` assigned to the engineer / new since the last run / with a branch `agent/inzynier-<temat>` or a PR / `stan.json → w_toku` entries | `inzynier/stan.json` | assigned plan without a branch > 1 window; work outside the plan (PR without a plan and an order) |
| | `PR własne otwarte` | `gh pr list --author` + branches `agent/inzynier-*` / with a new reviewer comment or red CI / with a reply (commit, comment) / `stan.json` entries | `inzynier/stan.json`, `recenzent/stan.json` (verdict by head SHA) | reviewer comment without a reply > 1 window; PR open > 3 days without movement |
| **tester** | `scalenia produktu` | `gh pr list --state merged` since the last run, **by number** / without a test result / tested (result recorded) / `stan.json → przetestowane` entries | `tester/stan.json`, `tester/lista-reczna.md` | merge without a test > 24 h; manual list item before a release without a result |
| | `błędy otwarte` | open `T-…` rows / new in the run / closed after a verified fix / rows; `otwarte` = critical | `tester/bledy.md` (T-…) | critical bug without an order to the author in the same run; critical open > 48 h |
| **lokalizacja** | `scalenia produktu` | merges into the main branch since the last run, by PR number / not in `stan.json → sprawdzone` / checked (numeric result, "0 missing" too) / `sprawdzone` entries | `lokalizacja/stan.json`, `lokalizacja/rejestr.md` (L-RRRRMMDD-NN) | merge without a check > 24 h; critical gap open > 6 h |
| | `Co nowego wersji w drodze` | versions on their way to the stores (`aplikacje/stan.json`) / without a full set of languages / completed / register rows | `lokalizacja/rejestr.md` (`<wersja>@<sklep>`) | version before submission without a full set of "What's new" |
| | `kraje dostępności` | countries from the aplikacje agent / new relative to the register / added with a language and a fallback language / rows | `lokalizacja/kraje.md` | availability country without a language or a fallback language |
| **compliance** | `bramki wydań` | versions on their way to release (`aplikacje/stan.json`, orders "bramka <wersja> <platforma>") / without a gate / with the gate done (passed or stopped, with a list of gaps) / `stan.json → bramki` entries | `compliance/bramka-wydania.md`, `compliance/stan.json` | version sent to a store or to production without a gate |
| | `PR-y obszarów wrażliwych` | merges in sensitive areas since the last run, by PR number / not assessed / assessed (requirement, gap or "no impact") / – | `compliance/rejestr-wymogow.md`, `compliance/luki.md` | sensitive merge without an assessment > 1 window; high gap without an owner and a deadline |
| | `kalendarz wymogów` | deadlines ≤ 30 and ≤ 90 days / new / with the action and owner prepared / rows | `compliance/kalendarz.md` | deadline < 14 days without preparation |
| | `zmiany przepisów i wytycznych` | sources from the register checked in the run (acts, authorities, stores) / new changes / assessed (requirement added or "not applicable" with a justification) / rows | `compliance/rejestr-wymogow.md` (source + read date) | change without an assessment > 7 days; source not read > 30 days |
| **wywiad** | `źródła i konkurenci` | register items checked in the run (`WebSearch`/`WebFetch`, URL + read date) / new findings / recorded as `WR-…` with EV and passed on by an order / rows | `wywiad/rejestr.md` (WR-…) | finding with EV without an order to the CEO; register source not read > 14 days; number without a URL and date |
| **pm** | `zlecenia/*.md (wszystkie)` | open headers in all mailboxes / new since the last run / included in the backlog / J rows; `otwarte` = past the deadline | `backlog.md` (J) | header without a J row; J without a header; past the deadline without a proposal |
| | `zadania zaplanowane + crony` | tasks in the app (list) + `cron.job` / new or changed rhythm / included / C rows | `backlog.md` (C), `koordynacja-odstepy.tsv` | task without a C row; rhythm ≠ tsv ≠ README |
| | `przebiegi/` (kontrakt) | lines from the last 24 h / agents without a line in their window / reported to the CEO / expected runs from the schedule | `przebiegi/*/`, tags `przebieg/*` | no line in the window; `NIEPEŁNY`/`BŁĄD` without a reaction; `rozbieznosci` open > 1 window |
| | `rejestr-sesji.md` | open sessions / new / reported (> 24 h, without a parent, duplicate) / rows | `rejestr-sesji.md`, `list_sessions` | as for the CEO (one owner of the review – proposal C-53) |

An agent that does not yet have a register for a source (e.g. mailbox register, calendar) creates it on the first run after rollout (a file in its own directory, with an ID column)
and until then reports `w_rejestrze: 0` with the discrepancy "no register – creating it". A source the agent cannot read because a tool is missing (e.g. Resend)
is `przeczytane: null` + `bledy`, **not** `0`.

## 5. Reconciliation and result rules

1. **Numbers always come from the source, in this run.** `stan.json`, the previous log and memory from the previous run are not a source of numbers (incidents 1 and 8 from the analysis).
2. **Discrepancy** = any of: `przeczytane ≠ w_rejestrze` where the register is meant to contain everything (ASC uwagi, kontakt@, PRs ↔ verdicts); `nowe > obsluzone`
   after the run; an item from the "Discrepancy =" column in section 4. Every discrepancy has a `dzialanie` – handled in this run or **a "Pilne" entry in `zlecenia/ceo.md`**
   (or with the relevant agent) **in the same run**; "I will do it next time" is allowed only with the ID in `stan.json` and an entry in `rozbieznosci`.
3. **CISZA is allowed only when all sources have `nowe = 0` and `rozbieznosci = []`.** Otherwise the mode is ZDARZENIE (event; even for a single item).
4. `wynik` is computed from the content (the validator enforces it): `BŁĄD` (run aborted) > `NIEPEŁNY` (source not read) > `ROZBIEŻNOŚĆ` > `OK`.
5. The line is created **before the final commit**, also when the run ends with a tool error – then `tryb: "BŁĄD"` and whatever could be established.
6. **Failed run** (for the watchdog, the PM and KPIs): no line within the schedule window (tolerance: window length + 30 min), `wynik: "BŁĄD"`, `NIEPEŁNY` two runs
   in a row for the same source, or a discrepancy without `dzialanie`. Failed run = a line in the PM's report for the CEO (daily) and in the Friday review.
7. Do not duplicate: you describe actions once (in `akcje` by ID); the log may summarise, it does not have to repeat the numbers.

## 6. The `pm/narzedzia/przebieg-raport.py` tool (no LLM, no network)

- `python3 -I pm/narzedzia/przebieg-raport.py dopisz <plik.json | ->` – reads a JSON object (file or stdin), fills in `wersja` and `koniec`, **validates**
  (required fields, ISO 8601 with time zone, mode and result from the list, integers or `null` with an entry in `bledy`, `obsluzone ≤ nowe`, `wynik` consistent with the content,
  no e-mail addresses or secrets, ≤ 8,000 characters) and appends the line to `przebiegi/<agent>/<data startu>.jsonl`. Code 1 + list of reasons on rejection – the agent fixes it.
- `sprawdz <plik.jsonl>…` – validates existing files (watchdog, PM). `podsumuj [dni]` – per-agent table: runs, modes, results, discrepancies, errors, latest
  (basis of the Friday review and the KPI row). Tested on 8.11 with the examples from section 3 (valid line accepted, invalid one rejected with 4 reasons).
- Recommended pattern in a run: write the JSON to a temporary file (`$TMPDIR/przebieg.json`), call `dopisz`, only then `git add`/commit.

## 7. Block to paste into `<agent>/agent.md` (pasted by the CEO; substitutes `<agent>` and the source list from section 4 for each agent)

```markdown
### Run contract – machine report after EVERY run, also CISZA (from 9.11.2026, stage 1 of reliability; spec: `pm/specyfikacje/2026-11-08-kontrakt-przebiegu.md`)

At the end of the run (before the commit; in CISZA instead of "I record nothing") append one JSON line to `przebiegi/<agent>/<RRRR-MM-DD>.jsonl` with the command
`python3 -I pm/narzedzia/przebieg-raport.py dopisz <plik.json>` (it rejects a line that breaks the contract – fix it and repeat). Fields: `agent: "<agent>"`, `zadanie`
(task id or `czat`), `konto` (A/B), `okienko` (`scripts/przebieg.sh okienko <agent>`), `start`, `koniec` (ISO 8601 with time zone, `date -Iseconds`), `tryb`
(CISZA/ZDARZENIE/PEŁNY/… as in item 2), `zrodla`, `akcje`, `rozbieznosci`, `bledy`, `wynik`, `raport` (closing sentence), `kpi` (content of the **KPI:** line).

**Mandatory sources** (each always in `zrodla`, with numbers from this run – never from memory or from `stan.json`): `zlecenia/<agent>.md`, `decyzje.md`
and: <LISTA ŹRÓDEŁ AGENTA Z SEKCJI 4 – nazwa · co liczyć · rejestr>. For each: `przeczytane` (how many you fetched – the whole list when the source requires it), `nowe`
(not in the register or new since the last run – compared **by ID**), `obsluzone` (of the new ones – dealt with now), `w_rejestrze` (register rows after the
run), optionally `otwarte`, `ids_bez_sladu`. The source could not be read → `przeczytane: null` + an entry in `bledy` (never `0`).

**Discrepancy** (`przeczytane ≠ w_rejestrze` where the register holds everything; `nowe > obsluzone`; the cases from your list) = work **in this run** or a
"Pilne" entry in `zlecenia/ceo.md` in this run – always with the `dzialanie` field. **CISZA only when all sources have `nowe = 0` and there is no discrepancy.**
In CISZA commit the file alone: `git add przebiegi/<agent>/ && git commit -m "<agent>: przebieg CISZA <data> <GG:MM>"`, then `git pull --rebase` and push.
Conflict in `.jsonl` → keep both lines. No personal data (no e-mails, names), no secrets, no e-mail content – only IDs and numbers.
No line, numbers without a source or `0` instead of `null` = failed run (the fleet watchdog and the PM count it as a failure, even when the session "succeeded").
```

For the PM (`pm/agent.md`) the CEO pastes the same block with the sources from the **pm** row in section 4; in addition, in every DZIENNY (daily) run the PM reads `przebiegi/*/` from the last 24 h
(`podsumuj 1`) and writes into the report for the CEO: agents without a line in the window, `BŁĄD`/`NIEPEŁNY`, discrepancies without action.

## 8. Weekly reliability review (Friday, PM's PEŁNY run) – `pm/przeglady/RRRR-MM-DD.md`, section "Niezawodność" (Reliability)

Sources: `przebiegi/*/*.jsonl` (7 days, `podsumuj 7`), tags `przebieg/*` on origin, `ceo/straznik-zdarzenia.md` (once the watchdog is deployed), "Pilne" entries in `zlecenia/ceo.md`,
agent logs, `decyzje.md`. Per-agent table and fleet summary:

| Metric | Definition | Target (CEO analysis, section 4) |
|---|---|---|
| Run coverage | lines in `przebiegi/` ÷ runs expected from the schedule (`koordynacja-odstepy.tsv`, 7 days; code 10 = expected run fulfilled by the second account) | 100 % |
| Runs without a report with numbers | no line in the window + `NIEPEŁNY` + `BŁĄD` | 0 % |
| CISZA share | lines with `tryb = CISZA` ÷ all – too high (> 90 % at a 3 h rhythm) = candidate for a sparser rhythm (review of recurring tasks) | – |
| Discrepancies | count, per source; how many handled in the same run; how many open > 1 window | items missed in reconciliations: 0 |
| Incidents | watchdog events + incidents from logs/decisions: start, detection, who detected (watchdog / agent / PM / CEO / owner), cause class A–H from the analysis | detected by the owner: 0 |
| MTTD | median (detection − start) of the week's incidents; MTTR – median (resolution − detection) | MTTD ≤ 30 min |
| PR without a verdict > 2 h | from reviewer/CEO and watchdog discrepancies (check 2) | 0 |
| Orders past the deadline without a reply | from the backlog (J past the deadline) | 0 |

Review output: (1) the table above; (2) the 3 biggest causes of the week (classes A–H) with examples by ID; (3) **max. 5 proposals** to `zlecenia/ceo.md`
(new watchdog check, invariant for CI tests, rhythm change, change to an agent's instructions – what, why, effect, risk); (4) a `**KPI:**` line with the metrics
(proposal of new organisation KPIs `K9 – przebiegi bez raportu z liczbami (%)` and `K10 – MTTD awarii automatów` for `ceo/kpi/kpi.md`, measured from `podsumuj` and the watchdog).
The metrics go onto the plan page in the KPI section once added to `kpi.md`/`pomiar.py` (CEO).

## 9. Rollout (proposed order for the CEO) and risks

1. 8.11: the CEO adds `.gitattributes` (`przebiegi/**/*.jsonl merge=union`), pastes the block from section 7 into 16 × `agent.md` (+ the instructions of the `org-sklepy-maile-a` task) –
   first **recenzent, aplikacje, recepcja, obsluga** (they already have reconciliations from 8.11 – format only), then the CEO and architekt, then the rest.
2. 9.11: first run of each agent with a line; the PM in the 6:45 and 9:45 runs checks `podsumuj 1` and reports agents without a line; the watchdog (architekt) reads
   `przebiegi/` in check 1 instead of the freshness of `stan.json`.
3. 14.11 (Friday): first reliability review (section 8); calibration: which discrepancies are noise (e.g. `w_rejestrze` for incremental sources).
- **Cost:** approx. 1,000 CISZA commits/week with 17 tasks every 3 h (line ~1 KB) – little space, a lot of noise in `git log`; if it gets in the way, alternative: the watchdog
  reads the files locally (same Mac), and CISZA commits only once a day (the PM merges) – CEO decision after a week.
- **Risk:** an agent enters numbers "off the top of its head" – hence registers by ID and the watchdog counting independently (ASC, PR, mailboxes); line and watchdog disagreeing = class A incident.
- **Risk:** `.jsonl` conflicts between accounts – `merge=union` + the "keep both lines" rule.
- **Out of scope (stages 2–3):** consent register with IDs (`decyzja` in `akcje` currently refers to `decyzje.md` by date and abbreviation), GitHub Issues as the queue (then the order `id` = issue number).
