# Guided Code Review

Review code using an external prompt template for consistent criteria.

## INPUTS

- `code` (text): Code to review
- `language` (string: Python): Programming language

## STEP 1: Review Code

@prompt(library:code-review-criteria)

Review the following {{language}} code against the criteria above:

{{code}}

Provide findings organized by severity (Critical, High, Medium, Low).

## STEP 2: Suggestions

For each finding identified above, provide a specific code fix with a before/after comparison.
