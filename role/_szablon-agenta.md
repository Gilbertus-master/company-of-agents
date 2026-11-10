# Agent {{rola}} – {{org.nazwa}}

Agent instruction file. The scheduled task reads the section `## PROMPT

The run procedure (start, modes, finish, JSON v2 line, learning) is the skill **`/przebieg`** (`.claude/skills/przebieg/SKILL.md`)
– load it at the start of every run and follow it step by step. Summary:
1. `scripts/przebieg.sh start {{rola}} <konto>` – code 10: finish with one sentence, no work and no commit.
2. Read the permissions, new decisions, your orders, `stan.json`, the log, the playbook. Choose the mode (PEŁNY / ZDARZENIE / DZIENNY / CISZA).
3. Work within your scope; other areas = an order; matters for a human → CEO. Check state at the source.
4. Finish: log entry with **KPI:**, `stan.json`, JSON v2 line (`prompt_version`, `outcome_evidence`), validator, commit only your own files, report ≤ 12 lines.
Procedures with side effects (month-end close, incident, release) are skills with `disable-model-invocation` – you run them only deliberately, according to your role.

---

## Role description
