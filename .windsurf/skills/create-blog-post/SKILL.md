---
name: create-blog-post
description: Guides creation of new blog posts with proper frontmatter and MDX content
---

# Blog Post Creation Guide

## Frontmatter Requirements

Create blog posts in `src/content/blog/` with the following frontmatter:

```yaml
---
title: "Post Title"
description: "Post description for SEO (150-160 characters)"
pubDate: 2024-01-15
author: "Your Dedicated Marketer"
category: "Category"
tags: ["tag1", "tag2"]
image: "/path/to/image.jpg"
featured: false
---
```

## Content Guidelines

- Write in markdown with MDX support
- Use proper heading hierarchy (##, ###, etc.)
- Include relevant code examples when applicable
- Add images with alt text
- Keep content concise (500-1000 words)
- Use consistent formatting

## Categories

Use existing categories:

- SEO
- Paid Media
- Content Marketing
- Email Automation
- Analytics & Reporting

## File Naming

- Use kebab-case: `ai-powered-seo-strategies.md`
- Use descriptive, SEO-friendly names
- Keep names under 60 characters

## Validation

The post will be validated against the Zod schema in `src/content/config.ts`:

- title: string (required)
- description: string (required)
- pubDate: date (required)
- author: string (default: "Your Dedicated Marketer")
- category: string (required)
- tags: array (default: [])
- image: string (optional)
- featured: boolean (default: false)

## After Creation

1. Run `npm run dev` to verify the post appears on `/blog`
2. Click the post to verify the detail page renders
3. Check that categories and tags display correctly
4. Verify reading time is accurate (or add it to frontmatter)
