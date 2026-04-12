# API Schema Generator

Generate a JSON API schema from a natural language description.

## INPUTS

- `api_description` (text): Description of the API endpoints and data models
- `version` (string: v1): API version prefix

## STEP 1: Parse Requirements

Extract all endpoints, request/response models, and validation rules from:

{{api_description}}

List each endpoint with its HTTP method, path, and data models.

@output(parsed_api)

## STEP 2: Generate Schema

Generate a complete OpenAPI 3.0 JSON schema for API version {{version}} based on the parsed requirements.

Include:
- All endpoint definitions
- Request and response schemas
- Validation constraints
- Example values for each field

Output valid JSON only.

## ARTIFACTS

type: json
