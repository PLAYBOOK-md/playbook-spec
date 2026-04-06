# Directives

Directives are special annotation lines within steps (or sub-steps) that control execution behavior. Each directive must appear on its own line. Directive lines are extracted during parsing and are not included in the prompt text sent to the AI.

## @output — Capture Step Output

Captures the AI response from a step as a named variable, making it available to downstream steps and branch conditions.

### Syntax

```
@output(varname)
@output(varname, extract:"field")
```

### Regex

```regex
^@output\((\w+)(?:,\s*extract:"(\w+)")?\)$
```

### Basic Capture

Stores the full AI response as a named variable:

```markdown
## STEP 1: Classify

Classify this issue as "bug", "feature", or "question": {{issue}}

@output(issue_type)
```

After execution, `issue_type` contains the full response text from Step 1.

### Field Extraction

Instructs the implementation to extract a specific field from a JSON object in the response:

```markdown
## STEP 1: Analyze

Analyze the severity. Include a JSON object with your assessment:
{"level": "critical", "confidence": 0.9}

@output(severity, extract:"level")
```

The implementation scans the response (bottom-up) for a JSON object containing the specified field. If extraction fails, the full response is stored as a fallback.

### Rules

- One `@output` per step (last one wins if duplicated)
- Variable name pattern: `\w+` (letters, digits, underscores)
- Extract field pattern: `\w+`
- Named outputs are available to:
  - Branch conditions in subsequent steps
  - `{{varname}}` interpolation in subsequent steps
  - Directive arguments (e.g., `@tool` JSON)

---

## @elicit — Human-in-the-Loop Input

Pauses execution and collects input from a human operator. This enables review checkpoints, approval gates, and interactive decision points within a workflow.

### Syntax

```
@elicit(input, "prompt text")
@elicit(confirm, "question?")
@elicit(select, "prompt", "option1", "option2", "option3")
```

### Regex

```regex
^@elicit\((\w+)(?:,\s*(.+))?\)$
```

### Types

| Type | UI Element | Response |
|------|-----------|----------|
| `input` | Free-text field | User-entered text |
| `confirm` | Yes/No buttons | `"yes"` or `"no"` |
| `select` | Dropdown / radio buttons | Selected option string |

Type matching is case-insensitive. Unrecognized types produce a **warning** and the directive is ignored.

### Argument Parsing

All arguments after the type must be **double-quoted strings**. Single quotes are not recognized.

For `select`: the first quoted string is the prompt, and all subsequent quoted strings are individual options.

```markdown
# Correct
@elicit(select, "Which framework?", "React", "Vue", "Svelte")

# Wrong — options must be separate quoted strings
@elicit(select, "Which framework?", "React, Vue, Svelte")
```

### Behavior

1. The step enters `awaiting_input` status
2. Execution pauses until the user responds
3. The response is stored as `__elicit_step_N` in named outputs (where N is the step number)

**Elicit-only steps:** If a step contains only `@elicit` (and optionally `@output`), the user's response becomes the step output directly — no AI call is made.

```markdown
## STEP 3: Choose Direction

@elicit(select, "Which approach?", "Conservative", "Aggressive", "Balanced")
@output(approach)
```

**Mixed steps:** If a step has both an `@elicit` and prompt content, the AI call runs after the user responds. The user's response is available as context.

### Example

```markdown
## STEP 2: Review Gate

@elicit(confirm, "The analysis above looks correct. Proceed to recommendations?")

## STEP 3: Get Feedback

@elicit(input, "What specific aspects should the revision focus on?")
```

---

## @prompt — External Prompt Reference

References prompt content from an external source and prepends it to the step's prompt text during execution. Supports multiple resolution schemes and dynamic variable-based prompts.

### Syntax

```
@prompt(scheme:identifier)
@prompt(identifier)
@prompt({{variable}})
```

### Regex

```regex
^@prompt\((.+)\)$
```

The argument is parsed in stages:

1. **Variable reference:** If the argument matches `^\{\{(\w+)\}\}$`, resolve as a dynamic prompt from the named variable's content.
2. **Scheme reference:** If the argument contains `:`, split on the first `:`. Left side is the scheme, right side is the identifier.
3. **Bare identifier:** Otherwise, treat as a bare identifier. The implementation decides how to resolve it.

### Standard Schemes

| Scheme | Identifier Pattern | Resolves To |
|--------|-------------------|-------------|
| `library` | `[a-zA-Z0-9\-]+` | Prompt content from a prompt management system |
| `file` | Relative path (no `..`, no absolute paths) | Content of a local file relative to the playbook |
| `mcp` | `server/prompt-name` | MCP prompt resource from a connected server |

Implementations MAY support additional custom schemes. Unrecognized schemes produce a **warning** and the directive is ignored (the step runs with its own content only).

### Variable Interpolation in Identifiers

Scheme identifiers support `{{variable}}` interpolation, resolved at execution time:

```markdown
@prompt(library:{{prompt_id}})
@prompt(file:prompts/{{review_type}}-criteria.md)
@prompt(mcp:{{server}}/{{prompt_name}})
```

### Dynamic Prompts from Variables

The `{{variable}}` form uses a variable's content directly as the prepended prompt text. The variable must be a declared input or a prior `@output` capture.

```markdown
## STEP 1: Generate Review Criteria

You are a prompt engineer. Write a detailed code review prompt
that checks for {{language}}-specific security vulnerabilities.

@output(review_prompt)

## STEP 2: Execute Review

@prompt({{review_prompt}})

Apply the review criteria above to the following code:
{{code}}
```

**Execution:** Step 1 runs, the AI generates review criteria, stored as `review_prompt`. Step 2 runs, the content of `review_prompt` is prepended to the step's prompt text. The AI for Step 2 sees the generated criteria followed by "Apply the review criteria above to the following code:".

This enables meta-prompting chains where one step generates the instructions for a subsequent step.

### Behavior

- Resolved content is **prepended** to the step's prompt text, separated by a blank line
- One `@prompt` per step (last one wins if duplicated)
- Resolution happens at step execution time (always the latest version for external sources)

### Failure Behavior

| Source | On Failure |
|--------|-----------|
| `scheme:identifier` | **Warning** — step runs with its own content only |
| Bare identifier | Implementation decides |
| `{{variable}}` | **Error** if variable is undefined; empty content if variable is empty string |

### Examples

**Prompt library reference (prompt management system):**

```markdown
## STEP 1: Security Review

@prompt(library:owasp-review-criteria)

Apply the review criteria above to the following code:
{{code}}
```

**Local file reference:**

```markdown
## STEP 1: Review

@prompt(file:prompts/review-template.md)

Review the following {{language}} code:
{{code}}
```

**MCP prompt resource:**

```markdown
## STEP 1: Analyze

@prompt(mcp:prompt-server/code-analysis)

Analyze this codebase for {{concern}}.
```

**User-supplied prompt via input:**

```markdown
## INPUTS

- `custom_prompt` (text): Analysis instructions to apply
- `code` (text): Code to analyze

## STEP 1: Analyze

@prompt({{custom_prompt}})

Apply the analysis above to:
{{code}}
```

**Meta-prompting chain (AI generates the prompt for a subsequent step):**

```markdown
## STEP 1: Generate Prompt

Based on the review type "{{review_type}}", generate a focused
review prompt with specific criteria and a scoring rubric.

@output(generated_prompt)

## STEP 2: Apply Review

@prompt({{generated_prompt}})

Score the following code against the criteria above:
{{code}}
```

### Implementation Notes

- **Scheme registry:** Implementations SHOULD document which schemes they support. The spec defines `library`, `file`, and `mcp` as standard schemes.
- **File scheme security:** Implementations MUST restrict `file:` paths to prevent directory traversal. Only relative paths from the playbook's location should be resolved. Absolute paths and `..` traversal MUST be rejected.
- **Variable timing:** `{{variable}}` in `@prompt` is resolved at step execution time, after all prior steps have completed.
- **Caching:** Implementations MAY cache resolved prompts within a single execution run.

---

## @tool — External Tool Invocation

Directly invokes an external tool (via MCP or equivalent protocol). When a step has a `@tool` directive, the AI call is **skipped entirely** — the tool result becomes the step output.

### Syntax

```
@tool(connection, tool_name)
@tool(connection, tool_name, {"key": "value"})
@tool("Connection Name", tool_name, {"query": "{{variable}}"})
```

### Regex

```regex
^@tool\((.+)\)$
```

### Arguments

| Position | Required | Description |
|----------|----------|-------------|
| 1 | Yes | Connection name — identifies the tool server. Bare identifier or double-quoted string. |
| 2 | Yes | Tool name — the specific tool to invoke. Bare identifier. |
| 3 | No | JSON arguments — an object passed to the tool. May contain `{{variable}}` placeholders. |

### Parsing Rules

- The argument string is split on the first two commas
- Everything after the second comma is treated as JSON (preserving internal commas)
- Connection and tool names can be bare identifiers (letters, digits, underscores, hyphens, dots) or double-quoted strings for names with spaces

### Behavior

1. Template variables (`{{var}}`) in JSON arguments are rendered before execution
2. The implementation connects to the named tool server and invokes the tool
3. The tool result becomes the step output
4. If the step also has `@output(varname)`, the result is stored as a named variable
5. The step does not make an AI call

### Combines With

- `@output` — capture tool result as named variable
- Breakpoints — execution can still pause after tool completion

### Does Not Combine With

- `@prompt` — tool steps skip AI, so prepending prompt content has no effect
- `@elicit` — tool execution is deterministic; human input before a tool call should be a separate step

### Example

```markdown
## STEP 2: Fetch Data

@tool(analytics_server, get_metrics, {"period": "{{timeframe}}", "metric": "latency"})
@output(metrics_data)

## STEP 3: Analyze

Based on the metrics data, identify trends and anomalies.
```

### Implementation Note

The `@tool` directive assumes an implementation of the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) or equivalent tool execution framework. Implementations without tool support may skip `@tool` steps or report them as errors. The spec defines the syntax and execution semantics but not the transport protocol for tool invocation.

---

## Directive Interaction Summary

| Directive | AI Call? | Can combine with |
|-----------|----------|-----------------|
| `@output` | Yes (captures response) | All other directives |
| `@elicit` | Only if step has other content | `@output` |
| `@prompt` | Yes (prepends content) | `@output`, `@elicit` |
| `@tool` | No (tool result is output) | `@output` |

When multiple directives appear in the same step, they are processed in this order:

1. `@tool` — if present, skip AI call entirely
2. `@elicit` — if present and no response yet, pause execution
3. `@prompt` — prepend referenced content to prompt
4. `@output` — capture the step result (from AI, tool, or elicitation)
