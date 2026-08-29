Supply Chain Schema Assistant
This project provides a lightweight workflow for generating JSON Schemas for supply‑chain events using IBM Bob. Developers describe a new event in plain language, and Bob produces:

a JSON Schema

an explanation of the schema

developer‑facing documentation

a short PR summary

The goal is to standardize schema creation and reduce the time spent writing and reviewing event definitions.

Project Structure
Code
data/
  schemas/
    shipment_event_schema.json     # Reference schema used as a pattern

bob_inputs/
  new_schema_request.md            # Developer-written event description

bob_outputs/
  <event>_schema.json
  <event>_schema_explained.md
  <event>_schema_docs.md
  <event>_schema_pr_summary.md
How It Works
Write the event description  
Developers add a Markdown file under bob_inputs/new_schema_request.md describing the event: purpose, required fields, optional fields, constraints, and an example payload.

Generate the schema with Bob  
Bob reads the request and the reference schema, then produces a JSON Schema following the same structure and conventions.

Review the explanation  
Bob generates a Markdown explanation of the schema, breaking down each field and its purpose.

Add documentation  
Bob creates developer‑facing documentation with example payloads and integration notes.

Create a PR summary  
Bob writes a short summary suitable for a pull request.

All generated files appear in bob_outputs/.

Reference Schema
The project includes a single reference schema (shipment_event_schema.json) that defines the structure and conventions Bob should follow when generating new schemas. This keeps output consistent across events.

Bob Prompts
Three prompts are used during the workflow:

Schema Generation

Schema Explanation

Documentation + PR Summary

These are included separately (PDF or Markdown) for use during demos.

Scope
This project focuses only on schema authoring.
It does not perform validation, DLQ simulation, or schema evolution.

Purpose
Supply‑chain systems rely on consistent event definitions. This workflow helps teams produce clear, well‑structured schemas quickly, with built‑in documentation and review materials.