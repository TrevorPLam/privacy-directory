---
name: setup-testing
description: Guides setup of Vitest for unit testing and Playwright for E2E testing
---

# Testing Framework Setup Guide

## 2026 Testing Stack

### Recommended Stack

- **Vitest 4.0**: Unit testing with native browser support, redesigned public API
- **Playwright**: E2E testing with AI automation
- **Default**: Vitest for unit tests, Playwright for browser tests
- **Note**: Jest 30 shipped June 2025 but Vitest is preferred for new projects

### Vitest 4.0 Features

- Native browser support backed by Playwright and WebdriverIO
- Redesigned public API
- Inline workspace configuration
- Mature browser mode
- Faster execution with Vite integration

## Install Dependencies

```bash
npm install -D vitest @vitest/ui @playwright/test
```

## Vitest Configuration

Create `vitest.config.ts` in project root:

```typescript
import { defineConfig } from "vitest/config";
import astro from "astro/config/vitest";

export default defineConfig({
  plugins: [astro()],
  test: {
    globals: true,
    environment: "node",
  },
});
```

## Playwright Configuration

Create `playwright.config.ts` in project root:

```typescript
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "./src/__tests__/e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: "html",
  use: {
    baseURL: "http://localhost:4321",
    trace: "on-first-retry",
  },
  projects: [{ name: "chromium" }, { name: "firefox" }, { name: "webkit" }],
  webServer: {
    command: "npm run dev",
    url: "http://localhost:4321",
    reuseExistingServer: !process.env.CI,
  },
});
```

## Add Test Scripts

Update `package.json`:

```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui"
  }
}
```

## Create Test Directory Structure

```
src/
└── __tests__/
    ├── unit/
    │   └── example.test.ts
    └── e2e/
        └── example.spec.ts
```

## Unit Test Example

Create `src/__tests__/unit/example.test.ts`:

```typescript
import { describe, it, expect } from "vitest";

describe("Example test", () => {
  it("should pass", () => {
    expect(true).toBe(true);
  });
});
```

## E2E Test Example

Create `src/__tests__/e2e/example.spec.ts`:

```typescript
import { test, expect } from "@playwright/test";

test("homepage loads", async ({ page }) => {
  await page.goto("/");
  await expect(page).toHaveTitle(/Your Dedicated Marketer/);
});
```

## Run Tests

```bash
# Run unit tests
npm run test

# Run unit tests with UI
npm run test:ui

# Run E2E tests
npm run test:e2e

# Run E2E tests with UI
npm run test:e2e:ui
```

## Notes

- Vitest is used for unit testing (better Astro compatibility than Jest)
- Playwright is used for E2E testing (works with static sites)
- Tests should be written for critical functionality
- Focus on testing user-facing features first
