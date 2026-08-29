# Shipment Delayed Event — Schema Documentation

This document describes the structure, fields, constraints, and design rationale of the **Shipment Delayed Event** schema (`shipment_delayed_schema.json`). This event is emitted by the transportation service whenever a shipment fails to meet its expected arrival time at a facility.

---

## Required vs Optional Fields

| Field | Required | Type |
|---|---|---|
| `shipment_id` | ✅ Yes | string |
| `delay_reason` | ✅ Yes | string (enum) |
| `timestamp` | ✅ Yes | string (date-time) |
| `location` | ✅ Yes | object |
| `delay_duration_minutes` | No | number |
| `carrier` | No | string |
| `notes` | No | string |

---

## Field Reference

### `shipment_id`

- **Type:** `string`
- **Required:** Yes

A unique identifier referencing the shipment that has been delayed. This field is required because every delay event must be traceable to a specific shipment record in the system. Without it, the event cannot be correlated with any downstream processing or alerting logic.

---

### `delay_reason`

- **Type:** `string`
- **Required:** Yes
- **Allowed values:** `"weather"`, `"mechanical_issue"`, `"traffic"`, `"customs"`, `"other"`

Describes the cause of the delay. This field is constrained to a controlled enumeration to ensure consistency across systems consuming the event. Free-text reasons are not permitted; use `"other"` for cases that do not fit a defined category. This design prevents categorisation drift and makes the field reliably filterable and aggregatable by downstream services.

---

### `timestamp`

- **Type:** `string`
- **Format:** `date-time` (ISO 8601)
- **Required:** Yes

Records the exact moment the delay was detected. The format is constrained to `date-time` to enforce a machine-parseable, timezone-aware representation. Per the business rules, this reflects detection time — not the shipment's departure time — so consumers must not interpret it as an origination timestamp.

---

### `location`

- **Type:** `object`
- **Required:** Yes

Captures the physical location where the delay was detected. The object structure mirrors the `location` shape used across all other shipment-related events, ensuring consistency in how location data is represented platform-wide.

#### Nested properties

All four sub-fields are **required** within `location`:

| Sub-field | Type | Description |
|---|---|---|
| `facility_id` | `string` | Identifier of the facility at or near which the delay occurred |
| `city` | `string` | City name of the delay location |
| `state` | `string` | State or province of the delay location |
| `country` | `string` | Country of the delay location |

**Rationale:** Requiring all four sub-fields avoids partial location records, which would reduce the utility of the event for geographic reporting and routing decisions.

---

### `delay_duration_minutes`

- **Type:** `number`
- **Required:** No
- **Constraint:** `minimum: 0`

The estimated or confirmed duration of the delay, expressed in whole or fractional minutes. This field is optional because the duration may not be known at the time the delay is first detected. When present, a `minimum` of `0` is enforced — a negative duration is physically meaningless and indicates a data error.

---

### `carrier`

- **Type:** `string`
- **Required:** No

The name of the carrier responsible for the shipment at the time of the delay. This field is optional because carrier information may already be available through the referenced `shipment_id` in the shipment registry, making duplication unnecessary in all cases. It is included as optional to support contexts where a denormalised snapshot is preferable.

---

### `notes`

- **Type:** `string`
- **Required:** No

A free-text field for supplementary information about the delay that does not fit into any structured field. This field is intentionally unstructured and optional. It is intended for human-readable context — for example, an operator comment or a brief incident description — and should not be used to encode structured data.

---

## Design Rationale Summary

- **Controlled vocabulary on `delay_reason`** prevents categorisation inconsistency across producers and consumers, making the field reliable for filtering, metrics, and alerting without additional normalisation.
- **`location` as a required nested object** follows the existing shipment event convention, ensuring that all shipment-related events carry location data in a uniform shape.
- **`timestamp` uses `date-time` format** rather than a plain string to enforce ISO 8601 compliance and timezone awareness at the schema validation layer.
- **`delay_duration_minutes` is optional with a floor of `0`** because duration is often unknown at detection time, but when provided it must be non-negative to be meaningful.
- **No fields were added beyond those specified in the event requirements**, keeping the schema minimal and avoiding assumptions about future needs.
