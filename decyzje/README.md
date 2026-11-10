# Decision register (directory)

One decision = one file `D-RRRRMMDD-NN.md` with frontmatter (`id`, `status`, `data`, `kto`, `scope`, `supersedes`,
`wazna_do`, `model_context`, `dotyczy`) and a body (owner's words, decision, effect). `decyzje.md` in the root is a **generated
index** (`python3 -I scripts/decyzje-indeks.py`, ≤ 200 lines) – the only layer injected into agents; do not edit by hand.
The permissions hook checks `DECYZJA=D-…` in this directory (status `accepted`).
Consistency check (`scripts/decyzje-indeks.py --sprawdz`, in CI): two `accepted` decisions with the same `scope` and no `supersedes`
= contradiction; `supersedes` pointing at a non-existent decision = error; `wazna_do` in the past = `expired`.
Archive: `superseded`/`expired` older than 90 days are moved by `--archiwizuj` to `decyzje/archiwum/RRRR/`.
