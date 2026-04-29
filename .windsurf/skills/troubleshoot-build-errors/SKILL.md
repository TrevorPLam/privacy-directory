---
name: troubleshoot-build-errors
description: Guides troubleshooting Astro build failures, TypeScript errors, and dependency issues
---

# Build Error Troubleshooting Guide

This skill guides you through troubleshooting build errors in the Astro marketing website.

## Common Build Errors

### Astro Build Failures

#### Error: "Failed to load module"

**Symptoms**: Build fails with module loading error

**Solutions**:

```bash
# Clear cache
rm -rf node_modules .astro dist
npm install

# Rebuild
npm run build
```

#### Error: "Collection not found"

**Symptoms**: Content collection not found during build

**Solutions**:

```bash
# Check content config
cat src/content/config.ts

# Verify content files exist
ls src/content/blog/
ls src/content/caseStudies/

# Validate content schema
npm run astro check
```

#### Error: "Invalid frontmatter"

**Symptoms**: Frontmatter validation error

**Solutions**:

```bash
# Check specific file
cat src/content/blog/problematic-post.mdx

# Validate against schema
npm run astro check

# Fix frontmatter fields
# Ensure all required fields are present
```

### TypeScript Errors

#### Error: "Cannot find module"

**Symptoms**: TypeScript cannot find imported module

**Solutions**:

```bash
# Check tsconfig.json path aliases
cat tsconfig.json

# Verify file exists
ls src/components/Button.astro

# Use correct path alias
# Change from: import Button from '../../../components/Button.astro'
# To: import Button from '@components/Button.astro'
```

#### Error: "Property does not exist"

**Symptoms**: TypeScript property not found on type

**Solutions**:

```bash
# Check type definitions
cat src/env.d.ts

# Add missing type definition
# Example: Add to src/env.d.ts
interface ImportMetaEnv {
  readonly VITE_FORM_SUBMISSION_URL: string;
}
```

#### Error: "Type 'any' is not assignable"

**Symptoms**: Strict TypeScript error with any type

**Solutions**:

```typescript
// Replace 'any' with proper type
// Bad: const data: any = response.data;
// Good: const data: DataType = response.data;
```

### Dependency Errors

#### Error: "Module not found"

**Symptoms**: Cannot find installed package

**Solutions**:

```bash
# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install

# Check package.json
cat package.json

# Install missing package
npm install missing-package
```

#### Error: "Peer dependency conflict"

**Symptoms**: Peer dependency version mismatch

**Solutions**:

```bash
# Check dependency versions
npm list

# Use --legacy-peer-deps flag
npm install --legacy-peer-deps

# Or resolve version conflict manually
npm install package@version
```

#### Error: "ESM import error"

**Symptoms**: CommonJS/ESM import error

**Solutions**:

```bash
# Add "type": "module" to package.json
# Already present in this project

# Check for CommonJS dependencies
npm list

# Use ESM-compatible versions
```

## Configuration Errors

### Astro Config Errors

#### Error: "Invalid configuration"

**Symptoms**: astro.config.mjs validation error

**Solutions**:

```bash
# Check configuration
cat astro.config.mjs

# Validate against schema
npm run astro check

# Check for syntax errors
node -c astro.config.mjs
```

#### Error: "Integration not found"

**Symptoms**: Astro integration not found

**Solutions**:

```bash
# Install missing integration
npm install @astrojs/react

# Check integrations in config
cat astro.config.mjs

# Verify integration is installed
npm list @astrojs/react
```

### Tailwind Config Errors

#### Error: "Tailwind class not found"

**Symptoms**: Tailwind class not recognized

**Solutions**:

```bash
# Check Tailwind config
cat tailwind.config.mjs

# Verify content paths
# Ensure src/**/*.{astro,html,js,jsx,md,mdx,svelte,ts,tsx,vue} is in content array

# Rebuild CSS
npm run build
```

## Content Errors

### MDX Errors

#### Error: "MDX parse error"

**Symptoms**: MDX file parsing error

**Solutions**:

```bash
# Check MDX syntax
cat src/content/blog/problematic-post.mdx

# Validate MDX
npm run astro check

# Fix syntax errors
# Check for unclosed tags, invalid JSX, etc.
```

#### Error: "Frontmatter validation failed"

**Symptoms**: Frontmatter doesn't match schema

**Solutions**:

```bash
# Check schema
cat src/content/config.ts

# Check frontmatter
cat src/content/blog/problematic-post.mdx

# Fix frontmatter to match schema
# Ensure all required fields are present
# Ensure field types match schema
```

## Environment Variable Errors

### Error: "Environment variable not found"

**Symptoms**: Missing environment variable

**Solutions**:

```bash
# Check .env.example
cat .env.example

# Check .dev.vars
cat .dev.vars

# Add missing variable to .dev.vars
echo "VITE_FORM_SUBMISSION_URL=https://example.com" >> .dev.vars

# Restart dev server
npm run dev
```

### Error: "Invalid environment variable"

**Symptoms**: Environment variable validation error

**Solutions**:

```bash
# Check type definitions
cat src/env.d.ts

# Ensure variable matches type
# Example: VITE_FORM_SUBMISSION_URL should be string

# Restart dev server after changes
```

## Debugging Steps

### Step 1: Clear Cache

```bash
# Clear Astro cache
rm -rf .astro

# Clear node modules
rm -rf node_modules

# Clear dist
rm -rf dist

# Reinstall
npm install
```

### Step 2: Check Dependencies

```bash
# Check for outdated packages
npm outdated

# Update packages
npm update

# Audit for vulnerabilities
npm audit
npm audit fix
```

### Step 3: Validate Configuration

```bash
# Check Astro config
npm run astro check

# Check TypeScript
npx tsc --noEmit

# Check ESLint (if configured)
npm run lint
```

### Step 4: Test Incremental Build

```bash
# Build single file
astro build --filter /blog/specific-post

# Build specific page
astro build --filter /contact
```

### Step 5: Check Logs

```bash
# Run build with verbose output
astro build --verbose

# Check for specific errors
astro build 2>&1 | grep -i error
```

## Platform-Specific Issues

### Windows Issues

#### Error: "ENAMETOOLONG"

**Symptoms**: File path too long on Windows

**Solutions**:

```bash
# Use shorter directory names
# Move project closer to root
# Enable long paths in Windows
```

#### Error: "Permission denied"

**Symptoms**: Permission error on Windows

**Solutions**:

```bash
# Run terminal as administrator
# Check file permissions
# Close file locks in other programs
```

### macOS Issues

#### Error: "Command not found"

**Symptoms**: Command not found on macOS

**Solutions**:

```bash
# Check PATH
echo $PATH

# Reinstall Node.js
# Use nvm for Node version management
```

### Linux Issues

#### Error: "EACCES"

**Symptoms**: Permission error on Linux

**Solutions**:

```bash
# Fix npm permissions
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'

# Or use sudo (not recommended)
sudo npm install
```

## Getting Help

### Check Documentation

- Astro Docs: https://docs.astro.build
- React Docs: https://react.dev
- Tailwind Docs: https://tailwindcss.com
- TypeScript Docs: https://www.typescriptlang.org

### Search Issues

- Search GitHub Issues for Astro
- Search Stack Overflow
- Check Astro Discord community
- Check project GitHub issues

### Create Minimal Reproduction

If issue persists:

1. Create minimal reproduction
2. Reproduce in fresh project
3. Document steps to reproduce
4. Include error messages
5. Include configuration files
6. Open GitHub issue

## Prevention

### Regular Maintenance

```bash
# Update dependencies monthly
npm update

# Audit for vulnerabilities
npm audit

# Run tests
npm test

# Run build
npm run build
```

### Code Quality

```bash
# Run linter
npm run lint

# Format code
npm run format

# Type check
npm run astro check
```

### CI/CD Integration

Add to `.github/workflows/ci.yml`:

```yaml
- name: Type Check
  run: npm run astro check

- name: Lint
  run: npm run lint

- name: Test
  run: npm test

- name: Build
  run: npm run build
```

## Quick Reference

### Common Commands

```bash
# Clear cache and rebuild
rm -rf .astro node_modules dist && npm install && npm run build

# Type check
npm run astro check

# Validate content
npm run astro check

# Build with verbose output
astro build --verbose

# Preview build
npm run preview

# Check dependencies
npm list
npm outdated
npm audit
```

### Error Patterns

| Error Pattern      | Common Cause           | Solution             |
| ------------------ | ---------------------- | -------------------- |
| Module not found   | Missing dependency     | npm install package  |
| Cannot find module | Wrong import path      | Check path alias     |
| Type error         | TypeScript strict mode | Fix type annotations |
| Build timeout      | Large bundle           | Optimize bundle size |
| Out of memory      | Memory limit           | Increase Node memory |

## After Fixing Errors

1. Verify build succeeds
2. Run all tests
3. Test in development
4. Test in preview
5. Deploy to staging
6. Monitor for issues
7. Document the fix
8. Update documentation if needed
