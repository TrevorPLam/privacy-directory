---
trigger: always_on
---

# Privacy Directory Project Architecture Rules

## Project Overview
This is a neutral, fact-based privacy directory website built with Astro 6. The site catalogs Technology (companies, software, devices), Privacy Incidents, Data Broker Information, Privacy Toolkit, and legal references without subjective judgments.

## Core Architecture Principles

### Domain-Driven Design (DDD)
- **Bounded Contexts**: Company Directory, Data Broker Management, Technology Catalog, Incident Tracking, Legal Guide, Web Infrastructure
- **Layer Structure**: `src/domain/`, `src/application/`, `src/infrastructure/`, `src/web/`, `src/shared/`
- **Dependency Direction**: Inward only - domain never imports infrastructure
- **Aggregate Roots**: Company, DataBroker, PrivacySoftware, PrivacyDevice, PrivacyIncident, PrivacyLaw

### Technology Stack
- **Primary Framework**: Astro 6 with Cloudflare adapter
- **Package Manager**: pnpm
- **Testing**: Vitest for unit tests, Cucumber for BDD
- **Type Safety**: TypeScript with strict mode
- **Error Handling**: neverthrow Result<T, E> pattern
- **Validation**: Custom ValidationError type

## File Organization Rules

### Domain Layer (`src/domain/`)
- Each bounded context has its own domain folder
- Aggregate roots use factory methods (e.g., `Company.create()`)
- Value objects are immutable
- Domain events are defined in separate files

### Application Layer (`src/application/`)
- Use cases and application services
- DTOs for data transfer
- No business logic - orchestrates domain objects

### Infrastructure Layer (`src/infrastructure/`)
- External integrations
- Repository implementations
- Event bus implementation (InMemoryEventBus)

### Web Layer (`src/web/`)
- Astro pages and components
- Static routing only (unless Next.js sub-app)
- No domain logic in web components

### Shared Layer (`src/shared/`)
- Result type re-exports from neverthrow
- Shared error types: ValidationError, NotFoundError, IntegrationError, ComplianceError
- Event bus interface

## Testing Requirements

### Unit Tests
- Location: `tests/unit/**/*.test.ts`
- Coverage: ≥90% for domain, ≥70% for infrastructure
- Use Vitest with TypeScript support

### Integration Tests
- Location: `tests/integration/**/*.test.ts`
- Test layer isolation and dependency direction

### BDD Tests
- Location: `features/**/*.feature`
- Use Gherkin syntax
- Test business scenarios and user workflows

## Code Quality Standards

### Import Patterns
- List imports exhaustively in task documentation
- No cross-context direct object references
- Use absolute imports with `@/` prefix where configured

### Error Handling
- All functions return Result<T, E> from neverthrow
- Define specific error types in `src/shared/errors.ts`
- Use early returns with error propagation

### Documentation
- All public APIs have JSDoc comments
- Domain terms defined in `docs/ubiquitous-language.md`
- Context map in `docs/context-map.md`

## Development Workflow

### Task Execution
- Follow phases sequentially (Phase 0 → Phase 0.5 → Phase 1)
- Subtasks within a phase can be parallelized
- Each subtask must have verification command

### Verification Standards
- All verification commands must exit with code 0
- Use `pnpm tsc --noEmit` for type checking
- Use `pnpm test` for unit tests
- Use `pnpm test:bdd` for BDD scenarios

## Next.js Integration (Optional)
- Next.js only for dynamic features (interactive opt-out wizard, data broker visualizer)
- Lives in `src/nextjs/` as separate deployable sub-app
- Communication via REST or shared DTOs
- Never import Next.js code into Astro domain

## Compliance Requirements
- All content must be factual and verifiable
- No subjective "good" or "bad" labels
- Source URLs required for all data
- Emphasis on neutrality in all presentations
