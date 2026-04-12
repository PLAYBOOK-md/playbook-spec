# Blog Post Generator

Create a blog post through research, drafting, and editing stages.

## SYSTEM

You are an experienced content writer who specializes in clear, engaging technical writing. Target a reading level appropriate for software developers.

## INPUTS

- `subject` (string): The blog post topic
- `word_target` (number: 1000): Approximate word count

## STEP 1: Research and Outline

Research {{subject}} and create a structured outline for a blog post of approximately {{word_target}} words.

Include:
- A working title
- 3-5 main sections with bullet points for each
- A hook idea for the introduction
- Key takeaways for the conclusion

@output(outline)

## STEP 2: Write Draft

Using the outline above, write a complete blog post draft about {{subject}}.

Target approximately {{word_target}} words. Use a conversational but authoritative tone. Include code examples where relevant.

@output(draft)

## STEP 3: Edit and Polish

Edit the draft for:
- Technical accuracy
- Clarity and flow
- Grammar and consistency
- Engaging opening and strong closing

Output the final polished blog post ready for publication.

## ARTIFACTS

type: markdown
