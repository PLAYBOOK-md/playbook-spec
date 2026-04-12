# Support Ticket Router

Route a support ticket to the appropriate handling workflow based on its category.

## INPUTS

- `ticket` (text): Support ticket description
- `category` (enum: billing, technical, account, other): Ticket category

## STEP 1: Classify Urgency

Read the support ticket and assess its urgency level.

{{ticket}}

Respond with a JSON object: {"urgency": "high" or "medium" or "low"}

@output(urgency_level, extract:"urgency")

## STEP 2: Handle Ticket

```if category == "billing"```

### STEP 2a: Billing Resolution

This is a billing-related ticket. Review the issue and provide:
1. Root cause of the billing discrepancy
2. Recommended resolution (refund, credit, adjustment)
3. Steps to prevent recurrence
4. Customer communication template

```elif category == "technical"```

### STEP 2b: Technical Investigation

This is a technical support ticket. Investigate and provide:
1. Likely root cause based on symptoms described
2. Diagnostic steps for the support team
3. Known workarounds if any
4. Whether engineering escalation is needed

```elif category == "account"```

### STEP 2c: Account Management

This is an account-related ticket. Address:
1. Account status verification steps
2. Required identity verification process
3. Resolution path
4. Policy references if applicable

```else```

### STEP 2d: General Support

This ticket does not fit standard categories. Provide:
1. Best-effort categorization
2. Suggested handling team
3. Draft response to the customer
4. Recommendation for new category if warranted

```endif```

## STEP 3: Summary

Write a concise internal summary of the ticket resolution for the support log.
