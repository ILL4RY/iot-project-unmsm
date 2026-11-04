# Phase 0 Research: Energy Monitoring System

**Feature**: `001-energy-monitoring-system`  
**Date**: 2026-10-14  
**Status**: Complete for planning

This record resolves the technical unknowns in the feature specification. It is
not an electrical installation approval. The prototype remains a controlled,
single-phase laboratory instrument whose mains-facing design and assembly require
review by a qualified person before energization.

## R-001 Measurement range and sensing architecture

**Decision**: Claim the MVP accuracy target only over `180–250 V RMS`,
`0.25–10.0 A RMS`, `0–2.5 kW` active power, and `55–65 Hz`. Protect the test
path at 10 A. Use a `SCT-013-020` 20 A/1 V current transformer and either a
certified isolated 220 V-to-low-voltage AC adapter (preferred) or a bare
`ZMPT101B` on a purpose-built, safety-reviewed PCB. Do not use a common exposed
ZMPT101B/LM358 module. Values outside the calibrated range may be observed but
MUST carry `out_of_range` and are outside the 5% claim.

**Rationale**: This range covers representative laboratory loads while retaining
sensor headroom. The SCT variant includes its burden, avoiding the open-secondary
risk of a current-output CT. A certified adapter reduces custom mains-design risk;
the ZMPT option is retained because it is obtainable and galvanically isolated,
but component ratings, creepage, fusing, enclosure, and assembly still determine
system safety.

**Alternatives rejected**:

- `SCT-013-000`: external burden and 100 A range add avoidable risk and reduce
  useful resolution for a 10 A prototype.
- `ACS712`: conductor insertion and PCB mains routing are unnecessary for a
  monitoring-only clamp-on prototype.
- Generic ZMPT101B module: component voltage ratings, layout, creepage, and output
  compatibility are not sufficiently controlled.

**Sources**: [Peru supply description](https://www.gob.pe/que/29306-informacion-general-sobre-lima),
[YHDC SCT-013 product information](https://en.yhdc.com/product/SCT013-401.html),
[SCT-013 series data](https://www.tinytronics.nl/product_files/005965_Datasheet_SCT013.pdf),
[ZMPT101B data](https://img.ozdisan.com/ETicaret_Dosya/625106_302431.pdf).

## R-002 Isolation, protection, and analog conditioning

**Decision**: Use an `ADS131M02` external simultaneous-sampling ADC in
high-resolution mode at 4,000 sample pairs/s. Connect voltage to AIN0 at PGA 2
and current to AIN1 at PGA 1. Each channel uses the datasheet differential
anti-alias network (50 ohms in each leg and 4.7 nF differential) plus the
manufacturer's external clamp topology. For the ZMPT option, use four series,
flameproof, voltage-rated approximately 27.4 kohm/0.5 W primary resistors
(109.6 kohm total, split across both leads) and a 100 ohm secondary burden.

The energized prototype requires an upstream 30 mA RCD/GFCI, two-pole 10 A
overcurrent protection, separately fused voltage-sense branch, certified isolated
low-voltage supply, touch-safe flame-retardant enclosure, strain relief, covered
terminals, and guarded mains PCB zone. No mains wiring may use solderless
breadboard or flying leads. A qualified person MUST review the schematic, parts'
ratings, creepage/clearance, slots, conductor sizing, enclosure, and test setup
before energization.

**Rationale**: Simultaneous voltage/current conversion removes multiplexing skew
from active-power calculation. The ADC provides differential inputs, programmable
gain, phase calibration support, and SPI CRC. Isolation-component ratings alone do
not certify the assembled prototype.

**Alternative rejected**: ESP32's internal SAR ADC is a fallback only. It lacks
simultaneous channels and introduces device-dependent nonlinearity and phase risk
that make the active-power target harder to defend.

**Sources**: [ADS131M02 product page](https://www.ti.com/product/ADS131M02),
[ADS131M02 datasheet](https://www.ti.com/lit/ds/symlink/ads131m02.pdf),
[ADS131M0x input protection](https://www.ti.com/lit/an/sbaa506/sbaa506.pdf),
[ESP32 datasheet](https://documentation.espressif.com/esp32_datasheet_en.html).

## R-003 Sampling, quantities, and energy continuity

**Decision**: Acquire synchronized voltage/current pairs continuously at 4 kS/s.
Remove offset and apply calibrated gain and fractional phase correction. Calculate
1-second internal blocks for quality and local-alert evaluation; publish a
5-second summary accumulated from all valid sample pairs.

For calibrated instantaneous values `v[n]` and `i[n]`:

```text
voltage_rms_V = sqrt(sum(v[n]^2) / N)
current_rms_A = sqrt(sum(i[n]^2) / N)
active_power_W = sum(v[n] * i[n]) / N
delta_energy_Wh = active_power_W * monotonic_elapsed_seconds / 3600
active_energy_import_Wh += max(active_power_W, 0) * dt / 3600
```

Active power is signed internally to detect reversed CT polarity. Sustained
negative power raises `polarity_reversed`; it is not silently converted into
consumption. Energy integration uses monotonic time and is never adjusted by a
wall-clock correction. Energy checkpoints use double-buffered records with
sequence number and CRC in `MB85RC256V` FRAM every 5 seconds. Pending telemetry
uses compact binary FRAM records and retains at least 256 summaries (more than 21
minutes), sufficient for the specified 10-minute outage test.

**Rationale**: High-rate paired samples are necessary to distinguish active power
from apparent power. One-second blocks meet local warning latency while 5-second
messages limit network and storage load. FRAM avoids rewriting internal flash on
every checkpoint.

**Alternative rejected**: `Vrms * Irms` is apparent power and MUST NOT be labeled
active power. Wall-clock differences are unsafe for integration because SNTP
corrections can jump.

**Sources**: [ADS131M02 datasheet](https://www.ti.com/lit/ds/symlink/ads131m02.pdf),
[MB85RC256V datasheet](https://www.fujitsu.com/us/Images/MB85RC256V-DS501-00017-3v0-E.pdf),
[ESP-IDF system time](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/system/system_time.html).

## R-004 Calibration and measurement evidence

**Decision**: Calibrate with an identifiable reference meter whose documented
uncertainty is at most 1%. Record hardware/firmware/calibration revisions,
operator, date, ambient conditions, wiring, sensor identity, ADC configuration,
reference evidence, and load identity. Warm up for 10 minutes; measure zero for
60 seconds; calibrate voltage and current gain; calibrate phase with a stable
non-unity-power-factor load; freeze coefficients before acceptance testing.

Acceptance uses a separate dataset containing low (about 50–100 W), medium
(400–800 W), high (1.5–2.0 kW) resistive loads and one stable inductive or
nonlinear load. Collect at least five minutes per power point and 15 minutes for
energy. Each required quantity at every accepted point—not merely the mean—must
be within 5% of reference. No-load behavior uses an absolute noise threshold.
Preserve reference readings, raw waveform excerpts, calculated blocks, summaries,
coefficients, scripts, hashes, and pass/fail tables.

**Rationale**: Separate calibration and validation data prevent circular evidence.
The phase-sensitive load is needed to validate active power rather than only
unity-power-factor apparent power.

**Sources**: [ADE7953 calibration guidance](https://www.analog.com/cn/resources/app-notes/an-1118.html),
[ADE7953 product information](https://www.analog.com/en/products/ade7953.html).

## R-005 Clock and timestamp model

**Decision**: SNTP UTC is authoritative, with battery-backed `DS3231` holdover
after cold restart. Every measurement states `sntp`, `holdover`, or
`unsynchronized`, plus the last synchronization age. The observation timestamp is
the end of the measurement window in RFC 3339 UTC with milliseconds. The server
adds `received_at` and `stored_at`; ordering and deduplication use observation
time together with `boot_id`, `sequence`, and immutable `message_id`.

**Rationale**: The three timestamps separate physical observation, network delay,
and persistence delay. Explicit clock quality prevents an unsynchronized value
from appearing trustworthy.

**Sources**: [RFC 3339](https://www.rfc-editor.org/info/rfc3339/),
[DS3231](https://www.analog.com/en/products/ds3231.html),
[ESP-IDF system time](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/system/system_time.html).

## R-006 Telemetry protocol and recovery

**Decision**: Use MQTT 5 over TLS on port 8883. Topics are:

```text
unmsm/energy/v1/nodes/{node_id}/measurements
unmsm/energy/v1/nodes/{node_id}/events
unmsm/energy/v1/nodes/{node_id}/status
unmsm/energy/v1/nodes/{node_id}/config
unmsm/energy/v1/nodes/{node_id}/acks/measurements
```

All messages use QoS 1. Retain only status and config; never retain measurements
or events. The node uses stable client identity, Clean Start false, 24-hour
session expiry, 15-second keepalive, retained offline last will, and 24-hour
message expiry. JSON payloads follow the versioned Draft 2020-12 contracts under
`contracts/` and stay below 16 KiB.

The node persists a record before publish and deletes it only after an application
ACK (`accepted`, `duplicate`, or terminal `rejected`) received after storage. The
server enforces unique `message_id`; reconnect drains oldest first at at least two
records/s with at most ten in flight. This gives at-least-once transport and an
exactly-once storage effect. A rejected record is retained in a bounded local
diagnostic/dead-letter area and raises health state rather than retrying forever.

**Rationale**: MQTT QoS 1 alone can duplicate delivery and its broker ACK does not
mean the database committed. The explicit application ACK makes the recovery
criterion observable and testable.

**Alternatives rejected**:

- QoS 0: cannot support recoverable delivery.
- QoS 2 alone: still does not prove application persistence and adds protocol
  complexity.
- Retained telemetry: makes stale data look current and is unnecessary with
  storage.

**Sources**: [MQTT 5.0 standard](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html),
[ESP-MQTT documentation](https://docs.espressif.com/projects/esp-idf/en/v5.4/esp32/api-reference/protocols/mqtt.html),
[JSON](https://www.rfc-editor.org/info/rfc8259/),
[JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12).

## R-007 Authentication, secrets, and network exposure

**Decision**: Operate a private laboratory CA. Give each node and service its own
certificate and key. Mosquitto requires client certificates, maps certificate
identity to the MQTT username, disables anonymous access, applies per-topic ACLs,
and accepts TLS 1.2 or newer. Devices may write only their own measurements,
events, and status and read only their own config and ACK topic.

Provision node credentials through a trusted wired flashing workflow from an
ignored per-device manifest. Store them in encrypted NVS; commit only the public
CA certificate and placeholder examples. Inject server secrets at runtime from
ignored files with restrictive permissions. Logs redact credentials and tokens.

Bind MQTT only to the device/laboratory LAN and firewall it to the device subnet.
Keep ingestion and databases on an internal container network. Expose only the
dashboard HTTPS endpoint on the trusted LAN, with one authenticated operator
account. Do not use public port forwarding, UPnP, public DNS, or browser-to-MQTT.
The notification worker may make outbound HTTPS requests only.

**Rationale**: Per-principal credentials and network segmentation limit the effect
of one compromise. Runtime injection satisfies the constitution's no-secret rule.

**Sources**: [Mosquitto configuration](https://www.mosquitto.org/man/mosquitto-conf-5.html),
[ESP-IDF security overview](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/security/security.html),
[ESP-IDF flash encryption](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/security/flash-encryption.html).

## R-008 Server platform, storage, and deployment

**Decision**: Run a reproducible Docker Compose deployment on one Linux host:
Mosquitto 2.1.2, a Python 3.14.8 ingestion/alert worker, InfluxDB OSS 2.9.1 for measurements, SQLite WAL
for the ingestion ledger/configuration/alert audit/outbox, Grafana for read-only
views (Grafana OSS 13.2.3), and Caddy for authenticated LAN-only HTTPS. Pin image digests or exact
versions and Python dependencies.

The worker validates JSON Schema, checks identity/topic agreement, applies an
idempotent persistence state machine, writes time-series data, evaluates remote
threshold state, creates notification outbox rows, and sends the application ACK.
SQLite is authoritative for discrete lifecycle state; InfluxDB is authoritative
for time series. Raw five-second telemetry has a 30-day retention policy. Hourly
and daily aggregates may be retained for one year but are derived, reproducible,
and never replace raw evidence during the 30-day window.

**Rationale**: InfluxDB and Grafana match time-series visualization needs, while a
small relational ledger supplies uniqueness and auditable state transitions that
a time-series store does not model cleanly. One host fits the single-node academic
scope and keeps deployment reproducible.

**Alternatives rejected**:

- Cloud-managed stack: adds public exposure, cost, and external availability to a
  LAN-scoped MVP.
- InfluxDB alone: poor fit for alert transitions, outbox transactions, and a
  durable idempotency ledger.
- PostgreSQL plus a time-series extension: capable but operationally heavier than
  required for one node.

**Sources**: [InfluxDB retention](https://docs.influxdata.com/influxdb/v2/reference/internals/data-retention/),
[InfluxDB releases](https://github.com/influxdata/influxdb/releases),
[Mosquitto releases](https://mosquitto.org/blog/),
[Python 3.14.8 release](https://www.python.org/downloads/release/python-3148/),
[Grafana OSS 13.2.3](https://grafana.com/grafana/download/13.2.3?edition=oss),
[Grafana provisioning](https://grafana.com/docs/grafana/latest/administration/provisioning/),
[Grafana alert provisioning](https://grafana.com/docs/grafana/latest/alerting/set-up/provision-alerting-resources/file-provisioning/).

## R-009 Dashboard, alerts, and health

**Decision**: Provision dashboard and datasource definitions from version control.
The main dashboard contains current V/I/P/import-energy cards with units and
quality, last-valid-update and clock status, active-power trend, voltage/current
trend, hourly/daily energy, quality-gap panel, active/resolved alert table, and
ingestion/recovery health. Never render a stale last value as live.

The node evaluates 1-second power blocks with state
`normal -> pending -> firing -> resolved`, using a default 10-second confirmation
and explicit 95% clear threshold/duration. LED/buzzer activates only in `firing`.
The server alert is `open -> acknowledged -> resolved`; acknowledgment records
operator receipt and does not electrically clear the condition. Every transition
is audited. Notifications use a transactional outbox and stable `alert_id` as the
idempotency key.

Publish retained status and a 30-second heartbeat. Mark telemetry stale at 15
seconds and node offline at 30 seconds or on last will. Expose structured logs and
metrics for acquisition gaps, invalid windows, queue depth, clock state, MQTT
reconnects, validation rejects, duplicates, ingest/storage latency, notification
failures, datasource health, and dashboard age.

**Rationale**: Separate physical condition, user acknowledgment, delivery, and
component health states prevent misleading alert semantics and local/remote
coupling.

## R-010 Future analytics dataset and evaluation

**Decision**: No anomaly or forecast model executes in the MVP. The MVP collects
validated five-second telemetry, quality flags, calibration revision, clock
quality, threshold events, and controlled experiment labels. A later feature
exports immutable Parquet datasets plus a manifest containing source query,
interval, exclusions, schema/calibration versions, hashes, code revision, and
environment lockfile.

For anomaly research, define event labels and tolerance windows before training;
split chronologically by complete experiment/day with no window leakage. Compare
against simple threshold and rolling robust-z baselines. Report precision, recall,
F1, false alerts/day, and detection delay on held-out data, stratified by load and
quality. For forecasting, compare against last-value and seasonal-naive baselines
using rolling-origin backtesting; report MAE, RMSE, and WAPE with prediction
horizon and coverage. Exclude invalid/unsynchronized data by declared rules rather
than silent cleaning. Record random seeds where applicable.

**Rationale**: Collection now avoids an unusable historical dataset, but model
claims require a separate specification, adequate history, held-out evaluation,
and explicit promotion criteria.

**Alternative rejected**: Random row splits leak adjacent time windows and inflate
results; training on unvalidated data makes an apparently accurate model inherit
measurement faults.

## R-011 Software baseline and verification

**Decision**: Firmware uses C/C++ with ESP-IDF 5.4 LTS and its Unity test support.
Server code uses Python 3.14.8 with Pydantic/JSON Schema validation, Paho MQTT,
InfluxDB client, SQLAlchemy/Alembic, Prometheus client, and pytest. Infrastructure
is Docker Compose with pinned container images. Exact patch versions and hashes
are fixed in implementation lockfiles.

Verification layers are calculation unit tests with recorded waveform fixtures,
JSON Schema contract tests, broker/ingestion/storage integration tests, outage and
clock-fault injection, dashboard provisioning checks, secret scanning, and
reference-meter hardware acceptance. Hardware-in-the-loop tests are separately
marked because they require qualified supervision.

**Rationale**: These tools have stable support for the selected device, protocol,
storage, and reproducible local deployment, while keeping the repository split
along independently testable boundaries.
