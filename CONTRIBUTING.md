# Contributing to Playbook Style

Thank you for your interest in contributing to the Playbook Style specification.

## Ways to Contribute

### Discuss

- **Open an issue** to propose a change, report an ambiguity, or ask a question about the spec
- **Join Discussions** on GitHub for open-ended conversations about direction and design

### Propose Changes

1. Fork the repo and create a branch
2. Make your changes
3. Open a pull request with a clear description of *what* changed and *why*

For substantive spec changes (new directives, new section types, behavioral changes), open an issue first to discuss before writing a PR.

### Write Examples

The [playbook-gallery](https://github.com/PLAYBOOK-MD/playbook-gallery) repo accepts community-contributed playbooks. This is the easiest way to get involved.

### Build Implementations

If you build a parser, executor, or tool that implements the spec, let us know — we'll list it in the ecosystem section.

## Spec Change Process

Changes to the specification follow this process:

1. **Issue** — Describe the problem or opportunity. Include concrete examples.
2. **Discussion** — Maintainers and community discuss the proposal. Design principles (see below) guide decisions.
3. **PR** — If the proposal has consensus, submit a PR against the spec documents.
4. **Review** — At least one maintainer reviews. Spec changes require careful consideration of backwards compatibility.
5. **Merge** — Changes are merged and included in the next tagged spec version.

### What Makes a Good Spec Change

- Solves a real problem encountered in practice (not hypothetical)
- Has at least one concrete use case
- Doesn't break existing valid playbooks
- Aligns with the design principles
- Is the simplest solution that works

### What We'll Push Back On

- Features that require a specific AI provider or platform
- Complexity that doesn't carry its weight in real-world usage
- Changes that make playbooks harder to read as plain text

## Design Principles

These guide all spec decisions:

1. **Plain text over proprietary formats** — Playbooks are markdown files.
2. **Readable before executable** — A non-technical person should understand the workflow.
3. **Convention over configuration** — Heading levels define structure, not metadata.
4. **Progressive complexity** — Zero-directive playbooks are valid.
5. **Implementation freedom** — The spec defines format and semantics, not transport or infrastructure.

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold these standards.

## License

Contributions to the spec are licensed under [CC BY 4.0](LICENSE). By submitting a pull request, you agree to license your contribution under the same terms.
