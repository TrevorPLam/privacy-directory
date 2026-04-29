---
name: performance-audit
description: Performs comprehensive performance audit focusing on Core Web Vitals, bundle size, and optimization opportunities
---

# Performance Audit

This skill guides you through performing a comprehensive performance audit of the Astro marketing website.

## Step 1: Core Web Vitals Assessment

### 1.1 Largest Contentful Paint (LCP)

**Target:** < 2.5 seconds

**What it measures:** Time it takes for the largest content element to load

**Common causes:**

- Slow server response times
- Render-blocking resources
- Large image files
- Unoptimized JavaScript

**Audit with Lighthouse:**

```bash
npx lighthouse https://yoursite.com --view --only-categories=performance
```

**Fixes:**

- Optimize images (use Astro Image component)
- Preload critical resources
- Use CDN for static assets
- Minimize JavaScript bundle size

### 1.2 Interaction to Next Paint (INP)

**Target:** < 200 milliseconds

**What it measures:** Time from user interaction to page response

**Note:** INP replaced FID (First Input Delay) as the Core Web Vitals responsiveness metric in March 2024. FID is no longer part of Core Web Vitals.

**Common causes:**

- Long JavaScript execution
- Heavy event handlers
- Complex animations
- Synchronous operations

**Fixes:**

- Use code splitting for React components
- Debounce event handlers
- Use web workers for heavy computations
- Optimize animation performance

### 1.3 Cumulative Layout Shift (CLS)

**Target:** < 0.1

**What it measures:** Visual stability of the page

**Common causes:**

- Images without dimensions
- Dynamic content injection
- Ads and embeds
- Fonts loading late

**Fixes:**

- Always specify image width and height
- Reserve space for dynamic content
- Use font-display: swap
- Avoid inserting content above existing content

## Step 2: Lighthouse Audit

### 2.1 Run Full Lighthouse Audit

```bash
npx lighthouse https://yoursite.com --view --output=html --output-path=./lighthouse-report.html
```

### 2.2 Analyze Scores

**Performance Categories:**

- Performance (target: 90+)
- Accessibility (target: 90+)
- Best Practices (target: 90+)
- SEO (target: 90+)

### 2.3 Review Opportunities

Lighthouse provides specific recommendations:

- Reduce unused JavaScript
- Eliminate render-blocking resources
- Properly size images
- Use efficient cache policy
- Minify CSS and JavaScript

## Step 3: Bundle Size Analysis

### 3.1 Analyze Bundle Size

**Tools:**

- Astro build output
- webpack-bundle-analyzer (if using bundler)
- Rollup visualizer (Astro uses Rollup)

**Check build output:**

```bash
npm run build
```

Review the `dist/` directory for bundle sizes.

### 3.2 Identify Large Dependencies

**Check package.json:**

```bash
npx why package-name
```

**Common large dependencies:**

- React and React DOM
- Heavy UI libraries
- Unnecessary packages

**Fixes:**

- Use tree shaking
- Code split routes
- Lazy load components
- Remove unused dependencies

### 3.3 Optimize JavaScript

**Strategies:**

- Use Astro's static rendering by default
- Only use React islands for interactivity
- Lazy load heavy components
- Minimize client-side JavaScript

**Example lazy loading:**

```tsx
const HeavyComponent = lazy(() => import("./HeavyComponent"));
```

## Step 4: Image Optimization

### 4.1 Audit Images

**Check:**

- Image file sizes
- Image formats (WebP, AVIF preferred)
- Image dimensions
- Alt text presence

**Tools:**

- Lighthouse image audit
- Chrome DevTools Network tab
- Image optimization tools

### 4.2 Optimize Images

**Strategies:**

- Use Astro Image component
- Serve WebP/AVIF formats
- Resize images to appropriate dimensions
- Compress images
- Use responsive images

**Example:**

```astro
---
import { Image } from "astro:assets";
import heroImage from "/images/hero.jpg";
---

<Image
  src={heroImage}
  alt="Hero"
  width={1920}
  height={1080}
  format="webp"
  quality={75}
  loading="eager"
/>
```

## Step 5: Network Performance

### 5.1 Check Network Requests

**Tools:**

- Chrome DevTools Network tab
- WebPageTest
- GTmetrix

**Check:**

- Number of requests
- Total transfer size
- Request waterfall
- Resource loading order

### 5.2 Optimize Network

**Strategies:**

- Enable compression (gzip/brotli)
- Use CDN for static assets
- Implement caching headers
- Minimize HTTP requests
- Use HTTP/2 or HTTP/3

**Cloudflare Workers:**
Cloudflare automatically provides:

- CDN edge caching
- Brotli compression
- HTTP/3 support
- Image optimization

## Step 6: Caching Strategy

### 6.1 Review Cache Headers

**Check current caching:**

```bash
curl -I https://yoursite.com/style.css
```

**Recommended cache headers:**

- Static assets: 1 year
- HTML: short cache or no cache
- API responses: appropriate TTL

### 6.2 Implement Service Worker (Optional)

For offline capability and faster repeat visits:

```javascript
// sw.js
self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open("v1").then((cache) => {
      return cache.addAll(["/", "/styles/global.css", "/scripts/main.js"]);
    })
  );
});
```

## Step 7: Mobile Performance

### 7.1 Test on Mobile

**Tools:**

- Chrome DevTools device emulation
- Real mobile devices
- Lighthouse mobile audit

**Check:**

- Page load speed on 3G
- Touch responsiveness
- Mobile-specific issues
- Battery impact

### 7.2 Optimize for Mobile

**Strategies:**

- Use responsive images
- Minimize JavaScript on mobile
- Optimize for slow connections
- Test on various devices

## Step 8: Monitoring Setup

### 8.1 Google Search Console

**Setup:**

- Add site to Google Search Console
- Enable Core Web Vitals report
- Set up alerts for issues

### 8.2 Real User Monitoring (RUM)

**Tools:**

- Google Analytics
- Cloudflare Web Analytics
- Custom RUM solutions

**Track:**

- Real user performance
- Geographic performance
- Device-specific performance
- Network condition impact

## Step 9: Generate Performance Report

### Report Structure

1. **Executive Summary**
   - Overall performance score
   - Core Web Vitals status
   - Critical issues
   - Quick wins

2. **Core Web Vitals**
   - LCP score and issues
   - INP score and issues
   - CLS score and issues
   - Field data vs lab data

3. **Bundle Analysis**
   - Total bundle size
   - Largest dependencies
   - Code splitting opportunities
   - Unused code

4. **Image Optimization**
   - Image size issues
   - Format recommendations
   - Lazy loading opportunities
   - Compression gains

5. **Network Performance**
   - Request count
   - Transfer size
   - Caching status
   - CDN utilization

6. **Recommendations**
   - Priority issues to fix
   - Quick wins
   - Long-term improvements
   - Estimated impact

## Common Issues and Fixes

### Slow LCP

**Fix:**

- Optimize hero image
- Preload critical CSS
- Use CDN
- Improve server response time

### Poor INP

**Fix:**

- Code split React components
- Debounce event handlers
- Use web workers
- Optimize animations

### High CLS

**Fix:**

- Add image dimensions
- Reserve space for dynamic content
- Use font-display: swap
- Avoid layout shifts

### Large Bundle Size

**Fix:**

- Remove unused dependencies
- Code split routes
- Lazy load components
- Use tree shaking

### Slow Mobile Performance

**Fix:**

- Optimize images for mobile
- Reduce JavaScript on mobile
- Use responsive images
- Test on slow connections

## Performance Budgets

### Recommended Budgets

**JavaScript:**

- Initial bundle: < 100KB gzipped
- Total per route: < 200KB gzipped

**CSS:**

- Initial CSS: < 50KB gzipped
- Total CSS: < 100KB gzipped

**Images:**

- Hero image: < 500KB
- Other images: < 200KB each

**Total Page Weight:**

- Desktop: < 2MB
- Mobile: < 1MB

## Next Steps

After audit:

1. Fix critical performance issues first
2. Implement quick wins
3. Set up performance monitoring
4. Establish performance budgets
5. Re-audit after changes
6. Monitor real user data
