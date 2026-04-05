# Playbook Style Specification

**Version: 0.1.0 (Draft)**

This directory contains the Playbook Style specification — an open format for defining multi-step AI workflows in plain markdown.

## Specification Documents

Read these in order for a complete understanding, or jump to the section you need:

### Core Format

1. **[Format](format.md)** — Document structure, required/optional sections, constraints, and fatal vs. warning errors. Start here.

2. **[Inputs](inputs.md)** — How to declare input variables with types, defaults, enum options, and validation rules.

3. **[Steps](steps.md)** — Step structure, sequential numbering, content body, and variable interpolation with `{{name}}` placeholders.

### Directives

4. **[Directives](directives.md)** — Annotations within steps that control execution behavior:
   - `@output(name)` — capture step output as a named variable
   - `@elicit(type, "prompt")` — pause for human input
   - `@prompt(library:id)` — reference an external prompt
   - `@tool(connection, tool_name, {args})` — invoke an external tool

### Control Flow

5. **[Branching](branching.md)** — Conditional execution with `` ```if``` ``/`` ```elif``` ``/`` ```else``` ``/`` ```endif``` `` blocks, sub-steps, and evaluation rules.

### Runtime

6. **[Execution](execution.md)** — How a conforming implementation should execute a playbook: sequential steps, context accumulation, state machine, pause/resume semantics, and streaming.

7. **[Artifacts](artifacts.md)** — Declaring expected output formats for the final result.

## Conformance Levels

An implementation may support the spec at different levels:

| Level | Requirements |
|-------|-------------|
| **Parse** | Can parse a playbook document into a structured representation. Reports fatal errors and warnings per spec. |
| **Execute (Basic)** | Parse + can execute linear playbooks (no branching, no directives). Steps run sequentially with context accumulation. |
| **Execute (Full)** | Basic + branching, all directives (`@output`, `@elicit`, `@prompt`, `@tool`), pause/resume. |
| **Stream** | Full + real-time token streaming during step execution. |

## Notation Conventions

Throughout the spec:

- `REQUIRED` / `OPTIONAL` indicate whether a section or field must be present
- Regex patterns are shown in code blocks and use Go/PCRE syntax
- `{{variable}}` denotes template interpolation at execution time
- Examples use fenced code blocks with `markdown` language hints
- "Fatal error" means parsing must fail; "Warning" means parsing succeeds with a diagnostic
