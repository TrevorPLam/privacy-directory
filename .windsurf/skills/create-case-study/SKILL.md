---
name: create-case-study
description: Guides creation of new case studies with proper frontmatter and measurable results
---

# Case Study Creation Guide

## Frontmatter Requirements

Create case studies in `src/content/caseStudies/` with the following frontmatter:

```yaml
---
title: "Case Study Title"
client: "Client Name"
industry: "Industry"
services: ["SEO", "Paid Media"]
challenge: "The challenge description"
solution: "The solution description"
results:
  - metric: "Metric Name"
    value: "Value"
    improvement: "Improvement"
testimonial:
  quote: "Testimonial quote"
  author: "Author Name"
  role: "Role"
publishedAt: 2024-01-15
---
```

## Content Structure

- **Challenge**: Describe the client's problem
- **Solution**: Explain the approach and implementation
- **Results**: Highlight measurable outcomes with metrics

## Industries

Use existing industries:

- E-commerce
- SaaS
- Healthcare
- Professional Services
- Technology

## Services

Use existing service names:

- SEO
- Paid Media
- Content Marketing
- Email Automation
- Analytics & Reporting
- Full Service

## Results Guidelines

- Include at least 3 metrics
- Use realistic values with percentage improvements
- Examples:
  - metric: "Organic Traffic", value: "+150%", improvement: "150% increase"
  - metric: "Conversion Rate", value: "3.2%", improvement: "From 1.8% to 3.2%"
  - metric: "Revenue", value: "$250K", improvement: "$250K increase in 6 months"

## Testimonial Guidelines

- Include quote, author name, and role
- Write testimonials that sound authentic
- Keep quotes concise (1-2 sentences)

## File Naming

- Use kebab-case: `ecommerce-revenue-growth.md`
- Use descriptive names that highlight the outcome
- Keep names under 60 characters

## Validation

The case study will be validated against the Zod schema in `src/content/config.ts`:

- title: string (required)
- client: string (required)
- industry: string (required)
- services: array (required)
- challenge: string (required)
- solution: string (required)
- results: array (required)
- testimonial: object (optional)
- publishedAt: date (required)

## After Creation

1. Run `npm run dev` to verify the case study appears on `/work`
2. Test industry and service filters with the new case study
3. Click the case study to verify the detail page renders
4. Verify results display correctly
5. Verify testimonial displays correctly
