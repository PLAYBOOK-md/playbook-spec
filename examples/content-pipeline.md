# Content Pipeline

Generate a polished article from a topic and target audience.

## SYSTEM

You are a professional content writer. Write clearly, cite sources where possible, and adapt tone to the target audience.

## INPUTS

- `topic` (string): Subject to write about
- `audience` (enum: technical, general, executive): Target audience
- `word_count` (number: 1500): Target word count

## STEP 1: Research

Research {{topic}} and identify the top 5 key themes relevant to a {{audience}} audience.

For each theme, provide:
- A one-sentence summary
- Why it matters to this audience
- One supporting data point or example

## STEP 2: Outline

Create a detailed article outline targeting approximately {{word_count}} words. Structure it with:

1. Hook / opening paragraph
2. Main sections (one per key theme)
3. Conclusion with takeaways

## STEP 3: Draft

Write the full article following the outline above. Target {{word_count}} words.

Use a tone appropriate for a {{audience}} audience.

## STEP 4: Polish

Edit the article for:
- Clarity and conciseness
- Grammar and punctuation
- Consistent tone for {{audience}} readers
- Smooth transitions between sections

Output the final polished article.

## ARTIFACTS

type: markdown
