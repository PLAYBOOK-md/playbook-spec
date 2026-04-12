# Content Adaptation Pipeline

Adapt content for different platforms, with multiple sub-steps per branch.

## INPUTS

- `content` (text): Original content to adapt
- `platform` (enum: blog, social, email): Target platform

## STEP 1: Analyze Content

Analyze the following content and identify:
- Core message and key points
- Tone and voice characteristics
- Target audience signals

{{content}}

@output(content_analysis)

## STEP 2: Adapt for Platform

```if platform == "blog"```

### STEP 2a: Blog Structure

Restructure the content for a blog format:
- Create an engaging headline
- Write a hook opening paragraph
- Organize into scannable sections with subheadings
- Add a call-to-action conclusion

@output(blog_structure)

### STEP 2b: Blog SEO

Optimize the blog post for search engines:
- Suggest 5 target keywords
- Write a meta description (155 characters max)
- Add internal linking suggestions
- Recommend image alt text

```elif platform != "email"```

### STEP 2c: Social Media Copy

Create social media versions of the content:
- Twitter/X post (280 characters max)
- LinkedIn post (professional tone, 1-3 paragraphs)
- Instagram caption (with emoji and hashtag suggestions)

@output(social_posts)

### STEP 2d: Social Media Hashtags

Generate a hashtag strategy:
- 5 primary hashtags (high relevance)
- 10 secondary hashtags (broader reach)
- 3 trending hashtags to consider

```else```

### STEP 2e: Email Draft

Convert the content into an email format:
- Subject line (50 characters max) with A/B variant
- Preview text (90 characters max)
- Email body with clear sections
- CTA button text suggestion

@output(email_draft)

### STEP 2f: Email Personalization

Suggest personalization tokens and segmentation:
- Variable fields for personalization
- Audience segments this would work best for
- Send time recommendations

```endif```

## STEP 3: Quality Check

Review the adapted content for:
- Message consistency with the original
- Platform-appropriate formatting
- Grammar and clarity
- Completeness of all required elements

Provide a final quality score (1-10) and any recommended fixes.
