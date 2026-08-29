# New Schema Request: shipment_delayed

## Event Description
This event represents a shipment that has been delayed during transit. It is emitted by the transportation service whenever a shipment fails to meet its expected arrival time at a facility.

## Business Rules
- A delay must always reference an existing shipment.
- The delay reason must come from a controlled list of values.
- The timestamp must reflect when the delay was detected, not when the shipment departed.
- Location information must follow the same structure used in other shipment-related events.
- Delay duration should be included if known, but it is optional.

## Required Fields
- shipment_id (string)
- delay_reason (string)
- timestamp (ISO 8601 datetime)
- location (object with facility_id, city, state, country)

## Optional Fields
- delay_duration_minutes (number)
- carrier (string)
- notes (string)

## Constraints
- delay_reason must be one of: ["weather", "mechanical_issue", "traffic", "customs", "other"]
- timestamp must be greater than the shipment’s departure timestamp (if known)
- delay_duration_minutes must be >= 0 if present

## Example Payload
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
