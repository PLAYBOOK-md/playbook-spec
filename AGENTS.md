# AGENTS.md -- PLAYBOOK.md Specification

This file teaches AI coding agents how to read, write, and validate PLAYBOOK.md files. It is a self-contained reference -- no additional context required.

Spec version: 0.1.0 (Draft)

## What is PLAYBOOK.md

PLAYBOOK.md is an open specification for multi-step AI workflows written in plain markdown. A playbook defines a sequence of steps -- each typically one AI call -- with inputs, conditional branching, human-in-the-loop checkpoints, external tool invocations, and declared output artifacts. The format is portable and tool-agnostic: any conforming implementation can parse and execute a playbook.

## Document Structure

A playbook is a UTF-8 encoded markdown file (`.md` or `.playbook.md`). Maximum size: 200,000 bytes (200 KB).

Sections are identified by `##` headings. The title uses a single `#` heading. Steps must be numbered sequentially starting from 1. Other sections can appear in any order.

```
# Title                          REQUIRED   First # heading in document
Description text                 OPTIONAL   Text between title and first ##
## SYSTEM                        OPTIONAL   System prompt applied to all steps
## INPUTS                        OPTIONAL   Input variable declarations
## STEP 1: Title                 REQUIRED   At least one step required
## STEP 2: Title                 ...        Sequential numbering (1, 2, 3...)
## STEP N: Title                 ...        No upper limit (subject to 200 KB)
## ARTIFACTS                     OPTIONAL   Expected output format declaration
```

Unrecognized `##` headings are silently skipped.

### Title

**Required.** The first `# ` heading in the document. Must appear before any `##` section. Fatal error if missing.

```markdown
# Code Review Pipeline
```

### Description

**Optional.** Plain text between the title and first `##` heading. Metadata only -- not sent to the AI during execution.

```markdown
# Code Review Pipeline

Automated code review with security focus and actionable feedback.

## INPUTS
```

### System Prompt

**Optional.** Heading: `## SYSTEM` or `## SYSTEM PROMPT` (case-insensitive). Content is all text until the next `##` heading, trimmed. Applied to every step during execution. If duplicated, last occurrence wins.

```markdown
## SYSTEM

You are a senior code reviewer. Focus on correctness, security, and maintainability.
Be specific -- cite line numbers and suggest fixes.
```

## Input Syntax

Inputs are declared in a `## INPUTS` section (case-insensitive heading). Each input is a markdown list item.

### Format

```
- `name` (type): Description
- `name` (type: default_value): Description
- `name` (enum: option1, option2, option3): Description
```

### Regex

```regex
^-\s+`([a-zA-Z][a-zA-Z0-9_]*)`\s+\(([^)]+)\)(?::\s*(.+))?$
```

### Variable Name Rules

Pattern: `[a-zA-Z][a-zA-Z0-9_]*`

- Must start with a letter
- May contain letters, digits, underscores
- Case-sensitive
- No hyphens, spaces, or leading underscores

### Types

| Canonical Type | Aliases | UI Element |
|----------------|---------|------------|
| `string` | *(any unrecognized type)* | Single-line text |
| `text` | -- | Multi-line textarea |
| `number` | `num`, `int`, `float` | Numeric input |
| `boolean` | `bool` | Toggle / checkbox |
| `enum` | `select`, `choice` | Dropdown with options |

Type matching is case-insensitive.

### Defaults

A default is specified after a colon inside the parentheses. Inputs with defaults are optional; inputs without defaults are required.

```markdown
- `language` (string: Go): Programming language
- `count` (number: 5): How many items to generate
- `verbose` (boolean: true): Include detailed output
```

### Enum Options

For enum types, the colon-separated value is a comma-separated list of options. Enum inputs do not support a separate default value -- the value is always parsed as the options list.

```markdown
- `tone` (enum: formal, casual, academic): Writing tone
- `focus` (select: security, performance, readability): Review focus
```

## Step Syntax

### Step Heading

```regex
^##\s+STEP\s+(\d+):\s+(.+)$
```

```markdown
## STEP 1: Research Phase
## STEP 2: Draft Content
## STEP 3: Polish and Review
```

### Numbering Rules

- Sequential integers starting from 1
- Non-sequential numbering produces a **warning** (not fatal)
- At least one step is **required** (fatal error if none)
- No upper limit on count (subject to 200 KB document size)

### Step Content

Everything between the step heading and the next `##` heading (or end of document) is the prompt text sent to the AI. Content is trimmed of leading/trailing whitespace. Directive lines and branch markers are extracted during parsing and excluded from the prompt.

```markdown
## STEP 1: Research Phase

Research {{topic}} and identify the top 5 key themes
relevant to a {{audience}} audience.

Organize findings by importance and include supporting evidence.
```

## Variable Interpolation

Use `{{variable_name}}` placeholders in step content. Resolved at execution time.

### Resolution Order

1. **Input values** provided by the user
2. **Input defaults** if no value was provided
3. **Named outputs** captured by `@output` directives in prior steps

If no value, default, or named output exists, the `{{name}}` placeholder remains as literal text.

### Extended Placeholder Syntax

```
{{name}}              Simple reference
{{name:type}}         With type hint (informational, not enforced)
{{name:type:default}} With type and inline default
```

### Where Variables Are Available

- Step content (prompt text)
- Branch conditions (`if variable == "value"`)
- `@tool` directive JSON arguments
- `@prompt` directive identifiers (`@prompt(library:{{id}})`)
- `@prompt` variable references (`@prompt({{variable}})`)

### Context Accumulation

Each step automatically receives all prior step outputs. Steps are **not** a conversation -- each step is a fresh AI call with accumulated context. Authors do not need to manually reference prior outputs. Named outputs (`@output`) provide explicit variable access; all other outputs are available through accumulated context.

## Directives

Directives are annotation lines within steps that control execution behavior. Each directive appears on its own line. Directive lines are extracted during parsing and are not included in the prompt text.

### @output -- Capture Step Output

Captures the AI response as a named variable for downstream steps and branch conditions.

**Syntax:**

```
@output(varname)
@output(varname, extract:"field")
```

**Regex:**

```regex
^@output\((\w+)(?:,\s*extract:"(\w+)")?\)$
```

**Basic capture** stores the full response:

```markdown
## STEP 1: Classify

Classify this issue as "bug", "feature", or "question": {{issue}}

@output(issue_type)
```

**Field extraction** extracts a specific field from a JSON object in the response (scanned bottom-up). Falls back to full response if extraction fails:

```markdown
## STEP 1: Analyze

Analyze the severity. Return JSON: {"level": "critical", "confidence": 0.9}

@output(severity, extract:"level")
```

**Rules:**
- One `@output` per step (last one wins if duplicated)
- Variable name pattern: `\w+`
- Named outputs are available to subsequent steps for interpolation, branch conditions, and directive arguments

### @elicit -- Human-in-the-Loop Input

Pauses execution and collects input from a human operator.

**Syntax:**

```
@elicit(input, "prompt text")
@elicit(confirm, "question?")
@elicit(select, "prompt", "option1", "option2", "option3")
```

**Regex:**

```regex
^@elicit\((\w+)(?:,\s*(.+))?\)$
```

**Types:**

| Type | UI Element | Response Value |
|------|-----------|----------------|
| `input` | Free-text field | User-entered text |
| `confirm` | Yes/No buttons | `"yes"` or `"no"` |
| `select` | Dropdown / radio | Selected option string |

Type matching is case-insensitive. Unrecognized types produce a **warning** and the directive is ignored.

**Argument rules:**
- All arguments after the type must be **double-quoted strings**
- For `select`: first quoted string is the prompt, subsequent quoted strings are individual options
- Wrong: `@elicit(select, "Pick one", "A, B, C")` -- options must be separate strings
- Right: `@elicit(select, "Pick one", "A", "B", "C")`

**Behavior:**
- Step enters `awaiting_input` status; execution pauses until user responds
- Response stored as `__elicit_step_N` in named outputs (N = step number)
- **Elicit-only steps** (only `@elicit` and optionally `@output`, no other content): user response becomes step output directly, no AI call
- **Mixed steps** (both `@elicit` and prompt content): AI call runs after user responds, with user response as context

```markdown
## STEP 3: Choose Direction

@elicit(select, "Which approach?", "Conservative", "Aggressive", "Balanced")
@output(approach)
```

### @prompt -- External Prompt Reference

References prompt content from an external source. Resolved content is **prepended** to the step's prompt text.

**Syntax:**

```
@prompt(scheme:identifier)
@prompt(identifier)
@prompt({{variable}})
```

**Regex:**

```regex
^@prompt\((.+)\)$
```

**Resolution order:**

1. **Variable reference:** If argument matches `^\{\{(\w+)\}\}$`, resolve the named variable's content as the prepended prompt
2. **Scheme reference:** If argument contains `:`, split on first `:` -- left is scheme, right is identifier
3. **Bare identifier:** No `:` and no `{{}}` -- implementation decides resolution

**Standard schemes:**

| Scheme | Identifier Pattern | Resolves To |
|--------|--------------------|-------------|
| `library` | `[a-zA-Z0-9\-]+` | Prompt from a prompt management system |
| `file` | Relative path (no `..`, no absolute) | Local file content relative to playbook |
| `mcp` | `server/prompt-name` | MCP prompt resource from a connected server |

Unrecognized schemes produce a **warning** and the directive is ignored.

**Variable interpolation in identifiers:**

```markdown
@prompt(library:{{prompt_id}})
@prompt(file:prompts/{{review_type}}-criteria.md)
@prompt(mcp:{{server}}/{{prompt_name}})
```

**Dynamic prompts from variables** -- uses a variable's content directly as the prepended prompt (meta-prompting):

```markdown
## STEP 1: Generate Review Criteria

Write a detailed code review prompt for {{language}}.

@output(review_prompt)

## STEP 2: Execute Review

@prompt({{review_prompt}})

Apply the review criteria above to:
{{code}}
```

**Rules:**
- One `@prompt` per step (last one wins if duplicated)
- Resolution happens at step execution time
- On failure: scheme references produce a **warning** (step runs with own content); variable references produce an **error** if undefined

### @tool -- External Tool Invocation

Invokes an external tool (via MCP or equivalent). When present, the AI call is **skipped entirely** -- the tool result becomes the step output.

**Syntax:**

```
@tool(connection, tool_name)
@tool(connection, tool_name, {"key": "value"})
@tool("Connection Name", tool_name, {"query": "{{variable}}"})
```

**Regex:**

```regex
^@tool\((.+)\)$
```

**Arguments:**

| Position | Required | Description |
|----------|----------|-------------|
| 1 | Yes | Connection name (bare identifier or double-quoted string) |
| 2 | Yes | Tool name (bare identifier) |
| 3 | No | JSON arguments object, may contain `{{variable}}` placeholders |

**Parsing:** Split on first two commas. Everything after second comma is JSON (internal commas preserved).

**Behavior:**
- `{{var}}` placeholders in JSON are rendered before execution
- Tool result becomes step output
- Combines with `@output` (capture result as named variable)
- Does NOT combine with `@prompt` (no AI call, so prepending has no effect)
- Does NOT combine with `@elicit` (use a separate step for human input before tool calls)

```markdown
## STEP 2: Fetch Data

@tool(analytics_server, get_metrics, {"period": "{{timeframe}}", "metric": "latency"})
@output(metrics_data)
```

### Directive Processing Order

When multiple directives appear in one step:

1. `@tool` -- if present, skip AI call entirely
2. `@elicit` -- if present and no response yet, pause execution
3. `@prompt` -- prepend referenced content to prompt
4. `@output` -- capture the step result (from AI, tool, or elicitation)

### Directive Interaction Matrix

| Directive | AI Call? | Combines With |
|-----------|----------|---------------|
| `@output` | Yes (captures response) | All other directives |
| `@elicit` | Only if step has other content | `@output` |
| `@prompt` | Yes (prepends content) | `@output`, `@elicit` |
| `@tool` | No (tool result is output) | `@output` only |

## Branching

Branching enables conditional execution within a step using fenced code markers.

### Syntax

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

### Marker Regex Patterns

| Marker | Regex |
|--------|-------|
| `if` | `` ^```if\s+(\w+)\s*(==\|!=)\s*"([^"]*)"\s*```$ `` |
| `elif` | `` ^```elif\s+(\w+)\s*(==\|!=)\s*"([^"]*)"\s*```$ `` |
| `else` | Exact match: `` ```else``` `` |
| `endif` | Exact match: `` ```endif``` `` |

### Operators

Only two operators: `==` (equals) and `!=` (not equals). Both perform exact string comparison. Values must be double-quoted. No numeric operators, pattern matching, or logical combinators.

### Condition Variables

Can reference:
- Input variables declared in `## INPUTS`
- Named outputs captured by `@output` in prior steps
- Undeclared variables produce a **warning** and evaluate as empty string at runtime

### Sub-steps

Inside branch bodies, use `###` headings with letter-suffixed labels:

```regex
^###\s+STEP\s+(\d+[a-z]):\s+(.+)$
```

```markdown
### STEP 2a: Deep Analysis
### STEP 2b: Quick Review
```

Label format: parent step number + lowercase letter (e.g., `2a`, `2b`). Sub-steps support all directives and variable interpolation.

### Evaluation Rules

1. Conditions evaluate top-to-bottom (`if`, then each `elif`)
2. First matching condition's branch executes
3. If no `if`/`elif` matches and `else` exists, it executes
4. If no match and no `else`, the step is **skipped** (no output, execution continues)
5. Only one branch executes per step

### Nesting

Branch blocks **do not nest**. For multi-level decisions, use sequential steps where each branches on a named output from a prior step.

### Mixed Content

A step can have content above the branch block (always included) and conditional branches (appended based on condition).

## Artifacts

The `## ARTIFACTS` section (also recognized: `## OUTPUT`, case-insensitive) declares the expected output format.

### Type Declaration

```markdown
## ARTIFACTS

type: markdown
```

**Regex:**

```regex
(?i)^type:\s*(\S+)$
```

### Valid Types

| Type | Description |
|------|-------------|
| `markdown` | Formatted text (reports, docs) |
| `json` | Structured JSON data |
| `mermaid` | Mermaid diagram syntax |
| `chartjs` | Chart.js configuration |
| `html_css` | HTML with CSS |
| `javascript` | JavaScript code |
| `typescript` | TypeScript code |

Unrecognized types produce a **warning**. If no `## ARTIFACTS` section, output is treated as untyped text.

The artifact type is metadata only -- it does not change step execution. It informs implementations about rendering, download format, and validation.

## Complete Example

This playbook demonstrates inputs, system prompt, steps, variable interpolation, @output with extraction, branching, @elicit, and artifacts.

````markdown
# Technical Debt Assessment

Evaluate a codebase area for technical debt and produce a prioritized remediation plan.

## SYSTEM

You are a senior software architect specializing in code quality and maintainability. Be specific and actionable -- cite concrete patterns, not vague advice.

## INPUTS

- `codebase_area` (string): Module or directory to assess
- `language` (string: TypeScript): Primary programming language
- `depth` (enum: quick, standard, thorough): Analysis depth
- `include_estimates` (boolean: true): Include time estimates for fixes

## STEP 1: Analyze

Analyze the {{codebase_area}} module ({{language}} codebase) for technical debt.

Perform a {{depth}} analysis covering:
1. Code duplication
2. Complex or deeply nested logic
3. Missing or outdated tests
4. Dependency issues
5. API design problems

Return a JSON object with your overall assessment:
{"severity": "low|medium|high|critical", "debt_items": <count>}

@output(assessment, extract:"severity")

## STEP 2: Route by Severity

```if assessment == "critical"```

### STEP 2a: Critical Path

This is a critical-severity debt assessment for {{codebase_area}}.

Identify the top 3 items that pose immediate risk (data loss, security, outages).
For each item, provide an emergency remediation plan with owners and deadlines.

```elif assessment == "high"```

### STEP 2b: High Priority Plan

This is a high-severity debt assessment for {{codebase_area}}.

Create a prioritized backlog of the top 5 debt items.
For each, estimate effort and business impact.

```else```

### STEP 2c: Standard Backlog

Create a prioritized backlog of all identified debt items for {{codebase_area}}.
Group by category and suggest a quarterly remediation schedule.

```endif```

@output(remediation_plan)

## STEP 3: Review Gate

@elicit(confirm, "Review the remediation plan above. Ready to generate the final report?")

## STEP 4: Final Report

Generate a technical debt report for {{codebase_area}} ({{language}}).

Structure the report:
- **Executive Summary**: 2-3 sentences on overall health
- **Severity**: {{assessment}}
- **Findings**: Key debt items with evidence
- **Remediation Plan**: From the analysis above
- **Timeline**: Estimated schedule (include time estimates: {{include_estimates}})

## ARTIFACTS

type: markdown
````

## Validation Rules

### Fatal Errors (parsing must fail)

| Condition | Description |
|-----------|-------------|
| No title | No `# ` heading found |
| No steps | No `## STEP N: Title` heading found |
| Duplicate input name | Two inputs share the same variable name |
| Content too large | Document exceeds 200,000 bytes |
| Empty content | Document is empty or whitespace-only |

### Warnings (parsing succeeds with diagnostics)

| Condition | Description |
|-----------|-------------|
| Non-sequential step numbers | Steps not numbered 1, 2, 3... |
| Malformed input line | List item in INPUTS does not match expected format |
| Unknown artifact type | Type not in the recognized set |
| Undeclared branch variable | Branch condition references undeclared variable |
| Invalid elicit type | Unrecognized `@elicit` type |
| Unrecognized @prompt scheme | Scheme not in `library`, `file`, `mcp` |

## Test Suite

The `tests/` directory contains 23 canonical test vectors for parser and validator implementations. Each vector defines an input `.playbook.md` file and the expected parse result, warnings, or fatal errors. Use these to verify conformance when building or modifying a parser. See `tests/README.md` for the full index.

## File Conventions

- **Extension:** `.md` or `.playbook.md`
- **Encoding:** UTF-8
- **Max size:** 200,000 bytes (200 KB)
- **MIME type:** `text/markdown`
- **Versioning:** Playbook documents do not embed a version number. Implementations parse any document and report errors/warnings for unsupported features.
- **Heading structure:** `#` for title, `##` for sections and top-level steps, `###` for branch sub-steps

## Quick Reference: Regex Patterns

Agents can use these patterns directly for parsing and validation.

```
Title:         ^#\s+(.+)$                                              (first match only)
System:        (?i)^##\s+SYSTEM(\s+PROMPT)?$
Inputs:        (?i)^##\s+INPUTS$
Input line:    ^-\s+`([a-zA-Z][a-zA-Z0-9_]*)`\s+\(([^)]+)\)(?::\s*(.+))?$
Step:          ^##\s+STEP\s+(\d+):\s+(.+)$
Sub-step:      ^###\s+STEP\s+(\d+[a-z]):\s+(.+)$
Variable:      \{\{(\w+)\}\}                                           (for interpolation)
Variable name: [a-zA-Z][a-zA-Z0-9_]*
Artifacts:     (?i)^##\s+(ARTIFACTS|OUTPUT)$
Artifact type: (?i)^type:\s*(\S+)$
@output:       ^@output\((\w+)(?:,\s*extract:"(\w+)")?\)$
@elicit:       ^@elicit\((\w+)(?:,\s*(.+))?\)$
@prompt:       ^@prompt\((.+)\)$
@tool:         ^@tool\((.+)\)$
Branch if:     ^```if\s+(\w+)\s*(==|!=)\s*"([^"]*)"\s*```$
Branch elif:   ^```elif\s+(\w+)\s*(==|!=)\s*"([^"]*)"\s*```$
Branch else:   ^```else```$
Branch endif:  ^```endif```$
```
