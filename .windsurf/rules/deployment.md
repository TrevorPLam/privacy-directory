---
trigger: glob
globs: .github/workflows/**/*,vercel.json,netlify.toml
---

<deployment_guidelines>

# Deployment Guidelines

When configuring deployments:

## Platform Selection

### Recommended Platforms

- **Vercel**: Best for Astro projects, zero-config deployment
- **Netlify**: Good alternative with excellent build tools
- **Cloudflare Pages**: Great for global edge deployment
- **GitHub Pages**: Free option for static sites

### Platform Considerations

- Build time limits
- Edge network coverage
- Custom domain support
- Environment variable management
- Preview deployments
- Team collaboration features

## Vercel Deployment

### Configuration

- Create `vercel.json` for custom configuration
- Set build command: `npm run build`
- Set output directory: `dist`
- Configure environment variables in dashboard
- Enable preview deployments for branches

### Environment Variables

- Add `VITE_FORM_SUBMISSION_URL` in Vercel dashboard
- Add `VITE_CALENDLY_URL` in Vercel dashboard
- Use different values for production/preview
- Never commit secrets to git
- Use Vercel CLI for local testing

### Deployment Commands

```bash
vercel login                    # Authenticate
vercel                          # Deploy to preview
vercel --prod                   # Deploy to production
vercel env add                  # Add environment variable
vercel env pull .env.local      # Pull env vars locally
```

### vercel.json Example

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "astro",
  "regions": ["iad1"],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        }
      ]
    }
  ]
}
```

## Netlify Deployment

### Configuration

- Create `netlify.toml` for configuration
- Set build command: `npm run build`
- Set publish directory: `dist`
- Configure redirects in netlify.toml
- Enable branch previews

### Environment Variables

- Add in Netlify dashboard under Site settings
- Use environment-specific values
- Never commit secrets
- Use Netlify CLI for local testing

### Deployment Commands

```bash
netlify login                   # Authenticate
netlify deploy                  # Deploy to preview
netlify deploy --prod           # Deploy to production
netlify env:set                 # Set environment variable
netlify dev                     # Local development
```

### netlify.toml Example

```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/old-path"
  to = "/new-path"
  status = 301

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
```

## Cloudflare Pages Deployment

### Configuration

- Use `wrangler.toml` for Workers deployment
- Configure build settings in Cloudflare dashboard
- Set build command: `npm run build`
- Set output directory: `dist`
- Enable preview deployments

### Environment Variables

- Add in Cloudflare Pages dashboard
- Use environment-specific values
- Never commit secrets
- Use Wrangler CLI for Workers

### Deployment Commands

```bash
npx wrangler pages project create  # Create project
npx wrangler pages deploy dist     # Deploy manually
# Or use Git integration for auto-deploy
```

## GitHub Pages Deployment

### Configuration

- Use GitHub Actions for deployment
- Configure in `.github/workflows/deploy.yml`
- Set source to GitHub Actions in repository settings
- Enable GitHub Pages in repository settings
- Use custom domain if needed

### Environment Variables

- Add in GitHub repository Settings > Secrets
- Use GitHub Actions secrets for sensitive data
- Never commit secrets to git
- Use environment-specific secrets

### GitHub Actions Example

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run build
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

## Environment Management

### Development Environment

- Use `.dev.vars` for local development
- Add to `.gitignore`
- Test locally before deploying
- Use environment-specific URLs

### Staging/Preview Environment

- Deploy to preview/branch deployments
- Test before production
- Use staging APIs and services
- Monitor for issues

### Production Environment

- Require manual approval for production
- Use production APIs and services
- Monitor after deployment
- Have rollback plan ready

## Pre-Deployment Checklist

### Build Verification

- [ ] Build succeeds locally
- [ ] All tests pass
- [ ] Linting passes
- [ ] Type checking passes
- [ ] Environment variables configured
- [ ] No console errors in dev mode

### Content Verification

- [ ] All pages build successfully
- [ ] Images load correctly
- [ ] Links work properly
- [ ] Forms submit correctly
- [ ] Search index generated
- [ ] Sitemap updated

### Performance Verification

- [ ] Lighthouse score > 90
- [ ] Core Web Vitals in green
- [ ] Bundle size within budget
- [ ] Images optimized
- [ ] CSS purged

### Security Verification

- [ ] No secrets in code
- [ ] HTTPS enabled
- [ ] Security headers configured
- [ ] CSP configured
- [ ] Dependencies updated

## Post-Deployment Verification

### Immediate Checks

- [ ] Site loads successfully
- [ ] All pages accessible
- [ ] Forms work correctly
- [ ] No console errors
- [ ] Analytics tracking works

### Monitoring

- [ ] Set up error monitoring
- [ ] Monitor performance metrics
- [ ] Check analytics data
- [ ] Monitor uptime
- [ ] Set up alerts

## Rollback Procedures

### When to Rollback

- Critical bugs affecting users
- Security vulnerabilities
- Performance degradation
- Data corruption
- Deployment failures

### Rollback Methods

- Revert to previous commit
- Use platform rollback feature
- Restore from backup
- Deploy previous artifact
- Hotfix if rollback not possible

### Rollback Commands

```bash
# Vercel
vercel rollback                  # Rollback to previous deployment

# Netlify
netlify deploy --prod --rollback  # Rollback to previous deploy

# GitHub Pages
git revert <commit>              # Revert commit
git push origin main             # Push revert
```

## CI/CD Integration

### GitHub Actions

- Build on every push
- Run tests on every push
- Deploy on main branch
- Require approval for production
- Notify on deployment status

### Workflow Triggers

- Push to main branch
- Pull requests
- Manual trigger
- Scheduled builds
- Tag releases

### Deployment Environments

- Development: on every push
- Staging: on pull requests
- Production: on main merge with approval

## Anti-Patterns

- Do not deploy without testing
- Do not commit secrets to git
- Do not skip environment variable configuration
- Do not deploy to production without approval
- Do not ignore build errors
- Do not skip rollback planning
- Do not deploy during high-traffic periods without testing
- Do not forget to monitor after deployment

## Troubleshooting

### Build Failures

- Check build logs for errors
- Verify dependencies are installed
- Check environment variables
- Verify Node.js version
- Test locally first

### Deployment Failures

- Check deployment logs
- Verify platform configuration
- Check authentication
- Verify repository permissions
- Check file size limits

### Runtime Errors

- Check browser console
- Verify environment variables
- Check API endpoints
- Verify CDN configuration
- Check CORS settings

</deployment_guidelines>
