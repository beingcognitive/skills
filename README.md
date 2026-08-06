# skills

Claude Code skills I actually run, shared as-is. Each folder is one skill: a `SKILL.md` you can
drop into `~/.claude/skills/<name>/`, plus a `README.md` explaining the thinking behind it.

These are patterns extracted from real daily use — the anecdotes in each write-up happened.

## Skills

| Skill | One line | Language |
|---|---|---|
| [`dialectic`](./dialectic/) | Harden a plan/design/decision with independent AI reviewers that don't know what you want — an unprimed thesis→antithesis→synthesis (정반합) loop. | English |
| [`133`](./133/) | 1+3+3 adversarial review fan-out — self-review first, then 3+3 independent reviewers across two model families, cross-table adjudication. For release-grade changes only. | Korean (English summary) |

## Install

```bash
# one skill
mkdir -p ~/.claude/skills/dialectic
curl -fsSL https://raw.githubusercontent.com/beingcognitive/skills/main/dialectic/SKILL.md \
  -o ~/.claude/skills/dialectic/SKILL.md
```

Or clone and copy the folders you want into `~/.claude/skills/`. Claude Code picks them up as
`/dialectic`, `/133`, etc.

Some skills assume tools beyond Claude Code itself (e.g. `133` fans out to the OpenAI Codex CLI
for cross-model independence). `133`'s README lists its prerequisites and reduced variants;
`dialectic` names its optional second-model dependency inside `dialectic/SKILL.md`.

## History

`dialectic` first shipped as the standalone repo
[`unprimed-dialectic`](https://github.com/beingcognitive/unprimed-dialectic) (essay + skill);
it now lives here, essay included.

## License

MIT — see [LICENSE](./LICENSE).
