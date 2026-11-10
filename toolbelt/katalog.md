# Toolbelt – catalogue of external plugins

Rule: **we do not write what can be plugged in.** Every area of the company is served by an off-the-shelf service driven by an agent
(MCP, CLI or API). We write our own code only where there is no service or where it is a competitive advantage.

An idea from other teams' practice: the agent has a toolbelt prepared in advance – services
it is already logged in to, with a usage description and ready code snippets – and sets up a project in them itself instead
of writing an integration from scratch.

Status legend: **MCP-ofic.** – the provider's official MCP server · **MCP-społ.** – community server · **CLI** ·
**API** – REST only. "verified" – used in the source organisation. "?" – unconfirmed in the 9.11.2026 research,
to check before choosing. Column "Who" – the agent that has access (least-privilege principle).

## Core (every organisation)

| Area | Default choice | Alternatives | Status | Who | Notes |
|---|---|---|---|---|---|
| Agent runtime | Claude Code: Routines (cloud) | scheduled tasks in the desktop app; Claude Agent SDK; Managed Agents | verified (desktop) | all | Routines do not need the laptop switched on; desktop only for GUI |
| Organisation layer | this repo (files + Issues) | Paperclip (open source, MIT: org chart, budgets, approvals, audit) | verified | all | Paperclip to be tested as a panel over Claude Code (stage 4 of the plan) |
| Repo and orders | GitHub + `gh` CLI | Linear (MCP-ofic.) | verified | all | one technical account per agent = accountability |
| Working method | superpowers plugin (brainstorming, writing-plans, TDD, code review, verification) | – | verified | architect, engineer | specifications and plans in the product repo |
| Secrets | 1Password CLI (`op run`) or Infisical | macOS keychain (single Mac only) | verified: keychain | per task | `toolbelt/sekrety.md`; a secret never in the model's context |
| Agent observability | OpenTelemetry from Claude Code → Langfuse (self-hosted in the EU) | – | – | pm, ceo | cost and result per agent and run |
| Fleet watchdog | `moduly/straznik` (Python without an LLM) in GitHub Actions every 15 min | launchd on a Mac (`instaluj.sh`) | verified (launchd) | – | works without the laptop; secret `STRAZNIK_TOKEN` |
| Notifications to people | PushNotification in the Claude app + e-mail | – | verified | ceo | only operator and owner |

## Communication

| Area | Default choice | Alternatives | Status | Who | Notes |
|---|---|---|---|---|---|
| Agent mailboxes | Resend (inboxes + sending) | AgentMail (MCP-ofic., Claude connector) | verified: Resend MCP-ofic. | reception, customer service, agents with an address | every agent sends only from its own address |
| Domain and DNS | Cloudflare | OVH | verified: OVH | security (read) | DNS changes → owner |
| Phone / SMS | Twilio | Zadarma | API; MCP ? | customer service | the company number is set up by the operator |
| Owner's mail | Gmail (connector, read and drafts) | – | verified | ceo | read only; we send nothing from the private mailbox |

## Growth and customers

| Area | Default choice | Alternatives | Status | Who | Notes |
|---|---|---|---|---|---|
| CRM | Attio (MCP-ofic.) | HubSpot (MCP-ofic.) | research | sales | |
| Helpdesk | Plain (MCP-ofic.) | Intercom Fin (MCP-ofic.), Zendesk (MCP ?) | research | customer service | rule of two: customer service does not refund money |
| Social | Typefully (MCP-ofic.) | Buffer (?) | research | marketing | publishing from a calendar approved by the CEO |
| Product analytics | PostHog (MCP-ofic.) | – | research | marketing, architect | |
| Website | static (GitHub Pages / Cloudflare Pages) | Fly.io, Vercel | verified: Pages, Fly | marketing, architect | |
| Graphic design | Claude Design / Stitch / generator with a component library | – | verified: Design | architect, marketing | ugly mock-ups → check against the specification → full design |

## Product and technology

| Area | Default choice | Alternatives | Status | Who | Notes |
|---|---|---|---|---|---|
| Database, sign-in, functions | Supabase (MCP-ofic., Management API) | WorkOS / Clerk (sign-in, ?) | verified | architect, engineer; others read aggregates | migrations according to `uprawnienia.yaml` |
| Hosting | Cloudflare / Fly.io | Vercel (MCP) | verified: Fly | architect | |
| Application errors | Sentry (MCP-ofic.) | – | research | tester, architect | |
| Mobile builds | Codemagic | EAS (Expo) | verified | architect | minutes budget! (minutes exhausted = error 402) |
| App stores | App Store Connect API, Google Play API | – | verified | apps agent | developer accounts are set up by the operator |
| Push | Firebase Cloud Messaging | – | verified | engineer | |
| Mobile subscriptions | RevenueCat | – | to check | finance | check the Apple Small Business Program (15%) |
| E2E tests | Playwright | – | verified | tester | |
| Automations without MCP | n8n (MCP Server Trigger, self-hosted) | Make | research | architect | a deterministic "pipe" called by agents |

## Money, books, law

| Area | Default choice | Alternatives | Status | Who | Notes |
|---|---|---|---|---|---|
| Payments | Stripe (MCP-ofic., Agent Toolkit) | – | verified: read | finance (read), CEO (refunds per policy) | **from 12.2026 the MCP accepts only keys tagged "Agent" – migrate by 30.11.2026** |
| PL invoices + KSeF | Fakturownia (REST; MCP-ofic. read only) | inFakt (REST, KSeF), wFirma (API ?) | verified: Fakturownia | accounting | the software handles KSeF; an invoice cannot be cancelled |
| KSeF directly | – | MCP-społ.: flippico/ksef-mcp, mcp-ksef-pl | research | accounting | only when the invoicing software is not enough |
| Bank | PSD2 aggregator (read) | Revolut Business API | research | finance, accounting (read) | transfers are made by the operator |
| Exchange rates | NBP API | – | verified | accounting | |
| Document signing | Docusign (MCP-ofic.) | Autenti (API ?) | research | legal (preparation) | a qualified signature is applied by the operator |
| Cloud credits | Google for Startups, AWS Activate, AI programmes | – | to check | finance | applications are submitted by the operator |

## The template's own tools (no LLM)

What could not be plugged in: deterministic code in the repo. Python standard library and bash, no network (unless
noted), no secrets, each with tests in `bash scripts/testy.sh`. The agent **runs** the tool and quotes the result –
it does not calculate "in its head".

| Tool | Command | Who | What it gives |
|---|---|---|---|
| Run lock | `scripts/przebieg.sh start <agent> <konto>` | every agent | one run per window with several accounts (git tag), capacity budget P1–P3 |
| Pace regulator | `python3 scripts/regulator.py --konto A --procent N --reset <ISO>` | task `limit` | even use of the account limit, speed up/slow down mode, handing work to the next account |
| Run without the app | `scripts/przebieg-headless.sh <agent>` | operator, server | `claude -p` with a cost and turn limit, run cost in `przebiegi/` |
| Run contract | `python3 -I pm/narzedzia/przebieg-raport.py dopisz\|sprawdz\|podsumuj` | every agent, pm | one JSON line per run, validation, 7-day table with cost |
| Backlog | `python3 -I pm/narzedzia/backlog.py [--sprawdz\|--json]` | pm | "Jednorazowe" (one-off) section from orders and Issues; overdue, duplicates, mismatches (network only via `gh`) |
| Session register | `python3 -I pm/narzedzia/sesje.py` | pm | sessions open > 24 h, without a parent, duplicates, closed without a result |
| PR scan | `bash recenzent/skrypty/skan-pr.sh <nr>` | reviewer | personal data, secrets, private addresses, commands for agents in added lines |
| Review reconciliation | `bash recenzent/skrypty/uzgodnienie.sh [--json]` | reviewer | open PRs ↔ verdicts for the current head SHA (network: `gh`) |
| Financial model | `python3 finansowy/model/model.py [--zaklad cena=1.1,churn=0.9]` | finance, ceo | BEP, EBITDA, runway, scenarios, lever ranking; "what if" variant with Δ BEP and Δ EBITDA |
| KPI measurement | `python3 -I ceo/kpi/pomiar.py [--sprawdz\|--trend F4]` | ceo, pm | values, colours and trend of indicators from `zrodla.yaml` (`--z-siecia` for API sources) |
| Plan and KPI page | `python3 ceo/plan/strona/zbuduj.py` | ceo | one HTML file: plan, KPI, BEP, portfolio, costs, safeguards; refuses to write with personal data |
| Decision index | `python3 -I scripts/decyzje-indeks.py [--sprawdz\|--archiwizuj]` | ceo | `decyzje.md` from `D-*.md` files, conflicting decisions = error |
| Fleet | `python3 flota/generuj.py [--sprawdz\|--routines\|--lista]` | ceo, operator | prompts and tasks from one manifest; mismatch = error |
| Watchdog | `python3 moduly/straznik/straznik.py` | CI (cron) | agent silence, queues, budgets, HALT, dead-letter – alert without an LLM |
| Permissions hook | `.claude/hooks/uprawnienia.py` (run by Claude Code) | – | enforces `uprawnienia.yaml`, gates from `polityki/progi.yaml`, audit chain |
| Evals | `python3 -I ewaluacje/uruchom.py [<agent>]` | reviewer, CI | adversarial scenarios before an instruction change (the only runner with a model) |
| New company / new agent | `scripts/init-org.sh`, `scripts/nowy-agent.sh <rola>` | operator, ceo | organisation start with one command; role with starter registers |
| Update from template | `scripts/aktualizuj-z-szablonu.sh [--sucho]` | ceo, operator | template fixes into the company without overwriting its files |
| Test suite | `bash scripts/testy.sh` | everyone, CI | tests of the tools above |

## How to add a service to the toolbelt
1. An agent proposes the service by an order to the architect: why, alternatives, cost, whether it has an MCP, what data it processes.
2. Compliance: data processing agreement (DPA), transfer outside the EEA, entry in the register of processing activities.
3. Security: key scope (minimal), where the secret lives, who has access.
4. The operator sets up the account (KYC, 2FA) and puts the secret into the secrets manager.
5. The architect adds a row here and a usage description in `toolbelt/uslugi/<usluga>.md` (how to log in from the CLI,
   example calls, limits, costs). Access check: `toolbelt/sprawdz-dostepy.md`.
6. Own tool (when there is no service): a script with a test, a row in the "Own tools" table and in `scripts/testy.sh` – in one PR.
