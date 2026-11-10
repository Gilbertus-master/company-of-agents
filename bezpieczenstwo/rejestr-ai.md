# Register of the organisation's AI systems (AI Act art. 26 – human oversight, logs ≥ 6 months: an obligation for high-risk systems, good practice here; ISO 42001 – inventory of systems)

| System | Provider / model | Purpose | Type (reader/actor/researcher) | Oversight person (authorised to HALT and rotate) | Personal data | DPA / ZDR | Logs (where, retention) |
|---|---|---|---|---|---|---|---|
| Agent fleet (Claude Code) | Anthropic – classes from `flota/zadania.yaml`, models from `org.yaml → runtime.modele` | running the organisation | per manifest | operator (org.yaml) | minimum (rules A5) | commercial plan with DPA: ☐ · model ZDR: ☐ checked | `bezpieczenstwo/audyt/` (git, indefinitely), `przebiegi/` |
| Fleet watchdog | script (no LLM) | reliability checks | – | operator | no | – | `ceo/straznik-zdarzenia.md` |
| (connectors/services from `toolbelt/katalog.md` that process personal data) | | | | | | DPA ☐ | |

Rules: entry before the system is started; quarterly review (compliance); pseudonymisation of PII in logs after 6 months;
the owner decides on the Claude plan (a consumer plan has no DPA) and on admitting models without ZDR – `decyzje/`.
