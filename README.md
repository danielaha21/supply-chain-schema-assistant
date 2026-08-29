# Supply Chain Schema Assistant
This project designs a workflow for generating JSON Schemas for supply‑chain events using IBM Bob. Developers describe a new event in plain language, and Bob produces:

1. a JSON Schema

2. an explanation of the schema

3. developer‑facing documentation

4. a short PR summary


## Scope
This project focuses only on schema authoring.

## Purpose
Supply‑chain systems rely on consistent event definitions. This workflow helps teams produce clear, well‑structured schemas quickly, with built‑in documentation and review materials. The goal is to standardize schema creation and reduce the time spent writing and reviewing event definitions.

## Project Structure Code
```
data/schemas/shipment_event_schema.json     
bob_inputs/new_schema_request.md           
bob_outputs/
  <event>_schema.json
  <event>_schema_explained.md
  <event>_schema_docs.md
  <event>_schema_pr_summary.md
```

## How It Works

### Write the event description  
Developers add a Markdown file under bob_inputs/new_schema_request.md describing the event: purpose, required fields, optional fields, constraints, and an example payload.

### Generate the schema with Bob  
Bob reads the request and the reference schema, then produces a JSON Schema following the same structure and conventions.

### Review the explanation  
Bob generates a Markdown explanation of the schema, breaking down each field and its purpose.

### Add documentation  
Bob creates developer‑facing documentation with example payloads and integration notes.

### Create a Pull Request summary  
Bob writes a short summary suitable for a pull request.

**All generated files appear in bob_outputs/.**

## Reference Schema
The project includes a single reference schema (shipment_event_schema.json) that defines the structure and conventions Bob should follow when generating new schemas. This keeps output consistent across events.

## Bob Prompts
Three prompts are used during the workflow:
- Schema Generation
- Schema Explanation
- Documentation & PR Summary

***These are included separately (see PDF) for use during demos.***
