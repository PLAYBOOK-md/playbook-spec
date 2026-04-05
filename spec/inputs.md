# Inputs

Inputs declare the variables that a playbook accepts before execution. They appear in the `## INPUTS` section and define the data a user (or calling system) must provide.

## Section Heading

```markdown
## INPUTS
```

Case-insensitive match. All lines until the next `##` heading are parsed as input definitions.

## Input Line Format

Each input is a markdown list item matching this pattern:

```
- `name` (type): Description
- `name` (type: default_value): Description
- `name` (enum: option1, option2, option3): Description
```

### Regex

```regex
^-\s+`([a-zA-Z][a-zA-Z0-9_]*)`\s+\(([^)]+)\)(?::\s*(.+))?$
```

### Components

| Part | Format | Required |
|------|--------|----------|
| List marker | `- ` | Yes |
| Name | `` `name` `` in backticks | Yes |
| Type spec | `(type)` or `(type: value)` in parentheses | Yes |
| Description | `: text` after closing parenthesis | No |

## Variable Names

Pattern: `[a-zA-Z][a-zA-Z0-9_]*`

- Must start with a letter
- May contain letters, digits, and underscores
- Case-sensitive

| Example | Valid |
|---------|-------|
| `topic` | Yes |
| `max_count` | Yes |
| `userInput2` | Yes |
| `_reserved` | No — starts with underscore |
| `2fast` | No — starts with digit |
| `my-var` | No — contains hyphen |
| `has spaces` | No — contains space |

## Types

| Canonical Type | Aliases | Resolves To |
|----------------|---------|-------------|
| `string` | *(any unrecognized type)* | Single-line text input |
| `text` | — | Multi-line textarea |
| `number` | `num`, `int`, `float` | Numeric input |
| `boolean` | `bool` | Toggle / checkbox |
| `enum` | `select`, `choice` | Dropdown with predefined options |

Type matching is case-insensitive. Any unrecognized type string defaults to `string`.

## Defaults

A default value is specified after a colon inside the parentheses:

```markdown
- `language` (string: Go): Programming language
- `count` (number: 5): How many items to generate
- `verbose` (boolean: true): Include detailed output
```

An input with a non-empty default is **not required** — if the user provides no value at execution time, the default is used.

An input without a default is **required** — execution should not proceed without a value.

## Enum Options

For enum types, the value after the colon is a comma-separated list of options:

```markdown
- `tone` (enum: formal, casual, academic): Writing tone
- `focus` (select: security, performance, readability): Review focus area
```

Each option is trimmed of whitespace. Empty options (from trailing commas) are discarded.

Enum inputs do not support a separate default value — the colon-separated value is always parsed as the options list. Implementations may choose to pre-select the first option as a default in their UI.

## Complete Example

```markdown
## INPUTS

- `technology` (string): Technology to evaluate
- `criteria` (text): Evaluation criteria (multi-line)
- `max_alternatives` (number: 3): Number of alternatives to suggest
- `depth` (enum: quick, thorough, comprehensive): Analysis depth
- `include_cost` (boolean: true): Include cost analysis
```

## Validation Rules

| Rule | Severity |
|------|----------|
| Duplicate input name | **Fatal error** — parsing fails |
| Line starts with `-` or `*` but doesn't match format | Warning |
| Lines not starting with `-` or `*` | Silently skipped |

## Runtime Behavior

At execution time, input values are available for:

1. **Variable interpolation** in step content via `{{name}}` placeholders
2. **Branch conditions** via `if name == "value"` comparisons
3. **Directive arguments** in `@tool` JSON via `{{name}}` placeholders

If no value or default exists for a referenced variable, the `{{name}}` placeholder remains as literal text in the rendered output. Implementations may choose to warn about unresolved placeholders.
