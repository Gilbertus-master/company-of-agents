# MCP servers: trust, pinning, screening (OWASP MCP Top 10 – MCP03 Tool Poisoning, MCP08 audit)

1. **Source, not name.** A server enters the toolbelt only from the provider's official address (`toolbelt/katalog.md`, column "Status");
   local servers from `.mcp.json` only from a signed/pinned source. The hook sees `mcp_server.source` – trust by source.
2. **Screening before connecting** (security agent, checklist): tool descriptions without instructions to the model ("ignore…", "always…"),
   without hidden Unicode, without paths to `~/.ssh`, `.env`; write tools listed and mapped to `uprawnienia.yaml`.
3. **Fingerprint pinning.** `toolbelt/mcp-odciski.tsv`: server · version · SHA-256 of the tool list (name+description+schema) · date · who checked.
   The fingerprint is computed by `python3 -I toolbelt/mcp-odcisk.py <plik-z-lista-narzedzi.json>` (the session exports the tool list: `ToolSearch`/`/mcp`).
   A fingerprint change without an entry = watchdog alert (planned check) and a ban on using write tools until review.
4. **Minimal scope.** A connector is enabled only for roles that need it (agent contract in `flota/zadania.yaml → narzedzia`);
   write tools behind `tool-approval`/the hook. No token passthrough (MCP spec): a server does not receive other parties' tokens.
5. **Audit.** Every MCP call is in `bezpieczenstwo/audyt/` (hook, pre and post).
