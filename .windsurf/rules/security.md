---
trigger: always_on
---

<security_guidelines>

# Security Guidelines

When writing or modifying code, follow these security best practices:

## OWASP Top 10:2025

Follow the OWASP Top 10:2025 security risks:

1. **A01: Broken Access Control** - Implement proper authorization checks
2. **A02: Security Misconfiguration** - Secure default configurations
3. **A03: Software Supply Chain Failures** - Validate dependencies and packages
4. **A04: Cryptographic Failures** - Use strong encryption and hashing
5. **A05: Injection** - Validate and sanitize all inputs
6. **A06: Insecure Design** - Design security from the start
7. **A07: Authentication Failures** - Implement strong authentication
8. **A08: Software or Data Integrity Failures** - Verify data and code integrity
9. **A09: Security Logging and Alerting Failures** - Implement comprehensive logging
10. **A10: Mishandling of Exceptional Conditions** - Handle errors securely

## Secrets Management

### Never Hardcode Secrets

- Never hardcode API keys, passwords, or tokens in code
- Never commit secrets to git (including .env files)
- Use environment variables for all sensitive configuration
- Never log sensitive data (API keys, tokens, user data)

### Environment Variables

- Use `VITE_` prefix for client-side environment variables
- Define all environment variables in `src/env.d.ts`
- Use `.env.example` to document required variables
- Add `.env` and `.dev.vars` to `.gitignore`

### Cloudflare Worker Security

- Secrets managed via Cloudflare Workers environment
- Never expose secrets in client-side code
- Use wrangler secrets command for production secrets
- Rotate secrets regularly

## API Security

### Input Validation

- Validate all user inputs on both client and server
- Use Zod schemas for runtime validation
- Sanitize data before processing
- Never trust client-side validation alone

### API Endpoints

- Never hardcode API endpoints in code
- Use environment variables for API URLs
- Validate API responses before processing
- Handle API errors gracefully without exposing internals

## Authentication & Authorization

### Form Security

- Contact form submits to Cloudflare Worker (not client-side)
- Never store credentials in localStorage
- Use secure cookies for authentication (if implemented)
- Implement CSRF protection for forms

### External Services

- Calendly URL configured via environment variable
- Form submission URL configured via environment variable
- Never expose third-party API keys in client code
- Use proxy endpoints when needed to hide keys

## Data Protection

### User Data

- Never log sensitive user information
- Mask personal data in logs (email, phone, etc.)
- Implement proper data retention policies
- Follow GDPR/CCPA compliance for user data

### Form Submissions

- Form data sent to Cloudflare Worker for processing
- Never store form data in client-side storage
- Validate form data before processing
- Implement rate limiting (via Cloudflare)

## Code Security

### Dependencies (A03: Software Supply Chain Failures)

- Keep dependencies updated regularly
- Run `npm audit` to check for vulnerabilities
- Review security advisories for dependencies
- Use npm ci for reproducible builds
- Verify package integrity (use npm provenance)
- Pin dependency versions in package-lock.json
- Review third-party packages before adding

### Static Analysis

- Run security scans before deployment
- Check for hardcoded secrets in code
- Validate third-party scripts and CDNs
- Review code for common vulnerabilities (XSS, injection)

## Deployment Security

### CI/CD

- Never commit secrets to CI/CD configuration
- Use GitHub Secrets for sensitive CI/CD data
- Require manual approval for production deployment
- Validate build artifacts before deployment

### Cloudflare Workers

- Use wrangler.toml for configuration
- Never commit wrangler secrets
- Use environment-specific configurations
- Enable logging for security monitoring

## Common Vulnerabilities

### XSS Prevention

- Sanitize user-generated content
- Use React's built-in XSS protection
- Validate and sanitize HTML inputs
- Use Content Security Policy headers

### Injection Prevention (A05: Injection)

- Use parameterized queries (if using databases)
- Validate all inputs before processing
- Escape user inputs in dynamic content
- Never concatenate user input into queries

### CSRF Prevention

- Use anti-CSRF tokens for forms
- Validate referrer headers
- Use SameSite cookie attributes
- Implement proper session management

### Security Logging (A09: Security Logging and Alerting Failures)

- Implement comprehensive logging for security events
- Log authentication attempts, authorization failures
- Set up alerts for suspicious activity
- Protect log files from unauthorized access
- Include sufficient context in logs for investigation

### Error Handling (A10: Mishandling of Exceptional Conditions)

- Never expose stack traces to users
- Use generic error messages for client-facing errors
- Log detailed errors server-side
- Implement proper error recovery mechanisms
- Test error conditions and edge cases

## Incident Response

If a security issue is discovered:

1. Immediately revoke exposed secrets
2. Rotate all affected credentials
3. Audit logs for unauthorized access
4. Patch vulnerabilities before deploying
5. Document the incident and response
6. Notify affected parties if data was compromised

</security_guidelines>
