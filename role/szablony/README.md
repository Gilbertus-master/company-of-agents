# Starter role registers

`role/szablony/<rola>/**` is copied by `scripts/nowy-agent.sh <rola>` into the agent's directory (recursively, **without overwriting** existing
files; markers `{{rola}}` and `{{org.nazwa}}` are substituted). The agent starts work with ready registers instead of inventing a format
in its first run – and the watchdog, the PM and the run contract (`przeczytane` / `w_rejestrze`) have fixed files for numeric reconciliation.

Each register has a header: **purpose**, **columns**, the **"no personal data"** rule (thread / review / case ID instead of people) and **who
updates it and when**. Format and content generalised from the source fleet's registers (November 2026).

| Role | Files | What for |
|---|---|---|
| aplikacje | `opinie.md`, `wydania.md` | testers' feedback by store ID; versions, builds, gates, rollback |
| architekt | `adr/README.md`, `tematy/README.md` | the product's architecture decisions; topic progress between sessions |
| bezpieczenstwo | `alerty.md`, `dostepy.md` | alerts and vulnerabilities by ID; quarterly review of identities and keys |
| compliance | `rejestr-wymogow.md`, `kalendarz.md`, `luki.md`, `bramka-wydania.md` | requirements per country (W-…), effective dates, gaps (C-…), gate G1–G12 |
| inzynier | `README.md` | branch convention, worktree, PR, claiming work |
| lokalizacja | `kraje.md`, `rejestr.md` | countries → languages → store listings; language gaps (L-…) |
| ochrona-danych | `rejestr-czynnosci.md`, `dostawcy-dpa.md`, `zadania-osob.md`, `naruszenia.md`, `dpia-szablon.md` | GDPR Art. 30, Art. 28, Art. 15–22, Art. 33–34, Art. 35 |
| prawny | `umowy.md`, `terminy.md`, `wzory/README.md` | contract register (U-…), action deadlines, templates with version and decision |
| recepcja | `rejestr-poczty.md` | incoming mail from general addresses by thread ID, triage time |
| tester | `lista-reczna.md`, `bledy.md` | what automation does not see; bugs (T-…) and escapes |
| wywiad | `rejestr.md` | sources (S-…), competitors (K-…), findings (WR-…) with EV |

The reviewer gets the rubric from `role/_rubryka-recenzenta.md` (a separate path in the script). A new register for a role: a file here + a row
in this table + a **Rejestry startowe** (starter registers) line in `role/<rola>.md`; the test `scripts/test-nowy-agent.sh` checks that every role with templates exists
in `role/` and that the copy succeeds.
