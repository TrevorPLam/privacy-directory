# TODO.md — Final Version

> **Execution Order & Dependencies**
> - Phases must be executed sequentially; tasks within a phase can be parallelized where dependencies allow.
> - Each parent task lists explicit `depends_on` IDs (comma‑separated).
> - `blocked_by` used when design decision is pending; currently only placeholder for external decisions (e.g., "await legal review on ComplianceService scope").
>
> **Project Conventions**
> - **Test files** use `tests/unit/...` or `tests/integration/...` and `.test.ts` extension.
> - **BDD feature files** live in `features/<bounded-context-slug>/` (e.g., `features/company/`). Cucumber glob: `features/**/*.feature`.
> - **Imports** are listed exhaustively in `imports_from` per subtask; no cross‑context direct object references.
> - **Depth metric** includes approximate internal complexity estimate (e.g., "deep – ~200 lines of algorithm").
> - **Verification** is a concrete command that must exit 0; every subtask has one.
> - **Result type** uses `neverthrow` (`Result<T, E>`); error types in `src/shared/errors.ts`.

---

## Phase 0 – Project & Architectural Foundation

**Node version requirement**: Project requires Node.js ≥ 20 (LTS); include `.nvmrc` with version.

### `ARCH‑001` Setup Astro 6 project structure | Status: `backlog`  
`depends_on`: none

- [ ] **Parent** `ARCH‑001` – Initialize the Astro 6 project with proper domain‑driven structure.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Project Architecture (enabling subdomain) `Aggregate root:` N/A (project scaffolding) `Ubiquitous language:` Domain, Infrastructure, Application, Web, Shared |
| **BDD** | N/A – infrastructure setup |
| **TDD** | `Test file(s):` tests/integration/project_structure.test.ts `Red‑Green‑Refactor cycle:` 1) verify project directory does not exist (red), 2) create directory structure (green), 3) verify dependency direction with `dependency‑cruiser` (green) |
| **Deep module** | N/A – configuration task |
| **Definition of done** | Astro 6 project created, domain‑driven directory structure in place, `dependency‑cruiser` validates inward‑only dependencies, `pnpm build` succeeds |
| **Out of scope** | CI/CD, deployment, dev environment setup |
| **Rules** | Domain never imports infrastructure; configuration externalized; `neverthrow` added to `package.json` |
| **Advanced patterns** | Dependency Injection setup, Module pattern |
| **Anti‑patterns** | Circular dependencies, scattered config |

#### Subtasks for ARCH‑001

- [ ] **ARCH‑001‑1** – `package.json` – Create project dependencies (include `neverthrow`, `@cloudflare/workers-types`, `vitest`, `@cucumber/cucumber`, `dependency‑cruiser`, `react`, `react-dom`, `@astrojs/react`, `@types/react`, `@testing-library/react`, `@testing-library/jest-dom`, `jsdom`)  
  `imports_from`: none  
  `verification`: `pnpm install && pnpm build`
- [ ] **ARCH‑001‑2** – `astro.config.mjs` – Configure Astro 6 with Cloudflare adapter and `@astrojs/react` integration (note: local dev uses miniflare/KV emulator)  
  `imports_from`: none  
  `verification`: `pnpm build` produces Astro output  
  Security: Configure Content Security Policy using Astro 6's stable CSP API
(security.csp in astro.config.mjs). Default policy: default-src 'self',
script-src 'self', style-src 'self', frame‑ancestors 'none'.
No third‑party tracking scripts permitted.
Note: for v1, output: 'static' is recommended. Hybrid mode (output: 'hybrid')
is available for v2 when SSR pages with KV/D1 bindings are needed.
- [ ] **ARCH‑001‑3** – `src/` – Create domain‑driven directory structure (domain/, application/, infrastructure/, web/, shared/)  
  `imports_from`: none  
  `verification`: `test -d src/domain && test -d src/application && test -d src/infrastructure && test -d src/web && test -d src/shared && echo "All layers present"`
- [ ] **ARCH‑001‑4** – `tests/integration/project_structure.test.ts` – Create structure tests: validate layer isolation (domain doesn't import infra)  
  `imports_from`: `vitest`  
  `verification`: `pnpm test -- project_structure` passes
- [ ] **ARCH‑001‑5** – `src/shared/result.ts` – Re‑export `Result`, `Ok`, `Err` from `neverthrow`  
  `imports_from`: `neverthrow`  
  `verification`: `pnpm tsc --noEmit` compiles
Note: neverthrow is class‑based; Result objects lose their prototype chain when
passed through structuredClone, postMessage, or Worker boundaries (relevant for
Cloudflare Workers SSR). For the v1 static site this is not an issue. If v2 adds
SSR API routes, consider verdict‑ts (491 B gzipped, plain‑object Result) or ensure
results are unwrapped before serialization.
- [ ] **ARCH‑001‑6** – `src/shared/errors.ts` – Define shared error types: `ValidationError`, `NotFoundError`, `IntegrationError`, `ComplianceError`  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit` compiles
- [ ] **ARCH‑001‑7** – `dependency-cruiser.cjs` – Configure import direction enforcement  
  `imports_from`: none  
  `verification`: `pnpm exec depcruise --validate` exits 0
- [ ] **ARCH‑001‑8** – `wrangler.toml` – Configure Cloudflare Workers bindings (KV namespace, environment variables) and Astro adapter settings  
  `imports_from`: none  
  `verification`: `test -f wrangler.toml && grep "kv_namespaces" wrangler.toml`

---

### `ARCH‑002` Configure testing infrastructure | Status: `backlog`  
`depends_on`: `ARCH‑001`

- [ ] **Parent** `ARCH‑002` – Set up comprehensive testing framework for TDD and BDD.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Testing Infrastructure (enabling subdomain) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/setup/test_config.test.ts `Red‑Green‑Refactor cycle:` 1) shell assertion: `pnpm test` fails before config (red), 2) configure vitest & cucumber (green), 3) verify `pnpm test` runs empty suites successfully |
| **Deep module** | N/A |
| **Definition of done** | `pnpm test` executes unit tests, `pnpm test:bdd` parses and runs `.feature` files, coverage thresholds enforced (≥90% domain, ≥70% infrastructure), a placeholder feature file exists |
| **Out of scope** | Performance testing |
| **Rules** | Unit tests isolated; BDD human‑readable; coverage thresholds in vitest.config.ts |
| **Advanced patterns** | Test doubles, Page object |
| **Anti‑patterns** | Test dependencies, missing cleanups |

#### Subtasks for ARCH‑002

- [ ] **ARCH‑002‑1** – `vitest.config.ts` – Configure unit runner (glob `tests/**/*.test.ts`, coverage thresholds, environment: 'jsdom', setupFiles: ['./tests/setup/vitest.setup.ts'])  
  `imports_from`: `vitest/config`  
  `verification`: `pnpm test` runs and collects zero tests
- [ ] **ARCH‑002‑2** – `cucumber.config.ts` – Configure BDD framework with glob `features/**/*.feature`, require: ['tests/**/*.steps.ts']  
  `imports_from`: `@cucumber/cucumber`  
  `verification`: `pnpm test:bdd` runs and parses the placeholder feature file (see ARCH‑002‑3)
- [ ] **ARCH‑002‑3** – `tests/setup/` + `features/example.feature` + `tests/setup/vitest.setup.ts` – Create test setup, mocks, vitest setup with jest-dom matchers, and a minimal `Example` feature so BDD runner has something to parse  
  `imports_from`: none  
  `verification`: `pnpm test:bdd` parses `example.feature` successfully
- [ ] **ARCH‑002‑4** – `tests/setup/test_config.test.ts` – Verify test runner configuration  
  `imports_from`: `vitest`  
  `verification`: `pnpm test -- test_config` passes

---

### `ARCH‑007` Create Cucumber step definition scaffold | Status: `backlog`  
`depends_on`: `ARCH‑002`, `COMP‑001‑7`, `BROK‑001‑6`, `TECH‑001‑7`, `VIOL‑001‑7`, `LAW‑001‑7`

- [ ] **Parent** `ARCH‑007` – Scaffold step definition files for BDD scenarios.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Testing Infrastructure `Aggregate root:` N/A |
| **BDD** | All `.feature` files have corresponding step definitions |
| **TDD** | `Test file(s):` tests/integration/step_definitions.test.ts |
| **Deep module** | N/A – scaffolding |
| **Definition of done** | Step definition files exist for all bounded contexts, BDD runner finds all steps |
| **Out of scope** | Full step implementations |
| **Rules** | Steps map to empty functions initially |
| **Advanced patterns** | Gherkin step organization |
| **Anti‑patterns** | Missing step files |

#### Subtasks for ARCH‑007

- [ ] **ARCH‑007‑1** – For each bounded context (company, broker, tech, violation, law), create a minimal step file that defines the steps referenced in the `.feature` files  
  `imports_from`: none  
  `verification`: `pnpm test:bdd --dry-run` lists all steps as defined/pending

---

### `ARCH‑003` Define Context Map & Ubiquitous Language | Status: `backlog`  
`depends_on`: `ARCH‑001`

- [ ] **Parent** `ARCH‑003` – Document strategic design artifacts.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Strategic Design `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | N/A |
| **Deep module** | N/A |
| **Definition of done** | `docs/context-map.md` covers all 15 bounded context pairs with integration patterns; `docs/ubiquitous-language.md` collects all domain terms with definitions |
| **Out of scope** | Detailed API contracts |
| **Rules** | Context Map template: context pair, integration pattern (ACL/Shared Kernel/etc.), direction, communication mechanism |
| **Advanced patterns** | ACL for external registries |
| **Anti‑patterns** | Unspecified cross‑context coupling |

#### Subtasks for ARCH‑003

- [ ] **ARCH‑003‑1** – `docs/context-map.md` – Create Context Map: one section per context pair (Company Evaluation, Data Broker Management, Technology Evaluation, Violation Tracking, Legal Compliance, Web Adapter)  
  `imports_from`: none  
  `verification`: `grep -c "Context Pair:" docs/context-map.md` equals 15, each section names an integration pattern  
  **Context pairs to enumerate**: Company↔Legal, Company↔Violation, Company↔Broker, Company↔Technology, Company↔Web, Legal↔Violation, Legal↔Broker, Legal↔Technology, Legal↔Web, Violation↔Broker, Violation↔Technology, Violation↔Web, Broker↔Technology, Broker↔Web, Technology↔Web
- [ ] **ARCH‑003‑2** – `docs/ubiquitous-language.md` – Collect all terms from task tables, define them canonically  
  `imports_from`: none  
  `verification`: file contains at least one entry per domain aggregate and value object

---

### `ARCH‑004` Event Bus infrastructure | Status: `backlog`  
`depends_on`: `ARCH‑001`

- [ ] **Parent** `ARCH‑004` – Provide a shared infrastructure for publishing and subscribing to domain events.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Infrastructure (shared kernel) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/unit/shared/event_bus.test.ts `Red‑Green‑Refactor cycle:` 1) subscribe and publish single event (in‑memory), 2) verify handler receives correct payload |
| **Deep module** | `Public interface:` EventBus.publish(event), .subscribe(type, handler) `Hidden complexity:` In‑memory delivery, future Cloudflare Queues adapter `Depth metric:` shallow‑moderate |
| **Definition of done** | In‑memory `EventBus` implementation; interface defined in `src/shared/event-bus.ts`; subscription registration mechanism (composition root) noted in docs |
| **Out of scope** | Production Cloudflare Queues, persistence of events |
| **Rules** | Events are plain objects with `type` and `payload`; subscribers registered at infrastructure wire‑up (not inside domain) |
| **Advanced patterns** | Observer pattern |
| **Anti‑patterns** | Domain directly depending on event bus implementation |

#### Subtasks for ARCH‑004

- [ ] **ARCH‑004‑1** – `src/shared/event-bus.ts` – Define `EventBus` interface and `InMemoryEventBus` class  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **ARCH‑004‑2** – `tests/unit/shared/event_bus.test.ts` – Tests for publish/subscribe  
  `imports_from`: `EventBus`  
  `verification`: tests pass
- [ ] **ARCH‑004‑3** – `docs/event-subscriptions.md` – Document where subscriptions are wired (composition root)  
  `imports_from`: none  
  `verification`: `test -f docs/event-subscriptions.md`

---

### `ARCH‑005` Data Flow Diagram & Error Strategy | Status: `backlog`  
`depends_on`: `ARCH‑001`, `ARCH‑004`

- [ ] **Parent** `ARCH‑005` – Document data flow and shared error vocabulary.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | N/A (documentation) |
| **BDD** | N/A |
| **TDD** | N/A |
| **Deep module** | N/A |
| **Definition of done** | `docs/data-flow.md` illustrates path: repository → aggregate → DTO mapper → DTO → web; `src/shared/errors.ts` updated with all needed error subtypes |
| **Out of scope** | Full architecture decision records |
| **Rules** | Each layer only touches types it owns |
| **Advanced patterns** | Mermaid diagrams |
| **Anti‑patterns** | Missing error documentation |

#### Subtasks for ARCH‑005

- [ ] **ARCH‑005‑1** – `docs/data-flow.md` – Create data flow diagram (Mermaid) showing query and command flows  
  `imports_from`: none  
  `verification`: file exists and renders a diagram  
  **Scope**: Diagram must show: query flow (repository → aggregate → DTO → web), command flow (web → application service → repository → event), and event‑driven cross‑context flow.
- [ ] **ARCH‑005‑2** – `src/shared/errors.ts` (finalize) – Ensure all error types used across tasks are defined  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit` compiles with imports from `errors.ts`

---

## Phase 1 – Domain Models

All of these can be developed in parallel after Phase 0.

### `COMP‑001` Define Company domain model | Status: `backlog`  
`depends_on`: `ARCH‑001`

- [ ] **Parent** `COMP‑001` – Establish core domain entities for privacy‑focused companies.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Company Evaluation `Aggregate root:` Company `Ubiquitous language:` Company, PrivacyScore, Practice, Category, CompanyEvaluated (event) |
| **BDD** | `Feature:` Administrator evaluates company privacy `Scenario 1:` Happy path – given practices with weights, score calculated correctly `Scenario 2:` Company with no practices → evaluation fails with ValidationError `Scenario 3:` Company with a single practice in each category → correct weighted score |
| **TDD** | `Test file(s):` tests/unit/company/domain/company.test.ts `Red‑Green‑Refactor cycle:` 1) test_company_creates_from_factory, 2) test_privacy_score_calculates_weighted_average, 3) test_category_assignment, 4) test_company_rejects_invalid_practices (negative), 5) test_evaluate_fails_with_no_practices, 6) test_algorithm_uses_configured_weights |
| **Deep module** | `Public interface:` Company.create(name, practices), evaluatePrivacy(), getCategory() `Hidden complexity:` Weighted scoring (~200 lines), validation, category classification `Depth metric:` deep – 3 public methods, ~200 lines internal logic |
| **Definition of done** | All BDD scenarios pass, tests ≥90% coverage, `.feature` file created, domain events defined |
| **Out of scope** | Company financials, employee data |
| **Rules** | `PrivacyScore` is an immutable value object (0‑100). Categories are mutually exclusive. Scoring weights (Data Collection & Sharing: 0.40, Transparency & User Control: 0.35, Retention & Security: 0.15, Legal Compliance History: 0.10) are defined as constants in `PrivacyScoringAlgorithm` based on Privacy Fairness Index methodology and can be configured per instance. Aggregate must have at least one Practice. Factory method `Company.create()` is the only public constructor. |
| **Advanced patterns** | Static Factory Method, Value Objects, Domain Events |
| **Anti‑patterns** | Anemic domain model; God object (Company delegates scoring to internal algorithm) |

#### Subtasks for COMP‑001

- [ ] **COMP‑001‑1** – `src/companies/domain/company.ts` – Define Company aggregate root (fields: id, name, practices, privacyScore, category; creates via static `create()`)  
  `imports_from`: Practice, PrivacyScore, Category  
  `verification`: `pnpm tsc --noEmit` compiles; later tests will cover
- [ ] **COMP‑001‑2** – `src/companies/domain/privacy-score.ts` – Define `PrivacyScore` value object (validates 0‑100, returns `Result<PrivacyScore, ValidationError>`)  
  `imports_from`: `Result` from `src/shared/result.ts`, `ValidationError` from `src/shared/errors.ts`  
  `verification`: `pnpm tsc --noEmit` and corresponding test suite passes
- [ ] **COMP‑001‑3** – `src/companies/domain/category.ts` – Define `Category` enum and assignment logic  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **COMP‑001‑4** – `src/companies/domain/company-events.ts` – Define `CompanyEvaluated` event (payload: { companyId, companyName, score, category })  
  `imports_from`: none (uses primitive companyId)  
  `verification`: `pnpm tsc --noEmit`
- [ ] **COMP‑001‑5** – `src/companies/domain/privacy-scoring-algorithm.ts` – Implement weighted algorithm as a pure function; reads constants based on Privacy Fairness Index methodology; accepts optional weights parameter  
  `imports_from`: Practice, `Result`  
  `verification`: unit tests pass; test_algorithm_uses_correct_weights  
  Real-world weights (mapped from the Privacy Fairness Index, terms.law 2026):
- Data Collection & Sharing: 0.40  (merges "Data Collection" 30% + "Third‑Party Sharing" 20% → 50% of base; scaled to 40%)
- Transparency & User Control: 0.35 (merges "User Control & Consent" 15% + "Transparency & Access" 10%)
- Retention & Security: 0.15 (merges "Retention & Deletion" 20% + "Security" 5%)
- Legal Compliance History: 0.10 (external regulatory record — not directly measured by policy‑only PFI)
Standard PFI categories: data-collection, third-party-sharing, retention-deletion,
user-control-consent, security-breach-notification, transparency-access.
- [ ] **COMP‑001‑6** – `tests/unit/company/domain/company.test.ts` – Full TDD suite with fixtures (Acme Corp, etc.)  
  `imports_from`: Company, PrivacyScore, etc.  
  `verification`: `pnpm test -- company.test.ts` passes, coverage ≥90%
- [ ] **COMP‑001‑7** – `features/company/company-evaluation.feature` – Gherkin scenarios  
  `verification`: `pnpm test:bdd` passes scenarios
- [ ] **COMP‑001‑8** – Depth refactor check: verify public method count vs. implementation LOC, ensure errors are `Result` types  
  `verification`: `pnpm lint && pnpm tsc --noEmit && pnpm test -- coverage`

---

### `BROK‑001` Define Data Broker domain model | Status: `backlog`  
`depends_on`: `ARCH‑001`

- [ ] **Parent** `BROK‑001` – Establish core domain entities for data broker tracking and deletion.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Data Broker Management `Aggregate root:` DataBroker `Ubiquitous language:` DataBroker, OptOutProcess, DeletionRequest, BrokerComplianceStatus, BrokerRegistered (event) |
| **BDD** | `Feature:` Administrator registers broker `Scenario 1:` Happy path – valid registration creates broker with pending opt‑out `Scenario 2:` Invalid registration data → rejected with ValidationError `Scenario 3:` Broker with existing registration → deduplication handled |
| **TDD** | `Test file(s):` tests/unit/broker/domain/data_broker.test.ts `Red‑Green‑Refactor cycle:` 1) create with valid data, 2) validate opt‑out process must be present, 3) compliance status tracking, 4) reject missing fields, 5) test duplicate detection |
| **Deep module** | `Public interface:` DataBroker.register(), updateCompliance(), getOptOutProcess() `Hidden complexity:` Registration validation, compliance rules (~150 lines) `Depth metric:` deep – 3 methods, ~150 lines |
| **Definition of done** | Domain model, events, `.feature` file, tests ≥90% |
| **Out of scope** | Real‑time DROP API |
| **Rules** | BrokerComplianceStatus is auditable; all deletions reference broker by identity; events published on state change |
| **Advanced patterns** | State pattern, Domain Events |
| **Anti‑patterns** | Primitive obsession, anemic model |

#### Subtasks for BROK‑001

- [ ] **BROK‑001‑1** – `src/brokers/domain/data_broker.ts` – DataBroker aggregate root  
- [ ] **BROK‑001‑2** – `src/brokers/domain/opt_out_process.ts` – OptOutProcess value object modeling CA DROP workflow  
  `imports_from`: `Result`, `ValidationError`  
  `verification`: `pnpm tsc --noEmit` and tests  
  CA DROP fields: activationDate (August 1, 2026), pollInterval (every 45 days),
determinationDeadline (date of download + 90 days), compensationDailyFine ($200),
status (pending/complete/suppressed).
Note: All identifiers go on a permanent suppression list regardless of match status.
  `verification`: `pnpm tsc --noEmit`
- [ ] **BROK‑001‑5** – `tests/unit/broker/domain/data_broker.test.ts` – Unit tests  
  `imports_from`: DataBroker, etc.  
  `verification`: pass, coverage ≥90%
- [ ] **BROK‑001‑6** – `features/broker/broker-management.feature` – BDD scenarios  
  `verification`: `pnpm test:bdd` passes
- [ ] **BROK‑001‑7** – Depth refactor check  
  `verification`: `pnpm lint && pnpm tsc --noEmit && pnpm test -- coverage`

---

### `TECH‑001` Define Privacy Technology domain model | Status: `backlog`  
`depends_on`: `ARCH‑001`

- [ ] **Parent** `TECH‑001` – Establish domain entities for privacy‑enhancing technologies.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Technology Evaluation `Aggregate root:` PrivacyTechnology `Ubiquitous language:` PrivacyTechnology, TechnologyType, MaturityLevel, UseCase, TechnologyEvaluated (event) |
| **BDD** | `Feature:` Researcher categorizes technology `Scenario 1:` Valid technology → classified with maturity and use cases `Scenario 2:` Missing source → rejected `Scenario 3:` Update maturity level when evidence changes |
| **TDD** | `Test file(s):` tests/unit/tech/domain/privacy_technology.test.ts `Red‑Green‑Refactor cycle:` 1) create with type, 2) validate maturity, 3) reject missing source, 4) test event publication |
| **Deep module** | `Public interface:` classify(), getMaturity(), getUseCases() `Hidden complexity:` Validation, maturity assessment (~120 lines) `Depth metric:` moderate – 3 methods, ~120 lines |
| **Definition of done** | Domain model, events, `.feature` file, tests |
| **Out of scope** | Real‑time monitoring, automated research analysis |
| **Rules** | Technologies must cite verifiable sources; maturity based on defined criteria |
| **Advanced patterns** | Specification for maturity, Domain Events |
| **Anti‑patterns** | Subjective classification |

#### Subtasks for TECH‑001

- [ ] **TECH‑001‑1** – `src/tech/domain/privacy_technology.ts` – PrivacyTechnology aggregate root  
  `imports_from`: TechnologyType, MaturityLevel  
  `verification`: `pnpm tsc --noEmit`
- [ ] **TECH‑001‑2** – `src/tech/domain/technology_type.ts` – TechnologyType value object  
  `imports_from`: `Result`, `ValidationError`  
  `verification`: `pnpm tsc --noEmit`
- [ ] **TECH‑001‑3** – `src/tech/domain/maturity_level.ts` – MaturityLevel value object  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`  
  **Allowed values**: EXPERIMENTAL, EMERGING, ESTABLISHED, MATURE
- [ ] **TECH‑001‑4** – `src/tech/domain/use_case.ts` – UseCase value object  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **TECH‑001‑5** – `src/tech/domain/tech-events.ts` – TechnologyEvaluated event  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **TECH‑001‑6** – `tests/unit/tech/domain/privacy_technology.test.ts` – Unit tests  
  `verification`: pass
- [ ] **TECH‑001‑7** – `features/tech/technology-categorization.feature` – BDD scenarios  
  `verification`: `pnpm test:bdd` passes
- [ ] **TECH‑001‑8** – Depth refactor check  
  `verification`: `pnpm lint && pnpm tsc --noEmit && pnpm test -- coverage`

---

### `VIOL‑001` Define Privacy Violation tracking domain | Status: `backlog`  
`depends_on`: `ARCH‑001` (CompanyReference uses only primitives; no cross‑context dependency)

- [ ] **Parent** `VIOL‑001` – Create domain model for tracking privacy violations.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Violation Tracking `Aggregate root:` PrivacyViolation `Ubiquitous language:` PrivacyViolation, ViolationType, Severity, CompanyReference, ViolationRecorded (event) |
| **BDD** | `Feature:` Compliance officer records violation `Scenario 1:` Valid violation with existing company ID → stored with CompanyReference `Scenario 2:` Invalid severity → rejected |
| **TDD** | `Test file(s):` tests/unit/violation/domain/privacy_violation.test.ts `Red‑Green‑Refactor cycle:` 1) create with required data, 2) severity classification, 3) company association via CompanyReference (valid ID), 4) reject invalid CompanyReference (format), 5) reject negative severity |
| **Deep module** | `Public interface:` record(), getSeverity(), getCompanyReference() `Hidden complexity:` Severity calculation, company ID validation (~100 lines) `Depth metric:` moderate – 3 methods, ~100 lines |
| **Definition of done** | Domain model, events, `.feature` file (2 scenarios), tests |
| **Out of scope** | Real‑time violation detection |
| **Rules** | Violations reference Company by CompanyReference (value object with { companyId, companyName snapshot }). All violations must have verifiable sources. CompanyReference validation checks only that ID format is valid; actual existence check happens in INT‑001 via events. |
| **Advanced patterns** | Domain Events, Specification for severity |
| **Anti‑patterns** | Violation duplication, missing evidence links, inconsistent severity |

#### Subtasks for VIOL‑001

- [ ] **VIOL‑001‑1** – `src/violation/domain/privacy_violation.ts` – PrivacyViolation aggregate root  
  `imports_from`: ViolationType, Severity, CompanyReference  
  `verification`: `pnpm tsc --noEmit`
- [ ] **VIOL‑001‑2** – `src/violation/domain/violation_type.ts` – ViolationType value object  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **VIOL‑001‑3** – `src/violation/domain/severity.ts` – Severity value object with research-backed tiers  
  `imports_from`: `Result`, `ValidationError`  
  `verification`: `pnpm tsc --noEmit`  
  **Research-backed tiers**: Critical (>10M records or immutable data like SSN, DNA), High (unauthorized sale, lack of consent for tracking), Moderate (insufficient disclosure), Low (technical violations without consumer harm)
- [ ] **VIOL‑001‑4** – `src/violation/domain/company_reference.ts` – CompanyReference value object (companyId: string, companyName: string — populated by CompanyEvaluated event handler; see INT‑001)  
  `imports_from`: none (primitives only)  
  `verification`: `pnpm tsc --noEmit`
- [ ] **VIOL‑001‑5** – `src/violation/domain/violation-events.ts` – ViolationRecorded event  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **VIOL‑001‑6** – `tests/unit/violation/domain/privacy_violation.test.ts` – Unit tests  
  `verification`: pass
- [ ] **VIOL‑001‑7** – `features/violation/violation-tracking.feature` – Gherkin scenarios  
  `verification`: `pnpm test:bdd` passes
- [ ] **VIOL‑001‑8** – Depth refactor check  
  `verification`: `pnpm lint && pnpm tsc --noEmit && pnpm test -- coverage`

---

### `LAW‑001` Define International Privacy Law domain | Status: `backlog`  
`depends_on`: `ARCH‑001`

- [ ] **Parent** `LAW‑001` – Establish domain model for international privacy laws.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Legal Compliance `Aggregate root:` PrivacyLaw `Ubiquitous language:` PrivacyLaw, Jurisdiction, AdequacyStatus, AdequacyDecision, LawUpdated (event) |
| **BDD** | `Feature:` Legal analyst adds privacy law `Scenario 1:` Valid law with jurisdiction → stored with adequacy decision `Scenario 2:` Missing source → rejected `Scenario 3:` Update adequacy decision when official source changes |
| **TDD** | `Test file(s):` tests/unit/law/domain/privacy_law.test.ts `Red‑Green‑Refactor cycle:` 1) create with jurisdiction, 2) track adequacy, 3) reject law without source, 4) reject invalid AdequacyDecision (no URL) |
| **Deep module** | `Public interface:` create(), getRequirements(), getEnforcement() `Hidden complexity:` Legal validation, requirement extraction (~130 lines) `Depth metric:` deep – 3 methods, ~130 lines |
| **Definition of done** | Domain model, events, `.feature` file, tests |
| **Out of scope** | Real‑time legal monitoring, automated legal advice |
| **Rules** | All laws must have verifiable sources; jurisdictions use ISO codes; AdequacyDecision contains `source_url` and `decision_date`; no legal advice without disclaimers. Legal compliance scoring follows a structured framework adapted from: EU AI Act 5‑dimension model (explainability, fairness, privacy, robustness, social well‑being); PET evaluation criteria (New America, 2026): absence of re‑identification, anonymity protections, linkage‑attack prevention, scalability, and implementation complexity; DROP broker compliance requirements (California 2026): 45‑day polling, 90‑day determination, permanent suppression list |
| **Advanced patterns** | Domain Events, Visitor for requirement analysis |
| **Anti‑patterns** | Legal advice without disclaimers, outdated data |

#### Subtasks for LAW‑001

- [ ] **LAW‑001‑1** – `src/law/domain/privacy_law.ts` – PrivacyLaw aggregate root  
  `imports_from`: Jurisdiction, AdequacyStatus, AdequacyDecision  
  `verification`: `pnpm tsc --noEmit`
- [ ] **LAW‑001‑2** – `src/law/domain/jurisdiction.ts` – Jurisdiction value object  
  `imports_from`: `Result` from `src/shared/result.ts`, `ValidationError` from `src/shared/errors.ts`  
  `verification`: `pnpm tsc --noEmit`
- [ ] **LAW‑001‑3** – `src/law/domain/adequacy_status.ts` – AdequacyStatus value object  
  `verification`: `pnpm tsc --noEmit`
- [ ] **LAW‑001‑4** – `src/law/domain/adequacy_decision.ts` – AdequacyDecision value object expanded for 2026 US state law landscape  
  `imports_from`: `Result`, `ValidationError`  
  `verification`: `pnpm tsc --noEmit`  
  **2026 updates**: Add fields: lawName (e.g., CCPA, CTDPA, MODPA), effectiveDate, keyRequirements (data minimization, universal opt-out, DPIA)  
  **Note**: 20+ US states now have comprehensive privacy laws; adequacy decisions are jurisdiction-specific; value object holds list of applicable laws
- [ ] **LAW‑001‑5** – `src/law/domain/law-events.ts` – LawUpdated event  
  `verification`: `pnpm tsc --noEmit`
- [ ] **LAW‑001‑6** – `tests/unit/law/domain/privacy_law.test.ts` – Unit tests  
  `verification`: pass
- [ ] **LAW‑001‑7** – `features/law/law-management.feature` – BDD scenarios  
  `verification`: `pnpm test:bdd` passes
- [ ] **LAW‑001‑8** – Depth refactor check  
  `verification`: `pnpm lint && pnpm tsc --noEmit && pnpm test -- coverage`

---

## Phase 2 – Repositories & Infrastructure

### `COMP‑002` Implement Company repository | Status: `backlog`  
`depends_on`: `COMP‑001`

- [ ] **Parent** `COMP‑002` – Create persistence layer for Company aggregate.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Company Evaluation `Aggregate root:` N/A (infrastructure service implements CompanyRepository domain interface) |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/integration/company/infrastructure/cloudflare_kv_repository.test.ts, tests/unit/company/infrastructure/memory_repository.test.ts `Red‑Green‑Refactor cycle:` for each implementation: save and retrieve full aggregate |
| **Deep module** | `Public interface:` CompanyRepository.save(), findById(), findByCategory() `Hidden complexity:` KV storage, mapping primitives to value objects `Depth metric:` moderate – 3 methods, ~100 lines of mapping |
| **Definition of done** | In‑memory repo (for tests), Cloudflare KV repo (for production), mapping layer converts stored primitives to domain objects, integration tests pass |
| **Out of scope** | Database migrations |
| **Rules** | Repositories return entire aggregates; use identity references; mapping inside infrastructure; explicit data mapping: primitives → Value Objects |
| **Advanced patterns** | Unit of Work (if needed later) |
| **Anti‑patterns** | Active Record, partial persistence, leaking infrastructure details |

#### Subtasks for COMP‑002

- [ ] **COMP‑002‑1** – `src/companies/infrastructure/company_repository.ts` – Repository interface  
  `verification`: `pnpm tsc --noEmit`
- [ ] **COMP‑002‑2** – `src/companies/infrastructure/memory_company_repository.ts` – In‑memory implementation  
  `verification`: unit tests pass
- [ ] **COMP‑002‑3** – `src/companies/infrastructure/cloudflare_kv_company_repository.ts` – Cloudflare KV implementation (uses KV emulator)  
  `verification`: integration tests pass with `@cloudflare/vitest-pool-workers`  
  Note: KV supports prefix‑based listing for secondary indexes
(key pattern: category:{value}:{companyId}). For v1 static build, write rate
limits (1/sec/key) are not a concern. For v2 with SSR, consider D1 or Durable
Objects for write‑heavy workloads.
- [ ] **COMP‑002‑4** – `tests/integration/company/infrastructure/cloudflare_kv_repository.test.ts` – Integration tests  
  `verification`: pass  
  (Note: Cloudflare KV is eventually consistent; tests must include retry/wait logic.)
- [ ] **COMP‑002‑5** – `tests/unit/company/infrastructure/memory_repository.test.ts` – Unit tests for in‑memory  
  `verification`: pass
- [ ] **COMP‑002‑6** – Depth refactor check (repository is inherently shallow; ensure it remains simple)  
  `verification`: no new public methods beyond interface

---

### `BROK‑002` Implement Data Broker registry integration | Status: `backlog`  
`depends_on`: `BROK‑001`

- [ ] **Parent** `BROK‑002` – Create integration layer for California DROP platform.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Data Broker Management `Aggregate root:` N/A (domain service interface in domain; implementation in infrastructure) |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/integration/broker/infrastructure/drop_adapter.test.ts |
| **Deep module** | `Public interface:` BrokerRegistry.sync(), getBrokers(), searchByName() `Hidden complexity:` HTTP, CSV parsing, error handling `Depth metric:` deep – 3 methods, ~180 lines |
| **Definition of done** | DROP adapter, CSV parser, error handling (idempotent), integration tests pass |
| **Out of scope** | Real‑time webhooks |
| **Rules** | Domain interface in `src/brokers/domain/broker_registry.ts`; implementation in `src/brokers/infrastructure/drop_broker_registry.ts`; CSV parsed robustly; `Result<T, IntegrationError>` for failures |
| **Advanced patterns** | Adapter, Circuit breaker |
| **Anti‑patterns** | Tight coupling, error swallowing |

#### Subtasks for BROK‑002

- [ ] **BROK‑002‑1** – `src/brokers/domain/broker_registry.ts` – Domain service interface  
  `verification`: `pnpm tsc --noEmit`
- [ ] **BROK‑002‑2** – `src/brokers/infrastructure/drop_broker_registry.ts` – Implementation (uses CSV parser internally)  
  `verification`: integration tests pass
- [ ] **BROK‑002‑3** – `src/brokers/infrastructure/csv_parser.ts` – Data‑broker registry adapter (handles hashed identifier lists from DROP; CSV parsing may be one possible format, but final format determined by CA DROP API).  
  `verification`: `pnpm tsc --noEmit`
- [ ] **BROK‑002‑4** – `tests/integration/broker/infrastructure/drop_adapter.test.ts` – Integration tests (mocked HTTP)  
  `verification`: pass
- [ ] **BROK‑002‑5** – Depth refactor check  
  `verification`: ensure interface depth, CSV parser remains shallow

---

## Phase 2.5 – Seed Data & Content Strategy

### `DATA‑001` Create shared seed data for all stubs | Status: `backlog`  
`depends_on`: none (independent research)

- [ ] **Parent** `DATA‑001` – Create comprehensive seed data for all application services.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Data Management (enabling) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/integration/seed_data.test.ts `Red‑Green‑Refactor cycle:` 1) verify seed data loads, 2) verify all entities have required fields, 3) verify neutral framing |
| **Deep module** | N/A – data task |
| **Definition of done** | `src/data/seed.json` with sections: technologies, violations, laws; each section has 3‑5 verified entries with source URLs; all entities follow neutral framing |
| **Out of scope** | Real‑time data updates |
| **Rules** | Neutral framing only: documented practices without judgment, factual data collection descriptions, verifiable sources required |
| **Advanced patterns** | JSON schema validation |
| **Anti‑patterns** | Subjective language, missing sources |

#### Subtasks for DATA‑001

- [ ] **DATA‑001‑1** – `src/data/seed.json` – Create seed data structure with sections: `technologies`, `violations`, `laws`  
  `imports_from`: none  
  `verification`: file exists and valid JSON with three sections
- [ ] **DATA‑001‑2** – Populate each section with at least 3‑5 verified entries using research sources  
  `imports_from`: none  
  `verification`: each entry has `source_url` field pointing to primary source
- [ ] **DATA‑001‑3** – Apply neutral framing convention to all entities  
  `imports_from`: none  
  `verification`: grep finds no "good/bad" language in seed data
- [ ] **DATA‑001‑4** – Align seed data with updated domain models (new PrivacyScore weights, CA DROP details, 2026 law list)  
  `imports_from`: none  
  `verification`: `pnpm test` (all stubs load seed data correctly)
Complete 2026 US state law abbreviations:
CCPA/CPRA, VCDPA, CPA, CTDPA, UCPA, ICDPA, INCDPA, TIPA, TDPSA, FDBR, MODPA,
MCDPA (MN), MCDPA (MT), OCPA, DPDPA, NHDPA, NJDPA, KYCDPA, NDPA, RHDPA.
Oklahoma (ODPA) effective Jan 1, 2027 (not included in v1 seed).
- [ ] **DATA‑001‑5** – `docs/neutral-language-guide.md` – Create style guide for factual, sourced approach  
  `imports_from`: none  
  `verification`: file exists with neutral framing guidelines

---

## Phase 3 – Application Layer (Query Services & DTOs)

### `APP‑001` Company query service and DTOs | Status: `backlog`  
`depends_on`: `COMP‑002`

- [ ] **Parent** `APP‑001` – Provide read‑side endpoint for the Web Adapter to fetch company data.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Company Evaluation (application layer) `Aggregate root:` N/A `Ubiquitous language:` CompanyDTO, filter, paginate |
| **BDD** | N/A – internal service |
| **TDD** | `Test file(s):` tests/unit/company/application/company_query_service.test.ts `Red‑Green‑Refactor cycle:` 1) test_get_all_companies_returns_DTOs, 2) test_filter_by_category, 3) test_search_by_name |
| **Deep module** | `Public interface:` getCompanies(filter, page) → CompanyDTO[] `Hidden complexity:` Repo access, DTO mapping `Depth metric:` shallow‑moderate |
| **Definition of done** | Service works with CompanyRepository; returns `CompanyDTO[]`; tests ≥70% coverage; DTO defined |
| **Out of scope** | Command handling |
| **Rules** | Never expose domain aggregates; DTOs are plain data objects |
| **Advanced patterns** | DTO mapping functions |
| **Anti‑patterns** | Returning domain objects directly |

#### Subtasks for APP‑001

- [ ] **APP‑001‑1** – `src/companies/application/company_query_service.ts` – Service implementation  
  `verification`: tests pass
- [ ] **APP‑001‑2** – `src/companies/application/company_dto.ts` – DTO definition  
  `verification`: `pnpm tsc --noEmit`
- [ ] **APP‑001‑3** – `tests/unit/company/application/company_query_service.test.ts` – Unit tests  
  `verification`: pass

---

### `APP‑002` Broker query service and DTOs | Status: `backlog`  
`depends_on`: `BROK‑002`

- [ ] **Parent** `APP‑002` – Provide broker data for the Web Adapter.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Data Broker Management (application) `Aggregate root:` N/A `Ubiquitous language:` BrokerDTO, delete, search |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/unit/broker/application/broker_query_service.test.ts `Red‑Green‑Refactor cycle:` 1) return all brokers, 2) search by name, 3) pagination |
| **Deep module** | `Public interface:` getBrokers(filter) → BrokerDTO[] `Hidden complexity:` Repo access, mapping `Depth metric:` shallow‑moderate |
| **Definition of done** | Service, DTO, tests |
| **Out of scope** | Delete command |
| **Rules** | DTOs only; no domain objects leaked |
| **Anti‑patterns** | Exposing aggregate |

#### Subtasks for APP‑002

- [ ] **APP‑002‑1** – `src/brokers/application/broker_query_service.ts` – Implementation  
  `verification`: tests pass
- [ ] **APP‑002‑2** – `src/brokers/application/broker_dto.ts` – DTO  
  `verification`: `pnpm tsc --noEmit`
- [ ] **APP‑002‑3** – `tests/unit/broker/application/broker_query_service.test.ts` – Tests  
  `verification`: pass

---

### `APP‑003` Technology query service and DTOs | Status: `backlog`  
`depends_on`: `TECH‑001`, `DATA‑001`

- [ ] **Parent** `APP‑003` – Provide technology data.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Technology Evaluation (application) `Aggregate root:` N/A `Ubiquitous language:` TechnologyDTO, category, maturity |
| **BDD** | N/A – internal service |
| **TDD** | `Test file(s):` tests/unit/tech/application/technology_query_service.test.ts `Red‑Green‑Refactor cycle:` 1) return all technologies, 2) filter by category, 3) filter by maturity |
| **Deep module** | `Public interface:` getTechnologies(filter) → TechnologyDTO[] `Hidden complexity:` In‑memory stub, DTO mapping `Depth metric:` shallow‑moderate |
| **Definition of done** | Service with in‑memory stub using shared seed data, DTO definition, tests ≥70% coverage |
| **Out of scope** | Write operations, persistence via Cloudflare KV deferred to v2; v1 uses static seed data loaded from `src/data/seed.json`. Repository implementation will be retrofitted in v2; stub uses the same interface. |
| **Rules** | DTOs only; no domain objects leaked; stub uses shared seed data from `DATA‑001` |
| **Advanced patterns** | DTO mapping functions |
| **Anti‑patterns** | Returning domain objects directly |

#### Subtasks for APP‑003

- [ ] **APP‑003‑1** – `src/tech/application/technology_query_service.ts` – Service implementation (in‑memory stub)  
  `imports_from`: TechnologyDTO  
  `verification`: tests pass
- [ ] **APP‑003‑2** – `src/tech/application/technology_dto.ts` – DTO definition (id, name, type, maturity, useCases)  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **APP‑003‑3** – `tests/unit/tech/application/technology_query_service.test.ts` – Unit tests  
  `imports_from`: TechnologyQueryService, TechnologyDTO  
  `verification`: pass, coverage ≥70%

---

### `APP‑004` Violation query service and DTOs | Status: `backlog`  
`depends_on`: `VIOL‑001`, `DATA‑001`

- [ ] **Parent** `APP‑004` – Provide violation data.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Violation Tracking (application) `Aggregate root:` N/A `Ubiquitous language:` ViolationDTO, severity, companyReference |
| **BDD** | N/A – internal service |
| **TDD** | `Test file(s):` tests/unit/violation/application/violation_query_service.test.ts `Red‑Green‑Refactor cycle:` 1) return all violations, 2) filter by severity, 3) filter by company |
| **Deep module** | `Public interface:` getViolations(filter) → ViolationDTO[] `Hidden complexity:` In‑memory stub, CompanyReference display data `Depth metric:` shallow‑moderate |
| **Definition of done** | Service with in‑memory stub using shared seed data, DTO with CompanyReference display fields, tests ≥70% coverage |
| **Out of scope** | Write operations, persistence via Cloudflare KV deferred to v2; v1 uses static seed data loaded from `src/data/seed.json`. Repository implementation will be retrofitted in v2; stub uses the same interface. |
| **Rules** | DTOs only; no domain objects leaked; ViolationDTO includes companyName for display; stub uses shared seed data from `DATA‑001` |
| **Advanced patterns** | DTO mapping functions |
| **Anti‑patterns** | Returning domain objects directly |

#### Subtasks for APP‑004

- [ ] **APP‑004‑1** – `src/violation/application/violation_query_service.ts` – Service implementation (in‑memory stub)  
  `imports_from`: ViolationDTO  
  `verification`: tests pass
- [ ] **APP‑004‑2** – `src/violation/application/violation_dto.ts` – DTO definition (id, type, severity, companyReference with companyName)  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **APP‑004‑3** – `tests/unit/violation/application/violation_query_service.test.ts` – Unit tests  
  `imports_from`: ViolationQueryService, ViolationDTO  
  `verification`: pass, coverage ≥70%

---

### `APP‑005` Legal query service and DTOs | Status: `backlog`  
`depends_on`: `LAW‑001`, `DATA‑001`

- [ ] **Parent** `APP‑005` – Provide legal data.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Legal Compliance (application) `Aggregate root:` N/A `Ubiquitous language:` LawDTO, jurisdiction, adequacy |
| **BDD** | N/A – internal service |
| **TDD** | `Test file(s):` tests/unit/law/application/legal_query_service.test.ts `Red‑Green‑Refactor cycle:` 1) return all laws, 2) filter by jurisdiction, 3) filter by adequacy status |
| **Deep module** | `Public interface:` getLaws(filter) → LawDTO[] `Hidden complexity:` Jurisdiction filtering, adequacy indicator `Depth metric:` shallow‑moderate |
| **Definition of done** | Service, DTO with jurisdiction filtering, tests ≥70% coverage |
| **Out of scope** | Write operations, persistence via Cloudflare KV deferred to v2; v1 uses static seed data loaded from `src/data/seed.json`. Repository implementation will be retrofitted in v2; stub uses the same interface. |
| **Rules** | DTOs only; no domain objects leaked; jurisdiction filtering uses ISO codes |
| **Advanced patterns** | DTO mapping functions |
| **Anti‑patterns** | Returning domain objects directly |

#### Subtasks for APP‑005

- [ ] **APP‑005‑1** – `src/law/application/legal_query_service.ts` – Service implementation  
  `imports_from`: LawDTO  
  `verification`: tests pass
- [ ] **APP‑005‑2** – `src/law/application/law_dto.ts` – DTO definition (id, name, jurisdiction, adequacyStatus, adequacyDecision)  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **APP‑005‑3** – `tests/unit/law/application/legal_query_service.test.ts` – Unit tests  
  `imports_from`: LegalQueryService, LawDTO  
  `verification`: pass, coverage ≥70%

---

## Phase 4 – Web Components (Presentation)

All WEB tasks depend on corresponding APP tasks and consume DTOs. Feature files are placed in `features/<context>/`.

Hydration strategy: Static pages use Astro .astro files (zero JS shipped).
Interactive components (CompanyListing, BrokerDashboard, etc.) use React islands
with targeted hydration directives: client:visible for filter panels and listings,
client:idle for search bars and auxiliary widgets.

### `WEB‑001` Company listing page | Status: `backlog`  
`depends_on`: `APP‑001`

- [ ] **Parent** `WEB‑001` – Build companies page with filtering and search.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (supporting) `Aggregate root:` N/A `Ubiquitous language:` Listing, Filter, Search, Pagination |
| **BDD** | `Feature:` Visitor browses companies `Scenario 1:` Happy path filter → list updates `Scenario 2:` No results message `Scenario 3:` Shareable URL preserves filter |
| **TDD** | `Test file(s):` tests/unit/web/components/company_listing.test.ts `Red‑Green‑Refactor cycle:` includes error state test |
| **Deep module** | `Public interface:` `<CompanyListing category={} searchQuery={} pageSize={} />` `Hidden complexity:` filter state, URL sync, debounce, pagination `Depth metric:` moderate – 3 props, ~150 lines internal |
| **Definition of done** | Component renders, tests pass, BDD scenarios in `.feature`, DTO consumed, loading/empty/error states handled |
| **Out of scope** | Analytics, real‑time updates |
| **Rules** | Fetches via `CompanyQueryService` (DTOs only); no domain objects; accessible |
| **Advanced patterns** | Composition, Observer for filters |
| **Anti‑patterns** | Monolithic component, direct DOM manipulation, missing states |

#### Subtasks for WEB‑001

- [ ] **WEB‑001‑1** – `src/web/components/company_listing.tsx` – Main component  
  `imports_from`: CompanyQueryService, CompanyDTO  
  `verification`: unit tests pass, BDD scenarios pass
- [ ] **WEB‑001‑2** – `src/web/components/company_card.tsx` – Card component  
  `imports_from`: CompanyDTO  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑001‑3** – `src/web/components/filter_panel.tsx` – Filters  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑001‑4** – `tests/unit/web/components/company_listing.test.ts` – Component tests  
  `imports_from`: CompanyListing  
  `verification`: pass
- [ ] **WEB‑001‑5** – `features/company/company-browsing.feature` – Gherkin scenarios (visitor actor)  
  `verification`: `pnpm test:bdd` passes

---

### `WEB‑002` Broker dashboard | Status: `backlog`  
`depends_on`: `APP‑002`

- [ ] **Parent** `WEB‑002` – Build broker dashboard with search and deletion guidance.

| Aspect | Detail |
| :--- | :--- |
| **BDD** | `Feature:` Visitor searches brokers `Scenario 1:` Search by name → list updated `Scenario 2:` No brokers found message |
| **TDD** | tests/unit/web/components/broker_dashboard.test.ts |
| **Deep module** | `Public interface:` `<BrokerDashboard query={} />` `Hidden complexity:` fuzzy search, async state `Depth metric:` moderate – 1 prop, ~130 lines |
| **Definition of done** | Component, tests, `.feature`, DTO consumption |
| **Rules** | Uses `BrokerQueryService`; deletion guide actionable |

#### Subtasks for WEB‑002

- [ ] **WEB‑002‑1** – `src/web/components/broker_dashboard.tsx` – Main component  
  `imports_from`: BrokerQueryService, BrokerDTO  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑002‑2** – `src/web/components/broker_search.tsx` – Search bar  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑002‑3** – `src/web/components/deletion_guide.tsx` – Guide display  
  `imports_from`: BrokerDTO  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑002‑4** – `tests/unit/web/components/broker_dashboard.test.ts` – Tests  
  `imports_from`: BrokerDashboard  
  `verification`: pass
- [ ] **WEB‑002‑5** – `features/broker/broker-dashboard.feature` – Scenarios  
  `verification`: `pnpm test:bdd`
- [ ] **WEB‑002‑6** – Depth refactor check: verify component structure and integration  
  `verification`: `pnpm lint && pnpm tsc --noEmit && pnpm test -- coverage && pnpm build`

---

### `WEB‑003` Technology showcase | Status: `backlog`  
`depends_on`: `APP‑003`

- [ ] **Parent** `WEB‑003` – Build technology showcase with categorization and maturity indicators.

| Aspect | Detail |
| :--- | :--- |
| **BDD** | `Feature:` Visitor browses technologies `Scenario:` Filter by category, see maturity badges |
| **TDD** | tests/unit/web/components/technology_showcase.test.ts |
| **Deep module** | `Public interface:` `<TechnologyShowcase category={} />` `Hidden complexity:` maturity visualization `Depth metric:` moderate – 1 prop, ~100 lines |
| **Definition of done** | Component, tests, `.feature` |
| **Rules** | Uses `TechnologyQueryService`; maturity badges visual |

#### Subtasks for WEB‑003

- [ ] **WEB‑003‑1** – `src/web/components/technology_showcase.tsx` – Main component  
  `imports_from`: TechnologyQueryService, TechnologyDTO  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑003‑2** – `src/web/components/technology_card.tsx` – Card component with maturity badge  
  `imports_from`: TechnologyDTO  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑003‑3** – `src/web/components/maturity_badge.tsx` – Maturity indicator  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑003‑4** – `tests/unit/web/components/technology_showcase.test.ts` – Component tests  
  `imports_from`: TechnologyShowcase  
  `verification`: pass
- [ ] **WEB‑003‑5** – `features/tech/technology-browsing.feature` – Gherkin scenarios  
  `verification`: `pnpm test:bdd` passes
- [ ] **WEB‑003‑6** – Depth refactor check: verify component structure and integration  
  `verification`: `pnpm lint && pnpm tsc --noEmit && pnpm test -- coverage && pnpm build`

---

### `WEB‑004` Violation tracker | Status: `backlog`  
`depends_on`: `APP‑004`

- [ ] **Parent** `WEB‑004` – Build violation tracking page with severity visualization.

| Aspect | Detail |
| :--- | :--- |
| **BDD** | `Feature:` Visitor tracks violations `Scenario 1:` Filter by company → timeline with severity colors `Scenario 2:` No violations message |
| **TDD** | tests/unit/web/components/violation_tracker.test.ts |
| **Deep module** | `Public interface:` `<ViolationTracker companyId={} />` `Hidden complexity:` timeline calculation, severity colors `Depth metric:` moderate – 1 prop, ~140 lines |
| **Definition of done** | Component, tests, `.feature` |
| **Rules** | Uses `ViolationQueryService`; severity colors consistent |

#### Subtasks for WEB‑004

- [ ] **WEB‑004‑1** – `src/web/components/violation_tracker.tsx` – Main component  
  `imports_from`: ViolationQueryService, ViolationDTO  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑004‑2** – `src/web/components/violation_timeline.tsx` – Timeline component  
  `imports_from`: ViolationDTO  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑004‑3** – `src/web/components/severity_indicator.tsx` – Severity color indicator  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑004‑4** – `tests/unit/web/components/violation_tracker.test.ts` – Component tests  
  `imports_from`: ViolationTracker  
  `verification`: pass
- [ ] **WEB‑004‑5** – `features/violation/violation-tracking.feature` – Gherkin scenarios  
  `verification`: `pnpm test:bdd` passes
- [ ] **WEB‑004‑6** – Depth refactor check: verify component structure and integration  
  `verification`: `pnpm lint && pnpm tsc --noEmit && pnpm test -- coverage && pnpm build`

---

### `WEB‑005` Legal compliance guide | Status: `backlog`  
`depends_on`: `APP‑005`

- [ ] **Parent** `WEB‑005` – Build legal guide with jurisdiction filter and adequacy indicators.

| Aspect | Detail |
| :--- | :--- |
| **BDD** | `Feature:` Visitor views legal guide `Scenario:` Select jurisdiction → see applicable laws with adequacy |
| **TDD** | tests/unit/web/components/legal_guide.test.ts |
| **Deep module** | `Public interface:` `<LegalGuide jurisdiction={} />` `Hidden complexity:` legal data aggregation, jurisdiction filtering, adequacy indicator, legal disclaimer rendering `Depth metric:` moderate |
| **Definition of done** | Component, tests, `.feature` |
| **Rules** | Uses `LegalQueryService`; jurisdiction filtering; adequacy indicators; legal disclaimers |

#### Subtasks for WEB‑005

- [ ] **WEB‑005‑1** – `src/web/components/legal_guide.tsx` – Main component  
  `imports_from`: LegalQueryService, LawDTO  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑005‑2** – `src/web/components/jurisdiction_selector.tsx` – Jurisdiction filter  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑005‑3** – `src/web/components/adequacy_indicator.tsx` – Adequacy status indicator  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑005‑4** – `src/web/components/legal_disclaimer.tsx` – Legal disclaimer component  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑005‑5** – `tests/unit/web/components/legal_guide.test.ts` – Component tests  
  `imports_from`: LegalGuide  
  `verification`: pass
- [ ] **WEB‑005‑6** – `features/law/legal-guide.feature` – Gherkin scenarios  
  `verification`: `pnpm test:bdd` passes
- [ ] **WEB‑005‑7** – Depth refactor check: verify component structure and integration  
  `verification`: `pnpm lint && pnpm tsc --noEmit && pnpm test -- coverage && pnpm build`

---

### `WEB‑006` Create Astro pages that wire the components | Status: `backlog`  
`depends_on`: `ARCH‑001`

- [ ] **Parent** `WEB‑006` – Build complete page structure using components.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | `Feature:` Visitor navigates site `Scenario 1:` Visit companies page → see listing `Scenario 2:` Visit data brokers page → see dashboard `Scenario 3:` Visit practices pages → see content |
| **TDD** | `Test file(s):` tests/integration/pages.test.ts |
| **Deep module** | N/A – page wiring |
| **Definition of done** | All Astro pages created and accessible via `pnpm build` |
| **Out of scope** | Dynamic routing |
| **Rules** | Pages consume components; components handle loading/empty/error states; Pages can be implemented as soon as their corresponding query service is ready; mocks/stubs may be used during development. |
| **Advanced patterns** | Astro static routing |
| **Anti‑patterns** | Missing pages, broken navigation |
| **Optional Enhancement** | Consider using Astro's View Transitions API for SPA‑like navigation between pages, preserving state where appropriate. |

#### Subtasks for WEB‑006

- [ ] **WEB‑006‑1** – `src/pages/companies.astro` – Uses `<CompanyListing />` and `CompanyQueryService`  
  `imports_from`: CompanyListing  
  `verification`: page renders at `/companies`
- [ ] **WEB‑006‑2** – `src/pages/technologies.astro` – Uses `<TechnologyShowcase />` with filter for all technologies  
  `imports_from`: TechnologyShowcase  
  `verification`: page renders at `/technologies`
- [ ] **WEB‑006‑3** – `src/pages/data-brokers.astro` – Uses `<BrokerDashboard />`  
  `imports_from`: BrokerDashboard  
  `verification`: page renders at `/data-brokers`
- [ ] **WEB‑006‑4** – `src/pages/practices/data-collection.astro` – Content page with fact‑based lists  
  `imports_from`: none  
  `verification`: page renders at `/practices/data-collection`
- [ ] **WEB‑006‑5** – `src/pages/practices/data-breaches.astro` – Displays violation data using `<ViolationTracker />`  
  `imports_from`: ViolationTracker  
  `verification`: page renders at `/practices/data-breaches`
- [ ] **WEB‑006‑6** – `src/pages/practices/regulatory-actions.astro` – Uses `<LegalGuide />` filtered for enforcement actions  
  `imports_from`: LegalGuide  
  `verification`: page renders at `/practices/regulatory-actions`
- [ ] **WEB‑006‑7** – `src/pages/practices/surveillance.astro` – Static content page on tracking methods  
  `imports_from`: none  
  `verification`: page renders at `/practices/surveillance`
- [ ] **WEB‑006‑8** – `src/pages/practices/smart-devices.astro` – IoT privacy risks  
  `imports_from`: none  
  `verification`: page renders at `/practices/smart-devices`
- [ ] **WEB‑006‑9** – `src/pages/practices/platforms.astro` – OS comparisons  
  `imports_from`: none  
  `verification`: page renders at `/practices/platforms`
- [ ] **WEB‑006‑10** – `src/pages/take-action.astro` – Consumer guidance page  
  `imports_from`: none  
  `verification`: page renders at `/take-action`
- [ ] **WEB‑006‑11** – `src/pages/methodology.astro` – Explains scoring and evaluation  
  `imports_from`: none  
  `verification`: page renders at `/methodology`
- [ ] **WEB‑006‑12** – `src/pages/glossary.astro` – Ubiquitous language terms  
  `imports_from`: none  
  `verification`: page renders at `/glossary`
- [ ] **WEB‑006‑13** – `src/pages/about.astro` – Standard about page  
  `imports_from`: none  
  `verification`: page renders at `/about`
- [ ] **WEB‑006‑14** – `src/pages/index.astro` – Homepage with site overview and navigation to main sections  
  `imports_from`: none  
  `verification`: page renders at `/`
- [ ] **WEB‑006‑15** – `src/pages/sitemap.astro` – XML sitemap for SEO and navigation  
  `imports_from`: none  
  `verification`: page renders at `/sitemap.xml`
- [ ] **WEB‑006‑16** – `tests/integration/pages.test.ts` – Verify all pages build and are accessible  
  `verification`: `pnpm build` succeeds and all pages accessible

---

### `WEB‑007` Create legal pages | Status: `backlog`  
`depends_on`: none (static content)

- [ ] **Parent** `WEB‑007` – Build legal compliance pages.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (legal) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | N/A |
| **Deep module** | N/A – static content |
| **Definition of done** | Legal pages with proper disclaimers |
| **Out of scope** | Dynamic legal content |
| **Rules** | All legal information includes disclaimers |
| **Advanced patterns** | Static page generation |
| **Anti‑patterns** | Missing legal pages |

#### Subtasks for WEB‑007

- [ ] **WEB‑007‑1** – `src/pages/privacy.astro` – Privacy policy stating the site's own data practices  
  `imports_from`: none  
  `verification`: page renders at `/privacy`
- [ ] **WEB‑007‑2** – `src/pages/terms.astro` – Terms of Use  
  `imports_from`: none  
  `verification`: page renders at `/terms`
- [ ] **WEB‑007‑3** – Ensure all pages that reference legal information include disclaimer: "This content is informational, not legal advice"  
  `imports_from`: none  
  `verification`: grep finds disclaimer on all legal pages  
  **Pages to check**: privacy.astro, terms.astro, any page containing legal interpretation like regulatory-actions.astro

---

## Phase 5 – Cross‑context Integration

### `INT‑001` Link Violations to Companies | Status: `backlog`  
`depends_on`: `COMP‑001`, `VIOL‑001`, `COMP‑002`, `ARCH‑004`

- [ ] **Parent** `INT‑001` – Ensure violations correctly reference companies via domain events.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Violation Tracking ↔ Company Evaluation (integration via events) |
| **BDD** | `Feature:` User views violation list `Scenario:` Given a violation with a company reference, when displayed, the company name is shown (based on event‑driven snapshot) |
| **TDD** | `Test file(s):` tests/integration/cross_context/violation_company_integration.test.ts |
| **Definition of done** | `CompanyReference` in Violation context resolved via event handler that listens to `CompanyEvaluated` and updates the reference snapshot; subscription registration in composition root (e.g., `src/infrastructure/event_subscriptions.ts`); integration test passes |
| **Out of scope** | Full ACID consistency. **Production limitation**: The current in‑memory EventBus (ARCH‑004) does not persist events across Workers requests. Full cross‑request event handling requires Cloudflare Queues, planned for v2. |
| **Rules** | Use identity reference only; never load Company aggregate directly; event handler reads `companyName` from `CompanyEvaluated` payload and updates the `CompanyReference` snapshot; no repository call needed |
| **Advanced patterns** | Domain Events, eventual consistency |
| **Anti‑patterns** | Direct aggregate dependency |

#### Subtasks for INT‑001

- [ ] **INT‑001‑1** – `src/violation/domain/company_reference.ts` – (already defined in VIOL‑001‑4; verify it exists)  
  `verification`: file exists
- [ ] **INT‑001‑2** – `src/infrastructure/event_subscriptions.ts` – Wire `CompanyEvaluated` handler: update `CompanyReference` in violation context (reads { companyId, companyName } from CompanyEvaluated payload (same shape as COMP‑001‑4))  
  `verification`: integration test passes
- [ ] **INT‑001‑3** – `tests/integration/cross_context/violation_company_integration.test.ts` – Test that when a Company event is published, the violation reference is updated  
  `verification`: pass

---

### `INT‑002` Evaluate Company compliance against Privacy Laws | Status: `backlog`  
`depends_on`: `COMP‑002`, `LAW‑001`

- [ ] **Parent** `INT‑002` – Minimal cross‑context check whether a company's practices align with a given law.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Legal Compliance → Company Evaluation |
| **BDD** | `Feature:` Compliance check (v1 minimal) `Scenario:` Given a company and a law, check if any practice category matches law requirements; return pass/fail status. |
| **TDD** | tests/integration/cross_context/compliance_check.test.ts |
| **Definition of done** | `ComplianceService` in legal context that takes companyId and lawId, returns `{ status: 'compliant' | 'non_compliant' }` by checking if company has at least one practice in the category required by the law (simplified). **Update**: Now considers expanded `AdequacyDecision` with multiple laws per jurisdiction. |
| **Out of scope** | Full legal analysis, multi‑jurisdiction |
| **Rules** | Uses identity references; service orchestrates queries without loading full aggregates across contexts |
| **Advanced patterns** | Domain service, query‑only integration |
| **Anti‑patterns** | Monolithic cross‑context logic |

#### Subtasks for INT‑002

- [ ] **INT‑002‑1** – `src/legal/application/compliance_service.ts` – Minimal compliance check  
  `imports_from`: CompanyRepository, LawRepository (domain interfaces)  
  `verification`: tests pass  
  **Implementation**: The compliance service checks if the company has at least one practice matching any of the law's keyRequirements (from AdequacyDecision). For v1, requirements are plain strings; structured matching to come in v2.
- [ ] **INT‑002‑2** – `tests/integration/cross_context/compliance_check.test.ts` – Test with stubs  
  `verification`: pass

---

### `INT‑003` Technology–Company association (optional v1) | Status: `backlog`  
`depends_on`: decision deferred (can be added later)

*This task is intentionally left as a placeholder; the integration between Technology and Company is out of scope for this iteration. If required, it should be modeled as a separate bounded context with its own aggregate (e.g., `CompanyTechnologyUsage`).*

---

### `INT‑004` Technology–Law compliance mapping (optional v1) | Status: `backlog`  
`depends_on`: decision deferred

*Out of scope for the initial release; similar placeholder as INT‑003.*

---

### `WEB‑008` Create navigation component | Status: `backlog`  
`depends_on`: `WEB‑006`

- [ ] **Parent** `WEB‑008` – Build global navigation component for site.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/unit/web/components/navigation.test.ts |
| **Deep module** | N/A – navigation component |
| **Definition of done** | Navigation component created and integrated into all pages |
| **Out of scope** | Dynamic navigation items |
| **Rules** | Navigation links to all main pages; accessible; responsive |

#### Subtasks for WEB‑008

- [ ] **WEB‑008‑1** – `src/web/components/navigation.astro` – Global header/footer component  
  `imports_from`: none  
  `verification`: page renders a navigation bar linking to all main pages
- [ ] **WEB‑008‑2** – Integrate the navigation into all pages created in `WEB‑006` and `WEB‑007`  
  `imports_from`: none  
  `verification`: `pnpm build` succeeds and navigation links work

---

## Meta‑conventions recap

- Every subtask has a `verification` command; if verification fails, halt and fix before proceeding.
- `blocked_by` is used only when a real‑world decision is pending; currently no tasks are blocked.
- `depends_on` exactly mirrors the phase‑based ordering; Phase 5 tasks may have cross‑phase dependencies.
- `blocked_by` is reserved for external decisions; tasks that are internally blocked by other tasks should use `depends_on`. Ensure all tasks follow this guideline.
- Depth refactoring is repeated as the final subtask inside every module; there is no separate global REF‑001.

This `TODO.md` is now fully executable by an AI agent following the stated conventions.
