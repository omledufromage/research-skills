# LaTeX proof expansions — Claude skills

Two Claude skills for working through proofs in LaTeX documents.

| Skill | Usage | What it does |
|---|---|---|
| [`expand`](skills/expand/SKILL.md) | `/expand [quick\|standard\|full] <target>` | Adds indented dark-orange blocks that fill in unclear steps and omitted calculations in a proof. The target can be a result name (e.g. `Lemma D.2`), a line reference (`@paper.tex#L120-145`) or the current editor selection. |
| [`expansions`](skills/expansions/SKILL.md) | `/expansions hide\|show\|strip [target]` | Hides, shows or removes the expansion blocks that `/expand` added, for the whole document or for one result or selection. |

## Example

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
cp -r skills/expand skills/expansions ~/.claude/skills/
```

**Claude.ai / desktop app:** zip each folder (`expand/`, `expansions/`) and upload it under *Settings → Capabilities → Skills*.
