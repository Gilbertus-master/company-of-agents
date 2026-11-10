# ADR 0002 · Deterministic gates in a hook instead of the CEO agent's judgement · status: accepted · 2026-11-09

## Context
Template v0.1 treated the CEO agent's approval ("[Decyzja CEO]") as the gate for money, merges and deployments. Project Vend 2 (Anthropic) showed
that a supervising agent on the same model "shares the shortcomings and blind spots" of its subordinate and approved concessions 8× more often than it rejected them; the literature
on self-preference bias in judges says the same. In the source organisation speed tempted people into Bypass mode, and policies lived in prompts and chat decisions.
A Claude Code hook can block a tool before it runs and record every decision.

## Decision
Policies and thresholds are **in data** (`uprawnienia.yaml`, `polityki/progi.yaml`, `polityki/*.md`), enforcement is **in the PreToolUse hook**
(`.claude/hooks/uprawnienia.py`, fail-closed, `matcher: "*"`, `onFailure: block`): gate `kwota` (refunds, spending up to the threshold), gate `pr`
(reviewer verdict OK for the head SHA + green CI), the rule of two (type `reader`/`researcher` from the manifest does not send, pay or deploy),
owner approval only as a `DECYZJA=D-…` that exists in `decyzje/`, kill switch `bezpieczenstwo/HALT`, audit of every decision with a SHA chain
(≥ 6 months). The CEO agent decides on priorities, delegation and ordering – **not on individual amounts or merges**.

## Consequences
- The agent cannot be "talked round": manipulation in an e-mail can change its intent, but not the threshold in the file or the hook's decision.
- The hook knows only command and tool patterns – the rule applies more widely than its enforcement (rules A1); new sending/payment tools
  need a pattern in the hook and a test (`test_uprawnienia.py`).
- Every permissions change is an owner decision with an ID; loosening happens on the basis of history (monthly review – `procesy/50`), not trust.
- Cost: a slower start (everything cautious), a dozen or so seconds per run for the hook, the audit grows (`merge=union`, archiving).
- A reviewer separated from the authors (different model, fresh context, rubric) is part of the same decision – its verdict is data for the hook.

## Source
Research "Standardy 2026": sections "Rada–CEO–liderzy" and "Bezpieczeństwo", recommendations 1 (fail-closed hook, audit, HALT), 2 (money gates outside the
CEO agent), 3 (separated reviewer), conclusion; `docs/raport/2026-11-09-raport.md` section 0 (Project Vend lessons), 3.2, 4.2 items 4 and 7.
