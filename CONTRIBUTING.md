# Contributing

Thank you for improving the core. Two ways to contribute:

1. **Automatically, from a running fleet** – if the company's owner has enabled sharing (see "Sharing improvements" in the
   README; the owner chooses in `scripts/init-org.sh --dziel-sie tak|nie`). Then every Monday the architect agent runs `scripts/propozycje.py` and files an Issue here with
   mechanism changes and proven rules. The proposal is a Contribution within the meaning of section 5 of the Apache-2.0
   licence and carries a statement of the owner's consent and of compliance with the Developer Certificate of Origin 1.1.
2. **By hand.** Open an Issue (template "Improvement proposal") or a pull request.

## Rules
- **Licence.** Contributions are accepted under Apache-2.0, the licence of this repository (section 5 of the licence), with
  no additional terms.
- **DCO, not a CLA.** Sign off every commit (`git commit -s`). The sign-off certifies the Developer Certificate of Origin 1.1
  (https://developercertificate.org): you wrote the change or have the right to submit it under the project licence.
- **No company data.** No names, e-mail addresses, tax or bank numbers, customer data, secrets or internal paths – in code,
  tests, issues or logs. The gate in `scripts/propozycje.py sprawdz` rejects them; reviewers do too.
- **Tests.** `bash scripts/testy.sh` must be green; a new mechanism comes with a test row in the `TESTY` array.
- **Scope.** The core is mechanisms: permissions hook, audit, HALT, watchdog, merge queue, run contract, cost measurement,
  evals, proposals. Roles and processes enter the core only after the maintainers generalise them (a fleet proposal may
  contain them); a specific company's policies and data do not belong here.
- **Language.** Code, identifiers, file names and script output are in Polish in both editions (`GLOSSARY.md`). Prose is
  maintained in Polish ([Gilbertus-master/firma-z-agentow](https://github.com/Gilbertus-master/firma-z-agentow)) and translated here; a change to a document is best
  proposed against the Polish edition, or here with a note that the Polish source needs the same change. Issues and pull
  requests may be in English or Polish.
