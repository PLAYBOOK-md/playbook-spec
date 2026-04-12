# Sentiment Analyzer

Analyze text sentiment and use the result in a follow-up step.

## INPUTS

- `text` (text): Text to analyze for sentiment

## STEP 1: Analyze Sentiment

Analyze the sentiment of the following text and respond with a single word: "positive", "negative", or "neutral".

{{text}}

@output(sentiment)

## STEP 2: Generate Response

The sentiment was classified as {{sentiment}}.

Write an appropriate response that acknowledges the tone and addresses the content constructively.
