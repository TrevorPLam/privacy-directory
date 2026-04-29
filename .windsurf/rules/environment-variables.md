---
trigger: glob
globs: .env*,.dev.vars,src/env.d.ts
---

<environment_variables_guidelines>

# Environment Variables Guidelines

When working with environment variables:

## Client-Side Variables

### VITE\_ Prefix

- All client-side environment variables must use `VITE_` prefix
- Only variables with `VITE_` prefix are exposed to client-side code
- Server-side variables (without prefix) are not available in browser

### Current Project Variables

- `VITE_FORM_SUBMISSION_URL` - Cloudflare Worker endpoint for contact form
- `VITE_CALENDLY_URL` - Calendly scheduling link

## Type Definitions

### src/env.d.ts

- Define all environment variable types in `src/env.d.ts`
- Use `ImportMetaEnv` interface for type safety
- Example:

```typescript
interface ImportMetaEnv {
  readonly VITE_FORM_SUBMISSION_URL: string;
  readonly VITE_CALENDLY_URL: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

## File Management

### .env.example

- Maintain `.env.example` with all required variables
- Include placeholder values (not real secrets)
- Document what each variable does
- Commit `.env.example` to git

### .env and .dev.vars

- Never commit `.env` or `.dev.vars` files
- Add both to `.gitignore`
- Use `.dev.vars` for local development
- Use environment-specific files for different stages

### .gitignore

Ensure these patterns are in `.gitignore`:

```
.env
.env.local
.env.*.local
.dev.vars
```

## Usage Patterns

### Accessing Variables

```typescript
// In client-side code
const formUrl = import.meta.env.VITE_FORM_SUBMISSION_URL;

// In Cloudflare Worker
const formUrl = env.FORM_SUBMISSION_URL;
```

### Validation

- Validate environment variables at startup
- Provide clear error messages for missing variables
- Use Zod schemas for runtime validation

## Security Best Practices

### Never Commit Secrets

- Never commit actual values to git
- Never include secrets in code comments
- Never log environment variable values
- Never expose secrets in error messages

### Least Privilege

- Only expose variables that are actually needed
- Use different values for different environments
- Rotate secrets regularly
- Use secrets managers for production (consider migrating from env vars)

## Cloudflare Workers

### wrangler.toml

- Define environment variables in `wrangler.toml` for Workers
- Use `[vars]` section for non-secret variables
- Use secrets command for sensitive data
- Example:

```toml
[vars]
FORM_SUBMISSION_URL = "https://example.com/api/submit"
```

### Secrets Command

```bash
npx wrangler secret put FORM_SUBMISSION_URL
```

## Common Patterns

### Default Values

```typescript
const apiUrl = import.meta.env.VITE_API_URL || "https://api.example.com";
```

### Required Variables

```typescript
const requiredEnv = (key: string): string => {
  const value = import.meta.env[key];
  if (!value) throw new Error(`Missing required env var: ${key}`);
  return value;
};

const formUrl = requiredEnv("VITE_FORM_SUBMISSION_URL");
```

## Testing

### Mock Environment Variables

- Mock environment variables in tests
- Use vi.stubEnv() for Vitest
- Test with missing variables to ensure proper error handling

## Documentation

### Adding New Variables

When adding a new environment variable:

1. Add to `.env.example` with placeholder
2. Add type definition to `src/env.d.ts`
3. Document the variable's purpose
4. Update this file if needed
5. Add to Cloudflare Workers configuration if applicable

</environment_variables_guidelines>
