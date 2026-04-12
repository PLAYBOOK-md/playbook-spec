# Email Drafter

Draft a professional email with configurable parameters.

## INPUTS

- `recipient` (string): Who the email is addressed to
- `subject_line` (string): Email subject
- `tone` (string: professional): Writing tone to use
- `max_length` (number: 200): Maximum word count
- `include_signature` (boolean: true): Whether to add a closing signature

## STEP 1: Draft Email

Write an email to {{recipient}} with subject "{{subject_line}}".

Use a {{tone}} tone and keep it under {{max_length}} words.

Include a signature: {{include_signature}}.
