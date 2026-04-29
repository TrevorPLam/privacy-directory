---
name: generate-search-index
description: Guides search index generation with MiniSearch configuration, content indexing, and deployment
---

# Search Index Generation Guide

This skill guides you through generating and deploying the search index for the Astro marketing website.

## Prerequisites

- Node.js 18+ installed
- Project dependencies installed (`npm install`)
- MiniSearch package installed (`npm install minisearch`)

## Overview

The search index is generated from blog posts and case studies using MiniSearch. The index is built during the build process and deployed as a static JSON file.

## Search Index Script

### Script Location

The search index generation script is located at `scripts/generate-search-index.mjs`.

### Script Structure

```javascript
import MiniSearch from "minisearch";
import { getCollection } from "astro:content";

// Configure MiniSearch
const miniSearch = new MiniSearch({
  fields: ["title", "description", "content"],
  storeFields: ["title", "description", "slug", "category", "publishedAt"],
  searchOptions: {
    boost: { title: 2, description: 1.5 },
    fuzzy: 0.2,
    prefix: true,
  },
});

// Load content
const blogPosts = await getCollection("blog");
const caseStudies = await getCollection("caseStudies");

// Add documents to index
const documents = [
  ...blogPosts.map((post) => ({
    id: post.slug,
    title: post.data.title,
    description: post.data.description,
    content: post.rendered?.html || "",
    slug: `/blog/${post.slug}`,
    category: post.data.category,
    publishedAt: post.data.pubDate,
  })),
  ...caseStudies.map((study) => ({
    id: study.slug,
    title: study.data.title,
    description: study.data.challenge,
    content: study.rendered?.html || "",
    slug: `/work/${study.slug}`,
    category: study.data.industry,
    publishedAt: study.data.publishedAt,
  })),
];

miniSearch.addAll(documents);

// Generate index
const index = JSON.stringify(miniSearch);

// Write to public directory
fs.writeFileSync("public/search-index.json", index);
```

## MiniSearch Configuration

### Fields to Index

Configure which fields to include in the search index:

```javascript
const miniSearch = new MiniSearch({
  fields: ["title", "description", "content"], // Fields to search
  storeFields: ["title", "description", "slug", "category", "publishedAt"], // Fields to return
});
```

### Search Options

Configure search behavior:

```javascript
const searchOptions = {
  boost: { title: 2, description: 1.5 }, // Boost specific fields
  fuzzy: 0.2, // Fuzzy matching (0 = exact, 1 = very fuzzy)
  prefix: true, // Match prefixes
  combineWith: "AND", // Combine terms with AND/OR
  maxResults: 10, // Maximum results to return
};
```

### Field Weights

Higher boost values give more importance to specific fields:

```javascript
boost: {
  title: 2,           // Title matches are 2x more important
  description: 1.5,   // Description matches are 1.5x more important
  content: 1,         // Content matches are baseline
}
```

## Content Indexing

### Blog Posts

Index blog posts with the following fields:

- `title`: Post title
- `description`: Post description from frontmatter
- `content`: Full rendered HTML content
- `slug`: URL slug
- `category`: Post category
- `publishedAt`: Publication date

### Case Studies

Index case studies with the following fields:

- `title`: Case study title
- `description`: Challenge description from frontmatter
- `content`: Full rendered HTML content
- `slug`: URL slug
- `category`: Industry
- `publishedAt`: Publication date

### Document Structure

Each document in the index should have:

```javascript
{
  id: 'unique-id',
  title: 'Document Title',
  description: 'Document description',
  content: 'Full content text',
  slug: '/url-path',
  category: 'Category',
  publishedAt: '2024-01-15'
}
```

## Running the Script

### Generate Index

```bash
npm run generate-search-index
```

This will:

1. Load all blog posts and case studies
2. Build the MiniSearch index
3. Write the index to `public/search-index.json`

### Build with Search Index

```bash
npm run build:with-search
```

This will:

1. Build the Astro site
2. Generate the search index
3. Deploy both together

## Search Index File

### File Location

The search index is generated at `public/search-index.json`.

### File Structure

```json
{
  "index": [
    {
      "id": "seo-strategies-2024",
      "title": "SEO Strategies for 2024",
      "description": "Learn the latest SEO strategies...",
      "slug": "/blog/seo-strategies-2024",
      "category": "SEO",
      "publishedAt": "2024-01-15"
    }
  ],
  "options": {
    "fields": ["title", "description", "content"],
    "storeFields": ["title", "description", "slug", "category", "publishedAt"]
  }
}
```

## Client-Side Search

### Loading the Index

```tsx
import { useEffect, useState } from "react";

export default function SearchModal() {
  const [searchIndex, setSearchIndex] = useState(null);

  useEffect(() => {
    fetch("/search-index.json")
      .then((res) => res.json())
      .then((data) => setSearchIndex(data));
  }, []);

  // Use searchIndex for searching
}
```

### Searching with MiniSearch

```tsx
import MiniSearch from "minisearch";

const miniSearch = new MiniSearch({
  fields: ["title", "description", "content"],
  storeFields: ["title", "description", "slug", "category"],
  searchOptions: {
    boost: { title: 2, description: 1.5 },
    fuzzy: 0.2,
    prefix: true,
  },
});

// Load index
miniSearch.load(searchIndex);

// Search
const results = miniSearch.search("SEO strategies");
```

## Performance Optimization

### Index Size

- Keep the index size under 1MB for fast loading
- Exclude unnecessary fields from indexing
- Compress the index if needed
- Consider lazy loading for large indexes

### Search Performance

- Use fuzzy matching sparingly (can be slow)
- Limit max results to 10-20
- Debounce search input
- Use Web Workers for large indexes

### Caching

- Cache the search index in localStorage
- Set cache expiration (e.g., 1 hour)
- Revalidate cache on page load
- Handle cache errors gracefully

## Troubleshooting

### Index Not Generating

```bash
# Check script exists
ls scripts/generate-search-index.mjs

# Check MiniSearch is installed
npm list minisearch

# Run script directly
node scripts/generate-search-index.mjs
```

### Content Not Indexed

```bash
# Check content collections
npm run astro check

# Verify content schema
cat src/content/config.ts

# Check content files
ls src/content/blog/
ls src/content/caseStudies/
```

### Search Not Working

```bash
# Check index file exists
ls public/search-index.json

# Verify index file size
du -h public/search-index.json

# Check index file content
cat public/search-index.json
```

## Best Practices

### Index Updates

- Regenerate index after content changes
- Automate index generation in build process
- Use `build:with-search` script for production builds
- Consider incremental index updates for large sites

### Search UX

- Show loading state while index loads
- Display search results as user types
- Highlight matching terms in results
- Show result categories
- Provide "no results" message
- Link to result pages

### Accessibility

- Ensure search is keyboard accessible
- Use proper ARIA attributes
- Provide search instructions
- Announce search results to screen readers
- Support screen reader navigation

## CI/CD Integration

### GitHub Actions

Add to `.github/workflows/ci.yml`:

```yaml
- name: Generate Search Index
  run: npm run generate-search-index

- name: Build
  run: npm run build
```

### Vercel

Add to `vercel.json`:

```json
{
  "buildCommand": "npm run build:with-search"
}
```

## Monitoring

### Index Size Monitoring

```bash
# Check index size
du -h public/search-index.json

# Set up alerts for large indexes
```

### Search Analytics

Track search usage:

- Search queries
- Click-through rates
- Popular search terms
- Zero-result searches

## After Generation

1. Verify index file exists in `public/`
2. Check index file size is reasonable
3. Test search functionality in development
4. Verify search results are accurate
5. Test search performance
6. Check accessibility
7. Deploy to production
8. Monitor search usage
