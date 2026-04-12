# Artifacts

The Artifacts section declares the expected output format of the playbook's final result. This metadata helps implementations render, download, or route the output appropriately.

## Section Heading

```markdown
## ARTIFACTS
```

Also recognized: `## OUTPUT` (case-insensitive).

## Type Declaration

The artifact type is declared with a `type:` line:

```markdown
## ARTIFACTS

type: markdown
```

### Regex

```regex
(?i)^type:\s*(\S+)$
```

Type matching is case-insensitive.

### Dynamic Type

The artifact type supports `{{variable}}` interpolation, resolved at execution time:

```markdown
## ARTIFACTS

type: {{output_format}}
```

The variable must be a declared input or a prior `@output` capture. The resolved value must be one of the valid types listed below.

## Valid Types

| Type | Description | Typical Use |
|------|-------------|-------------|
| `markdown` | Formatted text with headings, lists, code blocks | Reports, analyses, documentation |
| `json` | Structured JSON data | Data pipelines, API responses, configurations |
| `mermaid` | Mermaid diagram syntax | Flowcharts, sequence diagrams, architecture diagrams |
| `chartjs` | Chart.js configuration object | Data visualizations, dashboards |
| `html_css` | HTML with embedded or linked CSS | Rich formatted output, email templates |
| `javascript` | JavaScript code | Generated scripts, automation code |
| `typescript` | TypeScript code | Generated typed code |

Unrecognized types produce a **warning** listing all valid types.

## Behavior

The artifact type is **metadata only** — it does not change how steps execute. It informs the implementation about how to handle the final output:

- **Rendering**: A `mermaid` artifact could be rendered as a diagram; `markdown` as formatted HTML
- **Download**: The type suggests an appropriate file extension (`.md`, `.json`, `.html`, etc.)
- **Validation**: Implementations may validate that the final step output conforms to the declared type
- **Routing**: Output delivery systems can format payloads based on the artifact type
- **Dynamic resolution**: If the type contains `{{variable}}`, it is resolved at execution time from inputs or named outputs

## Default

If no `## ARTIFACTS` section is present, implementations should treat the output as untyped text. The recommended default rendering is plain text or markdown.

## Examples

### Markdown Report

```markdown
## ARTIFACTS

type: markdown
```

### JSON Data Pipeline

```markdown
## ARTIFACTS

type: json
```

### Diagram Generation

```markdown
## ARTIFACTS

type: mermaid
```

### Dynamic Format Selection

```markdown
## INPUTS

- `output_format` (enum: markdown, json, html_css): Desired format

## STEP 1: Generate

Create the report content.

## ARTIFACTS

type: {{output_format}}
```

## Future Considerations

- **Multiple artifacts**: A playbook could declare multiple output artifacts from different steps
- **Custom types**: Allow implementation-specific artifact types beyond the standard set
- **Schema validation**: For `json` artifacts, an optional JSON Schema could validate output structure
