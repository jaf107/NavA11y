# Results and Analysis Chapter — NavA11y

This folder holds the "Results and Analysis" chapter of the thesis in two formats. Both
files carry identical content and numbers; pick whichever your thesis is written in.

- `results.md` — Markdown version. Open in any Markdown viewer, or paste into Word/Google Docs.
- `results.tex` — LaTeX version, with `tabular`, `equation`, and `figure` environments.
- `figures/` — drop the four exported figures here (see file names in `results.tex`).

## Compiling the LaTeX file

`results.tex` begins with `\chapter{...}`, so it is meant to be included in a thesis with
`\input{results.tex}` or `\include{results}`. The preamble must load these packages:

```latex
\usepackage{booktabs}   % \toprule \midrule \bottomrule
\usepackage{amsmath}    % equation environment
\usepackage{graphicx}   % \includegraphics for figures
\usepackage{siunitx}    % \SI{16}{\giga\byte}, units
```

To compile it on its own as a quick standalone PDF, wrap it:

```latex
\documentclass{report}
\usepackage{booktabs}
\usepackage{amsmath}
\usepackage{graphicx}
\usepackage{siunitx}
\begin{document}
\input{results.tex}
\end{document}
```

Then run `pdflatex` twice (the second pass resolves the table/figure cross-references):

```bash
pdflatex main.tex
pdflatex main.tex
```

## Figures to export

The four figures are referenced but not yet rendered. Export them from the data in this
chapter and the generated site reports, and save them in `figures/` with these names:

- `fig5_1_confusion_matrix` — 2×2 heat map from Table 5.1 (off-diagonal cells are zero).
- `fig5_2_violations_by_sc` — bar chart from the aggregate-by-criterion table.
- `fig5_3_violations_per_site` — horizontal bar chart from the per-site table.
- `fig5_4_report_screenshot` — screenshot of any `reports/<site>/index.html`.

## Reproducing the numbers

- Labelled dataset (D_d), per-criterion metrics:
  `node evaluation/run-gds-evaluation.mjs` → writes `evaluation/results/rq1-results.json`.
  Expected result: TP=14, TN=18, FP=0, FN=0 → 100% precision/recall/F1/accuracy.
  Note: the GDS test pages depend on their `assets/` folder (shared stylesheet + script) at
  `dataset/focus-behavior-dataset/assets/`, which is now committed with the dataset. The CSS
  that removes the focus outline lives in that stylesheet, so the assets must stay in place
  for the violations to render and the 100% result to reproduce.
- Real-world dataset (D_r), per-site violations:
  `node evaluation/run-all-benchmark.mjs`, then `node evaluation/generate-benchmark-summary.mjs`
  → writes `benchmark-results/summary.{json,csv}`.

## Open items flagged in the chapter

- The True-Positive Rate on D_r (reported as ~90% in the project README) has no stored
  per-case verification file; locate or re-run the manual verification before the bound report.
- Per-site wall-clock timing was not captured in the recorded benchmark run; re-run with
  timing enabled if a seconds-per-site figure is needed for the Resource Utilization section.
