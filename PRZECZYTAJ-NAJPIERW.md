# Read first – starting a session in the agentic organisation

For Claude starting work in this organisation without conversation history (new account, new session, Routine).
No secrets – keys, passwords and tokens are not and must not be written to files.

## Who and what
- Company, people, mandate and runtime data: `org.yaml`.
- **Owner** – mandate, budget, boundaries; decisions via `ceo/do-decyzji.md` → `decyzje.md`.
- **CEO agent** – runs the organisation; all agents report to it.
- **Operator** – the "human interface": reports to the CEO, does only what an agent cannot (`operator/`).
- You work in English (this edition), in plain language, briefly.

## Reading order
1. `zasady.md` – always in force; `uprawnienia.yaml` – what is allowed without asking.
2. `README.md` – map of the repo and the agents.
3. `decyzje.md` (index; files in `decyzje/`), `ceo/do-decyzji.md`, `operator/kolejka.md`, `bezpieczenstwo/HALT` (if it exists – the fleet is stopped).
4. Your orders (Issues `agent:<ty>` or `zlecenia/<ty>.md`) and `<ty>/agent.md`.

## When you are a chat with a human
- First set the session role for the permissions hook: `echo ceo > .agent-rola` (or another role; the file is in `.gitignore`).
  Without it the hook blocks everything except `sam` actions.
- You talk to the owner or the operator – you record decisions as a file `decyzje/D-RRRRMMDD-NN.md` (frontmatter + a verbatim
  quote) and refresh the index `python3 -I scripts/decyzje-indeks.py`. An owner-level action is run with that ID: `DECYZJA=D-… <polecenie>`.
- You order execution work from agents or sessions (entry in `rejestr-sesji.md`); you do not do it in the chat.
