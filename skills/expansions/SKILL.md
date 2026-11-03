---
name: "expansions"
description: "Use when the user types /expansions hide, /expansions show or /expansions strip (for the whole document, or for one result or selection), to hide, show or remove the dark-orange expansion blocks that /expand added to a LaTeX document."
---

# /expansions — hide, show or remove expansion blocks

Mode: $ARGUMENTS

`/expand` tags every block with its result's key: `\begin{expansion}[lem:D2]`. The key is the `\label` in the result's statement, or the result's name without spaces if it has none. All switches live in the preamble region between `% >>> expansion blocks` and `% <<< end expansion blocks`. Hiding or showing therefore changes one preamble line and never touches a proof.

## Modes

Each mode works on the whole document, or on one result when a name is given (e.g. `/expansions hide Lemma D.2`) or text is selected in the editor.

- **`hide`**: the blocks stay in the .tex source but no longer appear in the PDF.
- **`show`**: the blocks appear in the PDF again.
- **`strip`**: remove the blocks for good. For the whole document this writes clean copies. For one result it edits the working files.
- **No mode given**: switch between hide and show, for the named result if there is one, otherwise for the whole document. Say which state is now active.

## 1. Find the document and the key

- Find the main .tex file: the one with `\documentclass` whose preamble has the expansion region, or otherwise defines the `expansion` environment. The preamble may also be in a separate .sty or preamble file it loads.
- Find every file the main file pulls in with `\input` or `\include`, because blocks can be in any of them.
- If there are several candidate documents, ask which one is meant.
- **Single result:** you don't need to locate its proof. Take the key from the `\label` in its statement (or its name without spaces), then search the files for `\begin{expansion}[KEY]`. If nothing matches, list the keys in use (`grep -o 'begin{expansion}\[[^]]*\]'`) and pick the one that clearly belongs to the result, or ask.
- **Selection:** the keys of the blocks inside the selection.

## 2. Migrate an older setup (once)

If the preamble has no expansion region, or any untagged `expansion` blocks or `hiddenexpansion` blocks remain, migrate first:

1. Replace the old definitions with the region below. That means the `expansion`/`hiddenexpansion` definitions, the `\ifshowexpansions` switch lines and `\excludecomment{…}` lines, plus `\usepackage{comment}`/`\usepackage{environ}` if they were added only for these. Keep the `expandorange` colour line only once. If the old switch was off (`\showexpansionsfalse`), set the new one off too.
2. Tag every untagged `expansion` block with the key of the result whose proof contains it.
3. For each result with `hiddenexpansion` blocks: rename them to `expansion` with that result's key, and add `\hideexpansionsof{KEY}` above `% <<< end expansion blocks`.
4. After compiling (step 5), confirm with `pdftotext` that the same blocks are visible as before.

The region (it needs `xcolor` loaded before it; on LaTeX older than 2020 also add `\usepackage{xparse}`):

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

If `expansion` is defined in some other way, show the user the definition and ask before changing it.

## 3. Hide or show

How a block's visibility is decided:

- If the region has an override line for the block's key (`\hideexpansionsof{KEY}` or `\showexpansionsof{KEY}`), that line decides.
- Otherwise the whole-document switch decides (`\showexpansionstrue` / `\showexpansionsfalse`).

So each result is independent. For example, "only Lemma D.2" is the switch off plus `\showexpansionsof{lem:D2}`.

### Whole document

- **hide:** change `\showexpansionstrue` to `\showexpansionsfalse`.
- **show:** change `\showexpansionsfalse` to `\showexpansionstrue`.

That is the only line that changes. Override lines stay. If any exist, list them in the reply (e.g. "Lemma D.2 is still hidden individually").

### Single result or selection

For each key:

1. Delete any existing override line for that key.
2. **hide:** if the whole-document switch is on, add `\hideexpansionsof{KEY}` above `% <<< end expansion blocks`.
3. **show:** if the whole-document switch is off, add `\showexpansionsof{KEY}` there.

Nothing outside the region changes. A selection acts on every result whose blocks it contains, not just the selected blocks. Say so if the selection covers only part of a proof.

## 4. Strip

Use these patterns (Python, flags `re.S | re.M`). `BLOCK` matches both `expansion` and `hiddenexpansion`, tagged or not, from the `\begin` line to the `\end` line inclusive:

```python
BLOCK  = r'^[ \t]*\\begin\{(?:hidden)?expansion\}(?:\[(?P<key>[^\]]*)\])?[ \t]*\n.*?^[ \t]*\\end\{(?:hidden)?expansion\}[ \t]*\n'
REGION = r'^% >>> expansion blocks.*?^% <<< end expansion blocks[ \t]*\n'
```

### A single result or selection

- **Result:** in every file, delete the blocks whose key matches, and that key's override line. Leave the region and all other blocks alone.
- **Selection:** delete only the blocks inside the selection. If no block with that key remains anywhere, also delete its override line.
- Check: compare each file before and after. The only differences must be the removed blocks and override line. No block with that key may remain, apart from blocks deliberately left outside a selection.
- In the reply, say how many blocks were removed. Mention that undo in the editor, Git or Overleaf's history restores them.

### The whole document

- Never strip the whole document in place. Write clean copies next to the originals with `-clean` added to each file name (e.g. `main.tex` → `main-clean.tex`). Update `\input`/`\include` lines in the clean copies so they point to the other clean copies.
- In the clean copies, delete every block (`BLOCK`, any key) and the region (`REGION`), and nothing else. Leave package lines such as `xcolor` and `amsmath` in place, since they are harmless and may be used elsewhere.
- Check: compare each clean copy with its original. The only differences must be the removed blocks and the region. Confirm that `expansion` no longer appears anywhere in the clean copies.

When the user first uses hide, mention once that hidden blocks are still in the .tex source. Anyone who receives the source, such as arXiv or a journal that takes LaTeX files, can read them, and so can anyone with access to the project on Overleaf. For anything submitted as source, use `strip`.

## 5. Check the build

If a LaTeX compiler is available (e.g. `latexmk` or `pdflatex`), compile the affected document once to confirm it still builds. Check with `pdftotext` that the blocks that should be hidden are absent and the others present. Report any errors instead of working around them.

## 6. Reply

Keep the reply to a couple of lines:

- which mode was applied, and to which document (and result and key, if one was named);
- for strip: the names of the clean files or the number of blocks removed, and whether the check passed;
- after a whole-document hide or show: any results with override lines;
- whether the document compiled, if you compiled it.