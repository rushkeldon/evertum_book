# Evertum Book — Tech Stack & Methodology

> **Purpose of this document.** This is a handoff/onboarding spec for a *fresh AI
> instance* (and the human author) collaborating on a technical book about physics,
> metaphysics, and new cosmology concepts. Read this first. It captures the stack,
> conventions, and working methods the author and a prior instance converged on,
> so a new instance can scaffold the project and start writing without re-litigating
> the foundational decisions. Where a decision is still open, it is flagged in
> **Section 12 (Open Decisions)** — confirm those with the author before scaffolding.

---

## 0. TL;DR for the next instance

- **Authoring format:** Quarto Markdown (`.qmd`). Quarto runs on Pandoc; you keep full
  LaTeX power for math and PDF.
- **Outputs:** HTML (rich/interactive), PDF (print-quality via LaTeX), EPUB optional.
- **Structure:** modular — one file per section/chapter, stitched with `{{< include >}}`.
  Hierarchy is **Part -> Chapter -> Section** (Parts optional; can start flat).
- **Math:** `$...$` / `$$...$$`, rendered by a math-aware Pandoc pipeline (this fixes the
  "Markdown munged my LaTeX" problem the author hit before).
- **Cross-references:** Quarto's `@fig-`, `@eq-`, `@sec-`, `@tbl-`, theorem refs. Stable,
  auto-numbered, resolve globally across files at build time.
- **Interactive examples:** geometric-algebra demos via **ganja.js**, embedded in the HTML
  edition, with a static image fallback in the PDF. Two-tier approach (inline include now,
  standalone-chunk + iframe later).
- **Version control:** git. Small files = clean diffs = good revision history.
- **AI guidance:** `.cursor/rules/` holds the voice + notation style guide.
- **Token economy:** never load the whole book. Load the "knowledge spine" (outline +
  glossary + notation sheet) plus the one section being edited. See **Section 11**.
- A canonical worked example already exists at [docs/example-chapter.qmd](example-chapter.qmd),
  written in proper Quarto syntax — use it as the living style exemplar (see Section 4.3).

---

## 1. Project overview

The book is a heavily mathematical, heavily cross-referenced work that will undergo many
revisions and import a large amount of existing material. Priorities, in the author's words:
strong cross-referencing, lots of iteration, importing existing material, and **not becoming
overly complex or burning too many tokens**.

Design goals that follow from that:

1. **Plain-text source** that diffs cleanly and is future-proof (no proprietary binary format).
2. **Modular files** so any one editing session touches a small surface.
3. **Full LaTeX-grade math** and **robust cross-referencing** (equations, figures, sections,
   theorems, citations).
4. **Multiple output editions** from one source: an interactive HTML edition and a
   print-quality PDF.
5. **Token discipline** as the book and its web of cosmology concepts grow.

---

## 2. The chosen stack

| Layer | Choice | Notes |
|-------|--------|-------|
| Authoring format | **Quarto Markdown (`.qmd`)** | Superset of Pandoc Markdown; plain text. |
| Conversion engine | **Pandoc** (under Quarto) | Math-aware parser; many output targets. |
| Math / PDF typesetting | **LaTeX** (via Quarto PDF) | Full LaTeX; custom preamble allowed. |
| Optional PDF backend | **Typst** (Quarto 1.9+) | Faster; optional. HTML-from-Typst still immature. |
| Interactive figures | **ganja.js** | Geometric algebra visualizations in HTML edition. |
| Preview | **Quarto preview** (and/or Markdown Preview Enhanced) | Live, math/crossref-accurate. |
| Version control | **git** | Revision history; small files. |
| Editor / AI | **Cursor** | Single canonical editor; AI lives here. |
| Bibliography | **`.bib`** (BibTeX/biblatex) + citeproc | Shared across chapters. |

Everything in this stack is **free and open source** and **runs locally** (no per-token cost
for rendering, works offline).

---

## 3. Why this stack (and what was rejected)

A new instance should not re-open these; they were vetted at length. Summary of the reasoning:

- **Quarto vs raw Pandoc + pandoc-crossref:** Quarto *is* Pandoc underneath, plus a polished
  book build system (Parts/Chapters, cross-chapter refs, simultaneous PDF/HTML/EPUB, search,
  themes) and a genuinely good live preview. It solves the preview pain "by design." Low
  lock-in: `.qmd` is ~99% plain Pandoc Markdown; you can always drop to raw Pandoc/LaTeX.
- **Quarto vs pure LaTeX:** pure LaTeX is the typesetting gold standard but weaker on
  multi-format output, slower to iterate, heavier syntax. Choose pure LaTeX **only** if a
  publisher demands `.tex` sources and PDF is the sole target. Note: choosing Quarto does
  **not** cap LaTeX — you can supply a full custom preamble and drop into raw LaTeX anywhere.
- **Quarto vs Typst (standalone):** Typst has excellent live preview and clean math syntax,
  but as of mid-2026 its **HTML export is experimental and math-in-HTML is broken by default**
  (MathML still landing). Since the book needs a rich HTML edition, Typst-alone is premature.
  Bonus: Quarto can render PDF *via* Typst, so its speed is available as a backend toggle.
- **Quarto vs Jupyter Book 2 / MyST:** a legitimate peer (community-governed, MIT, modular).
  Quarto is currently more polished for print (sub-figures, tables). Switch only if open
  governance / composable content-reuse matters more than print polish.
- **AsciiDoc:** strong technical-book markup but the weakest **math** path of the candidates —
  disqualifying for an equation-dense book.
- **Raw HTML as source:** rejected — verbose, diff-hostile, no equation auto-numbering, and
  its one advantage (stable IDs) is already provided by Quarto/LaTeX.
- **Adobe InDesign:** `.indd` is binary/proprietary (not git- or AI-friendly). Useful only as
  an *optional downstream* design destination via Pandoc's **ICML** export, and only for a
  lavishly designed edition. LaTeX already gives print quality for a math book; InDesign's
  equation support is weak. Not part of the core pipeline.

Mental model that recurs throughout: **Markdown is a host language with "escape regions" into
sub-languages**, each opened/closed by a delimiter that switches parsing rules:

- `---` ... `---` -> YAML (metadata)
- `$ ... $` / `$$ ... $$` -> LaTeX math
- ` ``` ` ... ` ``` ` -> verbatim code
- ` ```{=latex} ` -> raw passthrough to LaTeX (e.g. theorem environments)
- ` ```{=html} ` -> raw passthrough to HTML (HTML-only output)
- `:::` ... `:::` -> fenced divs (callouts, columns, conditional content)

This is *why* math stops getting munged: inside `$...$` a math-aware parser treats `_` as
subscript and `*` as multiplication, not Markdown emphasis.

---

## 4. Authoring conventions

### 4.1 Math
- Inline: `$H \equiv \dot{a}/a$`. Display: `$$ ... $$`.
- Multi-line/aligned derivations use a LaTeX `aligned` block inside `$$ ... $$`.
- For anything Markdown can't express (custom theorem environments, TikZ), drop into a raw
  ` ```{=latex} ` block — it passes straight through to the PDF.
- Residual LaTeX-escaping gotchas to teach the author and watch for:
  - literal `$` in prose -> `\$`
  - literal `%` -> `\%` (otherwise it's a LaTeX comment that eats the line)
  - `&` and `\\` belong inside proper environments, not loose in Markdown tables.

### 4.2 Cross-references and citations (Quarto syntax)
- Label elements with prefixed IDs: `{#sec-foundations}`, `{#eq-friedmann}`, `{#fig-flux}`,
  `{#tbl-symbols}`, theorems `{#thm-...}`.
- Reference with `@sec-foundations`, `@eq-friedmann`, `@fig-flux`, etc. Numbers auto-update on
  reorder and resolve **globally** across chapter files at build time.
- Citations: `@weinberg1989`, pulling from a shared `references.bib`. Set `bibliography:` in
  metadata.

### 4.3 The canonical example file
[docs/example-chapter.qmd](example-chapter.qmd) is the worked reference for this format,
written in proper **Quarto** syntax — dash-prefixed labels (`{#eq-friedmann}`) and bare
`@`-references (`@eq-friedmann`). It demonstrates: inline/display math, aligned derivations,
**native Quarto theorem environments** (`::: {#thm-...}`) which render in both HTML and PDF,
the raw `` ```{=latex} `` escape hatch for LaTeX-only content, captioned figures/tables with
stable IDs, citations, footnotes, and global cross-chapter references. Treat it as the living
style exemplar. The content (a placeholder "evertum term" modified Friedmann equation) is
illustrative only, not real physics.

### 4.4 Portable styling: fenced divs, not raw HTML
- Raw HTML is **output-targeted**: it survives to HTML output but is **dropped** from PDF/LaTeX.
- For anything that must appear in *both* editions, use **fenced divs** (`::: {.callout-note}`)
  and **bracketed spans** (`[text]{.smallcaps}`), which compile to both HTML and LaTeX.
- Markdown is **not** parsed inside raw `<div>` blocks; it **is** inside `:::` fenced divs — so
  prefer fenced divs for containers that hold math/refs.

### 4.5 Dual-edition / format-targeted content
Use conditional blocks so one source serves both editions:

```markdown
::: {.content-visible when-format="html"}
<!-- interactive / live content -->
:::

::: {.content-visible when-format="pdf"}
![Static fallback figure.](still.png){#fig-example}
:::
```

---

## 5. Source-tree layout

Recommended structure (Parts optional — see Open Decisions 12.1):

```text
evertum_book/
├── _quarto.yml                  # book config: title, part/chapter order, formats
├── metadata.yml                 # shared crossref prefixes, author, bibliography path
├── index.qmd                    # preface / landing page
├── references.bib               # shared bibliography (all citations)
│
├── parts/                       # (optional) Part intro pages
│   └── part-1-foundations.qmd
├── chapters/                    # one file per chapter (short chapters = one file)
│   ├── 01-foundations.qmd
│   ├── 02-notation.qmd
│   └── 03-cosmology.qmd         # long chapter = shell that includes sections
├── sections/                    # section fragments for long chapters
│   └── 03-cosmology/
│       ├── _01-friedmann.qmd    # leading underscore = not rendered standalone
│       └── _02-evertum-term.qmd
│
├── examples/                    # interactive ganja.js examples (see Section 7)
│   └── spacetime-rotor/
│       ├── index.qmd            # standalone "chunk" page (own URL)
│       ├── _widget.qmd          # reusable fragment (inline-include version)
│       └── still.png            # PDF fallback image
│
├── figures/                     # static images
├── assets/
│   ├── js/ganja.js              # vendored, version-pinned (offline-safe)
│   └── styles.css               # HTML edition styling
├── tex/preamble.tex             # LaTeX preamble for the PDF edition
│
├── .cursor/rules/book-style.mdc # AI voice + notation conventions
├── docs/                        # THIS methods doc + example-chapter.qmd (+ rendered output)
└── .git/                        # version control = revision history
```

`_quarto.yml` is the single place to (re)order the book:

```yaml
project:
  type: book
book:
  title: "The Evertum Hypothesis"   # placeholder title
  chapters:
    - index.qmd
    - part: "Part I: Foundations"   # remove parts wrapper for a flat list
      chapters:
        - chapters/01-foundations.qmd
        - chapters/02-notation.qmd
    - part: "Part II: Cosmology"
      chapters:
        - chapters/03-cosmology.qmd
format:
  html:
    toc: true
    resources: [assets]
  pdf:
    documentclass: scrbook
    include-in-header: tex/preamble.tex
bibliography: references.bib
```

A long chapter includes its sections:

```markdown
# Cosmology {#sec-cosmology}

{{< include ../sections/03-cosmology/_01-friedmann.qmd >}}
{{< include ../sections/03-cosmology/_02-evertum-term.qmd >}}
```

**Why this shape:** token economy (open only the section being edited), clean diffs (small
files), one-line reordering (`_quarto.yml`), and global cross-references at build time.

---

## 6. Modularity rules

- One file per section (or per short chapter). Keep files focused.
- Fragments meant only for inclusion get a **leading underscore** (`_name.qmd`) so Quarto
  doesn't render them as standalone pages. Fragments should **not** carry their own YAML front
  matter; the standalone wrapper page (`index.qmd`) holds the front matter.
- Cross-references are global at build time, so splitting into files never fragments numbering.
- Reordering = edit `_quarto.yml` only.

---

## 7. Interactive examples (ganja.js) methodology

The book will include working geometric-algebra demos (spacetime algebra, rotors, conformal
models) authored with **ganja.js** (Steven De Keninck's GA-for-JavaScript library; supports any
signature incl. spacetime algebra R(1,3), operator-overloaded GA notation, and interactive
SVG/Canvas/WebGL graphics). These belong in the **HTML edition**; the PDF gets a static fallback.

Two tiers — start simple, graduate when needed. The chapter only ever contains a one-line
reference, so migration is trivial.

### Tier 1 — inline include (start here)
`examples/<slug>/_widget.qmd` holds both editions via conditional content:

```markdown
::: {.content-visible when-format="html"}
```{=html}
<div id="ex-spacetime-rotor" class="ga-widget"></div>
<script type="module">
  import Algebra from "/assets/js/ganja.js";
  Algebra(1,3,0,()=>{ /* render into #ex-spacetime-rotor */ });
</script>
```
:::

::: {.content-visible when-format="pdf"}
![Spacetime rotor. *(Interactive version online.)*](still.png){#fig-spacetime-rotor}
:::
```

Placed in a chapter with:

```markdown
{{< include ../examples/spacetime-rotor/_widget.qmd >}}
```

### Tier 2 — standalone chunk + iframe (the author's "site of chunks" vision)
Each example is a standalone, independently publishable page (`examples/<slug>/index.qmd`)
deployed to a site. The HTML book embeds it inline via `<iframe>`; the PDF links to the same
published URL. A single base URL lives in `_quarto.yml`:

```yaml
example-base-url: "https://examples.evertumbook.org"   # placeholder
```

A small **Lua shortcode** (`_extensions/ga-example/`) renders per format:
- HTML -> `<iframe src="{base}/<slug>/" class="ga-embed" loading="lazy"></iframe>`
- PDF  -> static `still.png` + a hyperlink/QR to `{base}/<slug>/`

Used inline as: `{{< ga-example spacetime-rotor >}}`.

Benefits: the chunks *are* the website (single source); inline in HTML; centralized PDF links
that update from one variable; iframe isolation prevents CSS/JS collisions. Cost: iframes need
a defined height (small resize script) and are sandboxed if cross-origin.

**Implementation notes:** vendor ganja.js locally under `assets/js/` (version-pinned, offline);
add `resources: [assets]` so it's copied to output; give every widget a **unique DOM id**; use
`{ojs}` cells instead of raw `<script>` only if you need Quarto-native reactive controls (sliders
driving the figure) — ganja's own graphics are already interactive.

External-animation escape hatch: if an animation is heavy or you want it fully decoupled, host it
once at a stable, versioned URL (e.g. `.../anim/<name>/v1`) and link from both editions; the PDF
gets a still + QR. Keeps the manuscript lean.

---

## 8. Build & preview workflow

### One-time install (macOS)
```bash
# Quarto CLI
brew install quarto                 # or download from quarto.org
# LaTeX for PDF (lightweight); Quarto can also manage this:
quarto install tinytex
# Verify
quarto --version && quarto check
```
- ganja.js: download a pinned copy into `assets/js/ganja.js`.
- Optional in-editor preview extension: **Markdown Preview Enhanced** (Open VSX, installable in
  Cursor) with its **Pandoc parser** mode enabled — or just use Quarto's own preview.

### Daily loop
```bash
quarto preview          # live, math/crossref-accurate preview while writing
quarto render           # build all configured formats
quarto render --to html # or a single format
quarto render --to pdf
```
- Treat the **rendered output as ground truth**; any in-editor preview is an approximation.
- Add `docs/_book/` or the configured `output-dir` to `.gitignore` (don't commit build output).

### Draft vs full builds (token + speed lever)
Use **Quarto profiles** (e.g. `_quarto-draft.yml`) to build a fast, widgets-off draft edition
vs the full interactive edition.

---

## 9. Version control & revision methodology

- `git init` at the project root; commit early and often. Small files keep diffs legible.
- Suggested commit convention: `feat:` new section/content, `edit:` revision to existing prose,
  `fix:` correction, `build:` tooling/config, `import:` bringing in existing material.
- Use branches for large restructures or speculative drafts.
- The git history *is* the revision record the author asked for — reversible, diffable, free.

---

## 10. Cursor / AI collaboration conventions

- **Single canonical editor:** Cursor. Do not run a second editor (e.g. Typora) on the same
  files — it causes buffer-staleness/clobbering and WYSIWYG editors hide/garble the structural
  markup (labels, raw LaTeX) this book depends on. Use Quarto preview for "reading" views.
- **`.cursor/rules/book-style.mdc`** encodes voice, tense, POV, terminology, notation
  conventions, and "things to never do," so consistency holds across chapters without re-prompting.
- **Plan mode** for big/cross-cutting work (new parts, restructures, methodology changes);
  agent mode for focused writing/edits.
- Keep prompts scoped to the section at hand plus the knowledge spine (Section 11).

---

## 11. Protecting context size & token burn (REQUESTED — read carefully)

As the book grows and the cosmology concepts multiply, the single biggest risk is loading too
much into context. The architecture is explicitly designed to avoid that. Rules of thumb:

### 11.1 The "knowledge spine" — load this, not the whole book
Maintain three small, authoritative files that summarize the book so the AI rarely needs to read
full chapters:

1. **`outline.md`** — the master outline: parts, chapters, sections, and a one-line synopsis of
   each. The map of the whole book in a few hundred lines.
2. **`glossary.md`** — the **canonical definitions** of every coined term and cosmology concept
   (this book invents many). Each entry: term, one-paragraph definition, and the `@sec-` where
   it's introduced. This is the single source of truth for concept meaning.
3. **`notation.md`** — the symbol/notation table (every symbol, meaning, units). Prevents
   notation drift across chapters.

A normal editing session loads: **spine (these 3) + the one section file being edited + any
section directly cross-referenced.** Almost never the whole manuscript.

### 11.2 Modular files = bounded context
One section per file means the working set is naturally small. Resist the urge to merge sections
into big files "for convenience" — it directly inflates token usage.

### 11.3 Retrieve, don't slurp
- Use **search** (`Grep`, semantic search) to find the specific passage, then open only that
  file/range — do **not** read whole chapters to "get oriented."
- For broad exploration ("where do we discuss entropy bounds?"), prefer a **read-only explore
  subagent** that returns a short answer with file:line pointers, rather than pulling many files
  into the main context.

### 11.4 Concept index / cross-reference map
Keep a lightweight **concept index** (could live in `glossary.md`) mapping each concept to the
sections that define and use it. This lets the AI jump straight to the relevant 1-2 files instead
of scanning. Because Quarto cross-refs are global, the `@sec-`/`@eq-` IDs double as stable
addresses.

### 11.5 State/handoff notes for long efforts
For multi-session work, keep a short `WORKLOG.md` (current focus, open threads, decisions) so a
fresh instance reloads *context-as-summary* instead of re-reading chats or chapters.

### 11.6 Build-time, not context-time, assembly
Never paste multiple chapters together to "check cross-references" — that's what `quarto render`
is for. Cross-references resolve at build; verify them in the rendered output, not by loading
files into the prompt.

### 11.7 Draft profile for cheap iteration
Use the draft Quarto profile (widgets-off, maybe fewer formats) for fast rebuilds during heavy
writing; reserve full interactive builds for milestones.

### 11.8 Keep coined-term definitions DRY
Define each concept **once** in `glossary.md` and cross-reference it; do not re-explain concepts
inline in multiple chapters. This shrinks both the book and the context needed to stay consistent.

---

## 12. Open decisions (confirm with the author before scaffolding)

1. **Structure depth:** Part -> Chapter -> Section, or flat Chapter -> Section to start?
   (Recommendation: start flat, add Parts later — `_quarto.yml` change only.)
2. **Existing material format:** Word/Docs, LaTeX, plain text/Markdown, PDFs, or a mix? This
   drives the import strategy (e.g. Pandoc can convert most into Markdown/`.qmd`).
3. **Geometric algebra centrality:** is GA (spacetime algebra, rotors, conformal) a *core*
   mathematical framework? If so, set notation conventions in `notation.md` / `.cursor/rules`
   up front, and plan ganja.js infrastructure early (Section 7).
4. **Interactive-example hosting:** Tier 1 (inline) only for now, or stand up the Tier 2
   site-of-chunks early? Need the eventual base URL / hosting choice.
5. **Output targets & audience:** confirm HTML + PDF as primary; is EPUB wanted? Is there a
   publisher who will demand a specific format (e.g. `.tex`, JATS, DocBook)? Pandoc can export
   those downstream if so.
6. **Title / working title** for `_quarto.yml` (currently placeholder "The Evertum Hypothesis").

---

## 13. Quick-start checklist for the next instance

1. Read this document and [docs/example-chapter.qmd](example-chapter.qmd).
2. Confirm the **Open Decisions** (Section 12) with the author.
3. `git init` and create `.gitignore` (ignore the render output dir).
4. Scaffold: `_quarto.yml`, `metadata.yml`, `index.qmd`, `references.bib`, the
   `chapters/ sections/ figures/ assets/ examples/ tex/` tree, and `.cursor/rules/book-style.mdc`.
5. Create the **knowledge spine**: `outline.md`, `glossary.md`, `notation.md` (Section 11).
6. Use [docs/example-chapter.qmd](example-chapter.qmd) (already in Quarto syntax) as the living
   style exemplar and/or the seed for the first real chapter.
7. Verify the toolchain: `quarto check`, then `quarto preview`.
8. Begin importing existing material section-by-section, defining each new concept once in the
   glossary as you go.
