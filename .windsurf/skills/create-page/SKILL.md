---
name: create-page
description: Guides creation of new Astro pages with proper SEO meta tags and layouts
---

# Page Creation Guide

## File Location

Create pages in `src/pages/` with `.astro` extension.

## Page Structure

```astro
---
import BaseLayout from "@layouts/BaseLayout.astro";

const title = "Page Title";
const description = "Page description for SEO (150-160 characters)";
const image = "/og-image.jpg";
---

<BaseLayout title={title} description={description} image={image}>
  <main>
    <h1>Page Heading</h1>
    <p>Page content</p>
  </main>
</BaseLayout>
```

## Dynamic Routes

For dynamic routes (blog posts, case studies):

```astro
---
import { getCollection } from "astro:content";
import BaseLayout from "@layouts/BaseLayout.astro";

export async function getStaticPaths() {
  const posts = await getCollection("blog");
  return posts.map((post) => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await post.render();
---

<BaseLayout title={post.data.title} description={post.data.description} type="article">
  <article>
    <h1>{post.data.title}</h1>
    <Content />
  </article>
</BaseLayout>
```

## SEO Requirements

Every page must include:

**Required Props for BaseLayout:**

- `title`: Page title (format: "Page Title | Your Dedicated Marketer")
- `description`: Page description (150-160 characters)
- `image`: Featured image path (default: `/og-image.png`)
- `canonicalUrl`: Canonical URL (default: current URL)
- `type`: 'website' or 'article' (default: 'website')

**For Blog Posts:**

- `type: "article"`
- `publishedTime`: Publication date
- `author`: Author name

## Page Types

**Main Pages** (src/pages/):

- `index.astro` - Landing page
- `about.astro` - About page
- `contact.astro` - Contact page
- `pricing.astro` - Pricing page
- `team.astro` - Team page
- `faq.astro` - FAQ page

**Service Pages** (src/pages/services/):

- `index.astro` - Services overview
- `seo.astro` - SEO service page
- `paid-media.astro` - Paid Media service page
- etc.

**Content Pages**:

- `blog/index.astro` - Blog listing
- `blog/[slug].astro` - Blog post detail
- `work/index.astro` - Case studies listing
- `work/[slug].astro` - Case study detail

## JSON-LD Structured Data

BaseLayout automatically includes:

- Organization schema for main pages
- Article schema for blog posts

## After Creation

1. Run `npm run dev` to verify the page loads
2. Check page title and meta tags
3. Verify Open Graph tags
4. Test responsive design
5. Check accessibility (heading hierarchy, semantic HTML)
6. Verify canonical URL
7. Test in both light and dark modes
