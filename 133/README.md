# /133 — the 1+3+3 adversarial review fan-out

*Your own pass first, then six independent reviewers, before a release-grade change ships.*

**English summary.** `133` is a pre-release review pattern for Claude Code: the main agent
self-reviews first and fixes what it finds (the "1"), then fans the identical prompt out to
**three Claude Opus subagents and three OpenAI Codex CLI runs** (the "3+3") — six reviewers
across two model families, each blind to the others. Their reports are merged into a finding × reviewer cross-table; the main
agent adjudicates every finding (apply / adapt / reject, with reasons), applies the verdicts in
one batch, and — crucially — sends the *fixes* back for another (smaller) round of review.

Two empirical observations drove the design, across repeated real release reviews:

- **Unique true findings are scattered across reviewers.** In every round, some real bugs were
  caught by exactly one of the six — and *which* one varied. Any single reviewer, however strong,
  leaves a different hole. Cross-model diversity matters too: in practice Codex was consistently
  stronger on runtime/protocol/idempotency issues (with a severity-inflation habit), Opus on
  product intent and UX coherence.
- **Demanding evidence suppresses invented findings.** Once the prompt required a concrete
  failure scenario *and* a unified diff for every finding, unreproducible findings became rare —
  an impression from repeated use, not a measurement, which is why a finding reported by only
  one reviewer still gets reproduced directly before adoption (the 1-of-N rule below).

The costliest failure mode isn't missed bugs — it's **over-adoption**: six authoritative reports
create pressure to apply everything. Hence the adjudication rules (majority-agreement first;
1-of-N findings — caught by exactly one reviewer, whatever N you ran — need direct
reproduction; subtractive fixes beat additive ones; pre-existing
narrow-window defects default to the backlog) and the hard rule that **fixes get re-reviewed** —
in one real round, two of the applied fixes were themselves regressions, caught only by the
re-review.

Use it only for changes with real blast radius (store submissions, migrations,
protocol/contract changes, auth/E2EE). For localized single-surface fixes, a 1–2 reviewer
mini-check is the default — the full arc is deliberately expensive.

The skill body below is in Korean — it's the version I actually run.

## Prerequisites

- **Claude Code** with the Agent tool (for the Opus ×3 subagents).
- **[OpenAI Codex CLI](https://github.com/openai/codex)** installed and authenticated
  (`~/.codex/auth.json` or an API key) — this is what buys cross-model independence.
- A git repo: reviewers read a committed tree via `git diff <base>..HEAD`.

**Privacy.** The Codex half of the fan-out sends your diff — and whatever surrounding files the
reviewers read — to OpenAI. Don't run `/133` on a repo you can't share with a third-party model
provider; use the Opus-only reduced variant instead (labeled "single-model — reduced
independence").

**Reduced variants** if you lack a piece: without Codex, run Opus ×3 only (label the result
"single-model — reduced independence"); for small changes, run the 소검증 variant (1–2 reviewers
on the fix diff only). The self-review phase and the adjudication rules apply unchanged.

## Relation to [`dialectic`](../dialectic/)

Same engine — independent, unprimed reviewers — different shape. `dialectic` is a *convergence
loop* (two independent reviewers per round — four dispatches in round 1, which adds two blind
co-designers — until two consecutive clean rounds) for hardening plans and designs before you
build. `133` is a *wide one-shot fan-out* (6 reviewers, cross-table, adjudicate) for auditing a
finished release-grade diff. Harden the plan with `/dialectic`; audit the implementation with
`/133`.

## Did it survive itself?

Before this folder was published, the publication itself — this README, the sanitized
`SKILL.md`, and the monorepo around them — went through a reduced `/133` arc: self-review,
then a Codex + Opus pair on the same neutral prompt, then a fix-only mini-review. Three rounds
produced **18 real findings and zero hallucinated ones**, with the unique catches split across
reviewers exactly as described above (Codex ran the CLI to fact-check flags; Opus caught the
cross-document contradictions). And the fix round introduced **4 regressions that only the
re-review caught** — including a guard sentence that would have let reviewers legally skip
files, and an update command that silently overwrote user customizations. That is why Phase 3
exists.

## Install

The runnable version is a [Claude Code](https://claude.com/claude-code) skill
([`SKILL.md`](./SKILL.md)). **Paste this to Claude Code:**

> Install this skill into ~/.claude/skills:
> https://github.com/beingcognitive/skills/tree/main/133

Then invoke with **`/133`** (aliases: "1+3+3", "팬아웃 검수") on a release-grade branch.
Ask again to update — it overwrites `SKILL.md` in place, so back yours up first if you've
customized it.
