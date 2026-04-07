# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in the specification or any official implementation, please report it responsibly.

**Email:** security@playbook.style

Please include:
- Description of the vulnerability
- Steps to reproduce (if applicable)
- Affected spec section or repo
- Any suggested fix

## Response Timeline

- **Acknowledgment** within 48 hours
- **Assessment** within 7 days
- **Fix or advisory** within 30 days for confirmed issues

## Scope

This policy covers:
- The PLAYBOOK.md specification (`playbook-spec`)
- Official SDKs and tools maintained under the PLAYBOOK-MD organization
- The playbook.style website and infrastructure

Third-party implementations are outside our scope — please contact their maintainers directly.

## Security Considerations for Implementers

The spec itself is a document format, but implementations should be aware of:

- **Template injection**: `{{variable}}` interpolation must not execute arbitrary code
- **Tool directive safety**: `@tool` directives invoke external services — implementations must validate connections and sandbox execution
- **Content size limits**: The 200 KB document limit exists to prevent resource exhaustion during parsing
- **Elicitation trust**: User responses to `@elicit` directives are untrusted input and should be sanitized before use in prompts
