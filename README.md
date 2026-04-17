# PLAYBOOK.md

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![GitHub stars](https://img.shields.io/github/stars/PLAYBOOK-MD/playbook-spec)](https://github.com/PLAYBOOK-MD/playbook-spec/stargazers)
[![Spec](https://img.shields.io/badge/spec-v0.1.0--draft-blue)](https://github.com/PLAYBOOK-MD/playbook-spec)
[![Website](https://img.shields.io/badge/website-playbook.style-FF5F1F)](https://playbook.style)

**An open specification for multi-step AI workflows written in plain markdown.**

Playbooks are structured documents that define repeatable AI workflows. Each playbook chains multiple prompts together with input variables, conditional branching, human-in-the-loop checkpoints, and context accumulation across steps.

The format is deliberately simple: if you can write a README, you can write a playbook.

## Why a Specification?

AI workflows are proliferating across tools, platforms, and frameworks. Most are locked into proprietary formats, visual builders, or complex DSLs. PLAYBOOK.md proposes a different approach:

- **Markdown-native** — diffs cleanly in version control, readable without special tooling
- **Human-readable** — someone can understand what a workflow does just by reading it
- **Tool-agnostic** — any platform can parse and execute playbooks
- **Progressively complex** — a linear 3-step pipeline is as valid as a branching workflow with elicitation and tool calls

## Ecosystem

| Repo | Description |
|------|-------------|
| **[playbook-spec](https://github.com/PLAYBOOK-MD/playbook-spec)** | This repo — the specification |
| **[playbook-gallery](https://github.com/PLAYBOOK-MD/playbook-gallery)** | Curated example playbooks |
| **[playbook-schema](https://github.com/PLAYBOOK-MD/playbook-schema)** | TypeScript types + JSON Schema |
| **[playbook-integrations](https://github.com/PLAYBOOK-MD/playbook-integrations)** | SDKs (TS, Python, Go), MCP server, Agent Skills, Editor configs |
| **[playbook-playground](https://github.com/PLAYBOOK-MD/playbook-playground)** | Web-based editor and validator |
| **[playbook-cli](https://github.com/PLAYBOOK-MD/playbook-cli)** | CLI tool for validating, parsing, and scaffolding playbooks |
| **[playbook-vscode](https://github.com/PLAYBOOK-MD/playbook-vscode)** | VS Code extension with syntax highlighting and validation |
| **[playbook-action](https://github.com/PLAYBOOK-MD/playbook-action)** | GitHub Action for PR validation |
| **[playbook-docs](https://github.com/PLAYBOOK-MD/playbook-docs)** | Documentation site (docs.playbook.style) |
| **[playbook-site](https://github.com/PLAYBOOK-MD/playbook-site)** | Landing page (playbook.style) |

## Quick Example

```markdown
# Code Review Pipeline

Automated code review with configurable focus area.

## SYSTEM

You are a senior code reviewer. Be specific — cite line numbers and suggest fixes.

## INPUTS

- `code` (text): Code to review
- `focus` (enum: security, performance, readability): Review focus

## STEP 1: Analysis

Analyze the following code with a focus on {{focus}}:

{{code}}

Identify the top 3 issues.

@output(issues)

## STEP 2: Recommendations

For each issue identified, provide:
1. Severity (critical/warning/info)
2. Specific fix with code example
3. Rationale

## ARTIFACTS

type: markdown
```

## Specification

The full specification lives in [`spec/`](spec/):

| Document | Contents |
|----------|----------|
| [Format](spec/format.md) | Document structure, sections, constraints |
| [Inputs](spec/inputs.md) | Variable declarations, types, defaults, enums |
| [Steps](spec/steps.md) | Step structure, numbering, variable interpolation |
| [Directives](spec/directives.md) | `@output`, `@elicit`, `@prompt`, `@tool` |
| [Branching](spec/branching.md) | Conditional logic with `if`/`elif`/`else`/`endif` |
| [Execution](spec/execution.md) | Runtime semantics, context accumulation, state machine |
| [Artifacts](spec/artifacts.md) | Output format declarations |

## Examples

See [`examples/`](examples/) for complete playbooks demonstrating common patterns:

- [Linear Pipeline](examples/content-pipeline.md) — straightforward multi-step chain
- [Branching on Input](examples/adaptive-review.md) — conditional paths based on user input
- [Branching on AI Output](examples/issue-triage.md) — routing based on AI classification
- [Human-in-the-Loop](examples/decision-matrix.md) — elicitation checkpoints and tool calls

## Test Suite

The [`tests/`](tests/) directory contains 23 canonical test vectors for validating parser and validator implementations. Each test vector defines an input `.playbook.md` file and the expected parse result or error. See [`tests/README.md`](tests/README.md) for details.

## Tooling

### CLI

```bash
npm install -g @playbook-md/cli
```

### VS Code Extension

```bash
code --install-extension playbook-md.playbook-vscode
```

### GitHub Action

```yaml
- uses: PLAYBOOK-MD/playbook-action@v1
```

## Design Principles

1. **Plain text over proprietary formats.** A playbook is a `.md` file. No binary blobs, no JSON schemas required for authoring, no visual builder lock-in.

2. **Readable before executable.** A non-technical stakeholder should be able to read a playbook and understand the workflow. The format favors clarity over expressiveness.

3. **Convention over configuration.** Heading levels (`#`, `##`, `###`) define structure. Directives (`@output`, `@elicit`) are minimal annotations, not a programming language.

4. **Progressive complexity.** A playbook with zero directives is valid. Authors add branching, elicitation, and tool calls only when needed.

5. **Implementation freedom.** The spec defines the document format and execution semantics. How an implementation streams output, manages state, or connects to AI providers is not prescribed.

## Execution Targets

Playbooks run in a variety of environments: CLI tools, MCP servers embedded in editors, web runners, and autonomous cloud sessions like [Claude Code Routines](https://code.claude.com/docs/en/routines). The spec is deliberately silent on target-specific concerns (how an implementation streams, pauses, or resolves external prompt references) — see principle 5 above.

Two implementation-level concerns worth flagging for target authors:

- **Unattended execution.** `@elicit` directives cannot pause a run with no human in the loop. Implementations targeting autonomous execution (schedulers, batch runners, Claude Code Routines) should document fallback defaults — see the [Claude Code Routines guide](https://docs.playbook.style/guides/claude-code-routines/#elicit-in-autonomous-runs) for a working convention. A proposed optional `default:` argument for `@elicit` is tracked at [#2](https://github.com/PLAYBOOK-MD/playbook-spec/issues/2).
- **Extraction fidelity.** `@output(extract:"field")` implementations without a deterministic JSON parser in the loop (pure-LLM execution) occasionally miss the target field. Prefer `enum`-typed outputs for high-stakes branching when extraction is not guaranteed.

## Status

**Draft v0.1** — Extracted from a production implementation ([Promptmark](https://promptmark.ai)) where playbooks have been running in production since March 2026. This specification captures the format as-implemented and proposes it as an open standard.

The spec is stable for the core format (sections, inputs, steps, branching, directives). Extensions for advanced orchestration (parallel steps, loops, sub-playbook calls) are under consideration for future versions.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide. The short version:

- Open an issue to discuss format changes
- Submit a PR with example playbooks to the [gallery](https://github.com/PLAYBOOK-MD/playbook-gallery)
- Build an implementation and share it

## License

[CC BY 4.0](LICENSE) — Use it, build on it, just give credit.
