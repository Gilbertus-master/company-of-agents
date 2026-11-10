# Business policies

Hard rules the agent **does not negotiate** (zasady.md item 11). Lesson from Project Vend (Anthropic): an agent
talking to people can be talked into discounts, refunds and exceptions if the rules exist only in the prompt.

Every policy has a version, a date and the decision `D-…` that approved it. Change = a decision according to `uprawnienia.yaml`
(prices → owner).

| File | What | Who applies it |
|---|---|---|
| `cennik.md` | prices, plans, currencies, periods | sales, support, marketing |
| `zwroty.md` | when to refund, how much, up to the CEO's threshold; above it – owner | support (request), CEO (decision) |
| `rabaty.md` | which discounts exist; anything not on the list does not exist | sales |
| `komunikacja.md` | tone, signature, AI footer, what we do not promise | everyone writing externally |
| `outbound.md`, `blacklista.tsv` | outbound contact: consents (Art. 398 PKE), channels, limits | sales, marketing |
| `progi.yaml` | numeric thresholds read by the hook (refunds, expenses, outbound) | permissions hook |

Policy template:
```
# <nazwa> · version 0.1 · RRRR-MM-DD · approved: D-…
## Rules
1. …
## Exceptions
None. A request for an exception → an order to the CEO with a recommendation; the customer gets the reply "we will check and reply by <termin>".
```
