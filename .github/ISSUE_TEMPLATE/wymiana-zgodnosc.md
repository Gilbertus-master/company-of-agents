---
name: Exchange – skeleton conformance (tool only)
about: Process 16 records are created only by scripts/zgodnosc.py + scripts/wymiana.py – an issue written by hand will be "poza schematem" (off-schema)
title: "[zgodnosc] "
labels: ["wymiana", "typ:zgodnosc"]
---

<!-- Do not fill in by hand. In the project repo:
  python3 -I scripts/zgodnosc.py                                   # table: identical / changed / missing, areas
  python3 -I scripts/zgodnosc.py --rekord <projekt> --plan "<what next, one sentence>" --wyjscie zgodnosc.json
  python3 -I scripts/wymiana.py wyslij zgodnosc.json --zakazane "<names of companies and clients>"
The record contains only counts and skeleton paths shared with the template – no file contents and no paths that exist only in the project.
Description: procesy/16-wymiana-ceo.md, section "Zbieżność struktury" (structural convergence) -->
