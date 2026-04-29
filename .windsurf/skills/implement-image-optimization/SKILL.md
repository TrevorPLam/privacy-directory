---
name: implement-image-optimization
description: Replaces all HTML img tags with Astro Image component for automatic optimization
---

# Implement Image Optimization

This skill guides you through replacing all `<img>` tags with Astro's `<Image />` component for automatic optimization.

## Prerequisites

- Astro project with Sharp installed (`npm install sharp`)
- Images located in `public/` directory or imported from `src/`

## Installation

### 1. Install Sharp

```bash
npm install sharp
```

Sharp is required for Astro's image optimization.

## Step 1: Identify All img Tags

Search for all `<img>` tags in the codebase:

```bash
grep -r "<img" src/ --include="*.astro" --include="*.tsx"
```

Common locations:

- Component files in `src/components/`
- Page files in `src/pages/`
- Layout files in `src/layouts/`

## Step 2: Replace img Tags with Image Component

### Basic Replacement Pattern

**Before:**

```astro
<img src="/images/logo.png" alt="Company Logo" />
```

**After:**

```astro
---
import { Image } from "astro:assets";
import logo from "/images/logo.png";
---

<Image src={logo} alt="Company Logo" />
```

### For Remote Images

**Before:**

```astro
<img src="https://example.com/image.jpg" alt="Remote Image" />
```

**After:**

```astro
---
import { Image } from "astro:assets";
---

<Image src="https://example.com/image.jpg" alt="Remote Image" />
```

## Step 3: Update Component Files

### Components with Images

Check these components for img tags:

- `AuthorBio.astro` - Author avatar
- `ClientLogos.astro` - Client logos
- `CaseStudyCard.astro` - Case study images
- `TeamCard.astro` - Team member photos
- `PricingCard.astro` - Any images
- `ServiceCard.astro` - Service icons

### Example: AuthorBio.astro

```astro
---
import { Image } from "astro:assets";
import authorImage from "/images/author.jpg";
---

<Image src={authorImage} alt={authorName} width={200} height={200} class="rounded-full" />
```

## Step 4: Update Page Files

### Pages with Images

Check these pages for img tags:

- `index.astro` - Hero images, feature images
- `about.astro` - Team photos, office images
- `team.astro` - Team member photos
- `contact.astro` - Contact page images
- Service pages - Service-related images

### Example: index.astro

```astro
---
import { Image } from "astro:assets";
import heroImage from "/images/hero-bg.jpg";
---

<Image src={heroImage} alt="Hero background" width={1920} height={1080} class="w-full h-auto" />
```

## Step 5: Configure Image Options

### Common Image Attributes

- `width` and `height` - Required for aspect ratio
- `alt` - Required for accessibility
- `loading` - "eager" or "lazy" (default: lazy)
- `format` - "webp", "avif", or "png"
- `quality` - Image quality (1-100, default: 75)
- `sizes` - Responsive sizes

### Responsive Images

```astro
<Image
  src={image}
  alt="Description"
  width={1920}
  height={1080}
  sizes="(max-width: 768px) 100vw, 50vw"
/>
```

## Step 6: Update Content Collections

### Blog Post Images

For blog posts with frontmatter images:

**Before:**

```yaml
---
image: "/blog/post-image.jpg"
imageAlt: "Post description"
---
```

**In the blog post template:**

```astro
---
import { Image } from "astro:assets";
const { image, imageAlt } = Astro.props;
const postImage = image ? await import(image) : null;
---

{postImage && <Image src={postImage.default} alt={imageAlt} width={1200} height={630} />}
```

### Case Study Images

Similar approach for case studies in `src/pages/work/[slug].astro`.

## Step 7: Verify Configuration

### astro.config.mjs

Ensure image optimization is enabled (default in Astro 5+):

```javascript
export default defineConfig({
  image: {
    // Default configuration
    service: {
      entrypoint: "astro/assets/sharp",
    },
  },
});
```

## Testing

### 1. Build Verification

```bash
npm run build
```

Verify build succeeds with no image-related errors.

### 2. Local Testing

```bash
npm run dev
```

- Check that images load correctly
- Verify responsive behavior
- Test on different screen sizes
- Check browser console for errors

### 3. Image Quality Check

- Verify images are not blurry
- Check that WebP/AVIF formats are used
- Verify file sizes are reduced
- Test on slow connections

## Common Issues

### Module Not Found

If you see "module not found" for images:

- Ensure image paths are correct
- Check that images are in `public/` or imported from `src/`
- Verify file extensions are correct

### Sharp Installation Errors

If Sharp fails to install:

- Delete `node_modules` and `package-lock.json`
- Run `npm install` again
- Ensure you have build tools installed (on some systems)

### Build Errors

If build fails after Image component changes:

- Check that all images have `width` and `height`
- Verify all images have `alt` text
- Ensure `import` paths are correct
- Check for missing Sharp installation

## Performance Benefits

After implementing Image optimization:

- Images are automatically converted to WebP/AVIF
- Images are resized to appropriate dimensions
- Lazy loading is applied by default
- Images are served in modern formats
- Reduced bandwidth usage
- Faster page load times

## Accessibility

Ensure all images have:

- Descriptive `alt` text
- Empty `alt=""` for decorative images
- Proper aspect ratios to prevent layout shift
- Appropriate file sizes for mobile

## Next Steps

After implementation:

1. Run Lighthouse audit to verify performance improvements
2. Check Core Web Vitals (LCP, CLS)
3. Test on mobile devices
4. Verify images load correctly in production
5. Update documentation if needed
