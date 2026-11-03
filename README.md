# LaTeX proof expansions — Claude skills

Two Claude skills for working through proofs in LaTeX documents.

| Skill | Usage | What it does |
|---|---|---|
| [`expand`](skills/expand/SKILL.md) | `/expand [quick\|standard\|full] <target>` | Adds indented dark-orange blocks that fill in unclear steps and omitted calculations in a proof. The target can be a result name (e.g. `Lemma D.2`), a line reference (`@paper.tex#L120-145`) or the current editor selection. |
| [`expansions`](skills/expansions/SKILL.md) | `/expansions hide\|show\|strip [target]` | Hides, shows or removes the expansion blocks that `/expand` added, for the whole document or for one result or selection. |

## Installation

**Claude Code:** copy (or symlink) the skill folders into your skills directory:

```bash
cp -r skills/expand skills/expansions ~/.claude/skills/
```

**Claude.ai / desktop app:** zip each folder (`expand/`, `expansions/`) and upload it under *Settings → Capabilities → Skills*.
