---
name: seo-audit
description: Performs comprehensive SEO audit including technical SEO, on-page optimization, and structured data validation
---

# SEO Audit

This skill guides you through performing a comprehensive SEO audit of the Astro marketing website.

## Step 1: Technical SEO Audit

### 1.1 Crawlability and Indexation

**Check:**

- Google Search Console index status
- Robots.txt allows crawling of important pages
- XML sitemap is accessible and accurate
- No `noindex` tags on critical pages

**Commands:**

```bash
# Check robots.txt
curl https://yoursite.com/robots.txt

# Check sitemap
curl https://yoursite.com/sitemap.xml
```

**Common Issues:**

- Robots.txt blocking important pages
- Missing or incorrect sitemap
- Accidental noindex tags
- Blocked by Cloudflare firewall

### 1.2 Site Architecture and URL Structure

**Check:**

- Logical category structure
- Consistent URL naming (kebab-case)
- No unnecessary parameters
- Clear hierarchy between services and blog

**Best Practices:**

- URLs should be short and descriptive
- Use hyphens to separate words
- Keep URLs lowercase
- Avoid unnecessary parameters

**Example:**

- Good: `/services/seo-audit`
- Bad: `/services?id=123&category=seo`

### 1.3 Core Web Vitals and Page Speed

**Check:**

- Largest Contentful Paint (LCP) < 2.5s
- Interaction to Next Paint (INP) < 200ms
- Cumulative Layout Shift (CLS) < 0.1

**Tools:**

- Google PageSpeed Insights
- Lighthouse audit
- Google Search Console Core Web Vitals report

**Run Lighthouse:**

```bash
npx lighthouse https://yoursite.com --view
```

### 1.4 Mobile SEO Audit

**Check:**

- Responsive design works on mobile
- Text is readable without zooming
- Buttons are tap-friendly (min 44x44px)
- Mobile page speed is acceptable

**Test:**

- Use Chrome DevTools device emulation
- Test on actual mobile devices
- Check mobile-friendly test in Search Console

### 1.5 HTTPS and Security

**Check:**

- HTTPS is enforced sitewide
- No mixed content errors
- Forms and checkout pages are secure
- SSL certificate is valid

**Test:**

```bash
curl -I https://yoursite.com
```

### 1.6 Duplicate Content and Canonicalization

**Check:**

- Canonical tags are present on all pages
- No duplicate service pages
- No similar blog content targeting same keyword
- Trailing slash consistency

**Example Canonical:**

```astro
<link rel="canonical" href="https://yoursite.com/services/seo" />
```

### 1.7 Structured Data and Schema Markup

**Check:**

- Organization schema on main pages
- Article schema on blog posts
- Person schema on author pages
- Local business schema if applicable
- Use proper schema.org types

**Validate Schema:**

- Google Rich Results Test
- Schema.org validator
- Google Search Console structured data report

### 2026 Structured Data Updates

**Deprecated Features (Early 2026):**

- Practice Problems experience
- Nutrition Facts
- Nearby features
- Several small UI elements

**Focus Areas for 2026:**

- Organization schema
- Article schema
- Breadcrumb schema
- Product schema
- FAQ schema

**AI-Generated Schema:**

- AI tools available for schema generation
- Validate with Google Search Console
- Monitor for rich result eligibility

## Step 2: On-Page SEO Audit

### 2.1 Title Tags

**Check:**

- All pages have unique titles
- Titles under 60 characters
- Descriptive, keyword-rich titles
- Include site name suffix

**Format:**

- Main page: "Site Name"
- Other pages: "Page Title | Site Name"

**Example:**

```astro
<title>SEO Services | Your Dedicated Marketer</title>
```

### 2.2 Meta Descriptions

**Check:**

- All pages have unique descriptions
- Descriptions between 150-160 characters
- Include target keywords naturally
- Compelling, action-oriented descriptions

**Example:**

```astro
<meta
  name="description"
  content="Boost your search rankings with our expert SEO services. We deliver measurable results with data-driven strategies."
/>
```

### 2.3 Header Structure

**Check:**

- Proper heading hierarchy (h1 → h2 → h3)
- Only one h1 per page
- No skipped heading levels
- Descriptive, keyword-rich headings

**Example:**

```astro
<h1>SEO Services</h1>
<h2>Our Approach</h2>
<h3>Keyword Research</h3>
```

### 2.4 Image Optimization

**Check:**

- All images have alt text
- Descriptive file names
- WebP format when possible
- Compressed images
- Appropriate dimensions

**Example:**

```astro
<Image src={image} alt="SEO strategy diagram" width={1200} height={630} />
```

### 2.5 Internal Linking

**Check:**

- Logical internal linking structure
- No broken internal links
- Relevant anchor text
- Link to related content

### 2.6 Content Quality

**Check:**

- Content is unique and valuable
- No thin or duplicate content
- Proper keyword targeting
- Information gain for users

## Step 3: Open Graph and Social Tags

### 3.1 Open Graph Tags

**Check:**

- og:type (website or article)
- og:url (page URL)
- og:title (page title)
- og:description (page description)
- og:image (featured image)
- og:site_name (site name)

**Example:**

```astro
<meta property="og:type" content="website" />
<meta property="og:url" content="https://yoursite.com/services/seo" />
<meta property="og:title" content="SEO Services | Your Dedicated Marketer" />
<meta property="og:description" content="Boost your search rankings..." />
<meta property="og:image" content="https://yoursite.com/images/seo-og.jpg" />
<meta property="og:site_name" content="Your Dedicated Marketer" />
```

### 3.2 Twitter Cards

**Check:**

- twitter:card (summary_large_image)
- twitter:url (page URL)
- twitter:title (page title)
- twitter:description (page description)
- twitter:image (featured image)

**Example:**

```astro
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:url" content="https://yoursite.com/services/seo" />
<meta name="twitter:title" content="SEO Services | Your Dedicated Marketer" />
<meta name="twitter:description" content="Boost your search rankings..." />
<meta name="twitter:image" content="https://yoursite.com/images/seo-twitter.jpg" />
```

## Step 4: Tools and Validation

### 4.1 Google Search Console

**Check:**

- Index coverage report
- Mobile usability report
- Core Web Vitals report
- Manual actions
- Security issues

### 4.2 Rich Results Test

**Test structured data:**

- Google Rich Results Test
- Schema.org validator
- Check for rich result eligibility

### 4.3 PageSpeed Insights

**Run audit:**

- Desktop and mobile scores
- Core Web Vitals metrics
- Opportunities for improvement
- Diagnostics

### 4.4 Link Checker

**Check for broken links:**

```bash
npx broken-link-checker https://yoursite.com
```

## Step 5: Generate SEO Report

### Report Structure

1. **Executive Summary**
   - Overall SEO health score
   - Critical issues
   - Quick wins

2. **Technical SEO**
   - Crawlability issues
   - Performance metrics
   - Mobile optimization
   - Security status

3. **On-Page SEO**
   - Title tag issues
   - Meta description issues
   - Header structure problems
   - Content quality assessment

4. **Structured Data**
   - Schema markup validation
   - Rich result eligibility
   - Missing schema types

5. **Recommendations**
   - Priority issues to fix
   - Quick wins
   - Long-term improvements

## Common Issues and Fixes

### Missing Canonical Tags

**Fix:** Add canonical tags to all pages

```astro
<link rel="canonical" href={Astro.url} />
```

### Missing Alt Text

**Fix:** Add descriptive alt text to all images

```astro
<Image src={image} alt="Descriptive text" />
```

### Slow Page Load

**Fix:**

- Optimize images
- Minimize JavaScript
- Enable compression
- Use CDN

### Duplicate Content

**Fix:**

- Add canonical tags
- Consolidate similar pages
- Rewrite duplicate content

## Next Steps

After audit:

1. Fix critical issues first
2. Implement quick wins
3. Plan long-term improvements
4. Re-audit after changes
5. Monitor in Search Console
