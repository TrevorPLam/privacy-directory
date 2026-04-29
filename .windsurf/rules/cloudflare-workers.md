---
trigger: glob
globs: worker/**/*,wrangler.toml
---

<cloudflare_workers_guidelines>

# Cloudflare Workers Guidelines

When developing or modifying Cloudflare Workers:

## Configuration (wrangler.toml)

### Basic Configuration

- Set `name` to match project name
- Configure `main` to point to entry file (default: index.js)
- Set `compatibility_date` to current date
- Enable `compatibility_flags = ["nodejs_compat"]` for Node.js APIs

### Environment Configuration

- Use `[vars]` for non-secret environment variables
- Use `[secrets]` declaration for required secrets (validated at deploy)
- Configure separate environments with `[env.production]`
- Never commit actual secret values

### Example wrangler.toml

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

## Secrets Management

### Local Development

- Use `.dev.vars` file for local secrets
- Add `.dev.vars` to `.gitignore`
- Never commit `.dev.vars` or actual secret values

### Production Secrets

- Use `wrangler secret put <SECRET_NAME>` to add secrets
- Secrets are encrypted and stored securely
- Access secrets via `env` parameter in fetch handler
- Rotate secrets regularly

### Secret Access Pattern

```javascript
export default {
  async fetch(request, env, ctx) {
    const apiKey = env.API_KEY;
    // Use apiKey securely
  },
};
```

## Worker Code Patterns

### Request Handling

- Always return a Response object
- Use proper HTTP status codes
- Set appropriate CORS headers
- Handle errors gracefully

### Streaming Responses

- Stream request and response bodies for large payloads
- Use `Response.body` stream interface
- Avoid buffering large responses in memory

### Async Work After Response

- Use `ctx.waitUntil()` for background work
- Logging, analytics, or cache warming after response
- Do not block response on non-critical work

### Example Handler Pattern

```javascript
export default {
  async fetch(request, env, ctx) {
    try {
      const url = new URL(request.url);

      // Handle request
      const response = await handleRequest(request, env);

      // Background work
      ctx.waitUntil(logAnalytics(request, response));

      return response;
    } catch (error) {
      return new Response("Internal Server Error", { status: 500 });
    }
  },
};
```

## Architecture Patterns

### Bindings for Cloudflare Services

- Use KV, D1, R2 bindings instead of REST APIs
- Configure bindings in wrangler.toml
- Access via `env` parameter
- Better performance and security

### Service Bindings

- Use service bindings for Worker-to-Worker communication
- Avoid HTTP calls between Workers
- Configure in wrangler.toml with `[[services]]`

### Durable Objects for WebSockets

- Use Durable Objects for WebSocket connections
- Configure in wrangler.toml with `[[durable_objects.bindings]]`
- Maintain state across WebSocket connections

### Static Assets

- Use Workers Static Assets for new projects
- Configure in wrangler.toml with `assets`
- Better performance than serving from Worker

## Security Best Practices

### Web Crypto

- Use Web Crypto API for secure token generation
- Prefer Web Crypto over Node.js crypto module
- Generate secure random values with `crypto.getRandomValues()`

### Input Validation

- Validate all request inputs
- Sanitize user-provided data
- Use parameterized queries for D1
- Never trust client-side validation

### Error Handling

- Do not use `passThroughOnException()` as error handling
- Implement proper try/catch blocks
- Return appropriate error responses
- Log errors for debugging (without sensitive data)

## Observability

### Logging

- Enable Workers Logs and Traces
- Use `console.log()` for debugging
- Use structured logging for production
- Do not log sensitive data

### Monitoring

- Set up Workers Analytics
- Monitor request patterns and errors
- Set up alerts for error rates
- Track performance metrics

## Development and Testing

### Local Testing

- Use `wrangler dev` for local development
- Test with `--local` flag for local execution
- Use `--port` to specify port
- Test secrets with `.dev.vars`

### Testing Framework

- Use `@cloudflare/vitest-pool-workers` for testing
- Write unit tests for Worker logic
- Test with mock bindings
- Test error scenarios

### Deployment

- Use `wrangler deploy` to deploy
- Specify environment with `--env production`
- Test in staging before production
- Monitor deployment logs

## Code Patterns

### No Global State

- Do not store request-scoped state in global scope
- Workers are stateless between requests
- Use Durable Objects for persistent state
- Use KV for shared state

### Promise Handling

- Always await or waitUntil your Promises
- Do not leave floating promises
- Use Promise.all() for parallel operations
- Handle promise rejections

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
  },
};
```

## Common Anti-Patterns

- Do not store secrets in wrangler.toml or source code
- Do not use setTimeout for delays (use waitUntil)
- Do not buffer large responses in memory
- Do not use global variables for request state
- Do not ignore promise rejections
- Do not use passThroughOnException for error handling
- Do not hardcode API keys or tokens
- Do not log sensitive data

## Wrangler Commands

### Development

```bash
wrangler dev                    # Start local dev server
wrangler dev --local            # Local execution only
wrangler dev --port 8787        # Custom port
```

### Secrets

```bash
wrangler secret put API_KEY     # Add secret
wrangler secret list            # List secrets
wrangler secret delete API_KEY  # Delete secret
```

### Deployment

```bash
wrangler deploy                 # Deploy to production
wrangler deploy --env staging   # Deploy to staging
wrangler deploy --name my-worker # Custom name
```

### Testing

```bash
wrangler tail                   # View real-time logs
wrangler pages project create   # Create Pages project
wrangler types                  # Generate TypeScript types
```

</cloudflare_workers_guidelines>
