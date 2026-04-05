# Design Principles

These principles guide all decisions about the Playbook Style specification. When evaluating proposals, ask whether they align with these values.

## 1. Plain Text Over Proprietary Formats

A playbook is a `.md` file. It can be created in any text editor, diffed in any version control system, and read on any platform. No binary formats, no required tooling, no visual builder lock-in. The source of truth is always human-writable text.

## 2. Readable Before Executable

A non-technical stakeholder should be able to read a playbook and understand the workflow. If a feature makes playbooks harder to read as plain text, it needs a very strong justification. Clarity beats expressiveness.

## 3. Convention Over Configuration

Structure comes from markdown conventions — heading levels (`#`, `##`, `###`), list formatting, and fenced code blocks. Metadata and configuration are minimal. A reader who knows markdown already knows 90% of the format.

## 4. Progressive Complexity

A playbook with no directives, no branching, and no inputs is valid. It's just a sequence of prompts. Authors add `@output`, `@elicit`, branching, and `@tool` directives only when their workflow needs them. The simplest correct playbook should be trivially simple.

## 5. Implementation Freedom

The spec defines the document format and execution semantics. It does not prescribe:
- Which AI providers or models to use
- How to stream output to users
- How to persist execution state
- What transport protocol to use for tool calls
- How to authenticate users or manage access

Implementations make these decisions. The spec stays focused on what a playbook *is* and what it *means*.

## 6. Solve Real Problems

Every feature in the spec exists because someone needed it in practice — not because it seemed theoretically useful. Proposals must include concrete use cases from real workflows. "Someone might want this someday" is not sufficient justification.

## 7. Backwards Compatibility

Valid playbooks should remain valid. If a spec change would break existing documents, it must go through a deprecation cycle with a clear migration path. Adding new optional features is preferred over changing the meaning of existing syntax.

## 8. One Obvious Way

For any given workflow pattern, there should be one clear way to express it. Avoid introducing multiple syntaxes for the same concept. If two approaches could work, pick the simpler one and commit to it.

## Applying These Principles

When reviewing a proposal, ask:

- Does it make playbooks harder to read as plain text? (Principle 2)
- Can the same thing be achieved with existing features? (Principle 8)
- Does it require a specific platform or provider? (Principle 5)
- Is there a real workflow that needs this today? (Principle 6)
- Does it break any existing valid playbook? (Principle 7)
- Is this the simplest version of the feature that solves the problem? (Principle 4)

If the answer to any of these raises concern, the proposal needs refinement.
