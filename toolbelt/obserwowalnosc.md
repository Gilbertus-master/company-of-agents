# Fleet observability: cost and result per agent

Three sources, from the simplest:

| Source | What it gives | Where |
|---|---|---|
| **Run contract line** (agent) | mode, sources, actions, result OK/ROZBIEŻNOŚĆ, optionally `model` and `koszt_usd` | `przebiegi/<agent>/<data>.jsonl` |
| **Headless wrapper** (`scripts/przebieg-headless.sh`) | `total_cost_usd`, time, turns, error – from the JSON result of `claude -p` (client-side estimate) | `przebiegi/<agent>/koszty-<data>.jsonl` |
| **OpenTelemetry from Claude Code** | metrics `claude_code.cost.usage`, `claude_code.token.usage` (input/output/cacheRead/cacheCreation), `claude_code.session.count`, `claude_code.active_time.total`, `claude_code.commit.count`, `claude_code.pull_request.count`; events `claude_code.tool_result`, `claude_code.api_error` | OTLP collector → Langfuse (self-hosted, EU) or Prometheus/Grafana |

Weekly report: `python3 -I pm/narzedzia/przebieg-raport.py podsumuj 7` sums the cost from both files per agent.
KPI F2 "fleet cost / revenue" (`ceo/kpi/kpi.md`) – the finance agent takes the number from this report or from Langfuse.

## Enabling telemetry (operator, once per run machine)
OTEL variables in the repo's `.claude/settings.json` are **ignored** – they must be in the machine's `~/.claude/settings.json`
(or in the organisation's managed settings; in cloud scheduled tasks – in the cloud environment's environment variables):
```json
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_EXPORTER_OTLP_PROTOCOL": "http/protobuf",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "https://otel.<domena-firmy>",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Basic <token z 1Password, nie wpisywać na stałe – otelHeadersHelper>",
    "OTEL_RESOURCE_ATTRIBUTES": "org.prefiks=<prefiks>,deployment.environment=flota",
    "OTEL_METRIC_EXPORT_INTERVAL": "30000"
  },
  "otelHeadersHelper": "/usr/local/bin/otel-naglowki.sh"
}
```
- `otelHeadersHelper` – a script printing the headers (token from `op read`), so the secret does not sit in a file.
- Agent role: the headless wrapper adds `agent.role=<agent>` to `OTEL_RESOURCE_ATTRIBUTES` (environment variable per
  launch). In the desktop app and in the cloud one session = one agent, so the role is a session attribute; subagent
  metrics have their own `agent.name`.
- **Do not enable** `OTEL_LOG_USER_PROMPTS`, `OTEL_LOG_TOOL_CONTENT`, `OTEL_LOG_RAW_API_BODIES` – prompt and tool
  content contains personal data and secrets; metrics and events without content are enough for costs.

## Langfuse (self-hosted in the EU)
- Docker Compose on the company VPS (Postgres + ClickHouse + Redis + Langfuse); domain `langfuse.<domena>`; access only
  for the operator, the CEO (read via API) and the finance agent (read).
- The OTel collector (otel-collector-contrib) accepts OTLP from Claude Code and exports to Langfuse (endpoint
  `/api/public/otel`) – configuration in `toolbelt/uslugi/langfuse.md` after the first deployment (stage E).
- Alternative without your own server: Prometheus + Grafana Cloud (free tier) for cost metrics only.

## Budgets
- `flota/zadania.yaml`: `budzet_usd_przebieg` (hard limit `--max-budget-usd` in headless) and `budzet_usd_dzien`
  (soft: `scripts/przebieg.sh` prints `BUDŻET-AGENTA`, the agent works only in `ZDARZENIE` (event) mode).
- Whole-fleet budget: the account limit (task `limit`) and the capacity threshold (P3 ≥ 85%, P2 ≥ 95%).
- PM on Friday: cost per recurring task versus value (`backlog.md`, section "Cykliczne").
