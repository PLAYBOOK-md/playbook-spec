# Document Format

A playbook is a UTF-8 encoded markdown document (`.md` file) that defines a multi-step AI workflow. The parser reads top-to-bottom, matching headings and directives by regex. Steps execute sequentially with context accumulation — each step receives all prior step outputs automatically.

## Document Structure

A playbook contains sections identified by `##` headings. The title uses a single `#` heading. Sections can appear in any order relative to each other, but steps must be numbered sequentially starting from 1.

```
# Title                          REQUIRED   First # heading in document
Description text                 OPTIONAL   Text between title and first ##
## SYSTEM                        OPTIONAL   System prompt for all steps
## INPUTS                        OPTIONAL   Input variable declarations
## STEP 1: Title                 REQUIRED   At least one step
## STEP 2: Title                 ...        Sequential numbering
## STEP N: Title                 ...        No upper limit (subject to size constraint)
## ARTIFACTS                     OPTIONAL   Expected output format
```

Unrecognized `##` headings are silently skipped.

## Title

**REQUIRED.** The first level-1 heading (`# `) in the document.

```markdown
# Code Review Pipeline
```

Rules:
- Must use a single `#` (not `##` or deeper)
- Must appear before any `##` section
- Leading blank lines before the title are skipped
- Trailing whitespace is trimmed
- **Fatal error** if no `# ` heading is found

## Description

**OPTIONAL.** Plain text between the title and the first `##` heading.

```markdown
# Code Review Pipeline

Automated code review with security focus and actionable feedback.

## INPUTS
```

The description is metadata — it is not sent to the AI during execution. If no text exists between the title and the first section, the description is empty.

## System Prompt

**OPTIONAL.** Declared with the heading `## SYSTEM` or `## SYSTEM PROMPT` (case-insensitive).

```markdown
## SYSTEM

You are a senior code reviewer. Focus on correctness, security, and maintainability.
Be specific — cite line numbers and suggest fixes.
```

Content: all lines from after the heading until the next `##` heading, joined and trimmed.

The system prompt is applied to every step during execution. It provides consistent persona and behavioral instructions across the entire workflow. If duplicated, the last occurrence wins.

## Steps

**REQUIRED.** At least one step must be present. See [Steps](steps.md) for the full step specification.

## Inputs

**OPTIONAL.** See [Inputs](inputs.md) for the full input specification.

## Artifacts

**OPTIONAL.** See [Artifacts](artifacts.md) for the full artifacts specification.

## Constraints

| Constraint | Value |
|------------|-------|
| Encoding | UTF-8 |
| Maximum content length | 200,000 bytes (200 KB) |
| Minimum steps | 1 |
| Title | Required (fatal error if missing) |
| Step numbering | Sequential integers from 1 |

## Error Classification

Errors during parsing fall into two categories:

### Fatal Errors

Parsing must fail. The document cannot be executed.

| Condition | Description |
|-----------|-------------|
| No title | No `# ` heading found in the document |
| No steps | No `## STEP N: Title` heading found |
| Duplicate input name | Two inputs share the same variable name |
| Content too large | Document exceeds 200,000 bytes |
| Empty content | Document is empty or whitespace-only |

### Warnings

Parsing succeeds, but the implementation should report diagnostics.

| Condition | Description |
|-----------|-------------|
| Non-sequential step numbers | Steps are not numbered 1, 2, 3... |
| Malformed input line | A list item in INPUTS doesn't match the expected format |
| Unknown artifact type | Artifact type is not in the recognized set |
| Undeclared branch variable | A branch condition references a variable not declared as input or output |
| Invalid elicit type | An `@elicit` directive uses an unrecognized type |

## MIME Type

Playbook files use the standard `.md` extension and `text/markdown` MIME type. Implementations may additionally recognize `.playbook.md` as a convention for distinguishing playbook files from regular markdown.

## Versioning

This spec follows semantic versioning. The format version is declared in the [spec README](README.md). Playbook documents do not embed a version number — implementations should attempt to parse any document and report errors/warnings for unsupported features.
