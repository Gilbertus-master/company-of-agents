# ADR – register of the product's architecture decisions (agent architekt)

**Purpose.** Every decision that is costly to change (database, hosting, framework, data model, module boundaries, agent interface,
class B migrations) has a short record: context, decision, consequences, source. A new session reads the ADR instead of reconstructing the rationale
from logs. The organisation's ADRs (not the product's) are in the template's `docs/adr/`.

**Rules.** One file `NNNN-<temat>.md`, sequential number, status `proponowany` (proposed) → `przyjęty` (accepted; by the CEO or an owner's decision when it touches
`uprawnienia.yaml` / costs above the threshold) → `zastąpiony przez NNNN` (superseded by) / `wycofany` (withdrawn). An ADR does not change after acceptance – a new one supersedes it.
No personal data or secrets (service names – yes, keys and addresses – no). **Updated by:** the architect – with every specification from stage 8
of the process `procesy/10-budowa-produktu.md` and with every class B migration; the reviewer checks in the PR that the decision has an ADR.

| No. | Title | Status | Date | Supersedes |
|---|---|---|---|---|

## Template

```markdown
# ADR NNNN · <tytuł> · status: proponowany · RRRR-MM-DD · autor: architekt

## Context
What forces the decision (requirement, constraint, measurement, incident). Numbers with a source and a date.

## Options considered
1. … – pros / cons / cost
2. …

## Decision
One sentence. Who approved it (CEO entry „[Decyzja CEO]” / D-…).

## Consequences
What we gain, what we lose, what needs doing (orders, migrations, tests), when to review it again.

## Sources
Links (documentation, measurement, specification `docs/…`, PR).
```
