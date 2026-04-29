---
name: deploy-cloudflare-worker
description: Guides Cloudflare Worker deployment with wrangler, secrets management, and environment configuration
---

# Cloudflare Worker Deployment Guide

This skill guides you through deploying Cloudflare Workers for the Astro marketing website.

## Prerequisites

- Node.js 18+ installed
- Wrangler CLI installed (`npm install -g wrangler`)
- Cloudflare account
- Cloudflare API token with Workers permissions

## Authentication

### Login to Cloudflare

```bash
wrangler login
```

This will open a browser to authenticate with your Cloudflare account.

### Verify Authentication

```bash
wrangler whoami
```

## Wrangler CLI 2026 Updates

### New Features

- **Rebuilt CLI**: Broader API coverage with unified developer experience
- **Dev Registry**: Cross-process service bindings for Durable Objects and tail workers
- **RPC APIs**: Remote procedure calls for Workers bindings
- **Interactive Commands**: Combine local development with API requests
- **Enhanced Debugging**: Debug port for cross-process service bindings

### Dev Registry Usage

```bash
# Use dev registry for local development
wrangler dev --local

# Enable Durable Object RPC via dev registry
wrangler dev --experimental
```

### Interactive CLI Commands

```bash
# Interactive mode for complex deployments
wrangler deploy --interactive

# Combine local dev with API requests
wrangler dev --remote
```

## Configuration

### wrangler.toml Setup

Ensure `wrangler.toml` is configured correctly:

```toml
name = "form-submission-worker"
main = "index.js"
compatibility_date = "2024-01-01"
compatibility_flags = ["nodejs_compat"]

[vars]
FORM_SUBMISSION_URL = "https://api.example.com/submit"

[secrets]
required = ["API_KEY", "DATABASE_URL"]

[env.production]
name = "form-submission-worker-prod"
```

### Environment-Specific Configuration

Create separate environments for development, staging, and production:

```toml
[env.staging]
name = "form-submission-worker-staging"
vars = { ENVIRONMENT = "staging" }

[env.production]
name = "form-submission-worker-prod"
vars = { ENVIRONMENT = "production" }
```

## Secrets Management

### Local Development Secrets

Create `.dev.vars` file in the worker directory:

```env
API_KEY=your_api_key_here
DATABASE_URL=your_database_url_here
```

Add `.dev.vars` to `.gitignore`:

```
.dev.vars
```

### Production Secrets

Add secrets using wrangler CLI:

```bash
# Add a secret
wrangler secret put API_KEY

# Add secret for specific environment
wrangler secret put API_KEY --env production

# List secrets (names only, not values)
wrangler secret list

# Delete a secret
wrangler secret delete API_KEY
```

### Secret Validation

Declare required secrets in `wrangler.toml`:

```toml
[secrets]
required = ["API_KEY", "DATABASE_URL"]
```

This validates that secrets are set before deployment.

## Local Development

### Start Local Server

```bash
# Start local dev server
wrangler dev

# Start on specific port
wrangler dev --port 8787

# Start with local execution only (no network)
wrangler dev --local
```

### Test with .dev.vars

The `.dev.vars` file is automatically loaded for local development.

### Remote Development

Test against production environment locally:

```bash
wrangler dev --remote
```

## Deployment

### Deploy to Development

```bash
# Deploy to default environment
wrangler deploy

# Deploy with custom name
wrangler deploy --name my-worker
```

### Deploy to Specific Environment

```bash
# Deploy to staging
wrangler deploy --env staging

# Deploy to production
wrangler deploy --env production
```

### Deployment Verification

After deployment, verify:

```bash
# Check deployment status
wrangler deployments list

# View logs
wrangler tail
```

## Worker Code Patterns

### Basic Handler

```javascript
export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);

    if (url.pathname === "/api/submit") {
      return handleFormSubmit(request, env);
    }

    return new Response("Not Found", { status: 404 });
  },
};
```

### Environment Variable Access

```javascript
export default {
  async fetch(request, env, ctx) {
    const apiKey = env.API_KEY;
    const dbUrl = env.DATABASE_URL;

    // Use secrets securely
  },
};
```

### CORS Configuration

```javascript
const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET, POST, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type",
};

export default {
  async fetch(request, env, ctx) {
    if (request.method === "OPTIONS") {
      return new Response(null, { headers: corsHeaders });
    }

    // Handle other requests
    const response = await handleRequest(request, env);
    return new Response(response.body, {
      headers: { ...response.headers, ...corsHeaders },
    });
  },
};
```

## Testing

### Local Testing

```bash
# Start dev server
wrangler dev

# Test with curl
curl http://localhost:8787/api/submit
```

### Remote Testing

```bash
# Deploy to staging
wrangler deploy --env staging

# Test staging endpoint
curl https://form-submission-worker-staging.your-subdomain.workers.dev/api/submit
```

### Unit Testing

Use `@cloudflare/vitest-pool-workers` for testing:

```bash
npm install --save-dev @cloudflare/vitest-pool-workers vitest
```

Create test file:

```javascript
import { describe, it, expect } from "vitest";
import worker from "../index";

describe("Worker", () => {
  it("should handle form submission", async () => {
    const request = new Request("http://localhost/api/submit", {
      method: "POST",
      body: JSON.stringify({ name: "Test" }),
    });

    const response = await worker.fetch(request, { API_KEY: "test" }, {});
    expect(response.status).toBe(200);
  });
});
```

## Monitoring

### View Logs

```bash
# View real-time logs
wrangler tail

# View logs for specific environment
wrangler tail --env production
```

### Analytics

Enable Workers Analytics in Cloudflare dashboard:

- Navigate to Workers & Pages
- Select your worker
- Enable Analytics
- View metrics in dashboard

## Troubleshooting

### Build Failures

```bash
# Check wrangler version
wrangler --version

# Update wrangler
npm update -g wrangler

# Check configuration
wrangler dev --dry-run
```

### Secret Issues

```bash
# Verify secrets are set
wrangler secret list

# Re-add secret if needed
wrangler secret put API_KEY
```

### Deployment Failures

```bash
# Check deployment logs
wrangler deployments list

# View specific deployment
wrangler deployments list --name form-submission-worker

# Retry deployment
wrangler deploy --force
```

## Best Practices

### Security

- Never commit secrets to git
- Use wrangler secret for sensitive data
- Rotate secrets regularly
- Use environment-specific secrets
- Never log sensitive data

### Performance

- Use streaming for large responses
- Use `ctx.waitUntil()` for background work
- Cache responses when appropriate
- Use edge caching for static content

### Code Quality

- Use TypeScript for type safety
- Write tests for critical paths
- Use error boundaries
- Implement proper error handling
- Log errors for debugging

## Rollback

### Rollback to Previous Version

```bash
# List deployments
wrangler deployments list

# Rollback to specific deployment
wrangler rollback <deployment-id>
```

### Quick Rollback

```bash
# Revert to previous commit and redeploy
git revert HEAD
wrangler deploy
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Deploy Worker

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
      - run: npm test
      - run: npx wrangler deploy --env production
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

## After Deployment

1. Test the deployed worker endpoint
2. Verify logs show no errors
3. Monitor performance metrics
4. Check analytics data
5. Test error scenarios
6. Verify CORS configuration
7. Test with production data
8. Document any issues found
