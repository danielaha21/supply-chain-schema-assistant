# `shipment_delayed` — Developer Documentation & PR Summary

---

## Section 1 — Developer Documentation

### Event Purpose

The `shipment_delayed` event is emitted by the transportation service whenever a shipment fails to meet its expected arrival time at a facility. It signals that a delay has been detected during transit and provides structured context about the reason, location, and duration of the delay. Downstream consumers can use this event to trigger alerts, update ETAs, feed reporting dashboards, or initiate carrier follow-up workflows.

---

### Example Payloads

#### Minimal payload (required fields only)

```json
{
  "shipment_id": "SH12345",
  "delay_reason": "customs",
  "timestamp": "2026-08-18T14:32:00Z",
  "location": {
    "facility_id": "FC-204",
    "city": "Atlanta",
    "state": "GA",
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
  "notes": "Storm system over I-85 corridor. Dispatcher reference: INC-8821."
}
```

#### Delay with unknown duration

```json
{
  "shipment_id": "SH99001",
  "delay_reason": "mechanical_issue",
  "timestamp": "2026-09-02T08:15:00Z",
  "location": {
    "facility_id": "FC-110",
    "city": "Memphis",
    "state": "TN",
    "country": "USA"
  },
  "carrier": "Apex Freight"
}
```

---

### Usage Notes

- **`shipment_id`** must reference a shipment that already exists in the shipment tracking system. This event does not create a shipment record — it annotates one.
- **`delay_reason`** is an enumerated field. Only the following values are valid: `weather`, `mechanical_issue`, `traffic`, `customs`, `other`. Payloads containing any other value will fail schema validation. Use `other` for reasons not covered by the defined categories and capture detail in `notes`.
- **`timestamp`** must represent the moment the delay was detected, not the shipment's original departure time. Where possible, this value should be greater than the shipment's departure timestamp stored in the parent shipment record.
- **`location`** uses the same four-field object structure (`facility_id`, `city`, `state`, `country`) shared by all shipment-related events in this system. All four sub-fields are required.
- **`delay_duration_minutes`** should be omitted if the duration is not yet known at event emission time. Do not emit `0` as a placeholder — omit the field entirely until a real value is available.
- **`notes`** is an unstructured free-text field. It is intended for human-readable context only and should not be parsed programmatically.

---

### Integration Guidance

**Producers**

- Emit this event from the transportation service as soon as a delay is detected.
- Include `delay_duration_minutes` and `carrier` whenever they are available at emission time.
- Validate the payload against the `shipment_delayed` schema before publishing to the event bus.

**Consumers**

- Subscribe to `shipment_delayed` events on the transport domain topic.
- Index by `shipment_id` to correlate delays with parent shipment records.
- Use `delay_reason` for categorical routing (e.g., route `customs` delays to the compliance team, `weather` delays to the carrier ops team).
- Treat `delay_duration_minutes` as advisory — the field is optional and may be absent, especially for events emitted immediately after detection.
- Do not rely on `notes` for machine processing; treat it as a human-readable annotation only.

---

---

## Section 2 — PR Summary

### Summary

This PR introduces the `shipment_delayed` JSON Schema (Draft 2020-12). The schema defines the event structure emitted by the transportation service when a shipment fails to meet its expected facility arrival time. It follows the field naming and `location` object conventions established by the existing `shipment_event_schema.json`.

---

### Major Fields

| Field | Type | Description |
|---|---|---|
| `shipment_id` | `string` | References the delayed shipment. |
| `delay_reason` | `string` (enum) | Controlled vocabulary for the cause of delay. |
| `timestamp` | `string` (`date-time`) | ISO 8601 datetime when the delay was detected. |
| `location` | `object` | Facility location where the delay was detected. |
| `delay_duration_minutes` | `number` | Estimated delay length in minutes. Must be ≥ 0. |
| `carrier` | `string` | Logistics carrier responsible at time of delay. |
| `notes` | `string` | Free-text field for additional operational context. |

---

### Required vs Optional Fields

**Required**
- `shipment_id`
- `delay_reason`
- `timestamp`
- `location` (with all four sub-fields: `facility_id`, `city`, `state`, `country`)

**Optional**
- `delay_duration_minutes`
- `carrier`
- `notes`

---

### Example Payload

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
