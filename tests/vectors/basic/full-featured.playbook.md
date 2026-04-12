# Full-Featured Workflow

A comprehensive playbook demonstrating every spec feature for conformance testing.

## SYSTEM

You are a meticulous analyst. Always provide evidence-based reasoning with clear structure. Cite specifics rather than generalizations.

## INPUTS

- `topic` (string): Subject to analyze
- `detail_level` (enum: brief, standard, comprehensive): How much detail to include
- `max_items` (number: 5): Maximum items to return
- `include_sources` (boolean: true): Whether to include source references

## STEP 1: Research

Research {{topic}} and identify the top {{max_items}} most important aspects.

For each aspect provide:
- A clear title
- A one-paragraph explanation
- Relevance rating (high, medium, low)

@output(research_results)

## STEP 2: Evaluate

```if detail_level == "comprehensive"```

### STEP 2a: Deep Evaluation

Perform an exhaustive evaluation of each aspect of {{topic}} identified above.

For every item, provide:
1. Strengths with supporting evidence
2. Weaknesses and risk factors
3. Comparative analysis against alternatives
4. Long-term outlook (3-5 year horizon)

@output(evaluation)

```elif detail_level == "brief"```

### STEP 2b: Quick Summary

Provide a brief summary table of the findings about {{topic}}.

| Aspect | Rating | One-Line Take |
|--------|--------|---------------|

@output(evaluation)

```else```

### STEP 2c: Standard Evaluation

Evaluate each aspect of {{topic}} with balanced detail:
- Key strengths
- Key risks
- Recommendation

@output(evaluation)

```endif```

## STEP 3: Human Review

@elicit(confirm, "Does the evaluation look accurate? Proceed to final report?")

## STEP 4: Final Report

Synthesize everything into a final report on {{topic}}.

@prompt(library:report-template)

Structure the report with:
1. Executive Summary
2. Detailed Findings
3. Recommendations
4. Next Steps

@output(final_report)

## ARTIFACTS

type: markdown
