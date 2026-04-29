---
trigger: glob
globs: **/*.test.ts,**/*.spec.ts,**/src/test/**/*.ts
---

<testing_guidelines>

# Testing Guidelines

When writing or modifying tests for this project:

## Vitest 4.0 Compatibility

This project uses Vitest 4.0 (released 2025). Leverage new features when appropriate.

## Vitest Unit Tests

### Test Organization

- Organize tests close to source files: `Component.test.ts` next to `Component.tsx`
- Use `describe` blocks to group related tests
- Use `it` or `test` for individual test cases
- Follow AAA pattern: Arrange, Act, Assert

### Test Naming

- Use descriptive names that explain what is being tested
- Format: `should [expected behavior] when [condition]`
- Example: `should return user data when user exists`

### Mocking External Dependencies

- Mock external API calls with vi.fn()
- Mock third-party services (Stripe, Calendly, etc.)
- Use vi.mock() for module-level mocking
- Clean up mocks in afterEach hooks

### Test Structure

```typescript
describe("ComponentName", () => {
  beforeEach(() => {
    // Setup before each test
  });

  it("should do something when condition", () => {
    // Arrange
    const input = "test";

    // Act
    const result = functionUnderTest(input);

    // Assert
    expect(result).toBe("expected");
  });

  afterEach(() => {
    // Cleanup after each test
  });
});
```

### Vitest 4.0 New Features

#### Browser Mode (Stable)

- Use Browser Mode for component testing in real browser
- Install `@vitest/browser-play` for Playwright integration
- Better for testing components with browser-specific behavior
- Replaces jsdom for more accurate testing

#### Visual Regression Testing

- Use Vitest's built-in visual regression testing
- Compare screenshots across test runs
- Detect UI changes automatically
- Configure with `expect.toMatchSnapshot()`

#### Playwright Traces Support

- Generate detailed trace files for failed tests
- Analyze traces in Playwright Trace Viewer
- Debug complex test failures
- Traces available in HTML reporter

#### Type-Aware Hooks

- Use type-safe test hooks with TypeScript
- Better type inference in test helpers
- Improved autocomplete in test files

#### New Assertions

- `expect.assert()` - Assert number of assertions
- `expect.schemaMatching()` - Validate against schemas
- Enhanced custom matchers

### Coverage Requirements

- Aim for 80%+ code coverage on critical paths
- Focus on business logic and user-facing features
- Don't test implementation details
- Test error paths and edge cases

## Playwright E2E Tests

### Test Philosophy

- Test user-visible behavior, not implementation details
- Avoid testing third-party dependencies you don't control
- Make tests isolated - each test should run independently
- Use beforeEach hooks for setup (login, navigation)

### Locators

- Use user-facing locators: `getByRole()`, `getByText()`, `getByLabel()`
- Avoid CSS selectors and class names
- Generate locators with Playwright's codegen when unsure

### Page Object Pattern

- Create page objects for complex pages
- Encapsulate page-specific selectors and actions
- Reuse page objects across tests

### Test Isolation

- Each test should have its own browser context
- Clear local storage, session storage, cookies in beforeEach
- Use test isolation to prevent cascading failures

### Network Mocking

- Mock external API calls with `page.route()`
- Control third-party responses for reliability
- Don't test links to external sites

### Example Pattern

```typescript
test.beforeEach(async ({ page }) => {
  await page.goto("/");
  await page.getByRole("button", { name: "Contact" }).click();
});

test("should submit form successfully", async ({ page }) => {
  await page.getByLabel("Name").fill("Test User");
  await page.getByLabel("Email").fill("test@example.com");
  await page.getByRole("button", { name: "Submit" }).click();

  await expect(page.getByText("Message sent")).toBeVisible();
});
```

## Common Patterns

### Async Testing

- Use async/await for all async operations
- Await all Playwright actions
- Use proper assertions with expect()

### Test Data

- Use test fixtures for reusable test data
- Keep test data minimal and focused
- Don't use production data in tests

### Debugging

- Use `test.only` to run a single test
- Use Playwright's UI mode for debugging
- Check browser console for errors
- Use Vitest HTML reporter for trace files (v4.0)

## Performance

- Keep tests fast - avoid unnecessary waits
- Use parallel execution where possible
- Shard long-running test suites in CI
- Use test isolation to prevent flakiness
- Vitest 4.0 focuses on performance improvements

## Security

- Never commit real API keys or secrets
- Use environment variables for test credentials
- Mock sensitive operations in tests
- Don't log sensitive data in test output

</testing_guidelines>
