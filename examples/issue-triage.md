# Issue Triage

Classify an incoming issue and route it to the appropriate workflow.

## INPUTS

- `issue` (text): Issue description from the reporter
- `project` (string): Project name for context

## STEP 1: Classify

You are triaging issues for the {{project}} project.

Read the following issue report and classify it as exactly one of: "bug", "feature", or "question".

{{issue}}

Respond with a JSON object:
{"classification": "<type>", "confidence": <0.0-1.0>, "reasoning": "<brief explanation>"}

@output(issue_type, extract:"classification")

## STEP 2: Route

```if issue_type == "bug"```

### STEP 2a: Bug Analysis

Analyze this bug report for {{project}}:

{{issue}}

Provide:
1. **Summary**: One-line description
2. **Reproduction steps**: Inferred from the report (note gaps)
3. **Severity**: Critical / High / Medium / Low
4. **Affected area**: Which component or module
5. **Suggested investigation**: Where a developer should start looking

```elif issue_type == "feature"```

### STEP 2b: Feature Specification

Draft a lightweight feature specification for {{project}}:

{{issue}}

Include:
1. **Title**: Clear, concise feature name
2. **Problem statement**: What user need does this address?
3. **Proposed solution**: High-level approach
4. **Acceptance criteria**: 3-5 testable conditions
5. **Complexity estimate**: Small / Medium / Large

```else```

### STEP 2c: Answer

Provide a helpful answer to this question about {{project}}:

{{issue}}

If the question requires information you don't have, clearly state what's needed and suggest where the reporter might find it.

```endif```

## ARTIFACTS

type: markdown
