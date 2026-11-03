---
name: "expand"
description: "Use when the user types /expand, optionally with a depth (quick, standard or full), followed by a result name (e.g. Lemma D.2), a line reference such as @paper.tex#L120-145, or with text selected in the editor, to add indented dark-orange blocks that fill in unclear steps and omitted calculations in that proof."
---

# /expand — fill in unclear steps and omitted calculations in a proof

Arguments: $ARGUMENTS

## 1. Read the arguments

**Depth.** If the first word of the arguments is `quick`, `standard` or `full` (any case), it sets the depth, and the rest of the arguments is the target. Otherwise the depth is `standard`. Section 3 says what each depth means.

**Target.**

- **Result name** (e.g. `Lemma D.2`): find the statement and proof in the workspace. Search the .tex files for the name, number or `\label`. Go through the whole proof.
- **Line reference** (e.g. `@paper.tex#L120-145`, as inserted by Ctrl+Alt+K, or Cmd+Option+K on Mac): read those lines from the file and treat them as a selection.
- **Text selected in the editor** (the IDE attaches it to the message, e.g. as a note that the user selected certain lines of a file) **or pasted**: expand only that passage. Treat its sentences and displayed equations as the steps of the original proof. Use the rest of the proof and earlier results only as context.
- **Nothing came through** (no result name, line reference, selection or pasted text): do not guess. Say in one line that you can't see a selection. Tip the user that selecting the passage and pressing Ctrl+Alt+K (Cmd+Option+K on Mac) inserts a line reference that still works after clicking into the terminal. Ask which result or passage to expand.
- If the result cannot be found, say so and ask where it is. Never reconstruct a proof from memory.

**Key.** Every block is tagged with its result's key, so that `/expansions` can hide, show or strip one result without touching its proof. The key is the `\label` in the result's statement (e.g. `lem:D2`). If the statement has no label, use the name without spaces (e.g. `LemmaD.2`). Never add a label to the original. For a selection or line reference, use the key of the result whose proof contains it.

## 2. Ground rules

- **The original proof stays exactly as it is**, including its colours and formatting. Do not reword, reorder, shorten, recolour or "fix" it. If you notice a typo or an error, mention it in an expansion block instead.
- **Expansion blocks are indented and dark orange.** Each one sits between two consecutive steps of the original proof.
- Add a block only where the depth calls for one (section 3). Immediate steps never get a block.
- Gaps are always flagged, at every depth (section 9).

## 3. Depths

First label every transition between consecutive steps of the original proof as one of:

- **immediate**: one routine move (unfolding a definition, a substitution, a standard identity);
- **citation**: uses a result or hypothesis that is not named at that point;
- **calculation**: leaves out more than one manipulation;
- **idea**: needs a choice of constant, a case split, a construction or a non-obvious estimate;
- **gap**: does not follow (section 9).

Block sizes count sentences of prose and rows of `align*` together.

### quick

- Written for an expert in the area, such as a referee.
- Blocks only for idea and gap steps, and for calculation steps that leave out 4 or more manipulations.
- At most 4 blocks per proof (or per selection), keeping the steps that miss the most. Gap blocks do not count towards this cap.
- At most 3 sentences or rows per block.
- Content: the result or hypothesis that makes the step work, and the key intermediate expression. No step-by-step derivation.
- Context: the proof, the result's statement, and the statements of the results and hypotheses the proof cites.

Example (for a result labelled `lem:abscont`):

```latex
\begin{expansion}[lem:abscont]
Take $\delta = \varepsilon/L$; the Lipschitz bound on each interval gives $\sum_i |f(b_i)-f(a_i)| \le L\sum_i (b_i-a_i) < L\delta = \varepsilon$.
\end{expansion}
```

### standard

- Written for a PhD student in the field who knows the standard results.
- A block for every non-immediate step. A citation step gets one sentence naming the result or hypothesis.
- At most 8 sentences or rows per block.
- Content: a chain of implications stating each link, with a justification only for links that use a named result or hypothesis. Calculations show their main intermediate lines, not every manipulation.
- Context: as for quick, plus the definitions and notation (macros) the proof uses.

Example:

```latex
\begin{expansion}[lem:abscont]
$f$ is $L$-Lipschitz, so $|f(b_i)-f(a_i)| \le L(b_i-a_i)$ on each interval; summing gives the estimate below, which implies that $\delta = \varepsilon/L$ satisfies the definition of absolute continuity.
\begin{align*}
\sum_{i=1}^n |f(b_i) - f(a_i)|
  &\le L \sum_{i=1}^n (b_i - a_i) && \text{(Lipschitz bound)}\\
  &< L\,\delta = \varepsilon.
\end{align*}
\end{expansion}
```

### full

- Written for a reader checking every line.
- A block for every non-immediate step.
- No size limit. Each expansion is complete enough that a reader can go from one step of the original proof to the next by checking each line, without filling in anything themselves.
- Content: the preferred style below in full: every link of each chain stated separately, the result or hypothesis named after any link that is not immediate, and one manipulation per `align*` row.
- Context: the statements (not the proofs) of the definitions, lemmas and earlier results the proof uses, and the notation and macros the document defines.

Example:

```latex
\begin{expansion}[lem:abscont]
$f$ being $L$-Lipschitz implies $|f(y)-f(x)| \le L|y-x|$ for all $x,y$; this implies the estimate below for any finite family of disjoint intervals $(a_i,b_i)$ with $\sum_i (b_i-a_i) < \delta$; and the estimate implies that $\delta = \varepsilon/L$ satisfies the definition of absolute continuity.
\begin{align*}
\sum_{i=1}^n |f(b_i) - f(a_i)|
  &\le \sum_{i=1}^n L\,(b_i - a_i) && \text{(Lipschitz bound on each interval)}\\
  &= L \sum_{i=1}^n (b_i - a_i) \\
  &< L\,\delta = \varepsilon. && \text{(choice $\delta = \varepsilon/L$)}
\end{align*}
\end{expansion}
```

### Preferred style (all depths)

- **Chains of implications.** Wherever possible, phrase the argument as an explicit chain: "A implies B, B implies C, and C implies D".
- **Aligned calculations.** Put algebraic manipulations, estimates and chains of equalities or inequalities in an `align*` environment:
  - relation symbols (`=`, `\le`, `<`, …) aligned with `&`;
  - where a line isn't obvious, a short justification at the right, after `&&`, as `\text{(...)}`;
  - one manipulation per row at `full`; at `quick` and `standard` a row may combine routine manipulations.
- Use ordinary prose only where neither form fits.
- No `\verb` or other verbatim material inside a block.

## 4. Division of work

The expansion blocks are written by a different agent from the one that wrote the original proof, so they also serve as a second check of each step. For correctness, the writer is Opus at every depth.

Launch a fresh subagent with the Agent tool, with model `opus`, to write the blocks. Give it a self-contained brief with:

- the depth, that depth's rules and example from section 3, and the preferred style;
- the statement of the result, and its key;
- the original proof text, verbatim (for a selection: the selected passage verbatim, and the rest of the proof marked as context only);
- the context listed for that depth in section 3, and nothing more (a slim brief is what keeps `quick` fast);
- section 9 on gaps.

Ask the subagent to label each step first, write blocks only for the steps its depth covers, check its own counts against the depth's limits, and return the blocks ranked from the step that misses the most to the step that misses the least. Each block comes back already tagged with the key, with an anchor: a short verbatim quote of the original text the block comes after. The subagent only returns text, and you make all edits to the file.

At `quick`, keep gap blocks plus the first 4 other blocks in the subagent's ranking. Then splice each block after its anchor, so the blocks appear in the proof's order.

If no subagent is available (e.g. in a plain chat), write the blocks in a separate pass, working from the original proof text alone. Say in one line that a separate agent was not used.

## 5. Running again on the same result

Blocks are tagged with their result's key, so a new run replaces earlier blocks instead of adding to them:

- **Result name:** remove every `expansion` block tagged with that key, then splice in the new ones.
- **Selection or line reference:** remove only the blocks with that key that lie inside the selected lines, then splice in the new ones there.

This lets the user run `quick` over a whole proof and then `full` on a single step. If the preamble has `\hideexpansionsof{KEY}` for this key, leave it and mention in the reply that the new blocks are hidden.

## 6. Output format

### LaTeX file (the usual case)

Edit the file in place. Each block is `\begin{expansion}[KEY]` … `\end{expansion}`, with the `\begin` and `\end` lines each on a line of their own (indenting them is fine). Never leave a block untagged.

**Preamble.** `xcolor` and `amsmath` must be loaded; add `\usepackage{xcolor}` / `\usepackage{amsmath}` if they aren't. If the preamble has no region starting with `% >>> expansion blocks`, add this one after the package lines:

```latex
% >>> expansion blocks: managed by /expand and /expansions
\definecolor{expandorange}{RGB}{184,84,0}
\newif\ifshowexpansions
\showexpansionstrue
\newif\ifexpansionshown
\newcommand\hideexpansionsof[1]{\expandafter\def\csname expstate:\detokenize{#1}\endcsname{0}}
\newcommand\showexpansionsof[1]{\expandafter\def\csname expstate:\detokenize{#1}\endcsname{1}}
\long\def\expansionkeep#1{#1}
\long\def\expansiondrop#1{}
\NewDocumentEnvironment{expansion}{O{} +b}{%
  \ifshowexpansions\expansionshowntrue\else\expansionshownfalse\fi
  \ifcsname expstate:\detokenize{#1}\endcsname
    \if1\csname expstate:\detokenize{#1}\endcsname\expansionshowntrue\else\expansionshownfalse\fi
  \fi
  \ifexpansionshown\expandafter\expansionkeep
  \else\ignorespacesafterend\expandafter\expansiondrop\fi
  {\begin{quote}\color{expandorange}#2\end{quote}}}{}
% per-result overrides (\hideexpansionsof{KEY} / \showexpansionsof{KEY}) go here
% <<< end expansion blocks
```

`\NewDocumentEnvironment` is built into LaTeX since 2020 (Overleaf and TeX Live 2021+). On an older installation, also add `\usepackage{xparse}`.

**Older setup.** If the preamble instead defines `expansion` the older way (a plain `\newenvironment{expansion}{\begin{quote}\color{expandorange}}{\end{quote}}`, or an `\ifshowexpansions` switch built with `comment` or `environ`), migrate it once:

1. Replace the old definitions with the region above. That means the `expansion`/`hiddenexpansion` definitions, the switch lines and `\excludecomment{…}` lines, plus `\usepackage{comment}`/`\usepackage{environ}` if they were added only for these. Keep the `expandorange` colour line only once. If the old switch was off (`\showexpansionsfalse`), set the new one off too.
2. Tag every untagged `expansion` block with the key of the result whose proof contains it.
3. For each result with `hiddenexpansion` blocks: rename them to `expansion` with that result's key, and add `\hideexpansionsof{KEY}` above `% <<< end expansion blocks`.

If `expansion` is defined in some other way, show the user the definition and ask before changing it.

The first time, tip the user that `/expansions hide Lemma D.2` hides one result's blocks by adding a single preamble line, `/expansions hide` hides every block, and `/expansions strip` makes a clean copy for submission.

### Word (.docx)

Each expansion block is its own paragraph, with a left indent and font colour #B85400. Put calculations in an equation array with the relation symbols lined up. Leave the original paragraphs' formatting untouched. The depth limits apply as in LaTeX.

### Other formats (Markdown, plain text, chat)

Output LaTeX source using the same tagged `expansion` environment.

## 7. Verify the original proof is unchanged

After splicing, remove every block and the marked region from the edited file, using these patterns (Python, flags `re.S | re.M`):

```python
BLOCK  = r'^[ \t]*\\begin\{(?:hidden)?expansion\}(?:\[(?P<key>[^\]]*)\])?[ \t]*\n.*?^[ \t]*\\end\{(?:hidden)?expansion\}[ \t]*\n'
REGION = r'^% >>> expansion blocks.*?^% <<< end expansion blocks[ \t]*\n'
```

Compare the result with the file as it was before this run, treated the same way. They must be identical apart from any package lines you added or old definitions you migrated. If anything else differs, fix it before reporting. Run this check at every depth.

## 8. Reply

Keep the reply to a couple of lines:

- what was expanded, at which depth, in which file, and under which key;
- whether the check in step 7 passed;
- any gaps found.

## 9. Gaps in the proof

At every depth, if a step does not follow and cannot be fully justified, say so explicitly in an expansion block. For example: "Gap: (4) does not follow from (3) because …; it would need …". Leave the original proof unchanged.