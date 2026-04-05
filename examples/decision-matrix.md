# Technical Decision Matrix

Evaluate a technology choice with structured analysis, human review checkpoints, and a final recommendation.

## SYSTEM

You are a senior technical architect. Provide evidence-based evaluations. Consider trade-offs, ecosystem maturity, and long-term maintenance costs.

## INPUTS

- `technology` (string): Technology to evaluate
- `criteria` (text): Evaluation criteria
- `constraints` (text): Project constraints and requirements
- `depth` (enum: quick, thorough): Analysis depth

## STEP 1: Requirements Analysis

Analyze the following project constraints and extract technical requirements:

{{constraints}}

Focus on:
- Scalability needs
- Team expertise requirements
- Integration constraints
- Timeline pressures

Respond with a structured requirements list, and include:
{"priority_level": "high" or "medium" or "low"}

@output(requirements, extract:"priority_level")

## STEP 2: Technology Assessment

Evaluate {{technology}} against these criteria:

{{criteria}}

Rate each criterion on a 1-5 scale with justification.

```if depth == "thorough"```

### STEP 2a: Deep Dive

Perform a detailed analysis of {{technology}} including:
- Community health and contributor activity trends
- Security vulnerability history (last 2 years)
- Performance benchmarks vs. alternatives
- Migration complexity from common existing solutions
- License implications

@output(assessment)

```else```

### STEP 2b: Quick SWOT

Provide a concise SWOT analysis of {{technology}}:
- **Strengths**: Top 3
- **Weaknesses**: Top 3
- **Opportunities**: Key ecosystem trends in its favor
- **Threats**: Risks to long-term viability

@output(assessment)

```endif```

## STEP 3: Human Review

@elicit(confirm, "Does the assessment look accurate? Proceed to recommendation?")

## STEP 4: Choose Emphasis

@elicit(select, "What should the recommendation emphasize?", "Cost", "Performance", "Developer Experience", "Long-term Viability")
@output(chosen_focus)

## STEP 5: Final Recommendation

Based on the assessment of {{technology}}, provide a recommendation emphasizing {{chosen_focus}}:

1. **Decision**: Go / No-Go with confidence level (percentage)
2. **Key risks**: Top 3 risks with mitigation strategies
3. **Implementation timeline**: Phased rollout plan
4. **Alternatives**: If No-Go, recommend 2-3 alternatives with brief rationale

## ARTIFACTS

type: markdown
