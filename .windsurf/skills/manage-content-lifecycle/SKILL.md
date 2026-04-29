---
name: manage-content-lifecycle
description: Guides content updates, archival, and migration for blog posts and case studies
---

# Content Lifecycle Management Guide

This skill guides you through managing the lifecycle of blog posts and case studies.

## Content Updates

### When to Update Content

Update existing content when:

- Information becomes outdated
- New developments in the topic
- Statistics or data need refreshing
- Links are broken
- SEO improvements are needed
- User feedback indicates issues

### Update Process

1. **Identify Content to Update**
   - Review analytics for outdated content
   - Check for broken links
   - Review user feedback
   - Check industry developments

2. **Update Frontmatter**
   - Add `updatedDate` field with current date
   - Keep original `pubDate` unchanged
   - Update categories or tags if needed
   - Update featured status if appropriate

3. **Update Content**
   - Refresh outdated information
   - Add new insights or examples
   - Fix broken links
   - Improve readability
   - Add new code examples if applicable

4. **Add Changelog**
   - Add note at bottom of content
   - Document what was changed
   - Include date of update
   - Link to related new content if applicable

### Example Update

```yaml
---
title: "SEO Strategies for 2024"
description: "Learn the latest SEO strategies..."
pubDate: 2024-01-15
updatedDate: 2024-06-01  # Add this field
author: "Sarah Mitchell"
category: "SEO"
tags: ["SEO", "Marketing"]
---

# Content here...

---
*Updated June 1, 2024: Added new section on AI-powered SEO tools and updated statistics for 2024.*
```

## Content Archival

### When to Archive Content

Archive content when:

- Information is significantly outdated
- Topic is no longer relevant
- Content has been replaced by newer version
- Performance is poor and not worth updating
- Strategic decision to remove content

### Archival Process

1. **Add Archival Flag**
   - Add `archived: true` to frontmatter
   - Keep original content intact
   - Do not delete the file

2. **Add Redirect** (if URL changes)
   - Create redirect in astro.config.mjs
   - Point to updated content
   - Use 301 redirect for SEO

3. **Update Internal Links**
   - Find all internal links to archived content
   - Update to point to new content
   - Or add "archived" note to links

4. **Update Sitemap** (if needed)
   - Remove from sitemap if truly deprecated
   - Or keep with noindex meta tag

### Example Archival

```yaml
---
title: "SEO Strategies for 2023"
description: "Learn SEO strategies for 2023..."
pubDate: 2023-01-15
archived: true  # Add this flag
author: "Sarah Mitchell"
category: "SEO"
tags: ["SEO", "Marketing"]
---

# Content here...

---
*This post has been archived. See [SEO Strategies for 2024](/blog/seo-strategies-2024) for the latest information.*
```

## Content Migration

### When to Migrate Content

Migrate content when:

- Moving from another platform
- Restructuring content organization
- Changing URL structure
- Consolidating multiple posts
- Splitting large posts

### Migration Process

1. **Export Content**
   - Export from source platform
   - Preserve frontmatter
   - Keep original publish dates
   - Save images and media

2. **Convert to MDX**
   - Convert markdown to MDX format
   - Update frontmatter to match schema
   - Fix image paths
   - Update internal links

3. **Validate Content**
   - Check frontmatter fields
   - Verify images load
   - Test links
   - Check formatting

4. **Update Search Index**
   - Regenerate search index
   - Test search functionality
   - Verify results appear

### Migration Checklist

- [ ] Export content from source
- [ ] Convert to MDX format
- [ ] Update frontmatter to match schema
- [ ] Fix image paths
- [ ] Update internal links
- [ ] Preserve original publish dates
- [ ] Validate content builds
- [ ] Regenerate search index
- [ ] Test in development
- [ ] Deploy to production

## Featured Content Management

### Featured Content Rotation

Rotate featured content monthly:

- Select 2-3 blog posts to feature
- Select 2-3 case studies to feature
- Update homepage and index pages
- Remove old featured content

### Selection Criteria

Choose featured content based on:

- Recent publication (last 3-6 months)
- High engagement metrics
- Representative of services
- High-quality content
- SEO value

### Update Process

1. **Select New Featured Content**
   - Review analytics data
   - Identify top-performing content
   - Select diverse topics
   - Balance categories

2. **Update Frontmatter**
   - Set `featured: true` on selected items
   - Set `featured: false` on old featured items
   - Keep 2-3 items featured per collection

3. **Update Display**
   - Homepage will automatically pick up featured content
   - Index pages will show featured items
   - Test display in development

## Content Quality Review

### Review Checklist

Review content quarterly for:

- Accuracy of information
- Broken links
- Outdated statistics
- Formatting issues
- SEO optimization
- Accessibility compliance

### Review Process

1. **Audit Content**
   - List all blog posts and case studies
   - Check publish dates
   - Review analytics data
   - Identify outdated content

2. **Prioritize Updates**
   - High-traffic content first
   - Content with broken links
   - Outdated statistics
   - Poor performing content

3. **Execute Updates**
   - Update content systematically
   - Track changes
   - Test after updates
   - Monitor performance

## Content Deletion

### When to Delete Content

Delete content only when:

- Content is completely irrelevant
- No replacement exists
- Legal or compliance reasons
- Duplicate content
- Test content that should be removed

### Deletion Process

1. **Backup Content**
   - Save copy of content
   - Document reason for deletion
   - Note any redirects needed

2. **Remove Internal Links**
   - Find all internal links
   - Remove or update links
   - Check for orphaned content

3. **Delete File**
   - Delete content file
   - Update search index
   - Test build
   - Deploy changes

## Content Analytics

### Metrics to Track

Track these metrics for content decisions:

- Page views
- Unique visitors
- Time on page
- Bounce rate
- Social shares
- Search rankings
- Conversion rate

### Using Analytics

- Identify top-performing content
- Find content gaps
- Understand user behavior
- Optimize content strategy
- Plan future content

## Content SEO

### SEO Updates

Update SEO when:

- Keywords change
- Search intent shifts
- Competition increases
- Algorithm updates
- Performance drops

### SEO Checklist

- [ ] Update title and description
- [ ] Add new keywords
- [ ] Update meta tags
- [ ] Add internal links
- [ ] Update image alt text
- [ ] Check heading structure
- [ ] Verify schema markup
- [ ] Test search results

## Content Backup

### Backup Strategy

- Keep content in version control
- Export content regularly
- Backup images and media
- Document content structure
- Maintain change history

### Recovery Process

If content is lost or corrupted:

1. Restore from version control
2. Rebuild search index
3. Test in development
4. Deploy to production
5. Monitor for issues

## Troubleshooting

### Build Errors After Content Changes

```bash
# Check content schema
npm run astro check

# Validate frontmatter
npm run build

# Check specific file
astro build --filter /blog/specific-post
```

### Search Index Issues

```bash
# Regenerate search index
npm run generate-search-index

# Check index file
cat public/search-index.json

# Rebuild with search
npm run build:with-search
```

### Link Issues

```bash
# Find broken links
npm install --save-dev broken-link-checker
blc https://yourdomain.com

# Check internal links
grep -r "\[.*\](/" src/content/
```

## Best Practices

### Content Updates

- Always add `updatedDate` when updating
- Preserve original `pubDate`
- Document changes in changelog
- Test after updates
- Monitor performance

### Content Archival

- Never delete without archival
- Add redirects for URL changes
- Update internal links
- Keep content in version control
- Document archival reason

### Content Migration

- Preserve original dates
- Validate frontmatter
- Test thoroughly
- Update search index
- Monitor for issues

## After Content Changes

1. Test build succeeds
2. Verify content displays correctly
3. Check links work
4. Test search functionality
5. Verify SEO meta tags
6. Check accessibility
7. Deploy to production
8. Monitor analytics
9. Update documentation if needed
