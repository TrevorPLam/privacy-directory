---
trigger: glob
globs: **/*.test.ts,**/*.feature,**/vitest.config.ts,**/cucumber.config.ts
---

# Testing Standards for Privacy Directory

## Unit Testing with Vitest

### File Organization
- Unit tests: `tests/unit/**/*.test.ts`
- Integration tests: `tests/integration/**/*.test.ts`
- Test setup: `tests/setup/`

### Coverage Requirements
- Domain layer: ≥90% coverage
- Infrastructure layer: ≥70% coverage
- Web layer: ≥70% coverage

### Test Structure
```typescript
// Red-Green-Refactor cycle required
describe('Component/Domain', () => {
  it('should handle happy path', () => {
    // 1) Arrange - setup that fails (red)
    // 2) Act - make it pass (green)  
    // 3) Refactor - improve implementation
  })
})
```

### Verification Commands
- Type checking: `pnpm tsc --noEmit`
- Unit tests: `pnpm test`
- Coverage: `pnpm test -- coverage`
- Integration tests: `pnpm test -- integration`

## BDD Testing with Cucumber

### Feature File Organization
- Location: `features/**/*.feature`
- Structure: `features/<bounded-context-slug>/`
- Example: `features/company/company-management.feature`

### Gherkin Standards
```gherkin
Feature: Domain capability description
  Scenario: Specific behavior
    Given context setup
    When action occurs
    Then expected outcome
```

### BDD Execution
- Command: `pnpm test:bdd`
- Configuration: `cucumber.config.ts`
- Test data: Use realistic privacy domain examples

## Domain Testing Patterns

### Aggregate Root Testing
- Test factory methods (e.g., `Company.create()`)
- Validate invariants and business rules
- Test domain events publication
- Cover error cases with ValidationError

### Value Object Testing
- Test immutability
- Validate creation constraints
- Test equality/comparison logic
- Cover edge cases and boundary conditions

### Integration Testing
- Test layer isolation (domain doesn't import infrastructure)
- Validate dependency direction with `dependency-cruiser`
- Test event bus integration
- Verify repository patterns

## Test Data Management

### Fixtures and Mocks
- Location: `tests/fixtures/`
- Use realistic privacy domain data
- Mock external dependencies
- Keep test data independent

### Test Utilities
- Shared test helpers in `tests/setup/`
- Custom matchers for domain objects
- Factory functions for test data
- Cleanup utilities

## Quality Gates

### Pre-commit Checks
- All tests must pass
- Coverage thresholds met
- No TypeScript compilation errors
- Dependency direction validated

### Continuous Integration
- Run full test suite on PR
- Generate coverage reports
- Fail on coverage threshold drops
- Validate BDD scenario completeness
