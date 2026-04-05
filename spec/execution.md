# Execution Model

This document defines how a conforming implementation should execute a playbook. It covers the step execution lifecycle, context accumulation strategy, state machine, pause/resume semantics, and streaming.

## Overview

Playbook execution is **sequential and single-agent**. Steps run one at a time, in order, each producing output that flows into subsequent steps. There is no parallel execution, no multi-agent coordination, and no conversation-style multi-turn within a step.

This design makes playbooks:
- **Predictable** — token costs scale linearly with step count
- **Debuggable** — each step can be inspected independently
- **Testable** — the execution core doesn't depend on streaming or HTTP

## Step Execution Lifecycle

For each step in sequence:

```
1. Check for @tool directive  → if present, execute tool, skip to step 7
2. Check for @elicit directive → if present and no response yet, PAUSE (awaiting_input)
3. Evaluate branch conditions  → select matching branch, or skip step if no match
4. Resolve @prompt reference   → prepend referenced prompt content
5. Render template variables   → replace {{name}} placeholders with values
6. Call AI                     → send prompt with system message + accumulated context
7. Process @output directive   → store response as named variable if declared
8. Check breakpoints           → if step is a breakpoint, PAUSE (paused)
9. Record step result          → store output, tokens, latency, status
10. Advance to next step
```

## Context Accumulation

Each step receives the outputs of all previous steps. The implementation builds a context payload injected into the system message (or equivalent mechanism).

### What's Accumulated

| Source | Key | Value |
|--------|-----|-------|
| Step output | Step number (integer) | Full AI response text |
| Named output | Variable name (string) | Captured value (full response or extracted field) |

### How It's Used

- **Step outputs** (by number) provide the full conversation history
- **Named outputs** (by variable name) provide targeted data access via `{{name}}` interpolation

Both are available to downstream steps. The implementation should include accumulated context in a way that the AI can reference prior results without explicit prompting from the playbook author.

### Design Rationale

Steps are **not a conversation**. Each step is a fresh AI call with accumulated context injected into the system prompt area. This differs from a chat model where messages build on each other in a conversation thread.

Benefits:
- Token usage per step is predictable (system prompt + accumulated context + step prompt)
- Steps can be retried independently
- The execution engine is stateless between steps (all state is in the context maps)

## State Machine

A playbook execution moves through these states:

```
                    ┌──────────┐
                    │ pending  │
                    └────┬─────┘
                         │ start
                         v
                    ┌──────────┐
              ┌─────│ running  │─────┐
              │     └────┬─────┘     │
              │          │           │
         breakpoint   complete    error
              │          │           │
              v          v           v
         ┌────────┐ ┌─────────┐ ┌────────┐
         │ paused │ │completed│ │ failed │
         └───┬────┘ └─────────┘ └────────┘
             │
          resume
             │
             v
        ┌──────────┐
        │ running  │ (continues from paused step)
        └──────────┘
```

### States

| State | Description |
|-------|-------------|
| `pending` | Queued, not yet started |
| `running` | Actively executing steps |
| `completed` | All steps executed successfully |
| `failed` | An error occurred during execution |
| `cancelled` | User cancelled the execution |
| `paused` | Execution paused at a breakpoint |
| `awaiting_input` | Execution paused, waiting for elicitation response |
| `skipped` | Step was skipped (branch condition didn't match) |
| `tool_call` | Intermediate: step requires external tool execution |
| `expired` | Paused execution that exceeded the timeout |

### Per-Step States

Individual steps within an execution also have states:

| Step State | Meaning |
|------------|---------|
| `pending` | Not yet reached |
| `running` | Currently executing |
| `completed` | Finished successfully |
| `failed` | Error during this step |
| `skipped` | Branch condition didn't match |
| `paused` | Breakpoint hit after this step |
| `awaiting_input` | Waiting for elicitation response |

## Pause and Resume

Execution can pause for two reasons:

### Breakpoints

The caller specifies breakpoint step numbers before execution starts. After a breakpoint step completes, execution pauses with status `paused`.

- The full execution state (context maps, step outputs, current position) must be preserved
- Resume continues from the next step after the breakpoint
- Optional: the caller may override variable values when resuming

### Elicitation

When a step has an `@elicit` directive and no response has been provided, execution pauses with status `awaiting_input`.

- The elicitation prompt and type are communicated to the caller
- The caller provides a response
- Execution resumes:
  - If the step is elicit-only (no prompt content), the response becomes the step output
  - If the step has prompt content, the AI call runs with the response available as context

### State Persistence

Implementations should serialize the full execution state for pause/resume:

```json
{
  "execution_id": "...",
  "playbook_id": "...",
  "model_id": "...",
  "system_prompt": "...",
  "input_values": {"topic": "AI safety", "depth": "thorough"},
  "step_outputs": {"1": "...", "2": "..."},
  "named_outputs": {"analysis": "...", "severity": "critical"},
  "breakpoints": {"3": true, "5": true},
  "status": "paused",
  "paused_at_step": 3
}
```

### Expiration

Implementations should define a timeout for paused executions. The reference implementation uses 24 hours, after which the execution transitions to `expired` status.

## Streaming

Implementations that support real-time output should emit events during step execution:

| Event Type | Payload | When |
|------------|---------|------|
| `content` | Text chunk | As AI tokens arrive |
| `done` | Step result (tokens, latency, status) | Step completes |
| `error` | Error message | Step fails |

The streaming protocol is not prescribed — implementations may use Server-Sent Events (SSE), WebSockets, or any other streaming mechanism. The spec defines the event types and their semantics, not the transport.

## AI Provider Interface

The spec does not prescribe which AI provider or model to use. The implementation must support:

1. **System message** — for system prompt + accumulated context
2. **User message** — for the rendered step prompt
3. **Max tokens** — implementations should set a reasonable default (reference: 16,384)
4. **Temperature** — implementations may expose this as a playbook-level or step-level option

## Error Handling

| Scenario | Behavior |
|----------|----------|
| AI call fails | Step status: `failed`. Execution status: `failed`. Error message recorded. |
| Tool call fails | Step status: `failed`. Execution status: `failed`. Error message recorded. |
| Referenced prompt not found | Step executes with its own content only (no error). |
| Unresolved variable | Placeholder remains as literal text (no error). |
| Branch variable undefined | Evaluates as empty string (no match unless compared to `""`). |

Implementations should record detailed error information (message, step number, timing) for debugging.

## Rate Limiting

Implementations should consider rate limiting per user or per playbook to prevent resource exhaustion. The spec does not prescribe specific limits — this is an implementation concern.

## Idempotency

Playbook executions are **not idempotent**. The same playbook with the same inputs may produce different outputs due to AI model non-determinism. Each execution should be recorded as a distinct run with its own ID, timestamps, and results.
