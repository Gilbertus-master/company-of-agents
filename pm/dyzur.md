# PM blocker duty – RRRR-MM-DD GG:MM (performed GG:MM–GG:MM, account X)

Overwritten at each duty (`role/pm.md`, Daily item 2; procedure: skill `/dyzur-pm`). The file says **who is waiting for whom and since when**.
Every number and time has a source from this duty (command + time from `date`) – never from memory or from the previous duty.
A blocker > 6 h = an order to the CEO ("Dyżur blokad GG:MM"). No blockers: header, sources and the sentence "Dyżur: brak blokad."

**Sources:** `gh pr list` (repos from `org.yaml → repo.produkt` + the organisation) GG:MM · `recenzent/stan.json` · `zlecenia/*.md`
(`czeka:`, `Pilne`, `[w toku]`) · Issues `stan:czeka`/`typ:blokada` · `operator/kolejka.md` · `przebiegi/*/<dziś>.jsonl`
(`podsumuj 1`) · `przebiegi/dead-letter/` · `ceo/do-decyzji.md` · addressees' logs (only headers of entries since the blocker began).

**PRs:** open N (no drafts) · ready but unmerged (verdict OK + CI green + MERGEABLE): #… (for N h) · no verdict > 1 h: #… ·
with a conflict (author): #… · merged since the previous duty: N.
**Orders:** `czeka:` N · `Pilne` without an answer N · `[w toku]` > 6 h with no trace in the addressee's log N · overdue N (`backlog.py --sprawdz`).
**Operator:** items in the queue N, past SLA (`org.yaml → sla_godziny`) N. **For decision:** matters N, older than 7 days (expired – default "no") N.
**Fleet (24 h):** no line in the window: … · BŁĄD/NIEPEŁNY: … · dead-letter: … · F8 OK without evidence: N.
**Division reports (Mon):** N/M leads submitted `<lider>/pion/<data>.md`.

| # | What | Who blocks | Who waits | For how many h | What to do (who) |
|---|---|---|---|---|---|
| 1 | <PR #N – topic; verdict OK GG:MM, CI green> | <who is to merge per uprawnienia.yaml> | <author, dependent sessions> | N | <merge in the next run; if > 6 h – CEO> |
| 2 | <order Z-… / #N – topic> | <addressee> | <requester> | N | <Pilne reminder added GG:MM / wake up (CEO)> |
| 3 | <operator action #N> | operator | <agent> | N (SLA N h) | <CEO: escalation to the operator via the channel from org.yaml> |

**Pilne added to agents (once per 6 h):** `zlecenia/<agent>.md` – <list or "none">.
**Resolved since the previous duty:** <what and how – link to PR/commit/entry>.
**Within deadline (no Pilne):** <matters being watched whose deadline has not passed yet>.
**Recurring (repeat in ≥ 3 duties):** <list with the matter number in `ceo/do-decyzji.md` / `operator/kolejka.md`> – PM's proposal to the CEO: <…>.
**PM's own:** <PM backlog: DZIENNY not done, backlog not recalculated, Friday review>.
