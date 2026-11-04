# Evidence for proposed prior-work limitations

Scope: evidence below is limited to the selected papers' abstracts. An abstract's silence is recorded as `NR`, not treated as proof that the full paper omitted an item.

## Insufficient validation against calibrated instruments

- **Rohman et al. (2025)** reports tests with a phone charger, laptop charger, and no load, but names no reference instrument or numerical error in the abstract.
- Counterevidence: **de Sousa et al. (2026)** explicitly compares current with a “CW500 reference power analyzer” and reports MAE 0.23 A and MAPE 4.56%; **Cardenas Magaña et al. (2026)** compares against a calibrated commercial reference and reports MAE values of 0.31 V, 0.008 A, and 1.16 W; **Kizonde et al. (2023)** benchmarks against a DL10065N power measuring unit; **Munoz et al. (2024)** reports absolute error below 1% for all measurements. Therefore the defensible claim is that validation is *inconsistent*, not universally absent.

## Simple loads only

- **Rohman et al. (2025)** uses “cell phone charger, laptop charger, and no load”; this is a narrow set and does not establish performance across inductive/nonlinear laboratory loads.
- **Kizonde et al. (2023)** says “various home appliances” but does not identify them in the abstract; it additionally reports poor linearity for low-power appliances and considerable bias for high-power appliances.
- Counterevidence: **Cardenas Magaña et al. (2026)** explicitly tests inductive, resistive, and electronic household loads; **Munoz et al. (2024)** states validation with different loads and real-world deployment. A broad claim that all prior work uses simple loads is not supported.

## No power-factor analysis

- **Not supported as a general limitation.** **Munoz et al. (2024)** explicitly measures power factor, reactive/apparent power, harmonics, and THD. The other selected prototype abstracts generally do not say whether power factor was computed (`NR`), which is not enough to prove omission.

## Dependence on continuous connectivity

- **Not supported as a general limitation.** **Kizonde et al. (2023)** explicitly includes online and offline storage, while **Rohman et al. (2025)** uses an ESP32 access-point web server “eliminating the need for internet access.”
- The remaining system abstracts do not report controlled outage/recovery tests. This supports a narrower gap in *evaluated recovery behavior*, not a claim that every system requires continuous connectivity.

## Little latency evaluation

- Strong abstract-level support: only **de Sousa et al. (2026)** gives end-to-end components (gateway 1.53–12.8 ms; server 203.63–290.86 ms). **Nizam et al. (2024)** reports consistent 5-second updates but no latency distribution, and **Khan and Pirak (2021)** evaluates signal quality, registration, and packet loss but its abstract gives no latency value.
- The other selected system abstracts report no latency figures (`NR`). Thus “limited and nonuniform latency evaluation” is supported; “no latency evaluation exists” is contradicted.

## No measurement-uncertainty analysis

- **Not established from abstracts.** None of the 12 selected abstracts reports a measurement-uncertainty budget, traceability chain, confidence interval, or expanded uncertainty. Several report error metrics, which are not uncertainty analyses. Because full texts were deliberately not inspected, phrase this as “uncertainty analysis was not reported in the reviewed abstracts,” not as a categorical full-paper omission.

## Limited anomaly identification

- Counterevidence: **Oprea et al. (2021)** reports a hybrid anomaly pipeline with accuracy 90%, precision 0.875, and F1 0.894; **Zheng et al. (2023)** detects volatility and surge with dynamic thresholds and reports P/R/F1 of 72.1%/80.1%/73.1%.
- The limitation is supportable only for the *meter prototypes*: their abstracts generally provide monitoring and occasional threshold alerts/theft detection, but do not evaluate general anomalous-consumption behavior. Do not claim that the wider literature lacks anomaly-identification methods.

## Synthesis for the research gap

The selected literature supports a narrower, defensible gap: complete low-cost meters, metrological validation, communication performance, offline operation, and anomaly/pattern analysis are usually studied in separate works. Within these abstracts, no single work jointly demonstrates calibrated electrical accuracy, quantified end-to-end latency, controlled outage recovery, and consumption-pattern/anomaly analysis on the same low-cost prototype. This is an inference from the selected corpus, not proof of global novelty.
