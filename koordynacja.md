# Run coordination

Every scheduled run starts by locking its window. This way the same work is never done twice – even with
several Claude accounts, catch-up runs after the app starts, and a manual "Run now".
Chats with people (manual sessions) do not take the lock.

## Account
You take the account from the line "Konto: X" at the start of the task prompt (no line → `A`). Account list: `org.yaml → runtime.konta`.

## 1. Start – window lock
```
cd <repo organizacji> && git pull --rebase --quiet
scripts/przebieg.sh start <agent> <konto>      # e.g. scripts/przebieg.sh start ceo A
```
- **code 0** – the run is yours, carry on.
- **code 10** – in this window a run has already happened or is in progress. You finish with one sentence, no work, no commit.
  Code 10 with the message `BUDŻET:` – the account limit is high and the agent has a low priority (at ≥ 85% P3 does not run,
  at ≥ 95% neither does P2).
- **any other code** – you finish without work, report the script's message.

How it works: the script pushes the tag `przebieg/<agent>/<okienko>`. A tag can be created only once, so the first one wins.
Window = the interval from `koordynacja-odstepy.tsv` in the time zone from `org.yaml`. The script removes tags older than 14 days itself.
Test: `scripts/test-przebieg.sh`.

## 2. Work longer than one run
If an order may not fit in one run, mark it **before working**: `[w toku · konto X · RRRR-MM-DD GG:MM]`, commit
and push. Leave an entry younger than 6 h alone; treat an older one as abandoned – check what was done, mark it as yours
and finish it.

## 3. End
Log with the account in the header, `stan.json` with the field `konto`, a line in `przebiegi/`, commit only your own files.

## Chat sessions
- Every PR opened by a chat session has the line `Sesja: <session id>` in its description – the merge agent and the CEO
  use it to tell the author about the verdict (check B-13).
- After sending a PR for review the session **watches the verdict itself**: every few minutes (up to 2 h) it checks
  `recenzent/stan.json` for its PRs and fixes the remarks at once. It does not finish until its PR chain is merged or is
  waiting only for a dated decision. PR chains stall most often because the author does not see the verdict.
- Numbered resources (e.g. database migrations) come from an allocation, or you check the main branch and open PRs
  before choosing a number – two PRs with the same number only conflict after merging.

## Schedule change
In one commit: a row in `koordynacja-odstepy.tsv` (all columns, `od` = the moment of change), an entry in
`flota/zadania.yaml` (cron), `python3 flota/generuj.py`, tests: `python3 -m unittest discover flota` and
`scripts/test-przebieg.sh`. Changing scheduled tasks – according to `uprawnienia.yaml`.
