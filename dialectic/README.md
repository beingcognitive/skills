# The Unprimed Dialectic

*Hardening a decision with AI reviewers that don't know what you want.*

When you use an AI to harden a plan — a design, an algorithm, a decision — you hit a wall that
more capability won't knock down: a single model can't reliably surface its own blind spots. It
wrote the reasoning; the same priors that produced a gap are the priors that fail to notice it.
You can ask "are you sure?" and it will dutifully find something, or dutifully reassure you. Both
are mostly noise — you can't tell the signal from the reflex.

The pattern is old. It's usually taught as **thesis → antithesis → synthesis** and pinned on
Hegel — a well-known misattribution; the neat triad is closer to Fichte, and was spread as a
Hegel-summary by later expositors. The same triad, imported, is rendered 正反合 in Korean and
across the Sinosphere. Put up a position, attack it from the outside, fold the survivors back in,
and repeat until new critiques stop changing it. What's worth writing down isn't the idea — it's
the **operational discipline** that makes it work with AI reviewers, the parts that are easy to
get subtly, invisibly wrong.

## 1. Don't prime the reviewers

The mistake that most reliably ruins the exercise. Hand a reviewer your expected verdict — "I
think this is basically done," "we already rejected X, don't bring it up," "say READY if there's
nothing new" — and you haven't gotten a review. You've gotten **compliance.** The reviewer can
read the answer you want off your prompt and hand it back. "Convergence" becomes theater.

I learned this the hard way. Partway through hardening something, the AI running the loop framed a
round with the conclusion baked in: *we've done two rounds, nothing substantive should remain,
just confirm.* The reviewers confirmed. I pushed back — that's not a test, it's a leading question.
So we re-ran the **identical** artifact, changing only the prompt to a neutral "critique this," and
the reviewers immediately surfaced real bugs the primed run had buried. We can't fully separate
framing from ordinary run-to-run variance — but a primed run buried what a neutral re-run caught at
once.

So: give the reviewer the artifact and "critique this." Keep your prediction to yourself — tell the
human, never the reviewer. Neutral *context* (fixed constraints, non-goals) is fine; a prescribed
*conclusion* is not.

There's a real tension worth naming. A totally open prompt can nudge a reviewer to invent issues
just to look useful; any whiff of your verdict makes it rubber-stamp. The exit is a neutral prompt
that explicitly permits "if it's genuinely solid, say so" — without ever saying what "solid" looks
like.

## 2. Independence is earned, not assumed

The engine is independence, so be honest about how much you have. There are two kinds, and they are
**not** co-equal:

- **Model independence** — a *different* model (different weights, different training). It catches
  different *classes* of error. The only way to get it is to actually route a reviewer to a
  different model or provider.
- **Perspective independence** — the *same* model with no shared context and a different brief. A
  freshly spawned reviewer that never saw the draft's history catches what shared context would have
  quietly suppressed. But it buys you a lot against context-poisoning and **nothing** against the
  model's intrinsic blind spots; only model independence touches those.

And "no shared context" is itself a claim you have to *check*, not assume. Concretely: while
hardening this essay, a reviewer I had spawned to be context-free cited a function name from a
completely unrelated task earlier in the same conversation. The agent tool had quietly handed it the
whole session — the isolation I thought I'd arranged wasn't there. My "two independent reviewers"
were really one (a different model) plus a same-model second opinion that shared my context. The
trap has my fingerprints on it: I wrote the rule, then tripped it while writing about it. Name which
kind of independence you actually have; diversify what you can — model, review *axis*, evidence
source — and route each reviewer to its edge (checkable facts to one that can search; taste and
argument to a fresh critical reader).

There's a third move the same logic implies, and it surfaced while extending the method. Every
reviewer above sees your draft first, so every critique reasons *inside* its framing. That finds bugs
in your approach; it can't easily catch your *approach* being wrong. So spend the first round's
independence on **generation, not just critique**: hand a blind reviewer only the problem and the
constraints — never the draft — and ask for its own approach. Where its solution diverges from yours
is the highest-signal thing the round produces; it's precisely the framing assumption a draft-anchored
critic would have taken for granted. Treat that divergence as a *finding to adjudicate*, not a design
to merge — the instant you start merging whole rival solutions, the synthesizer's bias toward its own
draft (next section) quietly does the choosing. One round of this is enough; do it every round and the
draft never stops re-diverging long enough to converge.

## 3. The synthesizer is the weakest link — so audit it

Notice who does the work: *you* write the draft, *you* pick the reviewers, *you* decide which
findings are "genuinely new," *you* adjudicate disagreements, *you* declare convergence. Every
bias-prone judgment runs through the one party most invested in the draft passing.

The guardrail at the *stop* — don't quit because it "feels done" — is well known. The unguarded one
is *synthesis*: it is just as easy to wave a real finding away as "already handled" as it is to stop
early. So make the judgment auditable. **Log what you rejected and why, not just what you accepted.**
That rejection ledger is the only check anyone has on your "already addressed" reflex.

A sharp consequence, surfaced only when we ran the method on itself: a finding can be *genuinely new*
(the reviewer never saw it answered, because reviewers are blind to prior rounds) yet *correctly
produce no change* (the draft already answers it). Because blind reviewers keep re-raising points the
draft already answers, "no new finding" may never arrive — so a finished draft can fail to converge.
Define a clean round instead as **"synthesis accepted nothing; the draft did not change."**

## 4. Stop on a rule, never on a feeling

"This is probably done" is not a stop signal. It is what confirmation bias feels like from the
inside — the pull is strongest right when you most want to quit. So pre-commit the stop *before* you
start: a stated rule (e.g. two consecutive rounds where synthesis accepts nothing), a budget fixed in
advance, or a human's explicit call. Deciding the stop mid-loop, when the next round happens to look
expensive, is how the feeling wins.

You'll feel the feedback change character as you near the end: early rounds produce defects and design
changes; later, reviewers stop finding holes and start asking for traceability and tests. When
critique turns mostly to verification, that's a useful convergence signal — not proof you're done. And
keep the ceiling in view: this hardens your *reasoning*; it does not *prove correctness.* For
empirical, code, financial, security, or performance claims, dialectic agreement is not evidence — you
still have to run the thing.

## The method, applied to itself

We turned the method on the write-up of the method. It surfaced several real gaps in the first draft —
two of them the "clean round" definition above, and a contradiction we'd introduced while trying to
control cost. It also caught the loop priming its own reviewers, and the reviewer that wasn't really
isolated — the two failures that became rules #1 and #2. Running a method on itself doesn't *prove* it
works; that would be the opening blind-spot problem one level up. But "it kept finding real things,
including its own author's mistakes" is the behavior you'd want.

## A note on translation (正反合 ≠ thesis-antithesis-synthesis)

정반합 / 正反合 (*jeongbanhap*) is a modern rendering of that textbook triad, and the characters carry
connotations the usual English terms don't foreground. In the coinage, the final 合 (*hap*) stands in
for 綜合, "synthesis." But the character's field is wider than "compose a composite": the *Shuowen*
reads 合 as *closing/joining the mouth*, from 亼 ("gather") and 口 ("mouth"); later glyph explanations
gloss the shape as a lid fitting a vessel — *things fitting together.* It carries *fit, accord,
come-into-harmony*; and 合格, "to pass, to meet the bar," lends a secondary resonance of *measuring
up.* So where "synthesis" sounds like assembling a third thing, 合 leans toward *things clicking into
accord — and passing.* Translating back to English doesn't lose Korean meaning so much as shed the
resonance the characters had added. Worth holding onto even with only the English word: the aim isn't
to *build* a compromise. It's to reach the version that **fits** — the one that survives the attack
with nothing left to answer.

## The skill

The runnable version is a [Claude Code](https://claude.com/claude-code) skill
([`SKILL.md`](./SKILL.md)). **Paste this to Claude Code:**

> Install this skill into ~/.claude/skills:
> https://github.com/beingcognitive/skills/tree/main/dialectic

Then call it with **`/dialectic`** (or "정반합"). Ask again to update — it overwrites
`SKILL.md` in place, so back yours up first if you've customized it. (This
[skills monorepo](https://github.com/beingcognitive/skills) is canonical; the standalone
`unprimed-dialectic` repo is this skill's pre-monorepo home and is archived.)

It's deliberately heavier than a quick utility — several model calls per round, several rounds — so it
pays off on genuinely high-stakes convergence, not everyday edits. Reach for it when a single model's
blind spots actually worry you; skip it when they don't.

---

*Written with Claude Code. The skill was hardened by running itself on itself; this essay went through
the same loop — and the loop caught, among other things, the moment one of its own "independent"
reviewers turned out not to be.*
