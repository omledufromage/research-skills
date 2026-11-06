# Research skills for Claude

Claude skills for writing mathematical research papers in LaTeX: one that sets up a new research project with a team of Claude Code agents, and two for working through proofs.

| Skill | Usage | What it does |
|---|---|---|
| [`init-research-team`](skills/init-research-team/SKILL.md) | `/init-research-team` (Claude Code only) | Sets up a new project as a `Paper/` folder (LaTeX, synced with Overleaf) and a `Code/` folder (simulations, on GitHub), with a team of agents and the checks that work must pass before it reaches the paper. |
| [`expand`](skills/expand/SKILL.md) | `/expand [quick\|standard\|full] <target>` | Adds indented dark-orange blocks that fill in unclear steps and omitted calculations in a proof. The target can be a result name (e.g. `Lemma D.2`), a line reference (`@paper.tex#L120-145`) or the current editor selection. |
| [`expansions`](skills/expansions/SKILL.md) | `/expansions hide\|show\|strip [target]` | Hides, shows or removes the expansion blocks that `/expand` added, for the whole document or for one result or selection. |

## Setting up a research project

`init-research-team` runs in Claude Code (the desktop Code tab, the CLI or VS Code), because it writes configuration files and runs `git`. It first asks for the project folder, the paper's title and authors, a short summary, the simulation language (R by default, or Python), and the Overleaf and GitHub URLs. It shows the values back before writing anything. It then creates:

- **`Paper/`:** the manuscript, as a git clone of the Overleaf project. Its Claude Code session runs with `lead` as the main agent, which coordinates the others:
  - `prover` writes and repairs statements and proofs;
  - `latex-editor` handles wording, structure and the bibliography;
  - `referee` and `factchecker` check every deliverable before it reaches you;
  - `lit-scout` searches the literature;
  - `numerics` writes and runs the simulations;
  - `code-reviewer` reviews simulation code before its results go into the paper;
  - `hr` adds or changes agents when you ask.
- **`Code/`:** the simulation code, in its own git repository on GitHub, worked on by `numerics`. Numbers, tables and figures reach the paper only through a results gate: committed code, passing tests, code review, and a rerun that reproduces the stored result.
- **Notes and colour macros:** `NOTES.md`, `TODO.md` and `COORDINATION.md` for handing over between sessions, and LaTeX colour macros that mark changes awaiting your review.
- **A guard:** a Python hook that Claude Code runs before every command and file edit by the lead and every agent. It enforces the team's rules instead of only stating them:
  - each agent writes only its own files; for example, the referee and factchecker write only their reports;
  - only the owner of a repository commits to it, and never with `commit -a`;
  - nothing in the project is deleted;
  - every push asks you first.

  It includes a self-test, and agents whose calls are refused report the refusal instead of finding another way round. It reads command text, so a program that writes files itself, such as an R script, is still covered only by the agents' instructions.

The agent instructions and notes stay on your machine: they are git-ignored, so they never reach Overleaf. The skill never overwrites or deletes a file, and never pushes without asking. The team does not write proof expansions itself; it uses `/expand` and `/expansions` (below), so install those too.

## Working through proofs

`/expand` fills in the steps a proof leaves out, and `/expansions` hides, shows or removes what it added. They work in any LaTeX document, in Claude Code or in the desktop app.

### Example

Say `paper.tex` contains this lemma. Its proof chooses $\delta = \varepsilon/L$ but never shows why that choice works: it skips applying the Lipschitz bound to each interval and summing.

```latex
\begin{lemma}\label{lem:abscont}
Every $L$-Lipschitz function $f\colon[a,b]\to\mathbb{R}$ is absolutely continuous.
\end{lemma}
\begin{proof}
Let $\varepsilon > 0$ and take $\delta = \varepsilon/L$.
Then $\sum_i |f(b_i)-f(a_i)| < \varepsilon$ whenever $\sum_i (b_i-a_i) < \delta$.
\end{proof}
```

### Expanding the proof

```
/expand lem:abscont
```

At the default `standard` depth, Claude leaves the original proof untouched and inserts a block tagged with the lemma's `\label`:

```latex
\begin{proof}
Let $\varepsilon > 0$ and take $\delta = \varepsilon/L$.
\begin{expansion}[lem:abscont]
$f$ is $L$-Lipschitz, so $|f(b_i)-f(a_i)| \le L(b_i-a_i)$ on each interval; summing gives the estimate below, which implies that $\delta = \varepsilon/L$ satisfies the definition of absolute continuity.
\begin{align*}
\sum_{i=1}^n |f(b_i) - f(a_i)|
  &\le L \sum_{i=1}^n (b_i - a_i) && \text{(Lipschitz bound)}\\
  &< L\,\delta = \varepsilon.
\end{align*}
\end{expansion}
Then $\sum_i |f(b_i)-f(a_i)| < \varepsilon$ whenever $\sum_i (b_i-a_i) < \delta$.
\end{proof}
```

In the PDF the block appears indented and in dark orange. On the first run, Claude also adds a short `% >>> expansion blocks` region to the preamble that defines the `expansion` environment and its show/hide switches.

Other ways to call it:

- `/expand Lemma 2.1`: the result's name or number works too, as well as its `\label`.
- `/expand quick lem:abscont`: a few short hints for an expert reader.
- `/expand full lem:abscont`: every step written out, one manipulation per line.
- `/expand full @paper.tex#L120-125`: expand only those lines, for example one step that is still unclear after a `quick` pass. Running `/expand` again on the same result replaces its earlier blocks instead of adding to them.

If a step does not follow, the block says so (`Gap: … does not follow because …`) instead of patching the proof.

### Hiding, showing and removing the blocks

```
/expansions hide lem:abscont # hide this lemma's blocks: adds \hideexpansionsof{lem:abscont} to the preamble
/expansions show lem:abscont # show them again
/expansions hide             # hide every block: \showexpansionstrue becomes \showexpansionsfalse
/expansions strip            # write main-clean.tex (and clean copies of \input files) with no blocks
```

Hiding and showing change a single preamble line and never touch a proof. Hidden blocks are still in the `.tex` source, so use `strip` before you send the source to arXiv, a journal or co-authors.

## Installation

**Claude Code:** copy (or symlink) the skill folders into your skills directory:

```bash
cp -r skills/init-research-team skills/expand skills/expansions ~/.claude/skills/
```

**Claude.ai / desktop app:** zip each folder (`init-research-team/`, `expand/`, `expansions/`) and upload it under *Settings → Capabilities → Skills*. `init-research-team` only works when run from Claude Code.
