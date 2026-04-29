---
trigger: glob
globs: astro.config.mjs
---

<astro_config_guidelines>

# Astro Configuration Guidelines

When modifying `astro.config.mjs`:

## Integration Management

### Adding Integrations

- Use official Astro integrations when available (@astrojs/\*)
- Install integration package before adding to config
- Configure integrations with minimal required options
- Document custom integration options in comments

### Integration Order

- Framework integrations first (React, Vue, etc.)
- Content integrations second (MDX, RSS, etc.)
- Tooling integrations third (Tailwind, Partytown, etc.)
- Deployment adapters last

### Example Integration Pattern

```javascript
import { defineConfig } from "astro/config";
import react from "@astrojs/react";
import tailwind from "@astrojs/tailwind";
import mdx from "@astrojs/mdx";

export default defineConfig({
  integrations: [react(), tailwind(), mdx()],
});
```

## Build Configuration

### Output Configuration

- `output: 'static'` for static sites (default)
- `output: 'server'` for SSR with adapter
- `output: 'hybrid'` for mixed static/dynamic routes

### Build Options

- `build.format: 'directory'` for clean URL structure
- `build.assetsPrefix` for CDN asset hosting
- `build.inlineStylesheets: 'auto'` for critical CSS inlining

### Performance Settings

- Enable `compressHTML` for smaller HTML output
- Use `vite.build.rollupOptions` for bundle optimization
- Configure `build.assets` for asset fingerprinting

## Site Configuration

### Site Metadata

- `site`: Full canonical URL (e.g., https://example.com)
- `base`: Base path for subdirectory deployment
- `trailingSlash: 'ignore'` for flexible URL handling

### Redirects

- Use `redirects` array for simple redirects
- Prefer server-level redirects for complex rules
- Document redirect reasons in comments

## Server Configuration

### Development Server

- `server.host: true` for network access
- `server.port: 4321` (Astro default)
- `server.open: true` to auto-open browser

### Production Server

- Use adapter for production deployment
- Configure `server.headers` for security headers
- Set `server.allowedHosts` for restricted access

## Image Configuration

### Image Service

- Configure `image.service` for remote image optimization
- Add trusted domains to `image.domains`
- Use `image.remotePatterns` for dynamic sources

### Image Optimization

- Set `image.breakpoints` for responsive images
- Configure `image.layout` for sizing strategy
- Enable `image.objectFit` for aspect ratio handling

## Environment Variables

### Schema Validation

- Define `env.schema` for type-safe environment variables
- Use `env.validateSecrets: true` for CI validation
- Document required variables in comments

### Variable Access

- Client-side: `import.meta.env.VITE_*`
- Server-side: `process.env.*`
- Never expose secrets to client-side code

## Vite Configuration

### Build Optimization

- Use `vite.build.rollupOptions.output.manualChunks` for code splitting
- Configure `vite.build.chunkSizeWarningLimit` for bundle size limits
- Enable `vite.build.minify` for production builds

### Dev Server

- Configure `vite.server.proxy` for API proxying
- Set `vite.server.watch` for file watching options
- Use `vite.optimizeDeps` for dependency pre-bundling

## Security Configuration

### Security Headers

- Configure `server.headers` for CSP, HSTS, etc.
- Use `security` object for Astro security features
- Enable `security.checkOrigin` for origin validation

## Prefetch Configuration

### Prefetch Strategy

- `prefetch.prefetchAll: true` for aggressive prefetching
- `prefetch.defaultStrategy: 'viewport'` for viewport-based prefetching
- Consider bandwidth implications of prefetching

## Markdown Configuration

### Syntax Highlighting

- Configure `markdown.shikiConfig` for code highlighting
- Enable `markdown.syntaxHighlight: 'shiki'` for modern highlighting
- Add `markdown.remarkPlugins` for markdown transformations

### GFM Support

- Enable `markdown.gfm: true` for GitHub Flavored Markdown
- Configure `markdown.smartypants: true` for typography

## Font Configuration

### Font Optimization

- Use `font.provider: 'google'` for Google Fonts
- Configure `font.weights` for required font weights
- Enable `font.display: 'swap'` for font loading performance

## Migration Guidelines

### Upgrading Astro

- Check upgrade guide for breaking changes
- Update integrations to compatible versions
- Test build after version changes
- Update configuration for new features

### Configuration Validation

- Run `astro check` to validate configuration
- Use TypeScript for config file (astro.config.ts)
- Test configuration in dev mode before building

## Common Patterns

### Adapter Configuration

```javascript
import cloudflare from "@astrojs/cloudflare";

export default defineConfig({
  adapter: cloudflare({
    mode: "directory",
  }),
});
```

### Multi-Site Configuration

```javascript
export default defineConfig({
  site: "https://example.com",
  base: "/blog",
});
```

### Custom Vite Config

```javascript
export default defineConfig({
  vite: {
    build: {
      rollupOptions: {
        output: {
          manualChunks: {
            vendor: ["react", "react-dom"],
          },
        },
      },
    },
  },
});
```

## Anti-Patterns

- Do not hardcode environment-specific values
- Do not use deprecated configuration options
- Do not skip validation of environment variables
- Do not expose secrets in configuration
- Do not use incompatible integration versions

</astro_config_guidelines>
