# Claude Code plugin – company-of-agents

The agentic-organisation skills, available in any folder without cloning the template.

```
/plugin marketplace add Gilbertus-master/company-of-agents
/plugin install company-of-agents@company-of-agents
```

| Skill | Purpose |
|---|---|
| `/company-of-agents:new-organisation` | creates a private company repo from the template and walks you through the first start-up |
| `/company-of-agents:przebieg` | the agent run contract (start, modes, finish, report) |
| `/company-of-agents:dyzur-pm` | the PM blocker duty and fleet report |
| `/company-of-agents:recenzja` | PR review step by step |
| `/company-of-agents:incydent` | incident runbook (HALT, snapshot, report) |

`przebieg`, `dyzur-pm`, `recenzja` and `incydent` assume you are working in a company repo created from the template
(the same skills live there in `.claude/skills/`). Skill names stay Polish, as do other identifiers – see `GLOSSARY.md`.
The plugin ships no hooks and no permission rules – the fleet's safeguards work only inside the company repo
(`.claude/settings.json`, `.claude/hooks/uprawnienia.py`).

This directory is generated at release time from `.claude/skills/` – propose changes there. Licence: Apache-2.0.
