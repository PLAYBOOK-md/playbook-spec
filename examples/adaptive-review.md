# Adaptive Code Review

Code review that adapts its focus based on the selected mode.

## INPUTS

- `code` (text): Code to review
- `language` (string: Go): Programming language
- `mode` (enum: security, performance, readability): Review focus

## STEP 1: Review

Analyze the following {{language}} code:

{{code}}

```if mode == "security"```

### STEP 1a: Security Audit

Perform a security-focused audit:
- Check for OWASP Top 10 vulnerabilities
- Identify injection risks (SQL, command, XSS)
- Review authentication and authorization patterns
- Flag hardcoded secrets or credentials
- Assess input validation and sanitization

Rate each finding as Critical / High / Medium / Low.

@output(findings)

```elif mode == "performance"```

### STEP 1b: Performance Review

Perform a performance-focused review:
- Identify O(n^2) or worse algorithmic complexity
- Flag unnecessary allocations or copies
- Check for unbounded loops or queries
- Review concurrency patterns for race conditions
- Assess memory usage and potential leaks

Rate each finding by estimated impact: High / Medium / Low.

@output(findings)

```else```

### STEP 1c: Readability Review

Perform a readability-focused review:
- Evaluate naming conventions and consistency
- Assess function length and single-responsibility
- Review documentation and comments
- Check error handling patterns
- Evaluate code organization and modularity

Rate each finding by severity: Major / Minor / Suggestion.

@output(findings)

```endif```

## STEP 2: Recommendations

For each finding above, provide:
1. The specific issue with a code reference
2. A concrete fix with a code example
3. Brief rationale for why this matters

Prioritize by severity — address the most critical issues first.

## ARTIFACTS

type: markdown
