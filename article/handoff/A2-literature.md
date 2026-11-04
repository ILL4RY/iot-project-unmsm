# Codex handoff — A2 literature research for the article Introduction

You are doing the research part of step A2 for an IEEE conference paper (written
in Spanish later, by someone else). Your job is **evidence gathering only**: find,
verify, and structure literature and statistics. Do **not** write or edit any
`.tex` file and do not edit files outside the outputs listed below.

## Project context (for judging relevance)

Academic IoT prototype (UNMSM, Peru). One ESP32 node monitors one controlled
single-phase 220 V / 60 Hz laboratory load. Custom front end: ADS131M02
simultaneous-sampling ADC at 4 kS/s, SCT-013-020 current transformer, isolated
voltage sensing. It computes RMS voltage, RMS current, active power (mean of v·i,
phase-corrected) and active energy; publishes 5-second summaries over MQTT 5 with
mutual TLS, QoS 1 plus application ACK, and an offline FRAM queue; stores in
InfluxDB; shows a Grafana dashboard; raises local and remote threshold alerts.
Validation is against a reference meter (≤1% uncertainty), with a 5% accuracy
target, 95% of updates within 10 s, and 99% recovery after a 10-minute outage.
Power factor, anomaly detection, and forecasting are **not** claimed.
Details: `specs/001-energy-monitoring-system/spec.md` and `plan.md`.

The paper's title: *Design, Implementation, and Experimental Evaluation of an
IoT-Based System for Real-Time Electrical Energy Consumption Monitoring and
Analysis*. The Introduction requirements are in `article/SPEC.md` § I
(Antecedentes, Limitaciones, Brecha).

## Time budget (hard limit: 15 minutes, overrides any target below)

The run is killed at 15 minutes. Check the clock with `date` and follow these phases:

- **0–6 min, search**: about 2 queries per group on OpenAlex (use web search only
  to fill gaps). Aim for **~25 candidates**, not 40.
- **6–10 min, verify**: check DOIs with Crossref and fetch BibTeX in one batched
  shell loop. Select **12** papers (up to 15 only if time allows).
- **10–13 min, write**: fill `comparison.csv` from the **abstracts only** (put `NR`
  where a value isn't given; do not read full texts). Then write
  `limitations-evidence.md` and 3 Peru statistics.
- **13–15 min**: write the final summary.

Write each output file as soon as its data exists, not all at the end, so partial
results survive a timeout. If time runs short, drop the Peru statistics first,
then `limitations-evidence.md`. Never drop `references.bib` or `comparison.csv`.

## Tools

- Use your MCP connectors: **openalex**, **consensus**, **arxiv**, **dblp**, and
  live web search.
- Use network access (`curl`) for DOI verification and BibTeX:
  - Crossref: `https://api.crossref.org/works/<DOI>`
  - BibTeX: `curl -sL -H "Accept: application/x-bibtex" https://doi.org/<DOI>`
  - OpenAlex abstracts: `https://api.openalex.org/works/doi:<DOI>`
- Start from `article/research/seed-candidates.md` (Scopus/Consensus results
  already found); verify those too, and fill the missing DOIs.

## Search plan (query groups)

1. IoT-based energy monitoring systems
2. Smart energy meters (low-cost, ESP32, PZEM-004T, ADE/ATM90E/ADS1x ICs)
3. Real-time electricity consumption monitoring (latency, MQTT, reliability, data loss)
4. Energy data analytics / consumption patterns / peak demand
5. Low-cost power measurement systems with calibration or reference-meter validation
6. Electrical load monitoring with nonlinear/inductive loads
7. Energy consumption anomaly detection (threshold/statistical first, ML second)

Run several queries per group across connectors. Target ~40 candidates in total.

## Selection rules

- Publication years 2021–2026; peer-reviewed (journal or conference). arXiv only
  if no peer-reviewed version exists, and mark it as a preprint.
- Must have a DOI that resolves in Crossref. Drop anything you cannot verify.
- Select **12–15** final papers, covering at least:
  - ≥4 complete low-cost IoT monitoring systems (ESP32 or similar)
  - ≥3 with explicit validation against a reference instrument and reported error
  - ≥2 reporting latency, delivery reliability, or outage behavior
  - ≥2 on consumption-pattern analysis or anomaly detection
  - ≥1 recent review/survey
- Prefer IEEE, MDPI, Elsevier, and Springer venues and higher-citation work when relevance ties.
- Never invent, guess, or "complete" a reference, number, or quote. Use `NR`
  (not reported) when a value is not in the source.

## Outputs (all under `article/`)

1. `research/candidates.csv` — every candidate examined. Columns:
   `doi,title,first_author,year,venue,type,connector,query_group,decision,reason`
   (`decision` = include | exclude | maybe).
2. `research/comparison.csv` — one row per **included** paper. Columns:
   `bibkey,doi,setting,current_sensor,voltage_sensor,measurement_ic,mcu,quantities,sampling_or_update_rate,protocol,storage_platform,analysis_type,reference_instrument,load_types,error_metric,error_values,latency,reliability_or_outage,uncertainty_analysis,cost_usd,evidence_source,notes`
   (`evidence_source` = abstract | fulltext; quote numbers exactly as written).
3. `references.bib` — BibTeX for the included papers only, fetched from doi.org
   and then cleaned: keys `firstauthorYEARkeyword` (lowercase ASCII, e.g.
   `munoz2024smartmeter`), fields `author,title,journal/booktitle,year,volume,number,pages,doi`.
   Keep the existing header comment lines at the top of the file.
4. `research/limitations-evidence.md` — for each limitation in `SPEC.md` § I.C
   (insufficient validation against calibrated instruments, simple loads only,
   no power-factor analysis, dependence on continuous connectivity, little latency
   evaluation, no measurement-uncertainty analysis, limited anomaly identification):
   which included papers show it, with a short quote or exact evidence, or
   "not supported by the selected literature".
5. `research/peru-energy-stats.md` — 3–6 verifiable statistics on electricity
   consumption, demand, or metering/digitalization in Peru (prefer 2022–2026), from
   official or authoritative sources only (MINEM, COES-SINAC, Osinergmin, INEI,
   IEA, World Bank). For each one: exact figure, unit, year, organization, document
   title, URL, and access date. Add a BibTeX-ready `@misc`/`@techreport` entry for
   each source in a fenced block (do **not** add these to `references.bib`).
6. Final message: a summary covering the counts per query group, the included list
   with a one-line relevance note each, coverage of the selection rules, anything
   you could not verify, and any suggested research gap the literature supports or
   contradicts.
