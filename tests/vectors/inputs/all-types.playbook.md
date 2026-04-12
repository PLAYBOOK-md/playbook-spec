# Input Types Demo

Demonstrates all five input types supported by the spec.

## INPUTS

- `name` (string): A single-line text value
- `bio` (text): A multi-line biography
- `age` (number): Numeric age value
- `active` (boolean): Whether the profile is active
- `role` (enum: admin, editor, viewer): User role selection

## STEP 1: Display Profile

Create a formatted profile card for {{name}}.

Bio: {{bio}}
Age: {{age}}
Active: {{active}}
Role: {{role}}

Format it as a clean markdown card with all fields labeled.
