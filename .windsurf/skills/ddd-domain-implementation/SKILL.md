---
name: ddd-domain-implementation
description: Complete domain-driven design implementation with testing and validation
---

# DDD Domain Implementation Skill

## Overview
This skill provides comprehensive guidance for implementing Domain-Driven Design patterns in the Privacy Directory project, including aggregate roots, value objects, domain events, and complete testing coverage.

## When to Use
- Creating new bounded contexts and domain models
- Implementing aggregate roots with business rules
- Setting up domain events and event handling
- Creating comprehensive test coverage for domain logic
- Validating DDD patterns and architecture compliance

## Implementation Steps

### 1. Aggregate Root Creation
Create aggregate roots following these patterns:

```typescript
// Factory method pattern
export class Company {
  private constructor(
    private readonly id: CompanyId,
    private readonly name: string,
    private readonly practices: Practice[],
    private readonly dataSensitivityTier: DataSensitivityTier
  ) {}

  static create(name: string, practices: Practice[], dataSensitivityTier: DataSensitivityTier): Result<Company, ValidationError> {
    // Business rule validation
    if (practices.length === 0) {
      return Err(new ValidationError('Company must have at least one practice'));
    }
    
    const company = new Company(
      CompanyId.generate(),
      name,
      practices,
      dataSensitivityTier
    );

    return Ok(company);
  }

  // Domain methods
  getPractices(): Practice[] {
    return [...this.practices]; // Return copy for immutability
  }

  getCategory(): Category {
    // Business logic for categorization
    return this.practices[0]?.category ?? Category.OTHER;
  }
}
```

### 2. Value Object Implementation
Create immutable value objects:

```typescript
export class Practice {
  private constructor(
    public readonly category: Category,
    public readonly details: string
  ) {}

  static create(category: Category, details: string): Result<Practice, ValidationError> {
    if (!details || details.trim().length === 0) {
      return Err(new ValidationError('Practice details are required'));
    }

    return Ok(new Practice(category, details.trim()));
  }

  equals(other: Practice): boolean {
    return this.category === other.category && this.details === other.details;
  }
}
```

### 3. Domain Events
Define and publish domain events:

```typescript
export interface CompanyCreated {
  type: 'CompanyCreated';
  payload: {
    companyId: string;
    name: string;
    practices: Practice[];
    dataSensitivityTier: DataSensitivityTier;
    timestamp: Date;
  };
}

// In aggregate root
static create(...): Result<Company, ValidationError> {
  // ... validation logic
  
  const company = new Company(/* ... */);
  
  // Publish domain event
  const event: CompanyCreated = {
    type: 'CompanyCreated',
    payload: {
      companyId: company.id.value,
      name,
      practices,
      dataSensitivityTier,
      timestamp: new Date()
    }
  };

  // Event publishing handled by infrastructure
  return Ok(company);
}
```

### 4. Unit Testing Patterns
Create comprehensive unit tests:

```typescript
describe('Company', () => {
  describe('create', () => {
    it('should create company with valid data', () => {
      // Arrange
      const name = 'Test Company';
      const practices = [Practice.create(Category.DATA_COLLECTION, 'Collects user data')._unsafeUnwrap()];
      const tier = DataSensitivityTier.HIGH;

      // Act
      const result = Company.create(name, practices, tier);

      // Assert
      expect(result.isOk()).toBe(true);
      const company = result._unsafeUnwrap();
      expect(company.name).toBe(name);
      expect(company.getPractices()).toHaveLength(1);
    });

    it('should reject company without practices', () => {
      // Arrange
      const name = 'Test Company';
      const practices: Practice[] = [];
      const tier = DataSensitivityTier.HIGH;

      // Act
      const result = Company.create(name, practices, tier);

      // Assert
      expect(result.isErr()).toBe(true);
      expect(result._unsafeUnwrapErr()).toBeInstanceOf(ValidationError);
    });
  });
});
```

### 5. BDD Scenario Creation
Create realistic BDD scenarios:

```gherkin
Feature: Company Management
  As a privacy researcher
  I want to add companies to the directory
  So that users can find privacy-related companies

  Scenario: Successfully create a company with valid practices
    Given a company name "Privacy Corp"
    And a data collection practice "Collects email addresses"
    And a data sensitivity tier "HIGH"
    When I create the company
    Then the company should be created successfully
    And the company should have one practice
    And the company category should be "DATA_COLLECTION"

  Scenario: Reject company without any practices
    Given a company name "Empty Corp"
    And no practices
    And a data sensitivity tier "MODERATE"
    When I attempt to create the company
    Then the creation should fail
    And the error should indicate "at least one practice required"
```

### 6. Architecture Validation
Ensure compliance with DDD principles:

```typescript
// Integration test for layer isolation
describe('Domain Layer Isolation', () => {
  it('should not import infrastructure', () => {
    const domainFiles = glob.sync('src/**/*.domain.ts');
    
    domainFiles.forEach(file => {
      const content = fs.readFileSync(file, 'utf-8');
      expect(content).not.toMatch(/from\s+['"]\.\.\/infrastructure/);
      expect(content).not.toMatch(/import.*infrastructure/);
    });
  });

  it('should only use Result type from shared', () => {
    const domainFiles = glob.sync('src/**/*.domain.ts');
    
    domainFiles.forEach(file => {
      const content = fs.readFileSync(file, 'utf-8');
      expect(content).toMatch(/from\s+['"]@\/shared\/result['"]/);
    });
  });
});
```

## Verification Checklist

### Domain Model Validation
- [ ] All aggregates use factory methods
- [ ] Value objects are immutable
- [ ] Business rules are enforced in aggregates
- [ ] Domain events are defined and published
- [ ] Error handling uses neverthrow Result pattern

### Testing Validation
- [ ] Unit tests cover all business rules
- [ ] Test coverage ≥90% for domain layer
- [ ] BDD scenarios cover user workflows
- [ ] Integration tests validate layer isolation
- [ ] All tests follow Red-Green-Refactor cycle

### Architecture Validation
- [ ] Dependency direction enforced (inward only)
- [ ] Domain layer has no infrastructure dependencies
- [ ] Shared types used consistently
- [ ] Event bus interface properly abstracted

## Common Patterns

### Error Handling
```typescript
// Always return Result<T, E>
public someMethod(): Result<SomeType, ValidationError> {
  // Validation
  if (invalid) {
    return Err(new ValidationError('Specific error message'));
  }
  
  // Business logic
  return Ok(result);
}
```

### Immutable Collections
```typescript
// Return copies, not references
getPractices(): Practice[] {
  return [...this.practices];
}
```

### Domain Event Publishing
```typescript
// Events published at aggregate boundaries
static create(...): Result<Company, ValidationError> {
  // ... validation and creation
  
  // Event handled by infrastructure layer
  return Ok(company);
}
```

## Supporting Files

This skill includes templates for:
- Aggregate root structure
- Value object patterns
- Domain event definitions
- Unit test templates
- BDD scenario templates
- Architecture validation tests

## Usage Tips

1. **Start with BDD scenarios** - Define user workflows first
2. **Implement aggregates** - Create factory methods with validation
3. **Add value objects** - Ensure immutability and proper equality
4. **Define domain events** - Capture state changes
5. **Write comprehensive tests** - Cover all business rules
6. **Validate architecture** - Ensure layer isolation and dependency direction

This skill ensures consistent, high-quality DDD implementation across all bounded contexts in the Privacy Directory project.
