# Phase 1 Data Model: Energy Monitoring System

**Feature**: `001-energy-monitoring-system`  
**Canonical wire units**: volts (`V`), amperes (`A`), watts (`W`), watt-hours
(`Wh`)  
**Canonical time**: RFC 3339 UTC with milliseconds and `Z`

The JSON Schemas under `contracts/` are authoritative for boundary payloads.
This document defines persistence entities, relationships, invariants, and state
transitions.

## Identity and configuration entities

### MonitoringNode

| Field | Type | Rule |
|---|---|---|
| `node_id` | string | Primary key; `^[a-z0-9][a-z0-9-]{0,31}$` |
| `display_name` | string | Operator-facing, 1–80 characters |
| `certificate_subject` | string | Unique; must map to `node_id` |
| `hardware_revision` | string | Immutable for a physical build |
| `firmware_version` | string | Last reported version |
| `calibration_revision` | string | Must reference an accepted calibration before quality can be `valid` |
| `last_seen_at` | timestamp? | Server receive time, UTC |
| `last_valid_observed_at` | timestamp? | Latest valid measurement end |
| `status` | enum | `provisioning`, `online`, `stale`, `offline`, `degraded` |

One node belongs to exactly one `Circuit` in the MVP. A certificate may authenticate
only its matching node topics.

### Circuit

| Field | Type | Rule |
|---|---|---|
| `circuit_id` | string | Primary key; same identifier rules as `node_id` |
| `node_id` | string | Unique foreign key to `MonitoringNode` |
| `display_name` | string | 1–80 characters |
| `nominal_voltage_V` | decimal | Default 220; informational, not a substitute for measured voltage |
| `calibrated_voltage_min_V` | decimal | Default 180 |
| `calibrated_voltage_max_V` | decimal | Default 250 |
| `calibrated_current_min_A` | decimal | Default 0.25 |
| `calibrated_current_max_A` | decimal | Default 10.0 |
| `active` | boolean | Inactive circuits do not create consumption alerts |

### CalibrationArtifact

| Field | Type | Rule |
|---|---|---|
| `calibration_revision` | string | Primary key, immutable |
| `node_id` | string | Foreign key |
| `hardware_revision` | string | Exact build calibrated |
| `firmware_commit` | string | Full VCS revision |
| `reference_meter_id` | string | Serial or lab asset identity |
| `reference_uncertainty_percent` | decimal | Must be <=1 for acceptance evidence |
| `voltage_gain` | decimal | Applied after offset correction |
| `current_gain` | decimal | Applied after offset correction |
| `phase_correction_samples` | decimal | Signed fractional sample correction |
| `zero_noise_current_A` | decimal | Absolute no-load threshold |
| `procedure_version` | string | Versioned procedure reference |
| `dataset_manifest_sha256` | string | Hash of immutable evidence manifest |
| `accepted_at` | timestamp? | Null until every acceptance point passes |
| `accepted_by` | string? | Qualified reviewer/operator identity |

Calibration records are append-only. Editing coefficients creates a new revision.

### ThresholdConfiguration

| Field | Type | Rule |
|---|---|---|
| `config_version` | integer | Monotonically increasing per node |
| `node_id`, `circuit_id` | string | Must match the target node/circuit |
| `active_power_limit_W` | decimal | `>0` and `<=2500` for the calibrated MVP |
| `confirm_ms` | integer | 1,000–300,000; default 10,000 |
| `clear_below_W` | decimal | `>=0` and `< active_power_limit_W`; default 95% of limit |
| `clear_ms` | integer | 1,000–300,000 |
| `severity` | enum | `warning`, `critical` |
| `local_warning_enabled` | boolean | Does not authorize automatic switching |
| `effective_at` | timestamp | UTC |
| `created_by` | string | Operator identity |

Only the newest valid version is published as retained config. Firmware stores the
last accepted version and rejects rollbacks unless an explicit recovery procedure
authorizes one.

## Time-series entity

### MeasurementWindow

One record represents a five-second aggregation; it is not a raw ADC sample.

| Field | Type | Meaning and validation |
|---|---|---|
| `message_id` | UUID v4 | Globally unique and immutable across retries |
| `schema_version` | string | `1.0.0` for this contract |
| `node_id`, `circuit_id` | string | Identity and series tags |
| `boot_id` | UUID v4 | New on every firmware boot |
| `sequence` | uint64 | Strictly increases within one boot |
| `window_start` | timestamp | Inclusive observation start, UTC |
| `window_end` | timestamp | Exclusive observation end and canonical observation timestamp |
| `duration_ms` | integer | Monotonic elapsed duration; normally 5,000 |
| `sample_count` | integer | Number of paired samples included; normally 20,000 |
| `voltage_rms_V` | decimal? | `sqrt(mean(v^2))`; null when invalid |
| `current_rms_A` | decimal? | `sqrt(mean(i^2))`; null when invalid |
| `active_power_W` | decimal? | `mean(v*i)` after calibration/phase correction; signed internally |
| `active_energy_import_total_Wh` | decimal? | Monotonic imported energy for `counter_id`; never derived from wall-clock jumps |
| `counter_id` | UUID | Changes only when continuity cannot be established or counter is intentionally reset |
| `quality` | enum | `valid`, `degraded`, `invalid` |
| `quality_flags` | set | Enumerated evidence about range, sampling, clock, calibration, clipping, sensors, polarity |
| `valid_sample_ratio` | decimal | 0–1 |
| `calibration_revision` | string | Exact applied coefficients |
| `clock_source` | enum | `sntp`, `holdover`, `unsynchronized` |
| `last_sync_age_s` | integer? | Required unless unsynchronized with no previous sync |
| `uptime_ms` | uint64 | Device monotonic uptime |
| `received_at` | timestamp | Added by ingestion |
| `stored_at` | timestamp | Added after durable storage |

Invariants:

- `(node_id, boot_id, sequence)` and `message_id` are independently unique.
- `window_end > window_start`; `duration_ms` is monotonic duration, not their
  untrusted wall-clock subtraction.
- All numeric quantities use the canonical wire units. A dashboard may display
  kWh only by dividing Wh by 1,000 and labeling the result.
- `valid` requires all four values, accepted calibration, synchronized or
  permitted holdover clock, no clipping/sample-gap/disconnect/out-of-range flag,
  and `valid_sample_ratio >= 0.99`.
- `invalid` quantities are null; the last valid values must not be copied into a
  new invalid record.
- `degraded` may contain values for diagnosis but those values are excluded from
  accuracy, alert, and model evidence unless a procedure explicitly says otherwise.
- Negative active power is retained for diagnostics and raises
  `polarity_reversed`; it does not decrement imported energy.

InfluxDB tags are `node_id`, `circuit_id`, `quality`, and
`calibration_revision`; high-cardinality UUIDs remain fields. Raw records expire
after 30 days. Derived hourly/daily series carry the source interval and query
revision.

## Delivery and operational entities

### IngestionLedger

| Field | Type | Rule |
|---|---|---|
| `message_id` | UUID | Primary key |
| `node_id`, `boot_id`, `sequence` | identity | Unique tuple |
| `payload_sha256` | string | Same ID with different bytes is a security/contract error |
| `state` | enum | `received`, `stored`, `rejected` |
| `received_at`, `stored_at` | timestamp? | UTC lifecycle evidence |
| `reject_code`, `reject_detail` | string? | Bounded, secret-free diagnostics |
| `ack_count` | integer | Number of application ACK publications |

Transition rules:

```text
new -> received -> stored
new -> rejected
stored -> stored      (duplicate retry; publish duplicate ACK)
rejected -> rejected  (terminal contract rejection; publish rejected ACK)
```

If a process stops after time-series write but before ledger state `stored`,
recovery repeats the idempotent point write and finishes the ledger transition.
The application ACK is published only from `stored` or `rejected`.

### NodeStatus

Latest retained operational status, not historical measurement:

- identity, firmware/hardware/calibration revisions;
- state: `online`, `degraded`, `offline`;
- acquisition state, clock source, pending queue depth and oldest age;
- last valid window and error code;
- reported every 30 seconds and on material state change.

Server derives `stale` when no measurement arrives for 15 seconds and `offline`
after 30 seconds or an authenticated last will. A stale measurement is never
rendered as live.

## Alert entities and state machines

### DeviceThresholdEvent

An immutable node event records local threshold state changes with `event_id`,
config version, window evidence, measured W, threshold W, monotonic confirmation
duration, clock state, and local warning output state.

Local state machine:

```text
normal --value > limit--> pending
pending --confirmed for confirm_ms--> firing
pending --value <= limit or invalid--> normal
firing --value < clear_below_W--> clearing
clearing --below for clear_ms--> resolved --> normal
clearing --value >= clear_below_W--> firing
```

Invalid data never confirms or clears an electrical condition; it creates a
separate data-quality/health event. LED/buzzer is active only in `firing` or
`clearing`. There is no relay or cutoff transition.

### Alert

| Field | Type | Rule |
|---|---|---|
| `alert_id` | UUID | Primary key and notification idempotency key |
| `event_id` | UUID | Unique source event |
| `node_id`, `circuit_id` | string | Affected target |
| `kind` | enum | `active_power_threshold`, `measurement_quality`, `node_offline`, `pipeline_health` |
| `severity` | enum | `warning`, `critical` |
| `status` | enum | `open`, `acknowledged`, `resolved` |
| `opened_at`, `acknowledged_at`, `resolved_at` | timestamp? | UTC lifecycle |
| `observed_value`, `threshold_value`, `unit` | value | Evidence; nullable for health alerts |
| `quality` | enum | Quality of the evidence |

Server state machine:

```text
open --operator receipt--> acknowledged
open --physical clear--> resolved
acknowledged --physical clear--> resolved
```

Acknowledgment never clears the physical condition. `AlertTransition` is an
append-only audit row containing from/to state, actor, reason, and timestamp.

### NotificationOutbox

| Field | Type | Rule |
|---|---|---|
| `delivery_id` | UUID | Primary key |
| `alert_id` | UUID | Stable external idempotency key |
| `destination_type` | enum | MVP implementation-selected outbound HTTPS provider |
| `state` | enum | `pending`, `sending`, `delivered`, `failed_terminal` |
| `attempt_count`, `next_attempt_at` | retry | Exponential backoff with jitter |
| `last_error_code` | string? | Redacted and bounded |
| `delivered_at` | timestamp? | UTC |

No secret or personal destination is stored in source-controlled fixtures.

## Analytics evidence entities (future feature input)

### DatasetManifest

| Field | Type | Rule |
|---|---|---|
| `dataset_id` | string | Content-derived or immutable version |
| `created_from_query` | string | Versioned query or export command |
| `source_start`, `source_end` | timestamp | Closed historical interval |
| `schema_versions` | list | Input contract versions |
| `calibration_revisions` | list | Included calibration evidence |
| `exclusion_rules` | list | Explicit quality/missing-data rules |
| `label_definition_version` | string | Required for anomaly evaluation |
| `file_hashes` | map | SHA-256 for every data artifact |
| `code_commit`, `environment_lock_hash` | string | Reproduction identity |

### ExperimentRun

Records dataset ID, task (`anomaly` or `forecast`), chronological split boundaries,
baseline, preprocessing, parameters, random seed, runtime environment, metrics,
and artifact hashes. It is research evidence only until a later specification
defines promotion criteria and runtime behavior.

## Boundary-to-contract map

| Boundary | Contract/evidence |
|---|---|
| Sensor/ADC -> firmware | ADC register/config fixture, captured waveform fixture, calculation unit tests, calibration artifact |
| Firmware -> MQTT broker | `measurement.schema.json`, `node-status.schema.json`, `node-event.schema.json` |
| Server -> firmware | `threshold-config.schema.json` |
| Ingestion -> firmware | `measurement-ack.schema.json` |
| Ingestion -> storage | Ingestion ledger migrations plus idempotent point-write integration test |
| Storage -> dashboard | Versioned Flux queries, datasource provisioning, dashboard JSON snapshot tests |
| Threshold -> alerts | State-machine unit tests and `AlertTransition` audit rows |
| Dataset -> model result | Dataset manifest and experiment-run schema in the future analytics specification |
