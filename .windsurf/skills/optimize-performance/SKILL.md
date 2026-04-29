---
name: optimize-performance
description: Guides performance optimization including bundle analysis, lazy loading, caching strategies, and Core Web Vitals
---

# Performance Optimization Guide

This skill guides you through optimizing the Astro marketing website for performance.

## Performance Audit

### Run Lighthouse Audit

```bash
# Install Lighthouse CLI
npm install -g lighthouse

# Run audit
lighthouse https://yourdomain.com --view

# Run with specific categories
lighthouse https://yourdomain.com --only-categories=performance,accessibility,best-practices,seo
```

### Key Metrics to Monitor

- **LCP (Largest Contentful Paint)**: < 2.5s
- **FID (First Input Delay)**: < 100ms
- **CLS (Cumulative Layout Shift)**: < 0.1
- **TTFB (Time to First Byte)**: < 600ms
- **FCP (First Contentful Paint)**: < 1.8s
- **TTI (Time to Interactive)**: < 3.8s

## Bundle Analysis

### Analyze Bundle Size

```bash
# Install bundle analyzer
npm install --save-dev @rollup/plugin-visualizer

# Add to astro.config.mjs
import { visualizer } from '@rollup/plugin-visualizer';

export default defineConfig({
  vite: {
    plugins: [visualizer()],
  },
});
```

### Run Bundle Analysis

```bash
# Build with visualization
npm run build

# View report
open stats.html
```

### Bundle Optimization Strategies

1. **Code Splitting**
   - Use dynamic imports for heavy components
   - Split vendor code from application code
   - Lazy load routes

2. **Tree Shaking**
   - Remove unused code
   - Use ES modules
   - Avoid importing entire libraries

3. **Minification**
   - Enable minification in production
   - Minify HTML, CSS, JavaScript
   - Use terser for JavaScript

## Image Optimization

### Use Astro Image Component

```astro
---
import { Image } from "astro:assets";
import myImage from "../images/my-image.jpg";
---

<Image src={myImage} alt="Description" width={800} height={600} loading="lazy" format="webp" />
```

### Image Best Practices

- Use WebP format with fallbacks
- Compress images before upload
- Use responsive images with srcset
- Lazy load below-fold images
- Use appropriate dimensions
- Include alt text

### Image Optimization Commands

```bash
# Install sharp for image processing
npm install sharp

# Optimize images
sharp input.jpg --webp output.webp
sharp input.jpg --quality 80 output.jpg
```

## Lazy Loading

### Lazy Load Images

```astro
<img src="/image.jpg" alt="Description" loading="lazy" decoding="async" />
```

### Lazy Load Components

```astro
---
import HeavyComponent from "./HeavyComponent.astro";
---

<HeavyComponent client:visible />
```

### Lazy Load JavaScript

```tsx
import { lazy, Suspense } from "react";

const HeavyComponent = lazy(() => import("./HeavyComponent"));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

## Caching Strategies

### Browser Caching

Add to `astro.config.mjs`:

```javascript
export default defineConfig({
  build: {
    inlineStylesheets: "auto",
  },
});
```

### CDN Caching

- Deploy to CDN (Vercel, Cloudflare, Netlify)
- Configure cache rules
- Use long cache for static assets (1 year)
- Use short cache for HTML (no-cache)
- Enable compression

### Service Worker Caching

```javascript
// public/sw.js
const CACHE_NAME = "v1";
const urlsToCache = ["/"];

self.addEventListener("install", (event) => {
  event.waitUntil(caches.open(CACHE_NAME).then((cache) => cache.addAll(urlsToCache)));
});

self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => response || fetch(event.request))
  );
});
```

## Code Splitting

### Dynamic Imports

```tsx
const Component = lazy(() => import("./Component"));
```

### Route-Based Splitting

Astro automatically splits routes. Configure in `astro.config.mjs`:

```javascript
export default defineConfig({
  build: {
    format: "directory",
  },
});
```

### Manual Chunk Splitting

```javascript
// astro.config.mjs
export default defineConfig({
  vite: {
    build: {
      rollupOptions: {
        output: {
          manualChunks: {
            vendor: ["react", "react-dom"],
            ui: ["@components/Button", "@components/Card"],
          },
        },
      },
    },
  },
});
```

## CSS Optimization

### Purge Unused CSS

Tailwind automatically purges unused CSS. Verify in `tailwind.config.mjs`:

```javascript
export default {
  content: ["./src/**/*.{astro,html,js,jsx,md,mdx,svelte,ts,tsx,vue}"],
};
```

### Critical CSS Inlining

```javascript
// astro.config.mjs
export default defineConfig({
  build: {
    inlineStylesheets: "auto",
  },
});
```

### CSS Minification

Enabled by default in production builds.

## Font Optimization

### Font Loading Strategy

```astro
---
import "@fontsource/inter";
import "@fontsource/space-grotesk";
---

<style global>
  @font-face {
    font-family: "Inter";
    font-display: swap;
  }
</style>
```

### Font Subsetting

```javascript
// tailwind.config.mjs
export default {
  theme: {
    extend: {
      fontFamily: {
        sans: ["Inter", "sans-serif"],
      },
    },
  },
};
```

## Network Optimization

### Resource Hints

Add to `src/layouts/BaseLayout.astro`:

```astro
<head>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
</head>
```

### HTTP/2 and HTTP/3

Enabled by default on modern hosting platforms (Vercel, Cloudflare, Netlify).

## Performance Monitoring

### Web Vitals Monitoring

```tsx
import { useReportWebVitals } from "web-vitals";

export function WebVitals() {
  useReportWebVitals((metric) => {
    console.log(metric);
    // Send to analytics
  });
  return null;
}
```

### Real User Monitoring (RUM)

Set up RUM with:

- Google Analytics
- Cloudflare Web Analytics
- Vercel Analytics
- Custom monitoring solution

## Performance Budgets

### Set Budgets in Lighthouse

```json
{
  "budgets": [
    {
      "resourceSizes": [
        {
          "resourceType": "script",
          "budget": 200
        }
      ]
    }
  ]
}
```

### Bundle Size Budgets

```javascript
// astro.config.mjs
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        chunkSizeWarningLimit: 200,
      },
    },
  },
});
```

## Common Performance Issues

### Large JavaScript Bundle

**Symptoms**: Slow TTI, large bundle size

**Solutions**:

- Code split heavy components
- Remove unused dependencies
- Use tree shaking
- Lazy load routes
- Consider lighter alternatives

### Slow Image Loading

**Symptoms**: Slow LCP, large image sizes

**Solutions**:

- Optimize image sizes
- Use modern formats (WebP, AVIF)
- Implement lazy loading
- Use responsive images
- Serve from CDN

### Layout Shifts

**Symptoms**: High CLS score

**Solutions**:

- Reserve space for dynamic content
- Use aspect-ratio for media
- Avoid inserting content above
- Use transform animations
- Test with CLS monitoring

### Slow Time to Interactive

**Symptoms**: Slow TTI, long main thread work

**Solutions**:

- Reduce JavaScript execution
- Defer non-critical scripts
- Optimize main thread work
- Use web workers for heavy tasks
- Minimize render-blocking resources

## Performance Checklist

Before deploying optimizations:

- [ ] Lighthouse score > 90
- [ ] Core Web Vitals in green
- [ ] Images optimized and compressed
- [ ] CSS purged of unused styles
- [ ] JavaScript minified and split
- [ ] Critical resources preloaded
- [ ] Lazy loading configured
- [ ] Caching strategy implemented
- [ ] CDN configured
- [ ] Compression enabled
- [ ] Performance budgets met
- [ ] Bundle size analyzed

## Testing Performance

### Local Testing

```bash
# Build production version
npm run build

# Preview production build
npm run preview

# Run Lighthouse on preview
lighthouse http://localhost:4321
```

### Production Testing

```bash
# Run Lighthouse on production
lighthouse https://yourdomain.com

# Run PageSpeed Insights
# https://pagespeed.web.dev/
```

## Continuous Monitoring

### Set Up Alerts

Monitor:

- Lighthouse scores
- Core Web Vitals
- Bundle size
- Page load times
- Error rates

### Tools

- Google PageSpeed Insights
- Lighthouse CI
- WebPageTest
- GTmetrix
- Cloudflare Analytics

## After Optimization

1. Run Lighthouse audit
2. Verify Core Web Vitals
3. Test in multiple browsers
4. Test on mobile devices
5. Monitor performance over time
6. Set up alerts for regressions
7. Document changes
8. Share results with team
