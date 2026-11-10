# Security – shared files

- `audyt/RRRR-MM.jsonl` – a record of EVERY permissions-hook decision (pre: allow/deny, post: executed) with a SHA-256 chain (`prev_hash` → `hash`); retention ≥ 6 months (AI Act art. 26 – an obligation for deployers of high-risk systems; good practice for others), `merge=union`.
- `blokady.jsonl` – denials only (shortcut for the weekly review: the CEO uses it to decide on loosening `uprawnienia.yaml`).
- `HALT` – kill switch: while the file exists, the hook stops every agent session at the first tool call other than a read. Created by the watchdog (fleet budget ≥ 100 %) or by a human; removed by the operator after review.
