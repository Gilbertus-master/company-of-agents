# Fleet account, billing and the rate regulator

**Rule (since 10.11.2026):** the whole fleet works on **one account** – no account rotation and no switching the fleet to another
person's account that has limit to spare. Reason (our reading as of 10 November 2026 – not legal advice; check the current wording of the terms): consumer plans
(Free, Pro, Max, Ultra) are subject to the Consumer Terms – no account sharing and non-commercial use only (the version for
consumers in the EEA, §11; in other regions check your version); on the Team plan "Each seat is for one named individual", and limits "can't be transferred or shared
between team members"; the Usage Policy prohibits, among other things, creating multiple accounts to avoid detection or to circumvent product guardrails
([Claude Code: Legal and compliance](https://code.claude.com/docs/en/legal-and-compliance),
[Team plan](https://support.claude.com/en/articles/9266767-what-is-the-team-plan), [Usage Policy](https://www.anthropic.com/legal/aup)).
Only two people work in the organisation: the owner and the operator.

## Variants permitted under our reading (`org.yaml → runtime.rozliczenie`)
| Variant | How | When |
|---|---|---|
| `team` (default) | Team plan (min. 2 seats: owner and operator); the fleet runs on the operator's premium seat (`runtime.konto`); once the limit is exhausted – the organisation's paid overage (usage credits) at API rates, with a spend limit set at Anthropic equal to `runtime.nadwyzka_usd_miesiac` | scheduled tasks in the cloud (Routines) or in the Claude app |
| `api` | API key from the Claude Console: `scripts/przebieg-headless.sh` (Agent SDK) with its own schedule; USD budgets from the manifest and the fleet ceiling via HALT | own server (`runtime.glowny: agent-sdk`) |
| Usage-based Enterprise | no per-seat limits, everything at API rates (terms are set by Anthropic) | a larger company – set `rozliczenie: api` |

The owner has their own seat and their own usage (chat, decisions). Fleet tasks are not set up on the owner's seat.

## Rate regulator – task `<prefiks>-limit-<konto>` (every 30 min, the first task on the fleet account)
- Measurement `get_usage` → `python3 scripts/regulator.py --konto <K> --procent N --reset <ISO> [--procent5h N --reset5h <ISO>]`.
- `cel_teraz` (target now) = 100 % × hours since the start of the window / `okno_celu_h` (default 168 h – evenly across the whole week);
  `odchylenie` (deviation) = per cent used − target.
- Modes (file `~/.<prefiks>-limit-<K>.json`, read by `scripts/przebieg.sh`):

  | mode | condition | effect |
  |---|---|---|
  | przyspiesz | deviation < −10 pp | entry „Pilne: przyspiesz” in `zlecenia/ceo.md` (once per 2 h); the CEO adds work; second day in a row → class `lekki` gets a `mocny`-class model (F-04) |
  | normalnie | −10 … +5 pp | no change |
  | zwolnij | deviation > +5 pp (or the 5 h window ahead by > 10 pp / ≥ 90 %) | P3 does not work; at > +15 pp also P2; P1 always |
- Independently: ≥ 85 % → P3 stops, ≥ 95 % → P2 too (capacity budget). A mode older than 2 h is ignored.
- **Limit exhausted (≥ 98 %)**: one Pilne entry per window. With paid overage, work continues up to the spend limit; without it,
  only P1 works until the reset. Permanent exhaustion = an owner decision (higher seat, overage or `api` billing) – not a second account.

## Cost measurement (every run)
The `SessionStart` hook records the path of the session transcript; at the end of a run `python3 -I scripts/koszt.py zapisz --agent <agent>` sums
the tokens from the session transcript per `flota/ceny-modeli.yaml` and appends a line to `przebiegi/<agent>/koszty-<data>.jsonl` (the `SessionEnd` hook
appends the same locally if the agent did not manage to). Report: `python3 -I scripts/koszt.py raport --dni 14` – cost per agent and model,
monthly projection at API prices and comparison with the price of seats and overage. After 14 days enter the measured seat capacity
(`runtime.pojemnosc_miejsca_usd_miesiac`) and run `python3 flota/generuj.py` – agent budgets will become shares of the capacity.

## Fleet account
| Account | Person (seat) | Billing | State | Connectors connected |
|---|---|---|---|---|
| A | operator (Team premium seat) | team + overage | active | ☐ |

Connectors, plugins and Artifacts belong to the Claude account, not to the machine – list: `flota/konto-start.md`.
Moving the fleet to another account (change of operator or plan, not the limit): `ceo/handoff/START-nowe-konto.md`.
