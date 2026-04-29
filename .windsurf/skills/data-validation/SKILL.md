---
name: data-validation
description: Comprehensive data validation patterns for privacy directory entities with neverthrow error handling
---

# Data Validation Skill

## Overview
This skill provides comprehensive data validation patterns for all privacy directory entities, ensuring data integrity, source verification, and compliance with privacy standards. Uses neverthrow Result pattern for consistent error handling.

## When to Use
- Validating company data and practices
- Ensuring data broker information accuracy
- Verifying privacy incident details
- Validating software and device information
- Checking legal compliance data
- Implementing source verification workflows

## Core Validation Patterns

### Base Validation Interface
```typescript
// src/shared/validation.ts
import { Result, Err, Ok } from 'neverthrow';

export interface ValidationResult {
  isValid: boolean;
  errors: ValidationError[];
  warnings: ValidationWarning[];
}

export interface ValidationError {
  field: string;
  code: string;
  message: string;
  severity: 'error' | 'warning';
}

export interface ValidationWarning {
  field: string;
  code: string;
  message: string;
}

export abstract class Validator<T> {
  abstract validate(data: unknown): Result<T, ValidationError[]>;
  
  protected isValidUrl(url: string): boolean {
    try {
      new URL(url);
      return true;
    } catch {
      return false;
    }
  }

  protected isNotEmpty(value: string): boolean {
    return value != null && value.trim().length > 0;
  }

  protected isValidEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
}
```

### Company Data Validation
```typescript
// src/companies/validation/company-validator.ts
import { CompanyDto, PracticeDto, DataSensitivityTier } from '../../../shared/dto';
import { Validator } from '../../../shared/validation';

export class CompanyValidator extends Validator<CompanyDto> {
  validate(data: unknown): Result<CompanyDto, ValidationError[]> {
    const errors: ValidationError[] = [];
    const warnings: ValidationWarning[] = [];

    if (!data || typeof data !== 'object') {
      return Err([{
        field: 'root',
        code: 'INVALID_OBJECT',
        message: 'Company data must be an object',
        severity: 'error'
      }]);
    }

    const company = data as any;

    // Required field validation
    if (!this.isNotEmpty(company.name)) {
      errors.push({
        field: 'name',
        code: 'REQUIRED_FIELD',
        message: 'Company name is required',
        severity: 'error'
      });
    }

    if (!this.isValidUrl(company.url)) {
      errors.push({
        field: 'url',
        code: 'INVALID_URL',
        message: 'Company URL must be a valid URL',
        severity: 'error'
      });
    }

    if (!Array.isArray(company.practices) || company.practices.length === 0) {
      errors.push({
        field: 'practices',
        code: 'REQUIRED_PRACTICES',
        message: 'Company must have at least one practice',
        severity: 'error'
      });
    }

    // Validate practices
    if (company.practices) {
      const practiceValidator = new PracticeValidator();
      company.practices.forEach((practice: any, index: number) => {
        const result = practiceValidator.validate(practice);
        if (result.isErr()) {
          result.error.forEach(error => {
            errors.push({
              ...error,
              field: `practices[${index}].${error.field}`
            });
          });
        }
      });
    }

    // Validate data sensitivity tier
    if (!Object.values(DataSensitivityTier).includes(company.dataSensitivityTier)) {
      errors.push({
        field: 'dataSensitivityTier',
        code: 'INVALID_TIER',
        message: 'Invalid data sensitivity tier',
        severity: 'error'
      });
    }

    // Warnings for optional fields
    if (!this.isNotEmpty(company.description)) {
      warnings.push({
        field: 'description',
        code: 'MISSING_DESCRIPTION',
        message: 'Company description is recommended',
        severity: 'warning'
      });
    }

    if (errors.length > 0) {
      return Err(errors);
    }

    return Ok({
      id: company.id || this.generateId(),
      name: company.name.trim(),
      url: company.url,
      description: company.description?.trim() || '',
      practices: company.practices,
      dataSensitivityTier: company.dataSensitivityTier
    });
  }

  private generateId(): string {
    return `company_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

### Practice Validation
```typescript
// src/companies/validation/practice-validator.ts
export class PracticeValidator extends Validator<PracticeDto> {
  validate(data: unknown): Result<PracticeDto, ValidationError[]> {
    const errors: ValidationError[] = [];

    if (!data || typeof data !== 'object') {
      return Err([{
        field: 'root',
        code: 'INVALID_OBJECT',
        message: 'Practice data must be an object',
        severity: 'error'
      }]);
    }

    const practice = data as any;

    // Category validation
    const validCategories = [
      'DATA_COLLECTION',
      'DATA_SHARING',
      'DATA_RETENTION',
      'DATA_PROCESSING',
      'THIRD_PARTY_ACCESS',
      'USER_CONTROL',
      'TRANSPARENCY',
      'SECURITY'
    ];

    if (!validCategories.includes(practice.category)) {
      errors.push({
        field: 'category',
        code: 'INVALID_CATEGORY',
        message: `Invalid practice category. Must be one of: ${validCategories.join(', ')}`,
        severity: 'error'
      });
    }

    // Details validation
    if (!this.isNotEmpty(practice.details)) {
      errors.push({
        field: 'details',
        code: 'REQUIRED_FIELD',
        message: 'Practice details are required',
        severity: 'error'
      });
    }

    if (practice.details && practice.details.length > 1000) {
      errors.push({
        field: 'details',
        code: 'TOO_LONG',
        message: 'Practice details must be less than 1000 characters',
        severity: 'error'
      });
    }

    if (errors.length > 0) {
      return Err(errors);
    }

    return Ok({
      category: practice.category,
      details: practice.details.trim()
    });
  }
}
```

### Data Broker Validation
```typescript
// src/brokers/validation/data-broker-validator.ts
export class DataBrokerValidator extends Validator<DataBrokerDto> {
  validate(data: unknown): Result<DataBrokerDto, ValidationError[]> {
    const errors: ValidationError[] = [];
    const warnings: ValidationWarning[] = [];

    if (!data || typeof data !== 'object') {
      return Err([{
        field: 'root',
        code: 'INVALID_OBJECT',
        message: 'Data broker data must be an object',
        severity: 'error'
      }]);
    }

    const broker = data as any;

    // Required fields
    if (!this.isNotEmpty(broker.name)) {
      errors.push({
        field: 'name',
        code: 'REQUIRED_FIELD',
        message: 'Data broker name is required',
        severity: 'error'
      });
    }

    if (!this.isValidUrl(broker.url)) {
      errors.push({
        field: 'url',
        code: 'INVALID_URL',
        message: 'Data broker URL must be a valid URL',
        severity: 'error'
      });
    }

    // Opt-out process validation
    if (broker.optOutProcess) {
      const optOutValidator = new OptOutProcessValidator();
      const result = optOutValidator.validate(broker.optOutProcess);
      if (result.isErr()) {
        result.error.forEach(error => {
          errors.push({
            ...error,
            field: `optOutProcess.${error.field}`
          });
        });
      }
    } else {
      warnings.push({
        field: 'optOutProcess',
        code: 'MISSING_OPT_OUT',
        message: 'Opt-out process information is recommended',
        severity: 'warning'
      });
    }

    // Compliance status validation
    if (broker.complianceStatus) {
      const complianceValidator = new ComplianceStatusValidator();
      const result = complianceValidator.validate(broker.complianceStatus);
      if (result.isErr()) {
        result.error.forEach(error => {
          errors.push({
            ...error,
            field: `complianceStatus.${error.field}`
          });
        });
      }
    }

    if (errors.length > 0) {
      return Err(errors);
    }

    return Ok({
      id: broker.id || this.generateId(),
      name: broker.name.trim(),
      url: broker.url,
      optOutProcess: broker.optOutProcess,
      complianceStatus: broker.complianceStatus
    });
  }

  private generateId(): string {
    return `broker_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

### Opt-Out Process Validation
```typescript
// src/brokers/validation/opt-out-process-validator.ts
export class OptOutProcessValidator extends Validator<OptOutProcessDto> {
  validate(data: unknown): Result<OptOutProcessDto, ValidationError[]> {
    const errors: ValidationError[] = [];

    if (!data || typeof data !== 'object') {
      return Err([{
        field: 'root',
        code: 'INVALID_OBJECT',
        message: 'Opt-out process data must be an object',
        severity: 'error'
      }]);
    }

    const optOut = data as any;

    // Process type validation
    const validTypes = ['WEB_FORM', 'EMAIL', 'API', 'PORTAL', 'PHONE', 'MAIL'];
    if (!validTypes.includes(optOut.type)) {
      errors.push({
        field: 'type',
        code: 'INVALID_TYPE',
        message: `Invalid opt-out type. Must be one of: ${validTypes.join(', ')}`,
        severity: 'error'
      });
    }

    // Instructions validation
    if (!this.isNotEmpty(optOut.instructions)) {
      errors.push({
        field: 'instructions',
        code: 'REQUIRED_FIELD',
        message: 'Opt-out instructions are required',
        severity: 'error'
      });
    }

    // Type-specific validation
    if (optOut.type === 'EMAIL' && !this.isValidEmail(optOut.email)) {
      errors.push({
        field: 'email',
        code: 'INVALID_EMAIL',
        message: 'Valid email address is required for email opt-out',
        severity: 'error'
      });
    }

    if ((optOut.type === 'WEB_FORM' || optOut.type === 'PORTAL') && !this.isValidUrl(optOut.url)) {
      errors.push({
        field: 'url',
        code: 'INVALID_URL',
        message: 'Valid URL is required for web form or portal opt-out',
        severity: 'error'
      });
    }

    // Estimated time validation
    if (optOut.estimatedTime) {
      const validTimes = ['IMMEDIATE', 'MINUTES', 'HOURS', 'DAYS', 'WEEKS'];
      if (!validTimes.includes(optOut.estimatedTime)) {
        errors.push({
          field: 'estimatedTime',
          code: 'INVALID_TIME',
          message: `Invalid estimated time. Must be one of: ${validTimes.join(', ')}`,
          severity: 'error'
        });
      }
    }

    if (errors.length > 0) {
      return Err(errors);
    }

    return Ok({
      type: optOut.type,
      instructions: optOut.instructions.trim(),
      url: optOut.url?.trim(),
      email: optOut.email?.trim(),
      estimatedTime: optOut.estimatedTime || 'DAYS'
    });
  }
}
```

### Privacy Incident Validation
```typescript
// src/incident/validation/privacy-incident-validator.ts
export class PrivacyIncidentValidator extends Validator<PrivacyIncidentDto> {
  validate(data: unknown): Result<PrivacyIncidentDto, ValidationError[]> {
    const errors: ValidationError[] = [];

    if (!data || typeof data !== 'object') {
      return Err([{
        field: 'root',
        code: 'INVALID_OBJECT',
        message: 'Privacy incident data must be an object',
        severity: 'error'
      }]);
    }

    const incident = data as any;

    // Required fields
    if (!this.isNotEmpty(incident.title)) {
      errors.push({
        field: 'title',
        code: 'REQUIRED_FIELD',
        message: 'Incident title is required',
        severity: 'error'
      });
    }

    if (!this.isNotEmpty(incident.description)) {
      errors.push({
        field: 'description',
        code: 'REQUIRED_FIELD',
        message: 'Incident description is required',
        severity: 'error'
      });
    }

    // Incident type validation
    const validTypes = ['DATA_BREACH', 'LAWSUIT', 'REGULATORY_ACTION', 'INVESTIGATION'];
    if (!validTypes.includes(incident.incidentType)) {
      errors.push({
        field: 'incidentType',
        code: 'INVALID_TYPE',
        message: `Invalid incident type. Must be one of: ${validTypes.join(', ')}`,
        severity: 'error'
      });
    }

    // Date validation
    if (incident.date && !this.isValidDate(incident.date)) {
      errors.push({
        field: 'date',
        code: 'INVALID_DATE',
        message: 'Incident date must be a valid date',
        severity: 'error'
      });
    }

    // Source URLs validation
    if (!Array.isArray(incident.sourceUrls) || incident.sourceUrls.length === 0) {
      errors.push({
        field: 'sourceUrls',
        code: 'REQUIRED_SOURCES',
        message: 'At least one source URL is required',
        severity: 'error'
      });
    } else {
      incident.sourceUrls.forEach((url: string, index: number) => {
        if (!this.isValidUrl(url)) {
          errors.push({
            field: `sourceUrls[${index}]`,
            code: 'INVALID_URL',
            message: 'Source URL must be valid',
            severity: 'error'
          });
        }
      });
    }

    // Company reference validation (optional)
    if (incident.companyReference) {
      const companyRefValidator = new CompanyReferenceValidator();
      const result = companyRefValidator.validate(incident.companyReference);
      if (result.isErr()) {
        result.error.forEach(error => {
          errors.push({
            ...error,
            field: `companyReference.${error.field}`
          });
        });
      }
    }

    if (errors.length > 0) {
      return Err(errors);
    }

    return Ok({
      id: incident.id || this.generateId(),
      title: incident.title.trim(),
      description: incident.description.trim(),
      incidentType: incident.incidentType,
      date: incident.date || new Date().toISOString().split('T')[0],
      sourceUrls: incident.sourceUrls,
      companyReference: incident.companyReference,
      severity: incident.severity
    });
  }

  private isValidDate(dateString: string): boolean {
    const date = new Date(dateString);
    return !isNaN(date.getTime());
  }

  private generateId(): string {
    return `incident_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

## Source Verification Validation

### Source Validator
```typescript
// src/shared/validation/source-validator.ts
export class SourceValidator extends Validator<SourceData> {
  validate(data: unknown): Result<SourceData, ValidationError[]> {
    const errors: ValidationError[] = [];

    if (!data || typeof data !== 'object') {
      return Err([{
        field: 'root',
        code: 'INVALID_OBJECT',
        message: 'Source data must be an object',
        severity: 'error'
      }]);
    }

    const source = data as any;

    // URL validation
    if (!this.isValidUrl(source.url)) {
      errors.push({
        field: 'url',
        code: 'INVALID_URL',
        message: 'Source URL must be valid',
        severity: 'error'
      });
    }

    // Source type validation
    const validTypes = [
      'PRIVACY_POLICY',
      'TERMS_OF_SERVICE',
      'REGULATORY_FILING',
      'NEWS_ARTICLE',
      'GOVERNMENT_DATABASE',
      'COURT_DOCUMENT',
      'COMPANY_PRESS_RELEASE'
    ];

    if (!validTypes.includes(source.sourceType)) {
      errors.push({
        field: 'sourceType',
        code: 'INVALID_TYPE',
        message: `Invalid source type. Must be one of: ${validTypes.join(', ')}`,
        severity: 'error'
      });
    }

    // Reliability validation
    if (source.reliability && source.reliability < 1 || source.reliability > 5) {
      errors.push({
        field: 'reliability',
        code: 'INVALID_RELIABILITY',
        message: 'Reliability must be between 1 and 5',
        severity: 'error'
      });
    }

    // Last accessed validation
    if (source.lastAccessed && !this.isValidDate(source.lastAccessed)) {
      errors.push({
        field: 'lastAccessed',
        code: 'INVALID_DATE',
        message: 'Last accessed date must be valid',
        severity: 'error'
      });
    }

    if (errors.length > 0) {
      return Err(errors);
    }

    return Ok({
      url: source.url,
      sourceType: source.sourceType,
      title: source.title?.trim() || '',
      reliability: source.reliability || 3,
      lastAccessed: source.lastAccessed || new Date().toISOString().split('T')[0],
      notes: source.notes?.trim() || ''
    });
  }
}
```

## Batch Validation

### Entity Batch Validator
```typescript
// src/shared/validation/batch-validator.ts
export class BatchValidator<T> {
  constructor(private validator: Validator<T>) {}

  async validateBatch(items: unknown[]): Promise<{
    valid: T[];
    invalid: { item: unknown; errors: ValidationError[] }[];
    summary: BatchValidationSummary;
  }> {
    const valid: T[] = [];
    const invalid: { item: unknown; errors: ValidationError[] }[] = [];
    let totalErrors = 0;
    let totalWarnings = 0;

    for (const item of items) {
      const result = this.validator.validate(item);
      
      if (result.isOk()) {
        valid.push(result.value);
      } else {
        invalid.push({
          item,
          errors: result.error
        });
        totalErrors += result.error.filter(e => e.severity === 'error').length;
        totalWarnings += result.error.filter(e => e.severity === 'warning').length;
      }
    }

    return {
      valid,
      invalid,
      summary: {
        total: items.length,
        valid: valid.length,
        invalid: invalid.length,
        totalErrors,
        totalWarnings,
        successRate: (valid.length / items.length) * 100
      }
    };
  }
}

interface BatchValidationSummary {
  total: number;
  valid: number;
  invalid: number;
  totalErrors: number;
  totalWarnings: number;
  successRate: number;
}
```

## Integration with Domain Layer

### Validation in Domain Services
```typescript
// src/companies/application/company-service.ts
export class CompanyService {
  constructor(
    private companyValidator: CompanyValidator,
    private sourceValidator: SourceValidator,
    private companyRepository: CompanyRepository
  ) {}

  async createCompany(data: unknown, sources: unknown[]): Result<Company, ValidationError[]> {
    // Validate company data
    const companyResult = this.companyValidator.validate(data);
    if (companyResult.isErr()) {
      return companyResult;
    }

    // Validate sources
    const sourceValidator = new SourceValidator();
    const validatedSources: SourceData[] = [];
    
    for (const source of sources) {
      const sourceResult = sourceValidator.validate(source);
      if (sourceResult.isErr()) {
        return Err(sourceResult.error);
      }
      validatedSources.push(sourceResult.value);
    }

    // Create domain entity
    const companyDto = companyResult.value;
    const company = Company.create(
      companyDto.name,
      companyDto.practices.map(p => Practice.create(p.category, p.details)._unsafeUnwrap()),
      companyDto.dataSensitivityTier
    );

    if (company.isErr()) {
      return Err([new ValidationError('domain', 'CREATION_ERROR', company.error.message, 'error')]);
    }

    // Save to repository
    const savedCompany = await this.companyRepository.save(company.value);
    
    return Ok(savedCompany);
  }
}
```

## Testing Validation

### Validation Test Patterns
```typescript
// src/companies/validation/__tests__/company-validator.test.ts
describe('CompanyValidator', () => {
  let validator: CompanyValidator;

  beforeEach(() => {
    validator = new CompanyValidator();
  });

  describe('validate', () => {
    it('should validate valid company data', () => {
      const validCompany = {
        name: 'Test Company',
        url: 'https://example.com',
        description: 'A test company',
        practices: [{
          category: 'DATA_COLLECTION',
          details: 'Collects user email addresses'
        }],
        dataSensitivityTier: 'MODERATE'
      };

      const result = validator.validate(validCompany);

      expect(result.isOk()).toBe(true);
      expect(result.value.name).toBe('Test Company');
    });

    it('should reject company without required fields', () => {
      const invalidCompany = {
        name: '',
        url: 'invalid-url',
        practices: [],
        dataSensitivityTier: 'INVALID'
      };

      const result = validator.validate(invalidCompany);

      expect(result.isErr()).toBe(true);
      expect(result.error).toHaveLength(4); // name, url, practices, dataSensitivityTier
    });

    it('should reject company with invalid practices', () => {
      const companyWithInvalidPractices = {
        name: 'Test Company',
        url: 'https://example.com',
        practices: [{
          category: 'INVALID_CATEGORY',
          details: ''
        }],
        dataSensitivityTier: 'MODERATE'
      };

      const result = validator.validate(companyWithInvalidPractices);

      expect(result.isErr()).toBe(true);
      expect(result.error.some(e => e.field === 'practices[0].category')).toBe(true);
      expect(result.error.some(e => e.field === 'practices[0].details')).toBe(true);
    });
  });
});
```

## Usage Examples

### Single Entity Validation
```typescript
const validator = new CompanyValidator();
const companyData = {
  name: 'Privacy Corp',
  url: 'https://privacy-corp.com',
  practices: [{
    category: 'DATA_COLLECTION',
    details: 'Collects user analytics data'
  }],
  dataSensitivityTier: 'HIGH'
};

const result = validator.validate(companyData);

if (result.isOk()) {
  console.log('Valid company:', result.value);
} else {
  console.error('Validation errors:', result.error);
}
```

### Batch Validation
```typescript
const batchValidator = new BatchValidator(new CompanyValidator());
const companiesData = [/* array of company data */];

const { valid, invalid, summary } = await batchValidator.validateBatch(companiesData);

console.log(`Validated ${summary.total} companies`);
console.log(`Success rate: ${summary.successRate}%`);
console.log(`Valid: ${valid.length}, Invalid: ${invalid.length}`);
```

## Verification Checklist

### Validation Coverage
- [ ] All entity types have validators
- [ ] Required fields are properly validated
- [ ] Data type validation implemented
- [ ] Business rule validation included
- [ ] Source verification validation

### Error Handling
- [ ] Consistent error format
- [ ] Proper error categorization
- [ ] Detailed error messages
- [ ] Warning vs error distinction

### Integration
- [ ] Domain layer integration
- [ ] Repository validation
- [ ] API input validation
- [ ] Form validation

### Testing
- [ ] Unit tests for all validators
- [ ] Edge case coverage
- [ ] Batch validation testing
- [ ] Integration testing

This skill ensures comprehensive data validation across all privacy directory entities with consistent error handling and thorough source verification.
