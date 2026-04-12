# Document Formatter

Format a document based on the selected output style.

## INPUTS

- `content` (text): Raw content to format
- `style` (enum: formal, casual): Output style

## STEP 1: Format Document

Take the following content and reformat it:

{{content}}

```if style == "formal"```

### STEP 1a: Formal Formatting

Rewrite the content above in a formal, professional style:
- Use third person perspective
- Avoid contractions
- Use precise, technical vocabulary
- Structure with clear headings and numbered lists

@output(formatted)

```else```

### STEP 1b: Casual Formatting

Rewrite the content above in a friendly, conversational style:
- Use first or second person perspective
- Contractions are fine
- Keep sentences short and punchy
- Use bullet points and informal headings

@output(formatted)

```endif```

## STEP 2: Final Review

Review the formatted document for consistency and completeness. Fix any issues and output the final version.
