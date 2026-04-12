# Priority Classifier

Classify an incoming request and extract a priority level from structured JSON output.

## INPUTS

- `request` (text): The incoming request to classify

## STEP 1: Classify Priority

Analyze the following request and determine its priority.

{{request}}

Respond with a JSON object containing your assessment:
{"priority": "critical", "category": "infrastructure", "confidence": 0.95}

Use one of: "critical", "high", "medium", "low" for priority.

@output(classification, extract:"priority")

## STEP 2: Route Request

The request has been classified with priority: {{classification}}.

Based on this priority level, recommend:
1. Response SLA (time to first response)
2. Escalation path
3. Required team members
