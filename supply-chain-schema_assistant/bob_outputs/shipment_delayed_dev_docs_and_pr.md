# Shipment Delayed Event — Developer Documentation & PR Summary

---

## Section 1 — Developer Documentation

### Event Purpose

The `shipment_delayed` event is emitted by the transportation service whenever a shipment fails to meet its expected arrival time at a facility. It signals a detected delay during active transit and is intended for consumption by alerting systems, customer notification pipelines, SLA tracking services, and operational dashboards.

Every event is anchored to a specific shipment via `shipment_id`, a reason code via `delay_reason`, the moment of detection via `timestamp`, and the physical location via `location`. These four fields form the minimum viable payload; all other fields enrich the event when additional context is available at emission time.

---

### Example Payloads

#### Minimal payload (required fields only)

```json
{
  "shipment_id": "SH12345",
  "delay_reason": "customs",
  "timestamp": "2026-08-18T09:00:00Z",
  "location": {
    "facility_id": "FC-101",
    "city": "Chicago",
    "state": "IL",
    "country": "USA"
  }
}
```

#### Full payload (all fields present)

```json
{
  "shipment_id": "SH12345",
  "delay_reason": "weather",
  "timestamp": "2026-08-18T14:32:00Z",
  "location": {
    "facility_id": "FC-204",
    "city": "Atlanta",
    "state": "GA",
    "country": "USA"
  },
  "delay_duration_minutes": 90,
  "carrier": "BlueSky Logistics",
  "notes": "Severe thunderstorm activity closed the inbound loading dock."
}
```

#### Mechanical issue with unknown duration

```json
{
  "shipment_id": "SH99001",
  "delay_reason": "mechanical_issue",
  "timestamp": "2026-09-03T07:15:00Z",
  "location": {
    "facility_id": "FC-310",
    "city": "Dallas",
    "state": "TX",
    "country": "USA"
  },
  "carrier": "RedLine Freight"
}
```

---

### Usage Notes

- **`shipment_id` must reference an existing shipment.** Consumers should reject or dead-letter events where the referenced shipment cannot be resolved in the shipment registry.
- **`delay_reason` is an enum.** Only the values `"weather"`, `"mechanical_issue"`, `"traffic"`, `"customs"`, and `"other"` are valid. Producers must not emit free-text reasons; use `"other"` for uncategorised cases.
- **`timestamp` reflects detection time, not departure time.** Do not use this field to infer when the shipment left its origin. It must always be greater than the shipment's recorded departure timestamp where that is known.
- **`delay_duration_minutes` is optional and may be absent when the duration is not yet known.** When present, it must be `>= 0`. A value of `0` is valid and means the delay was detected but has not yet accumulated measurable time.
- **`notes` is unstructured and for human context only.** Do not encode machine-readable data in this field. Downstream systems should treat it as display-only.
- **`location` must always carry all four sub-fields** (`facility_id`, `city`, `state`, `country`). Partial location objects will fail schema validation.

---

### Integration Guidance

**Producers**

- Validate the payload against `shipment_delayed_schema.json` before publishing to the event bus.
- Set `timestamp` to the UTC wall-clock time at which the delay condition was first detected by your service.
- Include `delay_duration_minutes` only when a reliable estimate is available; omit it rather than publishing a speculative value.
- Populate `carrier` to enable carrier-level delay aggregation by downstream consumers, especially when a single shipment may change hands between carriers.

**Consumers**

- Index on `shipment_id` to join delay events back to shipment records.
- Filter on `delay_reason` to route events to the appropriate handling workflow (e.g., customs delays may require a compliance check; weather delays may trigger automatic customer notifications).
- Treat `delay_duration_minutes` as advisory when present — it reflects the estimate at detection time and may be superseded by a later resolution event.
- Do not assume `carrier` will always be present; fall back to the shipment record for carrier information when this field is absent.

---
---

## Section 2 — PR Summary

### Schema: `shipment_delayed`

**File:** `bob_outputs/shipment_delayed_schema.json`
**Draft:** JSON Schema Draft 2020-12

#### Purpose

Introduces a new JSON Schema for the `shipment_delayed` event. This event is emitted by the transportation service when a shipment misses its expected arrival time at a facility. The schema provides a validated, structured contract for producers and consumers of this event across the platform.

---

#### Major Fields

| Field | Type | Description |
|---|---|---|
| `shipment_id` | `string` | Identifies the shipment that has been delayed |
| `delay_reason` | `string` (enum) | Controlled reason code for the delay |
| `timestamp` | `string` (`date-time`) | ISO 8601 UTC timestamp of when the delay was detected |
| `location` | `object` | Facility and geographic location where the delay was detected |
| `delay_duration_minutes` | `number` | Estimated delay duration in minutes (`>= 0`) |
| `carrier` | `string` | Name of the carrier responsible at time of delay |
| `notes` | `string` | Free-text operator notes about the delay |

---

#### Required vs Optional

| Field | Required |
|---|---|
| `shipment_id` | ✅ Yes |
| `delay_reason` | ✅ Yes |
| `timestamp` | ✅ Yes |
| `location` | ✅ Yes |
| `delay_duration_minutes` | No |
| `carrier` | No |
| `notes` | No |

All four sub-fields of `location` — `facility_id`, `city`, `state`, `country` — are required when `location` is present.

---

#### Example Payload

```json
{
  "shipment_id": "SH12345",
  "delay_reason": "weather",
  "timestamp": "2026-08-18T14:32:00Z",
  "location": {
    "facility_id": "FC-204",
    "city": "Atlanta",
    "state": "GA",
    "country": "USA"
  },
  "delay_duration_minutes": 90,
  "carrier": "BlueSky Logistics"
}
```
