# What is not in the core

The core is extracted from the full template. Core documents – above all the run contract, the tool catalogue, the CEO
instructions and the risk register – describe the organisation in its full shape, so they mention elements that are not
here. That is not an installation error. An agent that meets such a reference skips it and does not recreate the missing
file from memory (`CLAUDE.md`).

| Area of the full template | Paths mentioned in the core | What to do in the core |
|---|---|---|
| Role library (engineer, tester, support, sales, marketing, finance, accounting, tax, legal, compliance and others) | `role/<rola>.md`, `role/szablony/<rola>/`, directories `obsluga/`, `sprzedaz/`, `marketing/`, `finansowy/`, `ksiegowy/`, `podatkowy/`, `inwestorzy/` | add your own role with `scripts/nowy-agent.sh` and `procesy/60-cykl-zycia-agenta.md` |
| CEO managerial controls | `ceo/kontrole/kontrole.md`, `ceo/kontrole/wtyczki.tsv`, `scripts/kontrole.py`, `procesy/55-…` | the core keeps only `ceo/kontrole/progi.yaml` – the thresholds used by the merge queue and the hook |
| KPIs, plan and portfolio of bets | `ceo/kpi/`, `ceo/plan/` | put the organisation's goal in `org.yaml`; keep the plan in `backlog.md` |
| Learning programme and CEO reports | `ceo/nauka/`, `ceo/raporty/` | collect rules in `ceo/wiedza/podrecznik.md` |
| Profitability model (BEP, EBITDA) | `finansowy/model/` | fleet cost is measured by `scripts/koszt.py` |
| Commercial policies | `polityki/cennik.md`, `rabaty.md`, `zwroty.md`, `komunikacja.md`, `outbound.md` | the core has `polityki/progi.yaml`; add your own policies as files in `polityki/` |
| Business processes | `procesy/10-…`, `20-…`, `30-…`, `40-…`, `50-…` | in the core: `00`, `05`, `15`, `16`, `60`, `90` |
| Periodic skills | `/skrot-dnia`, `/przeglad-tygodnia`, `/przeglad-zarzadzania`, `/zamkniecie-miesiaca`, `/wydanie` | in the core: `/przebieg`, `/dyzur-pm`, `/recenzja`, `/incydent` |
| Source report and research | `docs/raport/`, `docs/research/` | the architecture decisions in `docs/adr/` summarise them |

Test data (`scripts/testdane/`, `test_*` files) deliberately contains role names from outside the core: the tests check
how the mechanisms behave for a full fleet.
