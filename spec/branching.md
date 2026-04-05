# Branching

Branching enables conditional execution paths within a step. Based on the value of an input variable or a named output from a prior step, different sub-steps execute.

## Syntax

Branches use triple-backtick fenced markers on their own lines:

````markdown
## STEP 2: Route

```if variable == "value"```

### STEP 2a: Path A

Content for path A.

```elif variable != "other"```

### STEP 2b: Path B

Content for path B.

```else```

### STEP 2c: Default Path

Fallback content.

```endif```
````

## Marker Patterns

| Marker | Pattern | Example |
|--------|---------|---------|
| `if` | `` ^```if\s+(\w+)\s*(==\|!=)\s*"([^"]*)"\s*```$ `` | `` ```if mode == "security"``` `` |
| `elif` | `` ^```elif\s+(\w+)\s*(==\|!=)\s*"([^"]*)"\s*```$ `` | `` ```elif mode != "quick"``` `` |
| `else` | Exact: `` ```else``` `` | `` ```else``` `` |
| `endif` | Exact: `` ```endif``` `` | `` ```endif``` `` |

## Operators

**Only two operators are supported: `==` and `!=`.**

Both perform exact string comparison. There are no numeric operators (`>`, `<`, `>=`, `<=`), no pattern matching (`contains`, `matches`), and no logical combinators (`AND`, `OR`).

| Operator | Meaning | Example |
|----------|---------|---------|
| `==` | Equals (exact string match) | `mode == "security"` |
| `!=` | Not equals | `mode != "quick"` |

Values must be double-quoted. Single quotes are not recognized.

## Condition Variables

The variable in a condition can reference:

| Source | Description | Example |
|--------|-------------|---------|
| Input | A variable declared in `## INPUTS` | `depth == "thorough"` |
| Named output | A variable captured by `@output(name)` in a prior step | `issue_type == "bug"` |

If a condition references a variable that is neither a declared input nor a named output, the parser emits a **warning**. At runtime, an undeclared variable evaluates as an empty string.

## Sub-steps

Inside branch bodies, steps use `###` headings (level 3) with letter-suffixed labels:

```regex
^###\s+STEP\s+(\d+[a-z]):\s+(.+)$
```

```markdown
### STEP 2a: Deep Analysis
### STEP 2b: Quick Review
### STEP 3a: Bug Fix
### STEP 3b: Feature Spec
```

Label format: parent step number + lowercase letter (e.g., `2a`, `2b`, `3a`).

Sub-steps support all directives: `@output`, `@elicit`, `@prompt`, `@tool`, and `{{variable}}` interpolation. A branch body may contain multiple sub-steps.

## Evaluation Rules

1. Conditions are evaluated top-to-bottom (`if`, then each `elif`)
2. The **first** matching condition's branch executes
3. If no `if`/`elif` matches and an `else` branch exists, it executes
4. If no branch matches and there is no `else`, the entire step is **skipped** (status: `skipped`)
5. Only one branch executes per step — evaluation stops after the first match

## Branch Examples

### Branch on Input Variable

````markdown
## INPUTS

- `mode` (enum: security, performance, readability): Review focus

## STEP 1: Review

```if mode == "security"```

### STEP 1a: Security Audit

Audit this code for OWASP Top 10 vulnerabilities:
{{code}}

```elif mode == "performance"```

### STEP 1b: Performance Review

Profile this code for bottlenecks and O(n) issues:
{{code}}

```else```

### STEP 1c: Readability Review

Review this code for naming, structure, and documentation:
{{code}}

```endif```
````

### Branch on AI Output

````markdown
## STEP 1: Classify

Classify this issue as exactly one of: "bug", "feature", "question".
{{issue}}

@output(issue_type, extract:"classification")

## STEP 2: Route

```if issue_type == "bug"```

### STEP 2a: Bug Analysis

Analyze reproduction steps and severity.

```elif issue_type == "feature"```

### STEP 2b: Feature Spec

Draft a feature specification with acceptance criteria.

```else```

### STEP 2c: Answer

Provide a helpful answer to the question.

```endif```
````

### Mixed Content with Branches

A step may have both regular content (above the branch block) and conditional branches. The regular content is always included in the step; the branch content is appended based on the condition.

````markdown
## STEP 2: Evaluate

Evaluate {{technology}} against the criteria.

@output(eval_result, extract:"recommendation")

```if eval_result == "adopt"```

### STEP 2a: Adoption Plan

Create a detailed adoption roadmap.

```else```

### STEP 2b: Alternatives

Suggest three alternative technologies.

```endif```
````

## Nesting

Branch blocks **do not nest**. A branch body may not contain another `if`/`endif` block. To model multi-level decisions, use sequential steps where each step branches on a named output from a prior step.

## Future Considerations

The following are not part of v0.1 but are under consideration for future versions:

- **Logical combinators** (`AND`, `OR`) for compound conditions
- **Numeric operators** (`>`, `<`, `>=`, `<=`) with type coercion
- **Nested branching** within sub-steps
- **Pattern matching** (`contains`, `startsWith`, regex)
