# Article (IEEE)

LaTeX sources for the paper *Design, Implementation, and Experimental Evaluation
of an IoT-Based System for Real-Time Electrical Energy Consumption Monitoring and
Analysis*. The requirements are in [SPEC.md](SPEC.md). Overall project tracking is
in the [root README](../README.md#6-article-track).

## Build

Requires TeX Live with `IEEEtran`, `latexmk`, and `bibtex`.

```sh
make            # build/main.pdf
make watch      # rebuild on save
make clean      # remove aux files, keep PDF
make distclean  # remove build/
```

The bibliography is active (BibTeX with `IEEEtran.bst`). Every entry is DOI-verified,
or for official statistics checked at the source.

## Layout

```text
article/
├── main.tex                 preamble, abstract, section includes
├── metadata.tex             title, authors, affiliation, keywords
├── sections/
│   ├── 01-introduction.tex
│   ├── 02-methodology.tex
│   ├── 03-results-discussion.tex
│   └── 04-conclusions.tex
├── figures/                 figure sources (.svg/.drawio/.py) and exported PDF/PNG
├── tables/                  tables written as .tex, generated or hand-written
├── references.bib           verified entries only
├── research/                literature evidence (candidates, comparison, stats)
├── handoff/                 self-contained Codex task prompts
├── SPEC.md                  article requirements
├── latexmkrc, Makefile
└── build/                   output (git-ignored)
```

Conventions:
- The paper is written in **Spanish** (`babel` spanish). Code comments and repo docs stay in English.
- The specs in `../specs/` are the source of truth. `SPEC.md` is guidance only, and every deviation is recorded under Decisions below.
- One sentence per line in the `.tex` files, so diffs stay readable.
- Labels use `sec:`, `fig:`, `tab:`, and `eq:` prefixes.
- Numbers in results tables come from scripts under `../experiments/evaluation/`
  and are never typed by hand. Each table states the evidence artifact it comes from.
- Figures are vector (PDF) wherever possible, and each one has its source next to it.

## Section-to-evidence map

| Paper section | Project source |
|---|---|
| II-B Architecture | `plan.md` › Component and Deployment Design |
| II-C Prototype | `plan.md` › Hardware; `research.md` R-001, R-002 |
| II-D Acquisition model | `plan.md` › Quantities; `research.md` R-003; FR-003 |
| II-E Algorithm | `plan.md` › Threshold and alert lifecycle; future analysis module |
| II-F/G Experiments, metrics | `plan.md` › Calibration procedure; SC-001–SC-006 |
| III-B Accuracy | Calibration report (milestone M4) |
| III-C Patterns | Dataset export plus analysis (M7, `experiments/`) |
| III-D Performance | Recovery and latency reports (M5, M6) |

## Status

| # | Step | Status |
|---|---|---|
| A0 | Requirements received; repo setup | ✅ 2026-10-28 |
| A1 | Resolve the decisions that block writing (AD-2 to AD-5) | ✅ 2026-10-28 |
| A2 | **Introduction**: literature search (10–15 papers, 2021–2026), `references.bib`, draft of Section I | ✅ draft 2026-10-28 (awaiting review) |
| A3 | Outline with page budget and figure/table selection | ⬜ |
| A4 | Draft Methodology (no results needed) | ⬜ |
| A5 | Experiments, then Results/Discussion (depends on M4–M7) | ⬜ |
| A6 | Conclusions and abstract | ⬜ |
| A7 | Internal review and revision | ⬜ |
| A8 | Venue template check, PDF eXpress, submission | ⬜ |

<a id="decisions"></a>
## Decisions

| # | Decision | Date |
|---|---|---|
| AD-1 | Written in LaTeX with `IEEEtran` in conference mode, built with latexmk/BibTeX | 2026-10-28 |
| AD-2 | Language: Spanish, with a Spanish title (the proposal's translation) | 2026-10-28 |
| AD-3 | Hardware follows `plan.md`/`research.md` R-001 (ESP32 + ADS131M02 + SCT-013-020 + isolated voltage sensing), not the proposal's PZEM-004T. The proposal allows "u otro módulo adecuado" | 2026-10-28 |
| AD-4 | Measured quantities follow the spec: V_RMS, I_RMS, active power, active energy. P is defined as the mean of v·i (FR-003), not V·I·PF. Power factor and frequency get no accuracy claim; they appear only as a limitation and future work. The accuracy table has no PF row | 2026-10-28 |
| AD-5 | "Analysis" means what the spec delivers: hourly/daily/weekly aggregation, peak-period identification (SC-005), and threshold alerts with confirmation that reject startup surges (FR-011, SC-004). Statistical anomaly detection and forecasting are future work (plan › Future Dataset). "Detection accuracy" is measured as correct firing/non-firing of threshold alerts under scripted loads | 2026-10-28 |
| AD-6 | Venue not chosen. Keep the IEEE conference defaults (Letter, ~6 pages) and recheck when one is chosen | 2026-10-28 (deferred) |
| AD-7 | Numbers use a decimal comma (`siunitx` `output-decimal-marker={,}`), following the Spanish/INEI convention. Write numbers with `\SI{}`/`\num{}` | 2026-10-28 |
| AD-8 | Prior-work limitations are narrowed to what the evidence supports (`research/limitations-evidence.md`): heterogeneous validation, rarely measured latency, no controlled outage-recovery evaluation, and analytics evaluated apart from the hardware. The proposal's claims of "no power-factor analysis" and "always depends on connectivity" are contradicted by the literature and are not used | 2026-10-28 |
