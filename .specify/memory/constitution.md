<!--
Sync Impact Report
- Version change: unratified template -> 1.0.0
- Modified principles:
  - Placeholder Principle 1 -> I. Electrical Safety Before Convenience
  - Placeholder Principle 2 -> II. Measurement Integrity and Evidence
  - Placeholder Principle 3 -> III. Monitoring MVP Before Optional Complexity
  - Placeholder Principle 4 -> IV. Security and Secret Hygiene
  - Placeholder Principle 5 -> V. Contract-Tested Boundaries
- Added principles:
  - VI. Reproducible Experiments and Models
  - VII. Resilience and Observability
- Added sections:
  - Engineering and Operational Constraints
  - Development Workflow and Quality Gates
- Removed sections: none
- Follow-up TODOs: none
-->

# Intelligent Electrical Energy Monitoring System Constitution

## Core Principles

### I. Electrical Safety Before Convenience (NON-NEGOTIABLE)

All work involving mains voltage MUST identify voltage and current ranges, isolation
boundaries, overcurrent protection, enclosure requirements, and safe assembly and test
procedures before hardware is energized. Mains-connected assembly or review MUST involve a
qualified person. Testing MUST begin with simulation or an isolated low-energy setup wherever
practical. Automatic load switching is prohibited in the monitoring MVP; any future actuator
MUST have a separate specification, hazard analysis, fail-safe behavior, and manual override.
Schedule, cost, demonstration value, and convenience MUST NOT override an unresolved safety
risk.

**Rationale**: The system observes hazardous electrical energy. A monitoring feature is not
valuable if it exposes people or property to avoidable risk.

### II. Measurement Integrity and Evidence

Every reported electrical quantity MUST have a documented definition, unit, measurement range,
sampling or aggregation method, timestamp basis, and quality state. Each supported sensor and
device configuration MUST be calibrated against a trusted reference meter across representative
loads, with the procedure, raw results, error calculation, and acceptance decision preserved.
Invalid, stale, saturated, or uncertain readings MUST be labeled and MUST NOT be presented as
trustworthy measurements. The project MUST NOT claim active power, reactive power, power factor,
energy accuracy, or billing-grade behavior unless the instrumentation and reference evidence
directly validate that claim.

**Rationale**: Dashboards, alerts, and analytics inherit the quality of their measurements;
unverified numbers create false confidence rather than insight.

### III. Monitoring MVP Before Optional Complexity

The first releasable system MUST prove one complete vertical path: safe measurement of one
controlled single-phase circuit, local validation, reliable telemetry, durable history, dashboard
visibility, and threshold alerts. Optional environmental sensors, automatic load switching,
multi-node or three-phase support, anomaly models, forecasts, and autonomous recommendations
MUST NOT enter the MVP unless an approved specification changes its scope. Historical analytics
MUST be added only after the telemetry and data-quality acceptance criteria pass; forecasting
MUST be added only after data-sufficiency and baseline-accuracy gates pass.

**Rationale**: Staged delivery keeps the academic prototype testable and prevents optional
features from hiding failures in the core monitoring path.

### IV. Security and Secret Hygiene

Credentials, private keys, Wi-Fi passwords, notification tokens, private endpoints, and real
personal data MUST NOT be committed to version control, fixtures, screenshots, logs, or reports.
Tracked example configuration MUST contain placeholders only. Device and service identities MUST
be authenticated, data in transit MUST be protected from unauthorized disclosure or alteration,
and permissions MUST follow least privilege. Logs MUST redact secrets. Any accidental exposure
MUST trigger credential revocation, history-remediation assessment, and a documented incident
response before normal development resumes.

**Rationale**: An IoT system crosses physical and network trust boundaries; one leaked secret can
compromise the device, telemetry, infrastructure, and users.

### V. Contract-Tested Boundaries

Every system boundary MUST have a versioned, reviewable contract and an independently executable
test. This includes sensor-to-firmware inputs, firmware-to-telemetry messages,
telemetry-to-ingestion validation, storage-to-dashboard queries, and analytics-to-alert outputs.
Contracts MUST define field names, types, units, timestamps, identity, quality states, error
behavior, compatibility rules, and representative valid and invalid fixtures. A producer or
consumer change MUST NOT merge until its contract tests demonstrate compatibility or the change
includes an approved migration plan.

**Rationale**: Independently tested boundaries allow hardware, firmware, services, dashboards,
and analytics to evolve without relying on fragile end-to-end debugging.

### VI. Reproducible Experiments and Models

Every experiment and analytics result MUST be reproducible from versioned or content-addressed
inputs and a documented procedure. Dataset origin, collection interval, exclusions, labeling,
train/validation/test split, preprocessing, configuration, random seed where applicable, runtime
dependencies, evaluation metrics, and output artifacts MUST be recorded. Comparisons MUST use an
explicit baseline and held-out evaluation data. A model MUST NOT influence user-visible alerts,
forecasts, or recommendations until it meets the specification's acceptance criteria and defines
behavior for insufficient, drifting, or low-quality data.

**Rationale**: Reproducibility separates defensible engineering evidence from a one-time demo or
an irreproducible model result.

### VII. Resilience and Observability

Each component MUST expose enough status to distinguish measurement, connectivity, ingestion,
storage, analysis, and presentation failures. The design MUST define behavior for restarts,
network loss, late or duplicate messages, clock uncertainty, unavailable dependencies, and
partial data. Logs and metrics MUST be structured, timestamped, actionable, and free of secrets.
Critical recovery behavior MUST be verified through failure-injection or equivalent repeatable
tests before release.

**Rationale**: Continuous monitoring is credible only when users can tell the difference between
normal consumption, invalid data, and a broken monitoring path.

## Engineering and Operational Constraints

- The initial target is a controlled academic prototype for one indoor single-phase circuit or
  isolated test load; it MUST NOT be represented as a certified protection or utility billing
  device.
- Electrical calculations MUST distinguish sampled waveform data, RMS values, apparent power,
  active power, reactive power, power factor, and accumulated energy wherever those terms apply.
- Configuration, schemas, dashboards, infrastructure definitions, test fixtures, and experiment
  procedures MUST be versioned when they are required to reproduce a result or deployment.
- Dependencies MUST be pinned or locked, and the repository MUST document a repeatable local
  setup and verification path.
- Stored and transmitted data MUST use explicit units, timezone-aware timestamps, node and circuit
  identity, sequence or deduplication information, and a quality or validity state.
- Data retention, network exposure, notification destinations, and destructive operations MUST be
  explicit configuration rather than hidden defaults.
- Real hardware tests MUST document the equipment, wiring revision, calibration revision, load
  conditions, operator, date, and reference instrument used.

## Development Workflow and Quality Gates

1. **Specify**: Each feature MUST have testable user outcomes, explicit scope, safety impact,
   assumptions, failure cases, and measurable success criteria before planning.
2. **Plan**: Each plan MUST trace requirements to architecture decisions, contracts, security
   controls, calibration or experiment procedures, and verification evidence. Irreversible or
   safety-relevant decisions MUST include a recorded rationale and rejected alternatives.
3. **Task**: Tasks MUST be dependency ordered and divided into independently verifiable vertical
   milestones. Simulation and contract fixtures MUST precede integration with mains-connected
   hardware where practical.
4. **Verify**: Changes MUST include the lowest-cost effective evidence: unit tests for calculations
   and validation, contract tests for boundaries, integration tests for services and failure
   recovery, reference-meter tests for measurement claims, and acceptance tests for user outcomes.
5. **Review**: Reviews MUST verify constitution compliance, requirement traceability, secrets
   hygiene, documentation changes, and the evidence supporting affected safety, measurement, and
   analytics claims.
6. **Release**: Formatting, static checks, unit tests, contract tests, schema validation, secret
   scanning, and relevant integration or hardware acceptance tests MUST pass. Any skipped gate
   MUST have an approved, time-bounded exception recorded under Governance.
7. **Converge**: After implementation, the project MUST compare the delivered system against the
   approved specification and append missing work rather than silently narrowing acceptance.

## Governance

This constitution is the highest project-level engineering authority. Specifications, plans,
tasks, implementation changes, experiments, and reviews MUST demonstrate compliance with it.

Amendments MUST include the proposed text, rationale, affected artifacts, compatibility or
migration impact, and approval by the project maintainers. Safety, measurement-integrity, and
secret-hygiene requirements cannot be waived through an ordinary task or review; changing them
requires a constitution amendment with explicit risk justification.

Constitution versions follow semantic versioning:

- **MAJOR**: Removes or incompatibly redefines a principle or governance obligation.
- **MINOR**: Adds a principle, section, or materially stronger obligation.
- **PATCH**: Clarifies wording without changing required behavior.

Every feature specification and implementation plan MUST include a constitution check. Every
change review MUST identify the applicable principles and evidence. A temporary exception MUST
name an owner, scope, rationale, compensating control, approval, and expiry date; expired
exceptions block release. The Sync Impact Report at the top of this file is review scratch
material and MUST be removed before committing the ratified constitution.

**Version**: 1.0.0 | **Ratified**: 2026-10-14 | **Last Amended**: 2026-10-14
