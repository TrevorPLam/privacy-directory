---
trigger: manual
---

<git_workflow_guidelines>

# Git Workflow Guidelines

Follow these git workflow practices for this project:

## Branching Strategy

### Main Branch

- `main` is the production branch
- All features branch from `main`
- Direct commits to `main` are discouraged (use PRs)

### Feature Branches

- Create feature branches from `main`
- Use descriptive branch names: `feature/add-blog-post`, `fix/contact-form`
- Keep branches focused on a single feature or fix
- Delete branches after merging

### Branch Naming

- `feature/` - new features
- `fix/` - bug fixes
- `docs/` - documentation changes
- `refactor/` - code refactoring
- `test/` - test additions/changes
- `chore/` - maintenance tasks

## Commit Conventions

### Conventional Commits

Use Conventional Commits format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Common Types

- `feat` - new feature
- `fix` - bug fix
- `docs` - documentation changes
- `style` - formatting, missing semi colons, etc
- `refactor` - code refactoring
- `perf` - performance improvements
- `test` - adding or updating tests
- `chore` - maintenance
- `ci` - CI/CD changes

### Examples

```
feat(contact): add Calendly integration
fix(seo): update canonical URLs on blog posts
docs(readme): update deployment instructions
refactor(components): extract shared button styles
test(contact): add form submission tests
chore(deps): upgrade Astro to 5.18.1
```

### 50/72 Rule

- Keep subject line under 50 characters
- Wrap body at 72 characters
- Subject line should be imperative mood ("add" not "added")

### Commit Body

- Explain why, not what (diff shows what)
- Include context and alternatives considered
- Reference related issues: `Fixes #123`

## Pull Request Process

### PR Requirements

- All PRs must pass CI/CD checks
- Unit tests (Vitest) must pass
- E2E tests (Playwright) must pass
- Build must succeed
- Code review required before merge

### PR Description

Include in PR description:

- Description of changes
- Related issue number
- Screenshots for UI changes
- Testing performed
- Breaking changes (if any)

### PR Approval

- At least one approval required
- Self-approval discouraged
- Address all review comments before merge

## CI/CD Integration

### GitHub Actions Workflow

- Triggers on push to `main` and pull requests
- Runs build, test (Vitest), test-e2e (Playwright)
- Deployment requires manual approval via "production" environment
- Failed tests block deployment

### Deployment

- Production deployment requires manual approval
- Deploy to GitHub Pages
- Cloudflare Workers deployed separately via wrangler
- Never deploy failed builds

## Code Review Best Practices

### Review Focus

- Code correctness and logic
- Adherence to project conventions
- Security considerations
- Performance implications
- Test coverage

### Review Etiquette

- Be constructive and specific
- Ask questions, don't just dictate
- Explain reasoning for suggestions
- Acknowledge good work

## Common Git Commands

### Daily Workflow

```bash
# Create feature branch
git checkout -b feature/your-feature

# Make changes and commit
git add .
git commit -m "feat: add your feature"

# Push and create PR
git push origin feature/your-feature
```

### Syncing with Main

```bash
# Update main
git checkout main
git pull origin main

# Rebase feature branch
git checkout feature/your-feature
git rebase main
```

### Handling Conflicts

- Resolve conflicts carefully
- Test after resolving
- Continue rebase: `git rebase --continue`
- Abort if needed: `git rebase --abort`

## Git Ignore

### Files to Ignore

Ensure `.gitignore` includes:

```
node_modules/
dist/
.env
.env.local
.env.*.local
.dev.vars
*.log
.DS_Store
```

## Best Practices

### Commit Frequency

- Commit often, commit small
- Each commit should be logically complete
- Don't commit broken code
- Write meaningful commit messages

### History Cleanliness

- Avoid squashing unless necessary
- Keep history linear when possible
- Use rebase for cleaning up before merge
- Preserve important context in commits

### Collaboration

- Communicate in PRs, not commits
- Use draft PRs for work-in-progress
- Request review when ready
- Respond to review feedback promptly

</git_workflow_guidelines>
