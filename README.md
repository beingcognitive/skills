# skills

Claude Code skills I actually run, shared as-is. Each folder is one skill: a `SKILL.md` you can
drop into `~/.claude/skills/<name>/`, plus a `README.md` explaining the thinking behind it.

These are patterns extracted from real daily use — the anecdotes in each write-up happened.

## Skills

| Skill | One line | Language |
|---|---|---|
| [`dialectic`](./dialectic/) | Harden a plan/design/decision with independent AI reviewers that don't know what you want — an unprimed thesis→antithesis→synthesis (정반합) loop. | English |
| [`133`](./133/) | 1+3+3 adversarial review fan-out — self-review first, then 3+3 independent reviewers across two model families, cross-table adjudication. For release-grade changes only. | English |

## Install

**Paste this to Claude Code** (swap in the skill you want):

> Install this skill into ~/.claude/skills:
> https://github.com/beingcognitive/skills/tree/main/dialectic

Claude Code picks it up as `/dialectic`, `/133`, etc. Ask again to update — it overwrites in
place, so back up first if you've customized. Cloning the repo and copying folders into
`~/.claude/skills/` works too.

Some skills assume tools beyond Claude Code itself (e.g. `133` fans out to the OpenAI Codex CLI
for cross-model independence). `133`'s README lists its prerequisites and reduced variants;
`dialectic` names its optional second-model dependency inside `dialectic/SKILL.md`.

## History

`dialectic` first shipped as the standalone repo
[`unprimed-dialectic`](https://github.com/beingcognitive/unprimed-dialectic) (essay + skill);
it now lives here, essay included.

## License

MIT — see [LICENSE](./LICENSE).
