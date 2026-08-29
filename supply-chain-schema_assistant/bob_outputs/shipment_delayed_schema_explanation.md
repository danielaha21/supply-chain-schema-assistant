# Schema Explanation: `shipment_delayed`

This document describes the structure, constraints, and design rationale for the `shipment_delayed` JSON Schema (Draft 2020-12). The schema models an event emitted when a shipment fails to meet its expected arrival time at a facility.

---

## Required Fields

### `shipment_id`
- **Type:** `string`
- **Purpose:** Uniquely identifies the shipment that has been delayed. Every delay event must be traceable back to a known shipment, which is why this field is required. No format constraint is applied beyond `string`, deferring ID format enforcement to the upstream shipment service.

---

### `delay_reason`
- **Type:** `string`
- **Allowed values:** `"weather"`, `"mechanical_issue"`, `"traffic"`, `"customs"`, `"other"`
- **Purpose:** Categorises the root cause of the delay using a controlled enumeration. This enables downstream consumers (reporting tools, alerting systems) to group and filter delays by cause without free-text parsing. The `"other"` value acts as a catch-all for cases not covered by the defined categories.

---

### `timestamp`
- **Type:** `string`
- **Format:** `date-time` (ISO 8601)
- **Purpose:** Records the exact moment the delay was detected, not the shipment's departure time. Using `format: date-time` aligns with the convention established in `shipment_event_schema.json` and ensures consumers can reliably parse and compare times across events.

---

### `location`
- **Type:** `object`
- **Purpose:** Captures where the delay was detected, using the same four-field structure defined in `shipment_event_schema.json` for consistency across all shipment-related events.
- **Nested required fields:**

  | Field | Type | Description |
  |---|---|---|
  | `facility_id` | `string` | Identifier of the facility at which the delay was detected. |
  | `city` | `string` | City where the facility is located. |
  | `state` | `string` | State or province where the facility is located. |
  | `country` | `string` | Country where the facility is located. |

  All four nested fields are required within `location`. This mirrors the existing schema pattern and ensures location data is always fully qualified, preventing partial or ambiguous location records.

---

## Optional Fields

### `delay_duration_minutes`
- **Type:** `number`
- **Constraint:** `minimum: 0`
- **Purpose:** Represents the estimated or measured duration of the delay in minutes. Marked optional because the duration may not be known at the time the delay is first detected — it can be supplied once the shipment resumes movement. The `minimum: 0` constraint prevents negative durations, which have no physical meaning.

---

### `carrier`
- **Type:** `string`
- **Purpose:** Identifies the logistics carrier responsible for the shipment at the time of delay. Optional because carrier information may already be present on the parent shipment record and need not be repeated in every event.

---

### `notes`
- **Type:** `string`
- **Purpose:** A free-text field for additional context that does not fit the structured fields above (e.g., dispatcher comments, incident reference numbers). Intentionally unconstrained to accommodate varied operational notes without requiring schema changes for each new pattern.

---

## Design Summary

| Field | Required | Type | Constraint |
|---|---|---|---|
| `shipment_id` | ✅ | `string` | — |
| `delay_reason` | ✅ | `string` | `enum` of 5 values |
| `timestamp` | ✅ | `string` | `format: date-time` |
| `location` | ✅ | `object` | 4 required sub-fields |
| `delay_duration_minutes` | ❌ | `number` | `minimum: 0` |
| `carrier` | ❌ | `string` | — |
| `notes` | ❌ | `string` | — |

The required/optional split reflects operational reality: a delay event is only meaningful if you know *which shipment*, *why it was delayed*, *when it was detected*, and *where it occurred*. Duration and carrier details are useful enrichment but can be omitted when not yet available at event emission time.
