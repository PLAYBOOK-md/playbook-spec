# PLAYBOOK.md Conformance Test Suite

Canonical test vectors for PLAYBOOK.md parsers. Any conforming implementation (TypeScript, Python, Go, or otherwise) must produce identical parse results for these inputs.

## Purpose

These test vectors are the **source of truth** for parser conformance. They are derived directly from the PLAYBOOK.md specification (`spec/`), not from any single implementation. When a parser disagrees with a test vector, the test vector (and the spec it encodes) is authoritative.

## Structure

```
tests/
  README.md                          This file
  vectors/
    basic/                           Minimal and foundational playbooks
    inputs/                          Input declaration parsing
    directives/                      @output, @prompt, @elicit, @tool, @artifact
    branching/                       Conditional logic (if/elif/else/endif)
    edge-cases/                      Whitespace, unknown sections, missing optional fields
    invalid/                         Documents that MUST fail parsing (fatal errors)
```

## File Naming Convention

Each test case is a pair (or triple) of files sharing a base name:

| Suffix | Purpose |
|--------|---------|
| `.playbook.md` | The input document to parse |
| `.expected.json` | The expected `ParseResult` for valid documents |
| `.errors.json` | The expected `ParseResult` for invalid documents |

A test case has either `.expected.json` (valid) or `.errors.json` (invalid), never both.

## Expected JSON Format

### Valid documents (`.expected.json`)

```json
{
  "valid": true,
  "definition": {
    "title": "...",
    "description": "...",
    "system_prompt": "...",
    "inputs": [],
    "steps": [],
    "artifact_type": "..."
  },
  "warnings": [],
  "errors": []
}
```

### Invalid documents (`.errors.json`)

```json
{
  "valid": false,
  "definition": null,
  "errors": [
    { "line": 1, "message": "..." }
  ],
  "warnings": []
}
```

The JSON structure matches the Go SDK's `ParseResult` type with an added `valid` convenience field. Field names use `snake_case` as defined in the Go struct tags.

## Key Types Reference

From the Go SDK (`playbook.go`):

- **`ParseResult`**: `{ definition, warnings, errors }`
- **`PlaybookDefinition`**: `{ title, description, system_prompt, inputs, steps, artifact_type }`
- **`InputDef`**: `{ name, type, default, options, description, required, line }`
- **`Step`**: `{ number, label, title, content, prompt_ref, output_var, extract_field, elicitation, tool_call, is_branching, branches, line }`
- **`Branch`**: `{ condition, steps }`
- **`Condition`**: `{ variable, operator, value, source }`
- **`ElicitationDef`**: `{ type, prompt, options }`
- **`StepToolCall`**: `{ connection_name, tool_name, arguments }`
- **`PromptReference`**: `{ prompt_id }`

## How to Run

### General approach

For each `.playbook.md` file:

1. Read the file content as UTF-8 text
2. Pass it to your parser's `Parse()` / `parse()` function
3. Serialize the result to JSON
4. Compare against the corresponding `.expected.json` or `.errors.json`

### Comparison rules

- **`valid`**: `true` if `definition` is non-null and `errors` is empty
- **`line` fields**: Line numbers are 1-based. Implementations MAY omit line numbers (treat `0` or absent as "not specified"). Test runners SHOULD compare line numbers when both sides provide them, and skip the comparison when either side omits them.
- **`content` fields**: Trimmed of leading/trailing whitespace. Normalize line endings to `\n` before comparison.
- **`warnings`**: Order does not matter. Compare as sets by message content.
- **`errors`**: Order does not matter for invalid documents. At least the listed errors must be present; additional errors are acceptable.
- **Omitted fields**: Fields with `omitempty` in the Go struct tags (`description`, `system_prompt`, `default`, `options`, `prompt_ref`, `output_var`, `extract_field`, `elicitation`, `tool_call`, `branches`, `artifact_type`, `line`) may be absent from JSON when empty/zero. Treat absent and empty/zero as equivalent.

## Coverage Matrix

| Category | Test Case | Spec Section | What It Tests |
|----------|-----------|-------------|---------------|
| basic | minimal | format.md | Smallest valid playbook (title + 1 step) |
| basic | full-featured | all | Every spec feature in one document |
| basic | multi-step | steps.md | 3 steps with context accumulation |
| inputs | all-types | inputs.md | string, text, number, boolean, enum types |
| inputs | required-optional | inputs.md | Required vs optional (default) inputs |
| inputs | enum-with-options | inputs.md | Enum option parsing, aliases (select, choice) |
| directives | output-basic | directives.md | @output(var_name) basic capture |
| directives | output-field | directives.md | @output(var, extract:"field") |
| directives | prompt-ref | directives.md | @prompt(scheme:id) reference |
| directives | elicit | directives.md | @elicit types (input, confirm, select) |
| directives | output-typed | directives.md | @output(var: type) typed capture (number, string, json) |
| directives | output-enum | directives.md | @output(var: enum, "opt1", "opt2") enum capture |
| directives | output-typed-extract | directives.md | @output(var: type, extract:"field") typed + extract |
| directives | tool-call | directives.md | @tool(conn, name, {args}) |
| directives | artifact | artifacts.md | Artifact type declaration |
| branching | if-else | branching.md | Simple if/else branch |
| branching | if-elseif-else | branching.md | Full if/elif/else chain |
| branching | nested-steps | branching.md | Multiple sub-steps within branches |
| edge-cases | extra-whitespace | format.md | Leading blanks, trailing whitespace |
| edge-cases | unrecognized-sections | format.md | Unknown ## headings skipped silently |
| edge-cases | no-description | format.md | Title with no description text |
| edge-cases | empty-steps | steps.md | Step heading with empty body |
| invalid | missing-title | format.md | No # heading -- fatal error |
| invalid | no-steps | format.md | Title but no ## STEP -- fatal error |
| invalid | duplicate-step-numbers | steps.md | Two steps with same number -- warning (valid, not fatal) |
| invalid | gap-in-step-numbers | steps.md | Non-sequential numbering -- warning (valid, not fatal) |

## Notes on the `invalid/` Directory

The `invalid/` directory contains two categories of test cases:

1. **Fatal errors** (`missing-title`, `no-steps`): These use `.errors.json` and have `"valid": false` with `"definition": null`. Parsing MUST fail for these documents.

2. **Warning cases** (`duplicate-step-numbers`, `gap-in-step-numbers`): These use `.expected.json` and have `"valid": true` with a populated `definition` plus non-empty `warnings`. Per the spec, non-sequential step numbers produce a **warning**, not a fatal error. Parsing succeeds but implementations SHOULD report diagnostics. These are placed in `invalid/` because they violate spec recommendations, even though parsing does not fail.

## License

This test suite is part of the PLAYBOOK.md specification and is licensed under CC BY 4.0.
