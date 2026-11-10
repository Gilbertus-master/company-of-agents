# Claude plugins for the fleet – role → plugin map

Plugins work on the Claude account on which they are enabled (the fleet account – `flota/konto-start.md`). The operator installs them.
Rules (from the source organisation, 7.11.2026, generalised):
1. Skills from plugins are a **way of working** (checklists, templates) – they do not change `zasady.md` or `uprawnienia.yaml`:
   external actions still go through the hook and decisions.
2. Plugins with third-party MCP servers – only after an entry in `toolbelt/katalog.md`, a data protection opinion (DPA) and screening
   (`toolbelt/mcp-pinning.md`).
3. Priority: Anthropic and official partner plugins; community ones – local only, after a security review.
4. The agent records in its log which skill it used; the PM removes unused ones from the map every month.

## Recommended – level 1 (Anthropic / official partner)
| Plugin | Roles | Key skills |
|---|---|---|
| superpowers | architect, engineer, CEO | brainstorming, writing-plans, test-driven-development, code review, verification |
| Legal / commercial-legal / ip-legal | legal, sales | review-contract, compliance-check, NDAs and contracts, renewal-tracker, oss-review |
| privacy-legal / ai-governance-legal | data protection, compliance | dpa-review, dsar-response, DPIA, ai-inventory, vendor-ai-review |
| Finance / Data | finance, accounting, CEO | close-management, reconciliation, variance-analysis, sql-queries, dashboard |
| Marketing / Product Marketing | marketing | campaigns, brand voice, positioning, personas |
| Sales | sales | account-research, call-prep, draft-outreach (only with consent – `polityki/outbound.md`), pipeline-review |
| Customer Support | customer service, reception | ticket-triage, draft-response, kb-article, escalation |
| Product Management / Operations | architect, PM, CEO | write-spec, roadmap, capacity-plan, runbook, risk-assessment, status-report |
| Design | marketing, architect, apps | design-critique, UX copy, accessibility |
| official plugins of providers in the stack (e.g. Supabase) | architect, security | database best practices, RLS, auth |

## Level 2 (community, local – after a security review)
| Plugin | Roles | What for |
|---|---|---|
| security audit (OWASP, threat model) | security, architect | OWASP Top 10 / LLM, ship-check |
| dependency check (CVEs, licences) | security | dependency alerts |
| mobile app design | apps, architect | platform conventions |

## Not now
Paid integrations requiring accounts – only when needed and budgeted (decision D-…). There are no ready plugins for KSeF and Polish
law – the agents' own processes remain (`procesy/30-zamkniecie-miesiaca.md`, the legal role).
