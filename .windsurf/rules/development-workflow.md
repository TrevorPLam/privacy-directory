---
trigger: manual
---

# Development Workflow Rules

## Phase Execution Order

### Sequential Phase Dependencies
- **Phase 0** → **Phase 0.5** → **Phase 1**
- Phases must be executed completely before moving to next
- Tasks within a phase can be parallelized when dependencies allow

### Task Dependencies
- Each task lists explicit `depends_on` IDs (comma-separated)
- Use `blocked_by` for pending external decisions
- Verify all dependencies are complete before starting task

## Task Execution Standards

### Subtask Requirements
- Every parent task has detailed subtasks
- Each subtask has explicit `imports_from` declaration
- Verification command must exit with code 0
- Result type must use `neverthrow` Result<T, E>

### Verification Commands
```bash
# Type checking
pnpm tsc --noEmit

# Build verification  
pnpm build

# Unit tests
pnpm test

# BDD tests
pnpm test:bdd

# Dependency validation
pnpm exec depcruise --validate
```

### File Creation Standards
- Test files use `.test.ts` extension
- BDD files use `.feature` extension in `features/` directory
- Domain files follow bounded context structure
- All imports listed exhaustively in documentation

## Code Quality Gates

### Pre-commit Requirements
- TypeScript compilation succeeds
- All tests pass with coverage thresholds
- Dependency direction validated
- No placeholder text in implementation

### Definition of Done Checklist
- [ ] BDD scenarios pass
- [ ] Unit tests with ≥90% domain coverage
- [ ] Integration tests validate layer isolation
- [ ] Documentation updated (ubiquitous language, context map)
- [ ] Error handling uses neverthrow pattern
- [ ] Source URLs and citations included for all data

## Domain-Driven Development

### Bounded Context Implementation
- **Company Directory**: `src/companies/`
- **Data Broker Management**: `src/brokers/`
- **Technology Catalog**: `src/tech/`, `src/device/`
- **Incident Tracking**: `src/incident/`
- **Legal Guide**: `src/legal/`
- **Web Infrastructure**: `src/web/`

### Aggregate Development Pattern
1. Define aggregate root with factory method
2. Create value objects for attributes
3. Define domain events
4. Write unit tests (Red-Green-Refactor)
5. Create BDD scenarios
6. Verify with coverage and dependency checks

### Error Handling Standards
- Use `Result<T, E>` from neverthrow
- Define specific error types in `src/shared/errors.ts`
- ValidationError for input validation
- NotFoundError for missing entities
- IntegrationError for external service failures
- ComplianceError for regulatory violations

## Documentation Requirements

### Ubiquitous Language
- All domain terms defined in `docs/ubiquitous-language.md`
- Use consistent terminology across code
- Include definitions for all aggregates and value objects

### Context Map
- Document all bounded context relationships
- Specify integration patterns (ACL, Shared Kernel, etc.)
- Include communication mechanisms and directions

### Task Documentation
- Each task includes depth metric estimation
- List all imports explicitly
- Include verification commands
- Document out-of-scope items

## Next.js Integration Rules

### When to Use Next.js
- Interactive opt-out wizard
- Data broker visualizer
- Dynamic features requiring server-side rendering
- Complex user interactions

### Integration Patterns
- Next.js code lives in `src/nextjs/`
- Separate deployment from Astro static site
- Communication via REST APIs or shared DTOs
- Domain never imports Next.js infrastructure

## Privacy and Compliance Standards

### Content Requirements
- All content must be factual and verifiable
- No subjective "good" or "bad" labels
- Source URLs required for all data
- Emphasis on neutrality in presentations

### Data Handling
- Use Plausible Analytics (privacy-focused)
- No tracking cookies or invasive analytics
- Document all data practices in privacy policy
- Follow data minimization principles
