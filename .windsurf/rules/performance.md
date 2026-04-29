---
trigger: always_on
---

<performance_guidelines>

# Performance Optimization Guidelines

When optimizing site performance:

## Core Web Vitals Targets

### LCP (Largest Contentful Paint)

- Target: < 2.5 seconds
- Optimize hero images and above-fold content
- Use responsive images with proper sizing
- Preload critical resources
- Minimize render-blocking resources

### INP (Interaction to Next Paint)

- Target: < 200 milliseconds
- Minimize JavaScript execution time
- Use event delegation for many listeners
- Avoid long tasks (>50ms)
- Optimize input handlers

### CLS (Cumulative Layout Shift)

- Target: < 0.1
- Reserve space for dynamic content
- Use aspect-ratio for images and videos
- Avoid inserting content above existing content
- Use transform animations instead of layout changes

## Bundle Optimization

### Code Splitting

- Use dynamic imports for heavy components
- Split vendor code from application code
- Lazy load routes with Astro's prefetch
- Configure manual chunks in Vite config
- Analyze bundle size regularly

### Tree Shaking

- Remove unused code via tree shaking
- Use ES modules for better tree shaking
- Avoid importing entire libraries
- Use specific imports: `import { debounce } from 'lodash'`
- Configure sideEffects in package.json

### Minification

- Enable minification in production build
- Minify HTML, CSS, and JavaScript
- Use terser for JavaScript optimization
- Configure compression in Astro config
- Enable gzip/brotli compression

## Asset Optimization

### Images

- Use Astro Image component for optimization
- Serve WebP format with fallbacks
- Use responsive images with srcset
- Lazy load below-fold images
- Compress images before upload
- Use appropriate image formats (WebP, AVIF)

### Fonts

- Use font-display: swap for faster rendering
- Subset fonts to include only needed characters
- Use WOFF2 format for better compression
- Preload critical fonts
- Avoid font flash with proper loading strategy

### CSS

- Purge unused CSS with Tailwind
- Minify CSS in production
- Use critical CSS inlining for above-fold
- Avoid @import for CSS loading
- Use media queries for responsive styles

## Caching Strategies

### Browser Caching

- Set appropriate Cache-Control headers
- Use long cache for static assets (1 year)
- Use short cache for HTML (no-cache)
- Use ETags for cache validation
- Implement service worker for offline support

### CDN Caching

- Deploy to CDN for global distribution
- Configure CDN cache rules
- Use cache busting for asset updates
- Enable CDN compression
- Monitor cache hit rates

### API Caching

- Cache API responses when appropriate
- Use stale-while-revalidate strategy
- Implement cache invalidation
- Cache static data in build time
- Use KV storage for edge caching

## Lazy Loading

### Images

- Use loading="lazy" for below-fold images
- Use native lazy loading (browser support)
- Consider Intersection Observer for control
- Lazy load iframes and videos
- Use placeholder images for smooth loading

### JavaScript

- Lazy load non-critical JavaScript
- Use dynamic imports for code splitting
- Defer non-essential scripts
- Use async for independent scripts
- Load JavaScript after critical path

### Components

- Use Astro's client:load directive for interactivity
- Use client:visible for viewport-based loading
- Use client:idle for low-priority components
- Avoid client:visible for above-fold content
- Minimize React islands for performance

## Network Optimization

### HTTP/2 and HTTP/3

- Enable HTTP/2 on server
- Use HTTP/3 if available
- Multiplex requests over single connection
- Reduce connection overhead
- Use server push for critical resources

### Resource Hints

- Use preload for critical resources
- Use prefetch for likely next navigation
- Use preconnect for important origins
- Use dns-prefetch for DNS resolution
- Avoid overusing resource hints

### Reduce Requests

- Combine CSS files when possible
- Use sprites for small icons
- Use data URIs for tiny assets
- Minimize HTTP requests
- Use HTTP/2 multiplexing effectively

## Server-Side Optimization

### Static Generation

- Pre-render pages at build time
- Use ISR for dynamic content
- Cache build artifacts
- Optimize build time
- Use incremental builds

### Edge Computing

- Deploy to edge network
- Use edge functions for dynamic content
- Cache at edge for faster responses
- Reduce server round trips
- Use Cloudflare Workers for edge logic

### Compression

- Enable gzip compression
- Enable Brotli compression (better ratio)
- Compress text-based assets
- Configure compression levels
- Test compression effectiveness

## Monitoring and Analysis

### Performance Monitoring

- Use Lighthouse for performance audits
- Monitor Core Web Vitals in production
- Use RUM (Real User Monitoring)
- Track performance metrics over time
- Set up performance budgets

### Bundle Analysis

- Use webpack-bundle-analyzer or similar
- Identify large dependencies
- Find optimization opportunities
- Track bundle size over time
- Set bundle size budgets

### Performance Budgets

- Set budgets for JavaScript size
- Set budgets for CSS size
- Set budgets for image sizes
- Fail build if budgets exceeded
- Adjust budgets as needed

## Anti-Patterns

- Do not lazy load above-fold content
- Do not skip image optimization
- Do not load unnecessary JavaScript
- Do not ignore bundle size
- Do not use blocking resources
- Do not forget to compress assets
- Do not overuse animations
- Do not implement complex layouts with CLS
- Do not skip caching strategies
- Do not ignore performance monitoring

## Performance Checklist

Before deploying:

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

## Common Performance Issues

### Large JavaScript Bundle

- Split code into smaller chunks
- Remove unused dependencies
- Use tree shaking
- Lazy load routes
- Consider lighter alternatives

### Slow Image Loading

- Optimize image sizes
- Use modern formats (WebP, AVIF)
- Implement lazy loading
- Use responsive images
- Serve from CDN

### Layout Shifts

- Reserve space for dynamic content
- Use aspect-ratio for media
- Avoid inserting content above
- Use transform animations
- Test with CLS monitoring

### Slow Time to Interactive

- Reduce JavaScript execution
- Defer non-critical scripts
- Optimize main thread work
- Use web workers for heavy tasks
- Minimize render-blocking resources

</performance_guidelines>
