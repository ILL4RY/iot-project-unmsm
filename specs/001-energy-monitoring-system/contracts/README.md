# MQTT v1 contracts

All payloads are UTF-8 JSON, JSON Schema Draft 2020-12, `additionalProperties:
false`, and at most 16 KiB. Producers MUST NOT send duplicate object member names,
NaN, or infinities. Consumers MUST validate before use.

| Direction | Topic | Schema | Retained | QoS |
|---|---|---|---|---|
| Node -> server | `unmsm/energy/v1/nodes/{node_id}/measurements` | `measurement.schema.json` | No | 1 |
| Node -> server | `unmsm/energy/v1/nodes/{node_id}/events` | `node-event.schema.json` | No | 1 |
| Node -> server | `unmsm/energy/v1/nodes/{node_id}/status` | `node-status.schema.json` | Yes | 1 |
| Server -> node | `unmsm/energy/v1/nodes/{node_id}/config` | `threshold-config.schema.json` | Yes | 1 |
| Server -> node | `unmsm/energy/v1/nodes/{node_id}/acks/measurements` | `measurement-ack.schema.json` | No | 1 |

The authenticated certificate identity, topic `node_id`, and payload `node_id`
MUST agree. An exact duplicate measurement receives `duplicate`; a reused identity
with a different payload hash is rejected and raises a data-quality/security
event. Firmware removes a queued measurement only after `accepted` or `duplicate`.
A terminal contract rejection moves it to bounded diagnostics instead of causing a
retry loop.

Compatibility uses semantic schema versions. Additive changes require a new minor
schema accepted by consumers before producers emit it. Removed fields, renamed
fields, changed meanings/units, or stricter ranges require a new topic major
version. Canonical wire energy is always Wh; kWh is display-only conversion.

Contract fixtures under `examples/` intentionally include one valid and one
invalid measurement. CI validates the valid fixture successfully and asserts that
the invalid fixture fails for missing units and an unrecognized field.
