# Process 00 – a new agentic organisation from scratch

Goal: about 2 weeks after the owner's decision the organisation runs with a CEO, a management core and the first domain
agents. A human performs only the steps marked **[operator]** or **[właściciel]** (owner).

Start-up rule: **the fleet grows in stages.** Every stage has a gate – the next one starts only when the previous one has had
OK runs for 3 days (`przebiegi/`) and there are no open watchdog alerts. In the source organisation 21 agents
were created within a few days, and the reliability mechanisms (run contract, watchdog, claiming work) were added after
incidents. Here they are in place from day one.

## Stage 0 – decision and configuration (day 0, owner + Claude in chat)
1. **[właściciel]** A one-page business thesis: what, for whom, revenue model, what we do not do.
2. Copy the template: `gh repo create OWNER/<firma>-agenci --private --template Gilbertus-master/company-of-agents`
   and clone it to `~/projekty/<firma>-agenci`.
3. **One command:** `scripts/init-org.sh --nazwa "…" --domena … --prefiks … --github OWNER/<firma>-agenci --operator …`
   – fills in `org.yaml`, records the start-up decision `D-…-01`, creates the core agents (ceo, pm, recenzent), creates
   the Issues labels, generates prompts, runs the tests and prints the list of tasks to create and the steps for the operator.
   Add the mandate, budget and approval threshold to `org.yaml` by hand.
4. Review `uprawnienia.yaml` – keep it cautious at the start.
5. Fill in `polityki/` (price list, refunds, discounts, communication tone) – even as version 0.1.
6. `git push` (private repo) and the Actions secrets `STRAZNIK_TOKEN` (+ optionally `STRAZNIK_RESEND_KEY`) – the fleet
   watchdog (`.github/workflows/straznik.yml`) starts on its own every 15 min, without a laptop.

## Stage 1 – legal identity and accounts (days 1–5, operator)
The steps are prepared by the CEO agent (after stage 2 – until then Claude in chat), the operator performs them:
1. **[operator]** Company (e.g. sp. z o.o. via S24): the management board and the beneficial owner (CRBR) are natural persons.
2. **[operator]** Company bank account (KYC), read access through a PSD2 aggregator.
3. **[operator]** Domain, mail: functional addresses (kontakt@, support@, legal@, privacy@, security@, accounting@,
   tax@, sales@, investors@) via Resend/AgentMail.
4. **[operator]** Secrets manager (1Password / Infisical) with a service account for the agents.
5. **[operator]** Accounts for the services from `toolbelt/katalog.md` needed in stages 2–3 (GitHub, Stripe, invoicing
   software, KSeF – granting permissions with the trusted profile).
6. **[operator]** Claude account (a plan with Routines) and the MCP connectors from the toolbelt.

## Stage 2 – management core (days 3–7)
1. The core agents have existed since stage 0 (`init-org.sh`); fill in `ceo/kpi/kpi.md` (O1–O3) and `ceo/plan/` (year, quarter,
   first week).
2. Check in the Actions tab that the fleet watchdog has green runs (alerts: `zlecenia/ceo.md`, `ceo/straznik-zdarzenia.md`,
   mail to the operator when there is a Resend key).
3. Tasks of the core agents – depending on `org.yaml → runtime.glowny`:
   - **`claude-routines` (recommended, no laptop):** `python3 flota/generuj.py --routines --tylko limit,scalanie,ceo,pm,recenzent`
     → `ceo/handoff/routines/<agent>.md` with ready fields (cron `CRON_TZ=…`, model, effort, repo, connectors, prompt);
     **[operator]** creates each one at claude.ai/code/routines or via `/schedule` and clicks "Run now";
   - **`desktop-scheduled`:** `python3 flota/generuj.py --lista --konto A --tylko …` → tasks in the Claude app;
   - **`agent-sdk` (own server):** cron calls `scripts/przebieg-headless.sh <agent>` – a hard cost limit
     and cost recording to `przebiegi/<agent>/koszty-*.jsonl`.
   Run every task once manually so that it does not get stuck on a permission prompt.
4. Cost telemetry (`toolbelt/obserwowalnosc.md`): **[operator]** enables OTel in `~/.claude/settings.json` of the run
   machine or in the cloud environment variables; without it the cost is visible only from the headless wrapper and the account limit.
5. Gate: 3 days of CEO and PM runs with an OK result; the CEO keeps `operator/kolejka.md` and `ceo/do-decyzji.md`.

## Stage 3 – first line of revenue and risk (week 2)
1. Choice according to the thesis: usually `marketing`, `sprzedaz` or `obsluga` + `recepcja`, and `prawny`, `compliance`, `ochrona-danych` (when there is personal data – from the start),
   `ksiegowy`, `finansowy`.
2. Every agent: `scripts/nowy-agent.sh <rola>`, a row in `koordynacja-odstepy.tsv` and `flota/zadania.yaml`,
   the source table in `pm/specyfikacje/kontrakt-przebiegu.md` (section 4), `python3 -m unittest discover flota`.
3. Agents reading untrusted content (support, reception) – an injection test set in `ewaluacje/` before start.
4. Gate: 3 days without alerts, requests handled within SLA.

## Stage 4 – product and the full fleet (week 3+)
1. `architekt`, `inzynier`, `tester` – process `procesy/10-budowa-produktu.md`; with apps in stores and many languages also `aplikacje` and `lokalizacja` (releases: `procesy/40-wydanie.md`).
2. `bezpieczenstwo`, `podatkowy`, `wywiad`, `inwestorzy` – when there is something to guard. Every new agent according to `procesy/60-cykl-zycia-agenta.md` (3-day gate), monthly reviews according to `procesy/50-przeglad-agenta.md`.
3. Divisions with leads and division reports (once a division has 4 domain agents).

## Ongoing
- Every week the CEO: KPI "operator minutes", proposals to loosen `uprawnienia.yaml` and to remove the human from the loop.
- Every week the PM: fleet cost, justification of recurring tasks, contradictions between decisions.
- Every month the owner: review of the mandate, budget and permissions (30 min, based on the CEO's report).
