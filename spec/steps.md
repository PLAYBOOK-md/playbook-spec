# Steps

Steps are the core building blocks of a playbook. Each step represents a single unit of work — typically one AI call — that produces output which flows into subsequent steps.

## Step Heading

Steps are declared with `##` headings matching this pattern:

```regex
^##\s+STEP\s+(\d+):\s+(.+)$
```

```markdown
## STEP 1: Research Phase
## STEP 2: Draft Content
## STEP 3: Polish and Review
```

### Numbering Rules

- Steps must be numbered with sequential integers starting from 1
- Non-sequential numbering (e.g., 1, 2, 5) produces a **warning**, not a fatal error
- There is no upper limit on step count (subject to the 200 KB document size constraint)
- At least one step is **required** (fatal error otherwise)

## Step Content

Everything between the step heading and the next `##` heading (or end of document) is the step's content body — the prompt text sent to the AI.

```markdown
## STEP 1: Research Phase

Research {{topic}} and identify the top 5 key themes
relevant to a {{audience}} audience.

Organize findings by importance and include supporting evidence.
```

Content is trimmed of leading and trailing whitespace. Directive lines (`@output`, `@elicit`, `@prompt`, `@tool`) and branch markers are extracted from the content during parsing — they are not included in the prompt text.

## Variable Interpolation

Use `{{variable_name}}` placeholders in step content. These are resolved at execution time against:

1. **Input values** provided by the user
2. **Input defaults** if no value was provided
3. **Named outputs** captured by `@output` directives in prior steps

### Placeholder Syntax

```
{{name}}              Simple reference
{{name:type}}         With type hint (informational — not enforced at runtime)
{{name:type:default}} With type and inline default
```

If no value, default, or named output exists for a variable, the `{{name}}` placeholder remains as literal text. Implementations may warn about unresolved placeholders.

### Example

```markdown
## INPUTS

- `topic` (string): Subject to research
- `audience` (enum: technical, general, executive): Target audience

## STEP 1: Research

Research {{topic}} for a {{audience}} audience.

## STEP 2: Expand

Expand on the research above, focusing on the themes
most relevant to {{audience}} readers.
```

## Context Accumulation

Each step automatically receives all previous step outputs. The implementation injects prior outputs into the system message (or equivalent context mechanism), so downstream steps have full visibility into earlier results.

This is a key design choice: **steps are not a conversation.** Each step is a fresh AI call with accumulated context, not a multi-turn chat. This makes token costs predictable and steps independently testable.

Authors do not need to manually reference prior outputs — they are available automatically. Named outputs captured with `@output` provide explicit variable access; all other step outputs are available through the accumulated context.

## Step Labels

Each step has a display label:

| Type | Label Format | Example |
|------|-------------|---------|
| Top-level step | Step number as string | `"1"`, `"2"`, `"3"` |
| Branch sub-step | Number + lowercase letter | `"2a"`, `"2b"`, `"3a"` |

Labels are assigned by the parser and used for display purposes. See [Branching](branching.md) for sub-step details.

## Directives in Steps

Steps may contain directives — special annotation lines that modify execution behavior. Each directive must appear on its own line within the step body. See [Directives](directives.md) for the complete reference.

| Directive | Purpose |
|-----------|---------|
| `@output(name)` | Capture this step's output as a named variable |
| `@elicit(type, "prompt")` | Pause and collect human input |
| `@prompt(library:id)` | Prepend content from an external prompt |
| `@tool(conn, name, {args})` | Invoke an external tool (skip AI call) |

## Branching in Steps

Steps may contain conditional blocks that route execution based on variable values. See [Branching](branching.md) for the complete reference.

When a step has branches and no branch condition matches (and there is no `else` block), the step is **skipped** — it produces no output and execution continues to the next step.
