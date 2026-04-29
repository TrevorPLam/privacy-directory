---
trigger: glob
globs: src/content/**/*.md,src/content/**/*.mdx
---

<content_management_guidelines>

# Content Management Guidelines

When managing blog posts and case studies:

## Content Lifecycle

### Creating New Content

- Follow existing frontmatter schema in `src/content/config.ts`
- Use kebab-case for file names (e.g., `seo-strategies-2024.mdx`)
- Include all required fields in frontmatter
- Add relevant tags and categories
- Set appropriate publish dates

### Updating Existing Content

- Update `updatedDate` field when modifying content
- Keep original `pubDate` unchanged
- Add changelog note at bottom of content
- Consider creating new post for major updates
- Update related posts if content references change

### Archiving Content

- Add `archived: true` to frontmatter for archived posts
- Do not delete old content (preserve SEO value)
- Add redirect if URL changes
- Update internal links to point to new content
- Consider adding "updated" note with link to new version

### Featured Content

- Set `featured: true` for 2-3 items per collection
- Rotate featured content monthly
- Featured items should be recent and high-quality
- Update featured content in homepage and index pages

## Content Quality Standards

### Blog Posts

- Minimum 500 words for substantive posts
- Use proper heading hierarchy (##, ###)
- Include code examples when applicable
- Add relevant images with alt text
- Include reading time estimate
- Use consistent author names

### Case Studies

- Include measurable results with metrics
- Add testimonials with quotes
- Structure with Challenge, Solution, Results sections
- Include client industry and services
- Add realistic improvement percentages
- Use consistent date formats

## SEO Best Practices

### Titles and Descriptions

- Keep titles under 60 characters
- Write unique descriptions (150-160 characters)
- Include target keywords naturally
- Use action-oriented language
- Match user search intent

### Content Structure

- Use short paragraphs (2-3 sentences)
- Include bullet points for readability
- Add internal links to related content
- Use descriptive anchor text
- Include table of contents for long posts

### Images

- Use descriptive file names (e.g., `seo-audit-checklist.jpg`)
- Include alt text for all images
- Use appropriate dimensions (1200x630 for OG)
- Compress images for faster loading
- Use WebP format when possible

## Content Organization

### Categories

- Use existing categories from config
- Create new categories only when necessary
- Keep categories broad (5-10 total)
- Map categories to service offerings
- Update category filters in UI

### Tags

- Use 3-5 relevant tags per post
- Keep tags specific (not categories)
- Use consistent tag naming
- Update tag pages automatically
- Consider tag consolidation periodically

### Content Collections

- Blog posts in `src/content/blog/`
- Case studies in `src/content/caseStudies/`
- Follow Zod schema validation
- Use MDX for rich content
- Keep content in version control

## Content Workflow

### Draft Content

- Create draft files with `draft-` prefix
- Do not include in build (exclude in config)
- Review before publishing
- Update frontmatter when ready to publish
- Remove `draft-` prefix from filename

### Review Process

- Check for spelling and grammar errors
- Verify all links work
- Test code examples
- Validate frontmatter fields
- Check mobile responsiveness
- Verify SEO meta tags

### Publishing

- Set appropriate `pubDate`
- Remove `draft` status if applicable
- Generate search index if needed
- Test in development environment
- Build and verify output
- Deploy to production

## Content Maintenance

### Regular Reviews

- Review content quarterly
- Update outdated information
- Check for broken links
- Update statistics and data
- Refresh old posts with new insights

### Link Maintenance

- Check internal links quarterly
- Update links to moved content
- Fix broken external links
- Update anchor text for relevance
- Add links to new related content

### Performance Monitoring

- Track page views and engagement
- Identify low-performing content
- Update or improve underperforming posts
- Promote high-performing content
- Archive outdated content

## Content Migration

### Importing Content

- Validate frontmatter schema
- Convert to MDX format
- Update image paths
- Preserve original publish dates
- Set appropriate categories and tags
- Test build after import

### Exporting Content

- Export with frontmatter intact
- Preserve publish dates
- Include all metadata
- Maintain file structure
- Document migration process

## Content Analytics

### Metrics to Track

- Page views and unique visitors
- Time on page
- Bounce rate
- Social shares
- Conversion rate (if applicable)
- Search rankings

### Using Analytics

- Identify top-performing content
- Find content gaps
- Understand user behavior
- Optimize content strategy
- Plan future content

## Anti-Patterns

- Do not delete old content (archive instead)
- Do not skip required frontmatter fields
- Do not use Lorem ipsum placeholder text
- Do not duplicate content across posts
- Do not ignore broken links
- Do not publish without review
- Do not use unrealistic metrics in case studies
- Do not forget to update search index after content changes

## File Naming Conventions

### Blog Posts

- Use kebab-case: `seo-strategies-2024.mdx`
- Keep names under 60 characters
- Include year for time-sensitive content
- Use descriptive, SEO-friendly names
- Avoid special characters

### Case Studies

- Use client-name-service.md format
- Example: `techcorp-seo-growth.md`
- Keep names descriptive
- Include industry if relevant
- Avoid client-specific jargon

## Content Validation Checklist

Before publishing content:

- [ ] All required frontmatter fields present
- [ ] Title and description meet length requirements
- [ ] Images have alt text
- [ ] Internal links work
- [ ] Code examples tested
- [ ] Spelling and grammar checked
- [ ] Mobile responsive
- [ ] SEO meta tags configured
- [ ] Categories and tags appropriate
- [ ] Reading time accurate
- [ ] Author information correct
- [ ] Build succeeds without errors

</content_management_guidelines>
