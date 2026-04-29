---
name: deploy-to-production
description: Guides the deployment process to production with safety checks for GitHub Pages and Cloudflare Workers
---

# Deploy to Production

This skill guides you through deploying the Astro marketing website to production with proper safety checks.

## Pre-Deployment Checklist

### 1. Build Verification

- Run `npm run build` to ensure production build succeeds
- Run `npm run build:with-search` to include search index generation
- Verify no build errors or warnings
- Check that `dist/` directory is generated correctly

### 2. Test Verification

- Run `npm run test` to ensure all unit tests pass
- Run `npm run test:e2e` to ensure all E2E tests pass
- Verify test coverage is acceptable
- Check for any failing tests

### 3. Environment Variables

- Verify `.env.example` is up to date
- Ensure production environment variables are configured:
  - `VITE_FORM_SUBMISSION_URL` in Cloudflare Workers
  - `VITE_CALENDLY_URL` in Cloudflare Workers
- Never commit actual `.env` or `.dev.vars` files

### 4. Code Quality

- Run linting if ESLint/Prettier is configured
- Check for any console.log statements
- Verify no hardcoded secrets or API keys
- Review recent commits for breaking changes

## Deployment Steps

### GitHub Pages Deployment

1. **Push to GitHub**

   ```bash
   git add .
   git commit -m "chore: prepare for production deployment"
   git push origin main
   ```

2. **Trigger CI/CD**
   - Push to `main` branch triggers GitHub Actions workflow
   - Workflow runs: build, test (Vitest), test-e2e (Playwright)
   - Wait for all checks to pass

3. **Manual Approval**
   - Navigate to GitHub Actions tab
   - Find the workflow run for your commit
   - Click on the "deploy" job
   - Click "Review deployments"
   - Approve deployment to "production" environment

4. **Verify Deployment**
   - Wait for deployment to complete
   - Visit the deployed URL (configured in astro.config.mjs)
   - Test critical functionality (contact form, navigation, etc.)
   - Check GitHub Pages deployment status

### Cloudflare Workers Deployment

1. **Deploy Worker**

   ```bash
   npx wrangler deploy
   ```

2. **Configure Secrets**

   ```bash
   npx wrangler secret put FORM_SUBMISSION_URL
   npx wrangler secret put CALENDLY_URL
   ```

3. **Verify Worker**
   - Test form submission endpoint
   - Verify Calendly integration works
   - Check Cloudflare Workers dashboard for errors

## Post-Deployment Verification

### 1. Functionality Tests

- Test contact form submission
- Verify search functionality works
- Check all navigation links
- Test responsive design on mobile
- Verify Calendly integration

### 2. Performance Checks

- Run Lighthouse audit on deployed site
- Check Core Web Vitals (LCP, INP, CLS)
- Verify page load times are acceptable
- Check for any console errors

### 3. SEO Verification

- Check meta tags are present
- Verify canonical URLs
- Test with Google Rich Results Test
- Check sitemap is accessible
- Verify robots.txt is correct

## Rollback Procedure

If deployment fails or issues are discovered:

### GitHub Pages Rollback

1. Revert to previous commit:
   ```bash
   git revert <commit-hash>
   git push origin main
   ```
2. Wait for CI/CD to complete
3. Verify rollback succeeded

### Cloudflare Workers Rollback

1. Deploy previous version:
   ```bash
   git checkout <previous-commit>
   npx wrangler deploy
   ```
2. Restore secrets if needed
3. Verify functionality

## Troubleshooting

### Build Failures

- Check for missing dependencies: `npm install`
- Verify TypeScript errors: `npx astro check`
- Check for syntax errors in components

### Test Failures

- Run tests locally to reproduce: `npm run test` or `npm run test:e2e`
- Check test logs for specific failures
- Verify test environment is correct

### Deployment Failures

- Check GitHub Actions logs for errors
- Verify GitHub Pages is enabled in repository settings
- Check that `site` URL in astro.config.mjs is correct
- Verify GitHub token has correct permissions

## Security Considerations

- Never deploy with uncommitted secrets
- Always use environment variables for sensitive data
- Verify `.gitignore` excludes sensitive files
- Check that CI/CD secrets are properly configured
- Review deployment logs for leaked secrets

## Documentation

After successful deployment:

- Update deployment documentation if process changed
- Document any issues encountered and resolutions
- Note the deployed commit hash for reference
- Update changelog if applicable
