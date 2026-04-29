---
trigger: glob
globs: src/content/**/*.md
---

<content_collections>

# Content Collection Guidelines

When creating or modifying blog posts and case studies:

## Blog Posts (src/content/blog/)

Required frontmatter fields:

- `title`: string - Post title
- `description`: string - Post description for SEO
- `pubDate`: date - Publication date (YYYY-MM-DD format)
- `author`: string - Author name (default: "Your Dedicated Marketer")
- `category`: string - Category (SEO, Paid Media, Content Marketing, etc.)
- `tags`: array - Related tags

Optional fields:

- `image`: string - Featured image path
- `imageAlt`: string - Image alt text
- `authorImage`: string - Author avatar path
- `featured`: boolean - Featured post flag
- `readingTime`: string - Estimated reading time

## Case Studies (src/content/caseStudies/)

Required frontmatter fields:

- `title`: string - Case study title
- `client`: string - Client name
- `industry`: string - Industry (E-commerce, SaaS, Healthcare, etc.)
- `services`: array - Services provided (SEO, Paid Media, etc.)
- `challenge`: string - Client challenge description
- `solution`: string - Solution description
- `results`: array - Results metrics with value and improvement
- `publishedAt`: date - Publication date

Optional fields:

- `testimonial`: object - Quote, author, role
- `image`: string - Featured image path
- `imageAlt`: string - Image alt text
- `featured`: boolean - Featured case study flag

## Content Guidelines

- Write in markdown with MDX support
- Use proper heading hierarchy (##, ###, etc.)
- Include relevant code examples when applicable
- Add images with alt text
- Keep content concise and actionable
- Use consistent formatting

## SEO Best Practices

- Include descriptive titles and descriptions
- Use relevant tags and categories
- Add featured images for social sharing
- Include author information
- Use proper date formats

## File Naming

- Use kebab-case: `ai-powered-seo-strategies.md`
- Use descriptive, SEO-friendly names
- Keep names under 60 characters

</content_collections>
