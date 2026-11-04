Completed the A2 evidence-gathering run without using Consensus or modifying any `.tex` files.

Start: `19:03:31 -05`  
Final validation: `19:09:13 -05`

Outputs:

- [candidates.csv](/home/angela/dev/projects/iot-project-unmsm/article/research/candidates.csv) — 28 candidates
- [comparison.csv](/home/angela/dev/projects/iot-project-unmsm/article/research/comparison.csv) — 12 included papers, abstract-only evidence
- [references.bib](/home/angela/dev/projects/iot-project-unmsm/article/references.bib) — 12 DOI-verified entries
- [limitations-evidence.md](/home/angela/dev/projects/iot-project-unmsm/article/research/limitations-evidence.md)
- [peru-energy-stats.md](/home/angela/dev/projects/iot-project-unmsm/article/research/peru-energy-stats.md) — four official statistics with BibTeX-ready entries

Candidate counts by query group:

- IoT monitoring systems: 4
- Low-cost smart meters: 3
- Latency, MQTT, reliability: 8
- Consumption patterns/peak demand: 2
- Calibration/reference validation: 3
- Nonlinear/inductive loads: 1
- Anomaly detection: 7

Included literature:

- Sousa 2023 — low-cost ESP32 wireless meter with power-quality capability.
- Garcés 2025 — real-time IoT meter with noninvasive sensing and manual calibration.
- Munoz 2024 — three-phase meter, power quality, multiple loads, below-1% absolute error.
- de Sousa 2026 — ESP32-S3, MQTT/InfluxDB/Grafana backend, reference validation, and measured latency.
- Cardenas Magaña 2026 — ESP32/PZEM-004T tested with calibrated reference and inductive, resistive, and electronic loads.
- Kizonde 2023 — online/offline building node benchmarked against a DL10065N instrument.
- Nizam 2024 — ESP32, Modbus, ESP-NOW, MQTT, ThingsBoard, and consistent five-second updates.
- Khan 2021 — experimental MQTT/CoAP NB-IoT packet-loss and signal-performance evaluation.
- Rohman 2025 — three-phase PZEM-016 generator monitoring with internet-independent ESP32 access-point mode.
- Oprea 2021 — smart-meter anomaly analysis with reported accuracy, precision, and F1.
- Zheng 2023 — interpretable dynamic-threshold detection for industrial energy consumption.
- Rind 2023 — recent, highly cited IoT smart-meter review.

Selection-rule coverage:

- Complete low-cost systems: at least 8
- Reference-instrument validation: 4
- Numerical measurement errors: 3
- Latency, delivery, or communication reliability: 3
- Pattern/anomaly analysis: 3
- Recent review: 1
- All included DOIs resolved through Crossref.

Unverified items:

- No Crossref DOI was located for the seed citation attributed to Marin et al.
- The Cardenas Magaña DOI missing from the seed file was found and verified as `10.53375/ijecer.2026.527`.
- No full texts were used; `NR` marks details absent from abstracts.

The strongest supported gap is narrower than the original limitation list: the literature contains accurate meters, offline-capable systems, latency studies, and anomaly methods, but these capabilities are generally demonstrated separately. None of the selected abstracts jointly reports calibrated accuracy, quantified end-to-end latency, controlled outage recovery, and consumption-pattern/anomaly analysis on one low-cost prototype. Conversely, broad claims that prior systems universally omit power factor, depend on continuous connectivity, or only test simple loads are contradicted by the selected literature.