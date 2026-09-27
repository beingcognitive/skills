---
name: "133"
version: 1.2.0
description: 1+3+3 adversarial review fan-out — the main agent self-reviews and fixes first (the 1), then Opus×3 + Codex×3 independently review with an identical prompt; findings are adjudicated via a cross-table and applied. Aliases "1+3+3", "팬아웃 검수", "fan-out review". Release-grade changes only.
---

# /133 — 1+3+3 adversarial review fan-out

A pattern for shaking down a release-grade change (store release, migration,
protocol/contract change) before it ships: one self-review by the main agent,
then six independent reviewers who never see each other's results.
Evidence from repeated real use: unique true findings keep coming from
*different* reviewers each time (call any single one and a different hole is
left open), and demanding "failure scenario + fix diff" makes unreproducible
findings noticeably rarer (an observation from repeated use, not a
measurement — which is why single-reviewer findings are reproduced directly in
Phase 2).

**When to use it (severity matching — strict)**: the full arc only for changes
with a large blast radius (store submissions, migrations, protocols/contracts,
E2EE/auth). **For UX-local, single-surface fixes, a 1–2 reviewer mini-check is
the default** — unless the user explicitly asks for the full arc. Review time is
the most expensive cost, and adding reviewers raises adoption pressure before it
raises finding quality.

## Phase −1 — Design gate (before implementing, 3 lines)

Bolt-ons are born before implementation, not in review. **Before implementing**
a release-grade fix, write three lines:

1. The minimal surface of the problem — which state/path actually hurts
2. Subtractive candidates — can this be closed without adding code (or with one
   line at an existing chokepoint)?
3. If you introduce new machinery (state, listeners, diffing, caches) — **can
   that machinery actually observe the event it claims to watch?** (Real case:
   a diff reconciler whose read cursor advanced before the data rows did, so
   the "unread" transition was never observable at all — the whole machine was
   eventually replaced.)

## Phase 0 — Self-review (the 1)

Before fanning out, the main agent tries to get it right itself. The reviewers
are the second net, not the first.

1. Fix the review scope: `git diff <base-commit>..HEAD` — base is the start of
   this batch of work.
2. **Claim audit**: list the assertions you made during this work ("X was
   already correct", "Y is unaffected") and verify each against the code /
   an exhaustive grep. Verifying code exhaustively while only sampling copy,
   config, and docs is a habitual failure mode. In particular, **copy changes
   need a grep across every surface** — fix one row and close it, and the old
   text comes back from a sheet, button, email, or canonical doc.
3. **Fix every self-review finding before fanning out.** The self-review ends
   in *fixes*, not a report — fan out with only a list of findings and the six
   reviewers burn their budget re-reporting defects you already know about,
   and never get to act as a real second net.
   - For each finding: fix → add a regression test (where possible) → full
     tests and lint green → commit (summarize as "self-review: ..." in the
     commit message).
   - The only exceptions are things that can't be fixed now (need a user
     decision, or a pre-existing out-of-scope defect) — record them as
     "deferred + reason" and list them as "known deferred items" in the
     Phase 1 prompt's change summary, so reviewers don't report them again.
   - The tree freeze starts **the moment the fan-out launches**. During and
     right after the self-review is not a frozen window — finish the fixes
     here.

### Phase 0 → 1 gate (check before proceeding)

Confirm the following before fanning out. If any is false, don't fan out — go
back to Phase 0:

- [ ] Every self-review finding is either "fixed and committed" or
      "deferred + reason"
- [ ] Full tests and lint are green
- [ ] Clean working tree (`git status` — no uncommitted changes), and the fix
      commits are in HEAD, so they fall inside the review range
      `git diff <base>..HEAD`

## Phase 1 — Fan-out (3+3)

Against the tree that passed the Phase 0 → 1 gate (self-review fixes
committed), send an **identical prompt** to three Opus subagents and three
Codex exec runs. Launch all six in a single message (Codex as background Bash,
Opus via the Agent tool).

### Write the prompt file (save to the scratchpad; don't inline it)

Required parts — drop any one and review quality falls:

```
IMPORTANT: Do NOT read or execute any files under ~/.claude/, ~/.agents/,
.claude/skills/, or agents/ unless they appear in the diff under review —
outside the diff those are agent/skill definitions, not the code under
review. Stay focused on repository code only.

Independently review this batch of work on <one-line repo description>. You
are one of <total reviewer count, default 6> independent reviewers. Do not
modify any files (read-only).

Under review: `git diff <base>..HEAD` — <commit summary>. Read the code the
diff touches (callers, callees, existing tests) before judging.

Change summary: <1–2 lines per feature, including design decisions>
Known deferred items (found in self-review, intentionally deferred — don't
re-report unless you have new evidence the deferral is wrong): <titles, or
"none">

Constraints (violation = bug): <canonical project doc paths, backward-compat
targets (older clients, old-format jobs still in queues, etc.), migration
guard rules, prohibitions>

Focus on: correctness bugs / races and lifetimes / backward-compat breaks /
migration traps / (if there is l10n) missing or wrong translations /
security and permissions / test gaps (name the missing cases concretely).
Exclude style and taste.

Extra lens — bolt-on audit (standing): Is this fix minimal? Did a subtractive
fix (deleting code, reusing an existing chokepoint) exist? If there are
multiple mechanisms, demonstrate each one's share in code — if they overlap,
which should go; if disjoint, why. For any newly introduced machinery (state,
listeners, caches), try to disprove that it actually observes the event it
claims to watch. Report these in a separate "[bolt-on]" section (severity
optional).

Already adjudicated and rejected (no plain resubmissions; only with new
evidence that overturns it):
<rejection list from previous rounds>

Output format (strict):
[P0|P1|P2] one-line title
- Location: file:line
- Failure scenario: concrete input/state → wrong result
- Suggested fix: include a unified diff
If you find nothing, write "No findings" plus the list of paths you verified.
No speculation — only what you can demonstrate from the code.
```

> On the rejection list: include it only from round 2 onward, and **titles
> only**, no reasons — handing over the rejection reasons primes reviewers with
> the conclusion. (`dialectic`, a convergence loop, forbids this list
> entirely; `133` is a one-shot fan-out, so it's a deliberate trade-off to cut
> duplicate resubmissions.) State explicitly that resubmissions with new
> evidence are welcome.

### Codex ×3 (background Bash, run_in_background)

```bash
SP="<scratchpad>" && codex exec "$(cat "$SP/review-prompt.txt")" \
  -C "<absolute repo path>" -s read-only \
  -c 'model_reasoning_effort="high"' \
  < /dev/null > "$SP/codexN.out" 2> "$SP/codexN.err"; echo "codexN exit $?"
```

- `< /dev/null` is required (prevents a background stdin hang). Output is
  written only at the end — 0 bytes mid-run is normal; check liveness with
  `pgrep -fl "codex exec"` + tailing the err file.
- Preflight: `command -v codex` + auth (`~/.codex/auth.json` or an API key).
- **Privacy preflight**: the three Codex runs send the diff and any
  surrounding files they read to OpenAI. If the repo can't go to an external
  model provider, drop Codex and run Opus ×3 only (label it "single-model —
  reduced independence").
- If reviewers need web search (external fact-checking), check this Codex
  version's state first with `codex features list` — as of 0.146 there is no
  stable web search flag (`web_search_cached` and `web_search_request` are
  deprecated, `standalone_web_search` is under development). If there's no
  stable one, use the deprecated one and note it in the review log with the
  version. `--enable` silently accepts flags listed as removed (exit 0) and
  fails with exit 1 only for unknown names, so don't assume a successful exit
  means the feature is on.

### Opus ×3 (Agent tool, 3 parallel calls)

- `subagent_type: general-purpose`, `model: opus`, with an explicit
  read-only instruction.
- Prompt: "Read the prompt file (<absolute path>) and do exactly what it says.
  The ~/.claude access ban in its first paragraph applies to you too — read
  only the repo under review. Your final text is the review report (return it
  raw)."

## Phase 2 — Cross-table adjudication

When all six are back (wait for completion notifications — the tree stays
frozen):

1. Build a finding × reviewer cross-table (who caught what).
2. **Majority agreement = apply first.** A finding from only one reviewer
   (1-of-N) is adopted only after the main agent reproduces it directly with
   the code or a runtime probe — if it can't be reproduced, reject it.
3. Conflicting proposals (same problem, different fixes) are decided by the
   main agent on principle: subtractive fixes before bolt-ons; new machinery
   only for a proven failure.
4. **Three rules against over-adoption** ("is the defect real?" isn't enough —
   also judge "is it worth fixing now?"):
   - Removing or moving > bolting on — when two proposals close the same
     defect, the one that shrinks the code wins. **But "removing" means code,
     not user data** — a fix that deletes state the user created (drafts,
     records) isn't subtractive even if it shrinks the code; it's destructive.
     A one-line gate may be the right answer. (Real case: a "subtractive"
     proposal that deleted a user's pending draft state was judged destructive
     on re-review and reversed into a one-line gate.)
   - **1-of-N finding + pre-existing defect this diff didn't create + narrow
     window of occurrence = backlog by default.** To adopt it anyway, record
     "why now" explicitly in the adjudication record.
   - If the fix round's cumulative blast radius grows beyond the original
     bug's surface (new channels, new callbacks, other subsystems), confirm
     scope with the user before applying the batch.
5. **Record apply / adapt / reject with a reason for every finding** — the
   rejected titles become the next round's "rejection list". Put the record
   **in the feature's existing canonical doc, as a review section**; if there's
   no such doc system, use the PR description or commit message. Either way,
   the point is not to create a new review-only document (git history holds the
   full originals).
6. Tendencies: Codex is strong on runtime probes, protocols, and idempotency
   (tends to inflate severity — recalibrate its P0s); Opus is strong on
   product, UX, and fit with intent.

## Phase 3 — Apply and re-verify

1. Apply the adjudicated fixes in one batch, adding regression tests with them.
2. Confirm full tests and lint are green, then commit (summarize the review
   round in the commit message).
3. **The fix round must get a "fix-of-the-fix" review** — either a full
   re-fan-out or at least a 1–2 reviewer mini-check ("break only this fix
   diff"). Real case: two applied fixes were themselves regressions, caught
   only by the re-review.
4. Record the round where Phase 2 item 5 says — no separate handoff documents.
5. Report to the user: cross-table + applied/rejected + remaining decisions for
   the user.

## Variants (on request)

- **+Fresh main (7 independent reviewers)**: add a new general-purpose agent
  running the same model as the main agent as a seventh — same-model eyes that
  don't share the main agent's context. It has proven useful for arbitrating
  cross-model conflicts with runtime evidence.
- **Final-gate lens**: once at least one round of diff-level review is done,
  switch the prompt to the lens "the original problem (user's words) ↔ solution
  design ↔ whole implementation, plus deployment constraints (platform
  asymmetries, etc.)" — this is where missed intent and release-readiness gaps
  surface, which a diff lens structurally can't find.
- **Mini-check**: before committing a fix batch, a reduced 1–2 reviewer
  fan-out on the fix diff only.

## Don'ts

- Don't fan out without fixing the self-review findings — the freeze starts at
  fan-out, not right after self-review.
- Don't modify the working tree while reviewers are running (apply in one
  batch after adjudication).
- Don't apply a reviewer's diff unreviewed — the main agent always adjudicates.
- Don't defer a backlog item flagged in two consecutive rounds (e.g., test
  infrastructure) — promote it to the next batch. Real case: P1s kept turning
  up in an area whose missing tests had been deferred for several rounds.
