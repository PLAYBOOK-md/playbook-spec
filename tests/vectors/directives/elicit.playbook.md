# Interactive Report Builder

Build a report with human checkpoints at key decision points.

## INPUTS

- `data_source` (string): Data to analyze

## STEP 1: Initial Analysis

Analyze {{data_source}} and present the top 5 findings with supporting data.

@output(findings)

## STEP 2: Get Feedback

@elicit(input, "Review the findings above. What additional aspects should be explored?")

## STEP 3: Approval Gate

@elicit(confirm, "The expanded analysis is complete. Approve for final report generation?")

## STEP 4: Choose Format

@elicit(select, "What format should the final report use?", "Executive Summary", "Detailed Technical", "Visual Dashboard")
@output(chosen_format)

## STEP 5: Generate Report

Generate the final report on {{data_source}} in {{chosen_format}} format, incorporating all findings and feedback.
