# Unrecognized Section Test

A playbook with unknown section headings that should be silently skipped.

## METADATA

author: Test Author
version: 1.0

## CHANGELOG

- 2025-01-01: Initial version
- 2025-02-15: Added second step

## INPUTS

- `query` (string): Search query

## NOTES

These are internal notes that parsers should skip entirely.
This section has no spec meaning.

## STEP 1: Search

Search for information about {{query}} and return the top results.

## REFERENCES

- https://example.com/docs
- https://example.com/api

## STEP 2: Summarize

Summarize the search results into a concise answer.

## APPENDIX

Additional context that is not part of the spec.
