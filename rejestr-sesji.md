# Rejestr sesji

Session register. Every session (chat, run, development session, subagent for longer than one turn) has a parent and reports to it.
The chain ends at the CEO. Whoever starts a session records it; the parent closes it after checking the result at the source. Newest at the bottom.
Before starting, check that nobody is doing the same thing. Review: the PM in every run (`python3 -I pm/narzedzia/sesje.py`:
open > 24 h, no parent, unknown parent, duplicate titles, closed without a result) reports to the CEO; the CEO closes or assigns.

Columns (headers stay Polish – `sesje.py` reads them): **Data** – `RRRR-MM-DD GG:MM` of the start (without a time the tool counts age from the end of
the day); **Id** – `local_…` / task id / `sesja:<nazwa>`; **Rodzic** (parent) – an agent (`ceo`, `architekt (czat A)`) or a session Id from this register;
**Cel / zrobione gdy** (goal / done when) – a verifiable criterion; **Status**: `otwarta` (open) · `zamknięta` (closed; Wynik = link to the result) ·
`porzucona` (abandoned; reason, `przejmuje: <kto>` = taken over by); **Wynik** (result) – a path in the repo or a URL. Replace the `|` character in content with "/"
(the table has exactly 7 columns).

| Data | Id | Tytuł | Rodzic | Cel / zrobione gdy | Status | Wynik |
|---|---|---|---|---|---|---|
