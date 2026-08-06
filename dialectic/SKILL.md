---
name: dialectic
description: |
  Harden a plan/design/algorithm/decision via a thesis→antithesis→synthesis loop: each round, two
  independent reviewers (a different model + a freshly-spawned subagent) critique the current draft;
  fold in only findings the draft doesn't already resolve, adjudicating reviewer disagreements by
  evidence; stop after two consecutive clean rounds (cap 10, usually 3–5). Independence is the engine —
  a different model and an isolated perspective expose blind spots one model can't see in itself.
  Use when the user says "dialectic" or "정반합". NOT for trivial or no-real-tension changes.
---

# /dialectic — convergence loop

Harden a plan / design / algorithm / decision until independent review stops finding things. A single
model can't see its own blind spots; a different model and a fresh, uncontaminated perspective can.
That independence — and the disagreement it produces — is the engine.

It **hardens reasoning; it does not prove correctness.** For empirical / code / financial / security /
performance / UX claims, dialectic agreement is not evidence — carry through to real verification (run
it, test it, benchmark it, check the source). **Don't use** for trivial or no-real-tension changes.

## Step 0 — Seed the thesis
The loop *hardens* a draft; it doesn't author one from nothing. If the input isn't already a structured
draft, write **v0** yourself: goal · constraints · key assumptions · the approach · alternatives
considered · known risks · how it will be verified. If a **missing hard requirement could change the
plan, pause for the user's answer** before starting rounds; otherwise state your assumption and proceed.
Show v0 — that's round 1's thesis.

## Right-size
**Always two independent reviewers per round** — that pair is the method's load-bearing minimum (round 1
adds two *blind* co-designers on top; see "Each round"). What
you right-size is the *expected round count* and whether to add a **third** domain/human reviewer, not
the core two:
- Small or already-mature artifact → often hits the two-clean-round minimum in ~2–3 rounds.
- High-stakes → add a domain/human reviewer; may run to the cap.
- So trivial a single gut-check would do → below this skill's bar; don't run the loop.

State the expected round count to the user up front.

## Each round
1. **Send the complete current draft** (+ an optional neutral *Settled constraints / non-goals* block)
   to each reviewer — **in parallel, in one message, each blind to the other's output**. Nothing else:
   no prior rounds, ledgers, verdicts, or your rationale. ("Parallel" = the two dispatches; any
   verify-a-fact step happens later, in synthesis, after both return.)
2. **Reviewer A = model-independent:** a different model/provider when available (e.g. `/codex`). If none
   is available — or it's your own model family — use the most independent reviewer you have, label the
   round *"reduced independence/confidence,"* and don't call it model-independent.
   **Reviewer B = perspective-independent:** spawn a **new subagent** (Agent tool); pass it **only** the
   complete draft + the reviewer prompt; do **not** pass conversation history, prior rounds, ledgers, or
   your rationale (same base model as you, but isolated context and a different review *axis*).
   Give the two **different review axes, not different tones** — e.g. A = adversarial correctness;
   B = simplicity / over-engineering, or a risk axis the artifact needs (security / data-loss / ops / cost).
   Match each axis to the reviewer's **comparative advantage** — in particular, route *checkable/factual*
   claims to the web-search-capable reviewer (codex), and *judgment/taste/argument* axes to the other.

   Reviewer prompt (stable, neutral):
   > Critique this draft for correctness, missing constraints, hidden assumptions, edge cases,
   > over-engineering, under-specification, implementation risk, and verification gaps. Each finding must
   > cite a specific draft element (or a specific missing one) and its failure mechanism — no generic
   > checklist worries. Severity-grouped; for each: observation · why it matters · concrete fix. Don't
   > rewrite the draft. If it's solid, say so.
3. **Synthesize** into the per-round ledger (below); the revised draft becomes next round's thesis.

### Round 1 only — blind co-design pass
Critics handed v0 reason *inside* its framing, so they structurally can't catch the deepest blind spot — a
wrong *approach*. To reach it, **in round 1 only** add two **blind** dispatches alongside the two critics
(same independence pairing: one model-independent, one fresh subagent), handed **only the problem statement +
settled constraints, never v0**: *"Propose your own approach; assume no solution exists yet."* All four run in
parallel, each blind to the others. (R1 ⇒ four dispatches; from R2 on, revert to the standard two reviewers.)
- **Metabolize each blind solution as a *divergence finding*, not a thing to merge:** "an independent reviewer,
  blind to v0, chose X where v0 chose Y." Feed that divergence through the normal synthesis (adjudicate by
  evidence, don't average). **Never merge whole alternative designs wholesale** — that's exactly where the
  orchestrator's bias to protect its own v0 wins, the loop's weakest link. A divergence that exposes v0 as
  wrong-framed is just an accepted finding that revises the draft heavily; no special restart or recount.
- **Convergence of the blind solutions toward v0 does NOT by itself make the round clean or reset the
  clean-round counter** (same logic as a re-run, below). "They independently agreed with me" is real comfort
  but a confirmation-bias exit if it ends the loop.

### Per-round ledger (fill literally)
```
Round N
finding | source(A/B, or A-blind/B-blind in R1) | verdict(accept/reject) | one-line reason
...
Disagreements (only if any): claim A | claim B | deciding evidence/mechanism | resolution | confidence | needs-empirical?
Accepted this round: <n>   (the round is CLEAN iff 0)
Self-bias: <one way this synthesis could be wrong; if plausible, re-open the last reject>
```

### Synthesis rules
- **Accept** a finding only if the draft doesn't already *adequately resolve* it — adequate = the draft
  states the decision + rationale + constraints + a verification/mitigation path at an actionable level.
  Merely naming the topic ≠ resolved. Also accept: unresolved, materially under-specified, or contradicted.
- **Adjudicate disagreements; don't average.** A divergence is exactly where a lone model would adopt a
  wrong premise. If it turns on a fact you can verify now, **pause and verify before revising**; if not,
  record it as an unresolved risk, mark confidence, and make the draft conditional on the assumption.
- **Discard** non-actionable / out-of-scope / mechanism-less / already-resolved findings.
- **Re-run a reviewer at most once, and only for a format/process failure** (e.g. no severity grouping,
  empty, ignored the prompt) — never because findings are sparse, negative, or inconvenient (that's
  selection bias); log any re-run. A re-run's findings count normally toward acceptance, and a re-run
  never by itself **resets** the clean-round counter; a round whose only findings came from a re-run still
  counts as CLEAN if synthesis accepts none.

## Stop
- A round is **CLEAN** when synthesis accepts **no** finding (the draft needs no change). Findings discarded
  as already-resolved / cosmetic / out-of-scope / non-actionable do **not** count and do **not** reset the
  counter. **Stop after two consecutive clean rounds.**
- A clean round — **including round 1** — still triggers one more round with freshly-spawned reviewers on
  the unchanged draft (independence is what makes the confirmation meaningful). If the artifact had no real
  tension, note it was likely too trivial for this method.
- Hard cap = **10 rounds.** At **any** stop (clean-rounds or cap), the **exit checklist** must pass to
  declare convergence; if it doesn't (including hitting the cap), report the unresolved items to the user
  and stop **without** declaring convergence.
- **Exit checklist:** objectives met · assumptions explicit · risks handled · alternatives + why-rejected
  recorded · unresolved disagreements documented · verification plan named.
- **High-stakes:** the domain/human reviewer reviews at least the initial and the final pre-stop draft
  (and any round their concern is materially revised); their unresolved objections block convergence.
- **Mode-shift = the real signal:** early rounds change the draft (defects, design); near convergence,
  feedback turns to traceability/tests/verification. When critique becomes verification, you're there.
- On stop, deliver the **Output** (below). Proceed to implementation + verification **only if the user
  asked for it**; otherwise stop at the hardened draft.

## Guardrails (skip these and the result is fake — single source; don't restate them inline)
1. **Don't prime the reviewers.** Keep expected verdicts, "already-rejected" lists, "say READY if nothing's
   new," and round numbers OUT of the reviewer prompt. Priming hands them the answer and suppresses real
   findings — "convergence" becomes compliance. Neutral *context* (settled constraints, non-goals) is fine;
   a prescribed *conclusion* is not. Tell your prediction to the **user only**.
2. **Don't stop on a feeling.** Stop only by the rule, the stated budget cap, or a user-approved scope —
   never because the draft "feels done" (the classic confirmation-bias tell).
3. **Judge novelty against the draft's content, not the reviewer's wording.**
4. **Independence is earned, not assumed.** If both reviewers are effectively the same model, agreement
   proves little; diversify model / axis / evidence. **Verify** reviewer B's isolation — some harnesses
   spawn a subagent that inherits the whole conversation regardless of what you nominally pass it (observed
   with Claude Code's Agent tool: a "fresh" reviewer cited an unrelated detail from earlier in the session).
   Probe for it. If B can't be isolated, it is *fresh reasoning + a different axis*, not context-independent —
   your real independence then rests on the different-model reviewer (A).
5. **Privacy preflight.** Reviewers (especially `/codex` → OpenAI) see whatever you send. Redact secrets,
   credentials, PII, and proprietary context **from the draft itself**; scoping file reads isn't enough.
   "Complete draft" means complete w.r.t. the *sanitized* draft — replace decision-relevant redactions with
   abstract placeholders that preserve the constraint (e.g. `[regulated customer data]`, `[vendor contract limit]`).

## Output
`Final draft` · `Accepted findings by round` · `Rejected findings + reasons` · `Decision records` ·
`Unresolved risks` · `Verification plan` · `Implementation status`.

## Notes (read once — rationale, not steps)
- **Why two independent reviewers:** a single model can't see its own blind spots; a different model (A)
  catches different *classes* of error, and an isolated fresh perspective (B) catches what shared context
  would have suppressed. Their **disagreement is the highest-signal output** — it's exactly where a lone
  reviewer would have adopted a wrong premise.
- The orchestrator writes the draft *and* judges the critiques — the loop's weakest link. That's why the
  **rejection ledger + self-bias line are mandatory**: they're your only audit trail against your own bias.

## Appendix — codex (environment-specific; not part of the method)
- Use a fresh `codex exec` — its `resume` rejects the `-C` working-dir flag, so start fresh each round.
- Treat codex as GPT unless the user says otherwise.
- stderr capture: a fixed path like `/tmp/codex-err-$$.txt` (`$$` = shell PID; ensure it runs in a shell;
  BSD `mktemp` fails on a template with a suffix after the `X`s).
