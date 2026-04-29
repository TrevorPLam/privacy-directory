---
name: setup-linting
description: Sets up ESLint and Prettier for Astro, React, and TypeScript with proper configuration
---

# Setup Linting

This skill guides you through setting up ESLint and Prettier for the Astro marketing website.

## Prerequisites

- Node.js 18+ installed
- Project dependencies installed (`npm install`)

## Installation

### 1. Install ESLint Packages

```bash
npm install --save-dev eslint eslint-plugin-astro eslint-plugin-jsx-a11y @eslint/js globals typescript-eslint prettier prettier-plugin-astro eslint-config-prettier eslint-plugin-prettier
```

### 2. Install TypeScript ESLint Parser

```bash
npm install --save-dev @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

## Configuration Files

### 1. Create .eslintrc.json

Create `.eslintrc.json` in the project root:

```json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:astro/recommended",
    "plugin:jsx-a11y/recommended",
    "plugin:prettier/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "env": {
    "browser": true,
    "node": true,
    "es2021": true
  },
  "overrides": [
    {
      "files": ["*.astro"],
      "parser": "astro-eslint-parser",
      "parserOptions": {
        "parser": "@typescript-eslint/parser",
        "extraFileExtensions": [".astro"]
      }
    }
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "@typescript-eslint/no-explicit-any": "warn",
    "no-console": "warn"
  }
}
```

### 2. Create .prettierrc

Create `.prettierrc` in the project root:

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "plugins": ["prettier-plugin-astro"]
}
```

### 3. Create .prettierignore

Create `.prettierignore` in the project root:

```
node_modules
dist
.env
.env.*
public
coverage
.git
```

### 4. Create .eslintignore

Create `.eslintignore` in the project root:

```
node_modules
dist
.env
.env.*
public
coverage
.git
*.config.js
*.config.mjs
```

## Add Scripts to package.json

Add these scripts to the `scripts` section in `package.json`:

```json
{
  "scripts": {
    "lint": "eslint . --ext .ts,.tsx,.astro",
    "lint:fix": "eslint . --ext .ts,.tsx,.astro --fix",
    "format": "prettier --write \"**/*.{ts,tsx,astro,md,json}\"",
    "format:check": "prettier --check \"**/*.{ts,tsx,astro,md,json}\""
  }
}
```

## Verification

### 1. Run Linting

```bash
npm run lint
```

This should check all TypeScript, TSX, and Astro files for linting errors.

### 2. Run Formatting

```bash
npm run format
```

This should format all files according to Prettier rules.

### 3. Check Formatting

```bash
npm run format:check
```

This should verify that all files are properly formatted without making changes.

## VS Code Integration

### 1. Install VS Code Extensions

Install these extensions in VS Code:

- ESLint
- Prettier - Code formatter

### 2. Configure VS Code Settings

Add to `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[astro]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "eslint.validate": ["javascript", "javascriptreact", "typescript", "typescriptreact", "astro"]
}
```

## Common Issues

### TypeScript Errors in Astro Files

If you see TypeScript errors in `.astro` files:

- Ensure `@typescript-eslint/parser` is installed
- Check that the parser override in `.eslintrc.json` is correct
- Verify `tsconfig.json` includes Astro files

### Prettier Conflicts with ESLint

If Prettier and ESLint conflict:

- Ensure `eslint-config-prettier` is last in the extends array
- This disables ESLint rules that conflict with Prettier

### Astro Plugin Not Working

If Astro-specific linting doesn't work:

- Verify `eslint-plugin-astro` is installed
- Check that the parser override for `.astro` files is correct
- Ensure `astro-eslint-parser` is installed

## Best Practices

### Lint Before Commit

Add a pre-commit hook to run linting (optional but recommended):

```bash
npm install --save-dev husky lint-staged
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

Add to `package.json`:

```json
{
  "lint-staged": {
    "*.{ts,tsx,astro}": ["eslint --fix", "prettier --write"]
  }
}
```

### CI/CD Integration

Add linting to GitHub Actions workflow in `.github/workflows/ci.yml`:

```yaml
- name: Lint
  run: npm run lint

- name: Format Check
  run: npm run format:check
```

## Troubleshooting

### Module Not Found Errors

If you see "module not found" errors:

- Run `npm install` to ensure all dependencies are installed
- Check that package versions are compatible
- Delete `node_modules` and `package-lock.json`, then run `npm install`

### Linting Takes Too Long

If linting is slow:

- Exclude large directories in `.eslintignore`
- Use `--cache` flag with ESLint
- Consider using `eslint-plugin-import` for better performance

## Next Steps

After setup:

1. Run `npm run lint:fix` to auto-fix existing issues
2. Manually review and fix remaining issues
3. Commit the configuration files
4. Update documentation if needed
