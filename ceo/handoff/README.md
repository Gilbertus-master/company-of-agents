# Handing over work (handoff)

The fleet must survive: exhaustion of the fleet account's limit (reset or paid overage – no other accounts), a move to another account
(change of operator or plan), a computer failure, a model change.

- `handoff.md` – the organisation's state before the fleet is moved or before a planned break (written by the CEO).
- `sesje.md` – lines from agents and sessions: agent/session · what it was doing · where the result is · resume prompt.
- `prompty/` – task prompts generated from `flota/zadania.yaml` (`python3 flota/generuj.py`); do not edit by hand.
- `routines/` – instructions for setting up cloud tasks (scheduled tasks) per agent (`python3 flota/generuj.py --routines`);
  also generated – after a manifest change, regenerate and update the tasks (`/schedule update`).
- Moving to a new fleet account: `python3 flota/generuj.py --konto B --wyjscie /tmp/prompty-b` → a list of calls that create the tasks
  (first: `limit`, second: `scalanie` – the merge agent every 15 min), **[operator]** sets them up and runs each once; MCP connectors from `toolbelt/katalog.md`;
  `toolbelt/sprawdz-dostepy.md`.
- Chat memory does not carry over between accounts – the CEO's durable memory: `ceo/wiedza/pamiec-ceo.md`. The old account disables its tasks (does not delete them).
- Fleet account, compliant billing (team/api), rate regulator and cost measurement: `konto-floty.md`; account start: `flota/konto-start.md`.
