# Agent instruction changes – CEO register

Every operational change to `<agent>/agent.md`, `role/*.md`, `README.md`, `flota/szablony/` made by the CEO (max. 2 per week, `/przeglad-tygodnia` step 10) has a row here.
Safety rules (`zasady.md` A), `uprawnienia.yaml`, agent scopes and annual goals – **never** changed this way (only via a proposal to the owner). A PR with an instruction change is merged by the reviewer
or a human, never by the author (Doer–Verifier); after the merge `python3 flota/generuj.py` runs and `prompt_version` changes automatically in the contract lines (regressions are tied to the instruction version).
Knowledge source = a lesson from `ceo/wiedza/` or an AAR; no source, no change.

| Date | File | What | Why / knowledge source | PR / reviewer | Effect checked (date, metric) |
|---|---|---|---|---|---|
