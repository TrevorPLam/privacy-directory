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

**Node version requirement**: Project requires Node.js >= 22.12.0+; include `.nvmrc` with version.

### `ARCH‑001A` Project scaffolding | Status: `backlog`  
`depends_on`: none

- [ ] **Parent** `ARCH‑001A` – Initialize the Astro 6 project structure and core configuration.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Project Architecture (enabling subdomain) `Aggregate root:` N/A (project scaffolding) `Ubiquitous language:` Domain, Infrastructure, Application, Web, Shared |
| **BDD** | N/A – infrastructure setup |
| **TDD** | `Test file(s):` tests/integration/project_structure.test.ts `Red‑Green‑Refactor cycle:` 1) verify project directory does not exist (red), 2) create directory structure (green), 3) verify dependency direction with `dependency‑cruiser` (green) |
| **Deep module** | N/A – configuration task |
| **Definition of done** | Astro 6 project created, domain‑driven directory structure in place, `dependency‑cruiser` validates inward‑only dependencies |
| **Out of scope** | CI/CD, deployment, dev environment setup |
| **Rules** | Domain never imports infrastructure; configuration externalized; `neverthrow` added to `package.json` |
| **Advanced patterns** | Dependency Injection setup, Module pattern |
| **Anti‑patterns** | Circular dependencies, scattered config |

#### Subtasks for ARCH‑001A

- [ ] **ARCH-001A-1** - `package.json` - Create project dependencies (include `neverthrow`, `vitest`, `@cloudflare/vitest-pool-workers`, `@cucumber/cucumber`, `dependency-cruiser`, `react`, `react-dom`, `@astrojs/react`, `@types/react`, `@testing-library/react`, `@testing-library/jest-dom`, `jsdom`, `vi-axe`, `@axe-core/react`, `wrangler`)
  `imports_from`: none
  `verification`: `pnpm install && grep '"test:bdd"' package.json`
  Also add `"test:bdd": "cucumber-js --config cucumber.config.ts"` to the `scripts` section of `package.json`.
  Note: For v1 static output, `@astrojs/cloudflare` and `@cloudflare/workers-types` are not required. Keep `wrangler` as dev dependency for KV emulator in tests.
- [ ] **ARCH-001A-2** - `astro.config.mjs` - Configure Astro 6 with `@astrojs/react` integration. Use `import.meta.env.SITE` instead of deprecated `Astro.site` API.
  `imports_from`: none
  `verification`: `pnpm build` produces Astro output and grep finds no `Astro.site` usage
  Security: Configure Content Security Policy using Astro 6's stable CSP API (security.csp in astro.config.mjs). Default policy: default-src 'self', frame-ancestors 'none'. Do not set script-src or style-src – let Astro manage them automatically with hashes/nonces. No third‑party tracking scripts permitted.
  Note: for v1, output: 'static' is recommended. Hybrid mode (output: 'hybrid') is available for v2 when SSR pages with KV/D1 bindings are needed.
- [ ] **ARCH‑001A‑3** – `src/` – Create domain‑driven directory structure (domain/, application/, infrastructure/, web/, shared/) with barrel `index.ts` files per directory for clean imports
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit && pnpm astro check`
- [ ] **ARCH‑001A‑4** – `tests/integration/project_structure.test.ts` – Create structure tests: validate layer isolation (domain doesn't import infra)  
  `imports_from`: `vitest`  
  `verification`: `pnpm test -- project_structure` passes
- [ ] **ARCH-001A-5** - `.nvmrc` - Create Node version file specifying 22.12.0+  
  `imports_from`: none  
  `verification`: `test -f .nvmrc && grep -E "^22\." .nvmrc`

---

### `ARCH‑001B` Shared kernel & enforcement | Status: `backlog`  
`depends_on`: `ARCH‑001A`

- [ ] **Parent** `ARCH‑001B` – Establish shared types, error handling, and dependency enforcement.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Infrastructure (shared kernel) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/unit/shared/result.test.ts `Red‑Green‑Refactor cycle:` 1) test Result type creation, 2) test error propagation |
| **Deep module** | N/A – shared infrastructure |
| **Definition of done** | Shared Result type, error types defined, dependency-cruiser enforces inward-only dependencies, wrangler.toml configured, Cloudflare _headers in place |
| **Out of scope** | Production Cloudflare Queues (deferred to v2) |
| **Rules** | All domain functions return Result<T, E>; dependency-cruiser prevents circular imports |
| **Advanced patterns** | Result pattern, Dependency enforcement |
| **Anti‑patterns** | Direct error throwing, circular dependencies |

#### Subtasks for ARCH‑001B

- [ ] **ARCH‑001B‑1** – `src/shared/result.ts` – Re‑export `Result`, `Ok`, `Err` from `neverthrow`  
  `imports_from`: `neverthrow`  
  `verification`: `pnpm tsc --noEmit` compiles
  Note: neverthrow is class‑based; Result objects lose their prototype chain when passed through structuredClone, postMessage, or Worker boundaries (relevant for Cloudflare Workers SSR). For the v1 static site this is not an issue. If v2 adds SSR API routes, consider switching to verdict‑ts (491 B gzipped, plain‑object Result, survives serialization) or ensure results are unwrapped before serialization.
- [ ] **ARCH‑001B‑2** – `src/shared/errors.ts` – Define shared error types: `ValidationError`, `NotFoundError`, `IntegrationError`, `ComplianceError`  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit` compiles
- [ ] **ARCH‑001B‑3** – `dependency-cruiser.cjs` – Configure import direction enforcement
  `imports_from`: none
  `verification`: `pnpm exec depcruise --validate` exits 0
  Note: Be aware of @astroscope/airlock for future v2 SSR API routes that may need to bypass dependency rules for API route isolation
- [ ] **ARCH‑001B‑4** – `public/_headers` – Cloudflare security headers
  `imports_from`: none
  `verification`: `test -f public/_headers && pnpm astro check`
  Content:
  ```
  /*
    X-Frame-Options: DENY
    X-Content-Type-Options: nosniff
    Referrer-Policy: strict-origin-when-cross-origin
    Permissions-Policy: camera=(), microphone=(), geolocation=()
  ```
- [ ] **ARCH‑001B‑5** – `wrangler.toml` - Configure Cloudflare Workers bindings (environment variables) for v2 SSR
  `imports_from`: none
  `verification`: `test -f wrangler.toml && pnpm tsc --noEmit`
  Note: For v1 static output, wrangler.toml is optional - Cloudflare Pages auto-detects dist/. KV bindings deferred to v2. Do not require kv_namespaces in v1.

---

### `ARCH‑002` Configure testing infrastructure | Status: `backlog`  
`depends_on`: `ARCH‑001B`

- [ ] **Parent** `ARCH‑002` – Set up comprehensive testing framework for TDD and BDD.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Testing Infrastructure (enabling subdomain) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/setup/test_config.test.ts `Red‑Green‑Refactor cycle:` 1) shell assertion: `pnpm test` fails before config (red), 2) configure vitest & cucumber (green), 3) verify `pnpm test` runs empty suites successfully |
| **Deep module** | N/A |
| **Definition of done** | `pnpm test` executes unit tests, `pnpm test:bdd` parses and runs `.feature` files, coverage thresholds enforced (≥90% domain, ≥70% infrastructure), a placeholder feature file exists |
| **Out of scope** | CI/CD, deployment, dev environment setup, E2E testing (deferred to v2) |
| **Rules** | E2E testing with Playwright deferred to v2; v1 focuses on unit and integration tests only |
| **Rules** | Unit tests isolated; BDD human‑readable; coverage thresholds in vitest.config.ts |
| **Advanced patterns** | Test doubles, Page object |
| **Anti‑patterns** | Test dependencies, missing cleanups |

#### Subtasks for ARCH‑002

- [ ] **ARCH-002-1** – Vitest configuration (split unit/integration projects)
  `imports_from`: `vitest/config`, `@cloudflare/vitest-pool-workers`
  `verification`:
  - `test -f vitest.unit.config.ts`
  - `test -f vitest.integration.config.ts`
  - `pnpm test -- --list` shows both project suites
  **Implementation details**:
  - `vitest.unit.config.ts`: environment `jsdom`, glob `tests/unit/**/*.test.ts`, coverage thresholds, setupFiles [`./tests/setup/vitest.setup.ts`, `vi-axe/extend-expect`]
  - `vitest.integration.config.ts`: uses `@cloudflare/vitest-pool-workers` (workerd runtime), glob `tests/integration/**/*.test.ts`, lower coverage thresholds
  - `vitest.config.ts`: root file containing `projects: ['./vitest.unit.config.ts', './vitest.integration.config.ts']`
- [ ] **ARCH‑002‑2** – `cucumber.config.ts` – Configure BDD framework with glob `features/**/*.feature`, require: ['tests/**/*.steps.ts']  
  `imports_from`: `@cucumber/cucumber`  
  `verification`: `pnpm test:bdd` runs and parses the placeholder feature file (see ARCH‑002‑3)
- [ ] **ARCH‑002‑3** – `tests/setup/` + `features/example.feature` + `tests/setup/vitest.setup.ts` – Create test setup, mocks, vitest setup with jest-dom matchers, and a minimal `Example` feature so BDD runner has something to parse
  `imports_from`: none
  `verification`: `pnpm test:bdd` parses `example.feature` successfully and passes QABAGE checklist
- [ ] **ARCH‑002‑3‑1** – `docs/qabage-checklist.md` – QABAGE checklist with 7 yes/no questions for BDD scenario quality
  `imports_from`: none
  `verification`: `test -f docs/qabage-checklist.md && pnpm tsc --noEmit`
- [ ] **ARCH‑002‑4** – `tests/setup/test_config.test.ts` – Verify test runner configuration
  `imports_from`: `vitest`
  `verification`: `pnpm test -- test_config` passes
- [ ] **ARCH-002-5** – `package.json` – Add `test:bdd` script
  `verification`: `grep '"test:bdd"' package.json` returns a valid script
  Script: `"test:bdd": "cucumber-js --config cucumber.config.ts"`

---

### `ARCH-007-company` Create Company Cucumber step definitions | Status: `backlog`  
`depends_on`: `ARCH-002`, `COMP-001-7`

- [ ] **Parent** `ARCH-007-company` - Scaffold step definition file for Company BDD scenarios.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Testing Infrastructure `Aggregate root:` N/A |
| **BDD** | Company `.feature` file has corresponding step definitions |
| **TDD** | `Test file(s):` tests/integration/company/step_definitions.test.ts |
| **Deep module** | N/A - scaffolding |
| **Definition of done** | Company step definition file exists, BDD runner finds all company steps |
| **Out of scope** | Full step implementations |
| **Rules** | Steps map to empty functions initially |
| **Advanced patterns** | Gherkin step organization |
| **Anti-patterns** | Missing step file |

#### Subtasks for ARCH-007-company

- [ ] **ARCH-007-company-1** - Create `tests/company/company.steps.ts` with minimal step functions for company-evaluation.feature  
  `imports_from`: none  
  `verification`: `pnpm test:bdd --dry-run features/company/company-evaluation.feature` lists all steps as defined/pending

---

### `ARCH-007-broker` Create Data Broker Cucumber step definitions | Status: `backlog`  
`depends_on`: `ARCH-002`, `BROK-001-6`

- [ ] **Parent** `ARCH-007-broker` - Scaffold step definition file for Data Broker BDD scenarios.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Testing Infrastructure `Aggregate root:` N/A |
| **BDD** | Broker `.feature` file has corresponding step definitions |
| **TDD** | `Test file(s):` tests/integration/broker/step_definitions.test.ts |
| **Deep module** | N/A - scaffolding |
| **Definition of done** | Broker step definition file exists, BDD runner finds all broker steps |
| **Out of scope** | Full step implementations |
| **Rules** | Steps map to empty functions initially |
| **Advanced patterns** | Gherkin step organization |
| **Anti-patterns** | Missing step file |

#### Subtasks for ARCH-007-broker

- [ ] **ARCH-007-broker-1** - Create `tests/broker/broker.steps.ts` with minimal step functions for broker-management.feature  
  `imports_from`: none  
  `verification`: `pnpm test:bdd --dry-run features/broker/broker-management.feature` lists all steps as defined/pending

---

### `ARCH-007-tech` Create Technology Cucumber step definitions | Status: `backlog`  
`depends_on`: `ARCH-002`, `TECH-001-7`

- [ ] **Parent** `ARCH-007-tech` - Scaffold step definition file for Technology BDD scenarios.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Testing Infrastructure `Aggregate root:` N/A |
| **BDD** | Technology `.feature` file has corresponding step definitions |
| **TDD** | `Test file(s):` tests/integration/tech/step_definitions.test.ts |
| **Deep module** | N/A - scaffolding |
| **Definition of done** | Technology step definition file exists, BDD runner finds all tech steps |
| **Out of scope** | Full step implementations |
| **Rules** | Steps map to empty functions initially |
| **Advanced patterns** | Gherkin step organization |
| **Anti-patterns** | Missing step file |

#### Subtasks for ARCH-007-tech

- [ ] **ARCH-007-tech-1** - Create `tests/tech/tech.steps.ts` with minimal step functions for technology-categorization.feature  
  `imports_from`: none  
  `verification`: `pnpm test:bdd --dry-run features/tech/technology-categorization.feature` lists all steps as defined/pending

---

### `ARCH-007-violation` Create Violation Cucumber step definitions | Status: `backlog`  
`depends_on`: `ARCH-002`, `VIOL-001-7`

- [ ] **Parent** `ARCH-007-violation` - Scaffold step definition file for Violation BDD scenarios.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Testing Infrastructure `Aggregate root:` N/A |
| **BDD** | Violation `.feature` file has corresponding step definitions |
| **TDD** | `Test file(s):` tests/integration/violation/step_definitions.test.ts |
| **Deep module** | N/A - scaffolding |
| **Definition of done** | Violation step definition file exists, BDD runner finds all violation steps |
| **Out of scope** | Full step implementations |
| **Rules** | Steps map to empty functions initially |
| **Advanced patterns** | Gherkin step organization |
| **Anti-patterns** | Missing step file |

#### Subtasks for ARCH-007-violation

- [ ] **ARCH-007-violation-1** - Create `tests/violation/violation.steps.ts` with minimal step functions for violation-tracking.feature  
  `imports_from`: none  
  `verification`: `pnpm test:bdd --dry-run features/violation/violation-tracking.feature` lists all steps as defined/pending

---

### `ARCH-007-law` Create Law Cucumber step definitions | Status: `backlog`  
`depends_on`: `ARCH-002`, `LAW-001-7`

- [ ] **Parent** `ARCH-007-law` - Scaffold step definition file for Law BDD scenarios.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Testing Infrastructure `Aggregate root:` N/A |
| **BDD** | Law `.feature` file has corresponding step definitions |
| **TDD** | `Test file(s):` tests/integration/law/step_definitions.test.ts |
| **Deep module** | N/A - scaffolding |
| **Definition of done** | Law step definition file exists, BDD runner finds all law steps |
| **Out of scope** | Full step implementations |
| **Rules** | Steps map to empty functions initially |
| **Advanced patterns** | Gherkin step organization |
| **Anti-patterns** | Missing step file |

#### Subtasks for ARCH-007-law

- [ ] **ARCH-007-law-1** - Create `tests/law/law.steps.ts` with minimal step functions for law-management.feature  
  `imports_from`: none  
  `verification`: `pnpm test:bdd --dry-run features/law/law-management.feature` lists all steps as defined/pending

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

- [ ] **ARCH‑003‑1** – `docs/context-map.md` – Create Context Map: one section per context pair (Company Evaluation, Data Broker Management, Technology Evaluation, Violation Tracking, Legal Compliance, Web Adapter). Each context pair must explicitly name the integration pattern (ACL, Shared Kernel, Open Host, etc.).
  `imports_from`: none
  `verification`: `grep -c "Context Pair:" docs/context-map.md` equals 15, each section names an integration pattern (ACL/Shared Kernel/Open Host/etc.)
  **Context pairs to enumerate**: Company↔Legal, Company↔Violation, Company↔Broker, Company↔Technology, Company↔Web, Legal↔Violation, Legal↔Broker, Legal↔Technology, Legal↔Web, Violation↔Broker, Violation↔Technology, Violation↔Web, Broker↔Technology, Broker↔Web, Technology↔Web
- [ ] **ARCH-003-2** - `docs/ubiquitous-language.md` - Collect all terms from task tables, define them canonically  
  `imports_from`: none  
  `verification`: file contains at least one entry per domain aggregate and value object  
  **Privacy Score Categories**: The Privacy Fairness Index (PFI) uses 6 categories with the following weights:
  - Data Collection Scope: 25%
  - Third‑Party Sharing: 20%
  - Retention & Deletion: 20%
  - User Control & Consent: 15%
  - Transparency & Access: 10%
  - Security & Breach Notification: 10%
  Total = 100%

  Our 4‑category scoring system re‑maps these into aggregated dimensions, then
  re‑scales and adds an external Legal Compliance History dimension not measured by PFI:
  - Data Collection & Sharing: 0.40  (merges Data Collection 25% + Third‑Party Sharing 20% → 45% of PFI base)
  - Transparency & User Control: 0.35 (merges User Control & Consent 15% + Transparency & Access 10% → 25% of PFI base)
  - Retention & Security: 0.15 (merges Retention & Deletion 20% + Security & Breach Notification 10% → 30% of PFI base)
  - Legal Compliance History: 0.10 (external regulatory record — not derived from PFI)
  These four weights sum to 1.0. They are a separate methodological choice, not a
  direct arithmetic scaling of PFI.  
  **Monitoring item**: Add SECURE Data Act (discussion draft released April 22, 2026) to monitoring list - federal bill that would preempt state privacy laws if passed

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
| **Out of scope** | Production Cloudflare Queues, persistence of events  |
| **Rules** | Events are plain objects with `type` and `payload`; subscribers registered at infrastructure wire‑up (not inside domain) |
| **Limitations** | For v1 static output, the in-memory EventBus limitation has zero runtime impact (all data is pre-built). For v2 SSR, note that Cloudflare KV is eventually consistent and the in-memory EventBus does not survive worker restarts, meaning cross-request event handling will be degraded without Cloudflare Queues. |
| **Advanced patterns** | Observer pattern |
| **Anti‑patterns** | Domain directly depending on event bus implementation |

#### Subtasks for ARCH‑004

- [ ] **ARCH‑004‑1** – `src/shared/event-bus.ts` – Define `EventBus` interface and `InMemoryEventBus` class
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit`
  Additionally, define a discriminated union type `DomainEvent` aggregating all event payloads (CompanyEvaluated, BrokerRegistered, etc.) for type safety.
- [ ] **ARCH‑004‑2** – `tests/unit/shared/event_bus.test.ts` – Tests for publish/subscribe  
  `imports_from`: `EventBus`  
  `verification`: tests pass
- [ ] **ARCH‑004‑3** – `docs/event-subscriptions.md` – Document where subscriptions are wired (composition root)  
  `imports_from`: none  
  `verification`: `test -f docs/event-subscriptions.md && pnpm tsc --noEmit`

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

- [ ] **ARCH‑005‑1** – `docs/data-flow.md` – Create data flow diagram (Mermaid) showing query and command flows. Include error mapping strategy section documenting how domain errors (ValidationError, NotFoundError, etc.) map to HTTP responses and UI states.
  `imports_from`: none
  `verification`: `test -f docs/data-flow.md && pnpm tsc --noEmit`
  **Scope**: Diagram must show: query flow (repository → aggregate → DTO → web), command flow (web → application service → repository → event), event‑driven cross‑context flow, and error mapping strategy (domain → HTTP/UI).
- [ ] **ARCH‑005‑2** – `src/shared/errors.ts` (finalize) – Ensure all error types used across tasks are defined  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit` compiles with imports from `errors.ts`

---

### `ARCH‑006` Create navigation skeleton | Status: `backlog`
`depends_on`: `ARCH‑001`

- [ ] **Parent** `ARCH‑006` – Create global navigation component for site.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | N/A |
| **Deep module** | N/A – navigation component |
| **Definition of done** | Navigation component created with complete links to all main pages; integrated into BaseLayout |
| **Out of scope** | Dynamic navigation items |
| **Rules** | Navigation links to all main pages; accessible; responsive |
| **Advanced patterns** | Semantic HTML navigation |
| **Anti‑patterns** | Missing navigation, broken links |

#### Subtasks for ARCH‑006

- [ ] **ARCH‑006‑1** – `src/web/components/navigation.astro` – Global header/footer component with complete links to all main pages (homepage, companies, technologies, data brokers, violations, legal guide, practices)
  `imports_from`: none
  `verification`: `pnpm astro check` compiles and navigation renders with all links
- [ ] **ARCH‑006‑2** – Document navigation structure in `docs/navigation-structure.md`
  `imports_from`: none
  `verification`: `test -f docs/navigation-structure.md && pnpm astro check`

---

### `ARCH‑009` SEO Foundation | Status: `backlog`
`depends_on`: `ARCH‑001A`

- [ ] **Parent** `ARCH‑009` – Establish SEO infrastructure for all pages.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/unit/web/utils/jsonld.test.ts |
| **Deep module** | N/A – infrastructure |
| **Definition of done** | Head component with SEO meta tags, JSON-LD helpers configured, sitemap integration, robots.txt in place |
| **Out of scope** | Dynamic SEO (deferred to v2) |
| **Rules** | All pages use Head component; JSON-LD type-safe; sitemap auto-generated |
| **Advanced patterns** | Structured data, Sitemap generation |
| **Anti‑patterns** | Missing meta tags, broken structured data |

#### Subtasks for ARCH‑009

- [ ] **ARCH‑009‑1** – `src/web/components/Head.astro` – SEO head component with title, description, OG tags, JSON-LD slot
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit` compiles
- [ ] **ARCH‑009‑2** – `src/web/utils/jsonld.ts` – Type-safe helpers for JSON-LD (Organization, WebSite, BreadcrumbList, FAQPage)
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit` compiles and unit tests pass
- [ ] **ARCH‑009‑3** – Configure `@astrojs/sitemap` integration in `astro.config.mjs`
  `imports_from`: none
  `verification`: `pnpm build` generates sitemap.xml
- [ ] **ARCH‑009‑4** – `public/robots.txt` – Search engine crawler directives
  `imports_from`: none
  `verification`: `test -f public/robots.txt && pnpm astro check`

---

### `ARCH‑010` Privacy-First Font Strategy | Status: `backlog`
`depends_on`: `ARCH‑001A`

- [ ] **Parent** `ARCH‑010` – Configure privacy-first font loading strategy.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | N/A |
| **Deep module** | N/A – configuration |
| **Definition of done** | Astro 6 Fonts API configured with local font provider, font families applied via CSS custom properties |
| **Out of scope** | Web fonts from external CDNs (privacy concern) |
| **Rules** | Use local font provider (e.g., Fontsource); no external font requests; font-display: swap |
| **Advanced patterns** | Local font loading, CSS custom properties |
| **Anti‑patterns** | External font requests, font flash |

#### Subtasks for ARCH‑010

- [ ] **ARCH‑010‑1** – Enable Astro 6 Fonts API with local font provider (e.g., Fontsource) in `astro.config.mjs`
  `imports_from`: none
  `verification`: `pnpm build` succeeds and fonts are bundled locally
- [ ] **ARCH‑010‑2** – Apply font family in `BaseLayout.astro` via CSS custom properties
  `imports_from`: none
  `verification`: `pnpm build` succeeds and CSS custom properties are defined

---

### `ARCH‑012` Static page scaffold | Status: `backlog`
`depends_on`: `ARCH‑001A`

- [ ] **Parent** `ARCH‑012` – Create static page scaffold for incremental page development.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | N/A |
| **Deep module** | N/A – scaffolding |
| **Definition of done** | Static page scaffold with `<html>`, `<head>`, `<body>` and Astro `<slot />` for incremental page development |
| **Out of scope** | Dynamic content |
| **Rules** | Minimal scaffold; all pages extend this |
| **Advanced patterns** | Page template pattern |
| **Anti‑patterns** | Missing scaffold causing duplication |

#### Subtasks for ARCH-012

- [ ] **ARCH‑012‑1** – `src/layouts/PageLayout.astro` – Static page scaffold with `<html>`, `<head>`, `<body>` and Astro `<slot />` for incremental page development
  `imports_from`: none
  `verification`: `pnpm astro check` compiles successfully

---

### `ARCH‑013` Architectural Guardrails Check | Status: `backlog`
`depends_on`: `ARCH‑001B`, `ARCH‑002`

- [ ] **Parent** `ARCH‑013` – Create automated architectural guardrails to validate dependency direction and error handling patterns.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Infrastructure (shared kernel) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | N/A |
| **Deep module** | N/A – validation task |
| **Definition of done** | Dependency-cruiser validates inward-only dependencies; script checks Result usage; script checks no direct seed.json reads from domain/application layers |
| **Out of scope** | CI/CD integration (deferred to v2) |
| **Rules** | Domain never imports infrastructure; all domain functions return Result types; seed data only read in infrastructure layer |
| **Advanced patterns** | Architectural validation |
| **Anti‑patterns** | Circular dependencies, thrown errors in domain |

#### Subtasks for ARCH-013

- [ ] **ARCH‑013‑1** – `dependency-cruiser.cjs` – Ensure configuration validates that no domain file imports from infrastructure
  `imports_from`: none
  `verification`: `pnpm exec depcruise --validate` exits 0 with no violations
- [ ] **ARCH‑013‑2** – `scripts/check-result-usage.sh` – Script that checks all Result usage (grep for thrown errors in domain layer)
  `imports_from`: none
  `verification`: script exists and runs without finding thrown errors in domain layer
- [ ] **ARCH‑013‑3** – `scripts/check-seed-data-access.sh` – Script that checks that no file directly reads `seed.json` from the domain or application layers
  `imports_from`: none
  `verification`: script exists and runs without finding direct seed.json reads in domain/application layers

---

### `ARCH‑008` Configure accessibility testing | Status: `backlog`
`depends_on`: `ARCH‑002`

- [ ] **Parent** `ARCH‑008` – Verify accessibility testing infrastructure is properly configured.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Testing Infrastructure (enabling) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/setup/accessibility.test.ts |
| **Deep module** | N/A – configuration verification |
| **Definition of done** | `vi-axe` integration verified; `@axe-core/react` added for component-level checks; accessibility test passes |
| **Out of scope** | Full accessibility audit (deferred to dedicated audit task) |
| **Rules** | Accessibility testing integrated into test suite |
| **Advanced patterns** | Automated accessibility checks |
| **Anti‑patterns** | Accessibility testing not verified |

#### Subtasks for ARCH‑008

- [ ] **ARCH‑008‑1** – Verify `vi-axe` is properly configured in vitest setup
  `imports_from`: `vi-axe`
  `verification`: `pnpm test -- accessibility` passes with axe matcher available
- [ ] **ARCH‑008‑2** – Add `@axe-core/react` to dependencies for component-level accessibility checks
  `imports_from`: none
  `verification`: `pnpm install` succeeds and package is in package.json
- [ ] **ARCH‑008‑3** – `tests/setup/accessibility.test.ts` – Create baseline accessibility test
  `imports_from`: `@axe-core/react`
  `verification`: test passes with basic accessibility check

---

## Phase 1 – Domain Models

All of these can be developed in parallel after Phase 0.

### `COMP‑001` Define Company domain model | Status: `backlog`  
`depends_on`: `ARCH‑001`

- [ ] **Parent** `COMP‑001` – Establish core domain entities for privacy‑focused companies.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Company Evaluation `Aggregate root:` Company `Ubiquitous language:` Company, PrivacyScore, Practice, Category, CompanyEvaluated (event) **Repository interface location:** Domain layer (`src/companies/domain/company_repository.ts`) per DDD convention; infrastructure implements this interface |
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
  Note: Practice is a value object defined inline in `src/companies/domain/company.ts` (fields: id, name, category, weight, sourceUrl). No separate subtask required.
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
  The Privacy Fairness Index (PFI) uses 6 categories with the following weights:
  - Data Collection Scope: 25%
  - Third‑Party Sharing: 20%
  - Retention & Deletion: 20%
  - User Control & Consent: 15%
  - Transparency & Access: 10%
  - Security & Breach Notification: 10%
  Total = 100%

  Our 4‑category scoring system re‑maps these into aggregated dimensions, then
  re‑scales and adds an external Legal Compliance History dimension not measured by PFI:
  - Data Collection & Sharing: 0.40  (merges Data Collection 25% + Third‑Party Sharing 20% → 45% of PFI base)
  - Transparency & User Control: 0.35 (merges User Control & Consent 15% + Transparency & Access 10% → 25% of PFI base)
  - Retention & Security: 0.15 (merges Retention & Deletion 20% + Security & Breach Notification 10% → 30% of PFI base)
  - Legal Compliance History: 0.10 (external regulatory record — not derived from PFI)
  These four weights sum to 1.0. They are a separate methodological choice, not a
  direct arithmetic scaling of PFI.
  
  **Exact algorithm formula:**
  Score = sum over categories (category_weight × category_score)
  where category_score = (sum of practice weights in category) / (max possible sum in category) × 100
  practice weights are defined in seed data (0-100). If no practices in a category, category_score = 0.
  The max possible sum in a category is the sum of all practice weights if every practice were fully implemented (weight = 100).
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
- [ ] **BROK‑001‑3** – `src/brokers/domain/compliance_status.ts` – ComplianceStatus value object tracking broker compliance state  
  `imports_from`: `Result`, `ValidationError`  
  `verification`: `pnpm tsc --noEmit`
- [ ] **BROK‑001‑4** – `src/brokers/domain/broker-events.ts` – Domain events including `BrokerRegistered` event  
  `imports_from`: none  
  `verification`: `pnpm tsc --noEmit`
- [ ] **BROK‑001‑5** – `tests/unit/broker/domain/data_broker.test.ts` – Unit tests  
  `imports_from`: DataBroker, etc.  
  `verification`: pass, coverage ≥90%
- [ ] **BROK‑001‑6** – `features/broker/broker-management.feature` – BDD scenarios  
  `verification`: `pnpm test:bdd` passes
- [ ] **BROK‑001‑7** – `tests/unit/broker/domain/compliance_status.test.ts` – Unit tests for ComplianceStatus value object  
  `imports_from`: ComplianceStatus  
  `verification`: pass, coverage ≥90%
- [ ] **BROK‑001‑8** – Depth refactor check  
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
  **Allowed values (OECD PET taxonomy, 2026):**
  - DATA_OBFUSCATION (e.g., differential privacy, noise addition)
  - ENCRYPTED_PROCESSING (e.g., homomorphic encryption, confidential computing)
  - FEDERATED_ANALYTICS (e.g., federated learning, split learning)
  - DATA_ACCOUNTABILITY (e.g., privacy dashboards, consent mechanisms)
  - PRIVACY_INFRASTRUCTURE (e.g., identity management, anonymization)
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
- [ ] **TECH‑001‑9** – `src/tech/domain/maturity_specification.ts` – Define maturity specification with clear rules for assigning maturity levels
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit`
  **Maturity criteria rules:**
  - EXPERIMENTAL: Concept or proof-of-concept only; no peer-reviewed sources; no implementation references; zero adoption evidence
  - EMERGING: At least one peer-reviewed source or academic paper; has implementation reference (GitHub repo, prototype); limited adoption (< 10 organizations)
  - ESTABLISHED: Multiple peer-reviewed sources (3+); has production implementation references; measurable adoption (10-100 organizations); documented use cases
  - MATURE: Extensive peer-reviewed literature (5+); multiple production implementations; widespread adoption (100+ organizations); industry standard or W3C/IETF specification; comprehensive documentation and tooling

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
- [ ] **VIOL‑001‑4** – `src/violation/domain/company_reference.ts` – CompanyReference value object (companyId: string, companyName: string, scoreAtViolation: number — populated by CompanyEvaluated event handler; see INT‑001). The `scoreAtViolation` field captures the company's privacy score at the time of the violation for historical accuracy.
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

- [ ] **LAW‑001‑1** – `src/law/domain/privacy_law.ts` – PrivacyLaw aggregate root with `lastVerifiedDate` field to track when legal data was last verified for accuracy
  `imports_from`: Jurisdiction, AdequacyStatus, AdequacyDecision
  `verification`: `pnpm tsc --noEmit`
  **Legal data freshness note**: Legal data has a cut-off date due to its static nature. The site should display a visible banner on legal pages like "Legal information last verified: [date]. This is not legal advice." The date should be populated from the `lastVerifiedDate` field or from `src/data/seed.json`.
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
  **EU-Brazil adequacy**: Include Brazil (BR) with decisionDate: "2026-01-27", sourceUrl: EC press release, full public+private scope (670M combined consumers)
- [ ] **LAW‑001‑5** – `src/law/domain/law-events.ts` – LawUpdated event  
  `verification`: `pnpm tsc --noEmit`
- [ ] **LAW‑001‑6** – `tests/unit/law/domain/privacy_law.test.ts` – Unit tests  
  `verification`: pass
- [ ] **LAW‑001‑7** – `features/law/law-management.feature` – BDD scenarios  
  `verification`: `pnpm test:bdd` passes
- [ ] **LAW‑001‑8** – Depth refactor check  
  `verification`: `pnpm lint && pnpm tsc --noEmit && pnpm test -- coverage`

---

### `LAW‑002` Implement Law repository | Status: `backlog`
`depends_on`: `LAW‑001`, `DATA‑001`

- [ ] **Parent** `LAW‑002` – Create persistence layer for PrivacyLaw aggregate.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Legal Compliance `Aggregate root:` N/A (infrastructure service implements LawRepository domain interface) |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/unit/law/infrastructure/memory_law_repository.test.ts `Red‑Green‑Refactor cycle:` 1) save and retrieve full aggregate, 2) find by jurisdiction |
| **Deep module** | `Public interface:` LawRepository.save(), findById(), findByJurisdiction() `Hidden complexity:** Mapping primitives to value objects `Depth metric:` moderate – 3 methods, ~80 lines of mapping |
| **Definition of done** | In‑memory repo loads from seed.json, mapping layer converts stored primitives to domain objects, tests pass |
| **Out of scope** | Database migrations |
| **Rules** | Repository returns entire aggregates; mapping inside infrastructure; explicit data mapping: primitives → Value Objects |
| **Advanced patterns** | Repository pattern |
| **Anti‑patterns** | Active Record, partial persistence |

#### Subtasks for LAW-002

- [ ] **LAW‑002‑1** – `src/law/infrastructure/law_repository.ts` – Repository interface
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit`
- [ ] **LAW‑002‑2** – `src/law/infrastructure/memory_law_repository.ts` – In‑memory implementation that loads from `src/data/seed.json`
  `imports_from`: LawRepository, PrivacyLaw
  `verification`: unit tests pass and repository loads seed data correctly
- [ ] **LAW‑002‑3** – `tests/unit/law/infrastructure/memory_law_repository.test.ts` – Unit tests
  `verification`: pass

---

### `SOFT‑001` Define Software/Applications domain model | Status: `backlog`
`depends_on`: `ARCH‑001`

- [ ] **Parent** `SOFT‑001` – Establish domain entities for consumer-facing software applications with privacy characteristics.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Software Evaluation `Aggregate root:` SoftwareApplication `Ubiquitous language:** SoftwareApplication, SoftwareCategory, Platform, SoftwareEvaluated (event) |
| **BDD** | `Feature:` Researcher categorizes software `Scenario 1:` Valid software → classified with category and platforms `Scenario 2:` Missing source → rejected `Scenario 3:** Update evaluation when evidence changes |
| **TDD** | `Test file(s):` tests/unit/software/domain/software_application.test.ts `Red‑Green‑Refactor cycle:` 1) create with category, 2) validate platforms, 3) reject missing source, 4) test event publication |
| **Deep module** | `Public interface:** classify(), getPlatforms(), getDataCollectionPractices() `Hidden complexity:** Validation, practice assessment (~120 lines) `Depth metric:** moderate – 3 methods, ~120 lines |
| **Definition of done** | Domain model, events, `.feature` file, tests |
| **Out of scope** | Real-time monitoring, automated research analysis |
| **Rules** | Software must cite verifiable sources; categories distinguish consumer software from enabling technologies (TECH-001) |
| **Advanced patterns** | Specification for categories, Domain Events |
| **Anti‑patterns** | Subjective classification, overlap with Technology domain |

#### Subtasks for SOFT‑001

- [ ] **SOFT‑001‑1** – `src/software/domain/software_application.ts` – SoftwareApplication aggregate root
  `imports_from`: SoftwareCategory, Platform
  `verification`: `pnpm tsc --noEmit`
- [ ] **SOFT‑001‑2** – `src/software/domain/software_category.ts` – SoftwareCategory value object
  `imports_from`: `Result`, `ValidationError`
  `verification`: `pnpm tsc --noEmit`
  **Allowed values**: BROWSER, MESSAGING, EMAIL, VPN, PASSWORD_MANAGER, CLOUD_STORAGE, VIDEO_CONFERENCING, SEARCH_ENGINE
- [ ] **SOFT‑001‑3** – `src/software/domain/platform.ts` – Platform value object
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit`
- [ ] **SOFT‑001‑4** – `src/software/domain/data_collection_practices.ts` – DataCollectionPractices value object with structure: collectionTypes (array of strings), purposes (array of strings), sharing (boolean), thirdParty (boolean), retentionPeriod (string), sourceUrl (string), lastVerifiedDate (string)
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit`
- [ ] **SOFT‑001‑5** – `src/software/domain/software-events.ts` – SoftwareEvaluated event
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit`
- [ ] **SOFT‑001‑6** – `tests/unit/software/domain/software_application.test.ts` – Unit tests
  `verification`: pass
- [ ] **SOFT‑001‑7** – `features/software/software-evaluation.feature` – BDD scenarios
  `verification`: `pnpm test:bdd` passes
- [ ] **SOFT‑001‑8** – Depth refactor check
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
  (Note: Cloudflare KV is eventually consistent (up to 60s). Use `vi.waitFor` with timeout to retry assertions until the KV change propagates, e.g. `await vi.waitFor(() => expect(...), { timeout: 10_000 })`.)
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
- [ ] **BROK‑002‑3** – `src/brokers/infrastructure/csv_parser.ts` – Data‑broker registry adapter (handles hashed identifier lists from DROP; CSV parsing may be one possible format, but final format determined by CA DROP API). Add `src/brokers/infrastructure/json_parser.ts` stub with same interface as fallback if API returns JSON.
  `verification`: `pnpm tsc --noEmit` and both CSV and JSON parser interfaces match
  **⚠️ Provisional**: This parser implementation is provisional and must be revisited once the CA DROP API specification is available (expected August 2026). The actual API may return JSON or a different identifier manifest format. JSON parser stub provided as fallback.
- [ ] **BROK‑002‑4** – `tests/integration/broker/infrastructure/drop_adapter.test.ts` – Integration tests (mocked HTTP)  
  `verification`: pass
- [ ] **BROK‑002‑5** – Depth refactor check  
  `verification`: ensure interface depth, CSV parser remains shallow

---

## Phase 2.5 – Seed Data & Content Strategy

### `DATA‑001` Create shared seed data for all stubs | Status: `backlog`
`depends_on`: `DATA‑002`

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
  `verification`: `test -f src/data/seed.json && pnpm tsc --noEmit`
- [ ] **DATA‑001‑2** – Populate each section with exact entries specified in content brief (DATA-002): 5 companies, 5 technologies, 5 devices, 5 software, 5 violations, 5 laws
  `imports_from`: none
  `verification`: each entry has `source_url` field pointing to primary source and counts match content brief
  **Legal data freshness note**: Legal data has a cut-off date due to its static nature. The site should display a visible banner on legal pages like "Legal information last verified: [date]. This is not legal advice." The date should be populated from `src/data/seed.json` or a constant.
- [ ] **DATA‑001‑3** – Apply neutral framing convention to all entities  
  `imports_from`: none  
  `verification`: grep finds no "good/bad" language in seed data
- [ ] **DATA-001-4** - Align seed data with updated domain models (new PrivacyScore weights, CA DROP details, 2026 law list)  
  `imports_from`: none  
  `verification`: `pnpm test` (all stubs load seed data correctly)  
  **Schema validation**: Use Zod from local import (add to package.json if not already present) to define schema for seed.json and add validation step
Complete 2026 US state law abbreviations:
CCPA/CPRA, VCDPA, CPA, CTDPA, UCPA, ICDPA, INCDPA, TIPA, TDPSA, FDBR, MODPA,
MCDPA (MN), MCDPA (MT), OCPA, DPDPA, NHDPA, NJDPA, KYCDPA, NDPA, RHDPA.
Oklahoma (ODPA) effective Jan 1, 2027 (not included in v1 seed).
- [ ] **DATA‑001‑5** – `docs/neutral-language-guide.md` – Create style guide for factual, sourced approach  
  `imports_from`: none  
  `verification`: `test -f docs/neutral-language-guide.md && pnpm tsc --noEmit`
- [ ] **DATA-001-6** – Add `devices` section to seed data with 5 exact entries:
  1. Punkt MC03 (minimalist privacy phone, launched Jan 2026)
  2. Purism Librem 5 (PureOS Crimson, Mar 2026)
  3. PinePhone Pro (Linux-based, hardware kill switches)
  4. Jolla Phone (Sailfish OS, European privacy focus)
  5. Hiroh smartphone (privacy-focused, 2026)
  `imports_from`: none  
  `verification`: `src/data/seed.json` contains these 5 entries with source_url fields

---

### `DATA‑002` Create content brief specifying exact seed data entries | Status: `backlog`
`depends_on`: `DATA‑001`

- [ ] **Parent** `DATA‑002` – Create explicit content brief to prevent AI execution ambiguity for seed data.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Data Management (enabling) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | N/A |
| **Deep module** | N/A – documentation task |
| **Definition of done** | `docs/content-brief.md` explicitly lists every entry for seed data with names, sources, and key attributes |
| **Out of scope** | Actual content creation (done in DATA-001) |
| **Rules** | Content brief specifies exact entities to include; prevents AI execution ambiguity |
| **Advanced patterns** | Content specification |
| **Anti‑patterns** | Ambiguous "3–5 entries" without specifics |

#### Subtasks for DATA‑002

- [ ] **DATA‑002‑1** – `docs/content-brief.md` – Create content brief with exact entries
  `imports_from`: none
  `verification`: `test -f docs/content-brief.md && pnpm tsc --noEmit`
  - Companies: 5 entries with names, practices, sources
  - Technologies: 5 entries (e.g., differential privacy, homomorphic encryption, federated learning, confidential computing, secure multi-party computation)
  - Devices: 5 entries (Punkt MC03, Purism Librem 5, PinePhone Pro, Jolla Phone, Hiroh smartphone)
  - Software: 5 entries (Signal, Brave Browser, Proton Mail, Mullvad VPN, Bitwarden)
  - Violations: 5 entries with dates, companies, severity, sources
  - Laws: 5 entries covering major jurisdictions
  - California DROP statistics: Include key statistics about the California DROP platform (e.g., number of registered data brokers, deletion request volume, compliance rate)

---

### `DATA‑003` Content Audit Checklist | Status: `backlog`
`depends_on`: `DATA‑002`

- [ ] **Parent** `DATA‑003` – Create neutral-framing audit checklist for content validation.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Data Management (enabling) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | N/A |
| **Deep module** | N/A – documentation and tooling |
| **Definition of done** | Neutral-framing checklist document created, automated grep script for banned adjectives in place |
| **Out of scope** | Automated content generation |
| **Rules** | All content must be neutral and factual; no subjective labels like "good", "bad", "ethical", "unethical" |
| **Advanced patterns** | Automated validation |
| **Anti‑patterns** | Subjective language, opinionated descriptions |

#### Subtasks for DATA‑003

- [ ] **DATA‑003‑1** – `docs/content-audit-checklist.md` – 5-point neutral-framing checklist for content validation
  `imports_from`: none
  `verification`: `test -f docs/content-audit-checklist.md && pnpm tsc --noEmit`
- [ ] **DATA‑003‑2** – `scripts/audit-neutral-language.sh` – Automated grep script for banned adjectives (good, bad, ethical, unethical, etc.)
  `imports_from`: none
  `verification`: script exists and runs without errors

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
  `imports_from`: CompanyQueryService, CompanyDTO, CompanyRepository
  `verification`: tests pass
- [ ] **APP‑001‑2** – `src/companies/application/company_dto.ts` – DTO definition  
  `verification`: `pnpm tsc --noEmit`
- [ ] **APP‑001‑3** – `tests/unit/company/application/company_query_service.test.ts` – Unit tests  
  `verification`: pass
- [ ] **APP‑001‑4** – `tests/unit/company/application/company_query_service_error.test.ts` – Error path tests
  `imports_from`: CompanyQueryService, IntegrationError, ValidationError
  `verification`: tests pass; covers repository throws → IntegrationError, invalid filter → ValidationError

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
  `imports_from`: BrokerQueryService, BrokerDTO, BrokerRepository
  `verification`: tests pass
- [ ] **APP‑002‑2** – `src/brokers/application/broker_dto.ts` – DTO  
  `verification`: `pnpm tsc --noEmit`
- [ ] **APP‑002‑3** – `tests/unit/broker/application/broker_query_service.test.ts` – Tests  
  `verification`: pass
- [ ] **APP‑002‑4** – `tests/unit/broker/application/broker_query_service_error.test.ts` – Error path tests
  `imports_from`: BrokerQueryService, IntegrationError, ValidationError
  `verification`: tests pass; covers repository throws → IntegrationError, invalid filter → ValidationError

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
- [ ] **APP‑003‑4** – `tests/unit/tech/application/technology_query_service_error.test.ts` – Error path tests
  `imports_from`: TechnologyQueryService, IntegrationError, ValidationError
  `verification`: tests pass; covers repository throws → IntegrationError, invalid filter → ValidationError

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
- [ ] **APP‑004‑4** – `tests/unit/violation/application/violation_query_service_error.test.ts` – Error path tests
  `imports_from`: ViolationQueryService, IntegrationError, ValidationError
  `verification`: tests pass; covers repository throws → IntegrationError, invalid filter → ValidationError

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
- [ ] **APP‑005‑4** – `tests/unit/law/application/legal_query_service_error.test.ts` – Error path tests
  `imports_from`: LegalQueryService, IntegrationError, ValidationError
  `verification`: tests pass; covers repository throws → IntegrationError, invalid filter → ValidationError

---

### `APP‑006` Software query service and DTOs | Status: `backlog`
`depends_on`: `SOFT‑001`, `DATA‑001`

- [ ] **Parent** `APP‑006` – Provide software application data.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Software Evaluation (application) `Aggregate root:` N/A `Ubiquitous language:** SoftwareDTO, category, platform |
| **BDD** | N/A – internal service |
| **TDD** | `Test file(s):` tests/unit/software/application/software_query_service.test.ts `Red‑Green‑Refactor cycle:** 1) return all software, 2) filter by category, 3) filter by platform |
| **Deep module** | `Public interface:** getSoftware(filter) → SoftwareDTO[] `Hidden complexity:** In‑memory stub, DTO mapping `Depth metric:** shallow‑moderate |
| **Definition of done** | Service with in‑memory stub using shared seed data, DTO definition, tests ≥70% coverage |
| **Out of scope** | Write operations, persistence via Cloudflare KV deferred to v2; v1 uses static seed data loaded from `src/data/seed.json`. Repository implementation will be retrofitted in v2; stub uses the same interface. |
| **Rules** | DTOs only; no domain objects leaked; stub uses shared seed data from `DATA‑001` |
| **Advanced patterns** | DTO mapping functions |
| **Anti‑patterns** | Returning domain objects directly |

#### Subtasks for APP‑006

- [ ] **APP‑006‑1** – `src/software/application/software_query_service.ts` – Service implementation (in‑memory stub)
  `imports_from`: SoftwareDTO
  `verification`: tests pass
- [ ] **APP‑006‑2** – `src/software/application/software_dto.ts` – DTO definition (id, name, category, platforms, dataCollectionPractices, encryptionDefault, openSource, sourceUrl)
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit`
- [ ] **APP‑006‑3** – `tests/unit/software/application/software_query_service.test.ts` – Unit tests
  `imports_from`: SoftwareQueryService, SoftwareDTO
  `verification`: pass, coverage ≥70%
- [ ] **APP‑006‑4** – `tests/unit/software/application/software_query_service_error.test.ts` – Error path tests
  `imports_from`: SoftwareQueryService, IntegrationError, ValidationError
  `verification`: tests pass; covers repository throws → IntegrationError, invalid filter → ValidationError

---

## Phase 4 – Web Components (Presentation)

All WEB tasks depend on corresponding APP tasks and consume DTOs. Feature files are placed in `features/<context>/`.

Hydration strategy: Static pages use Astro .astro files (zero JS shipped).
Interactive components (CompanyListing, BrokerDashboard, etc.) use React islands
with targeted hydration directives: client:visible for filter panels and listings,
client:idle for search bars and auxiliary widgets.

### `WEB‑012` Base Layout, Error Handling & 404 | Status: `backlog`
`depends_on`: `ARCH‑006`, `ARCH‑009`, `ARCH‑010`

- [ ] **Parent** `WEB‑012` – Create base layout and error handling infrastructure.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/unit/web/components/error_boundary.test.ts |
| **Deep module** | N/A – infrastructure |
| **Definition of done** | BaseLayout with Head, Navigation, Footer, CSP meta, view-transition meta; ErrorBoundary for React islands; custom 404 page using BaseLayout |
| **Out of scope** | Dynamic error pages (deferred to v2) |
| **Rules** | All pages use BaseLayout; ErrorBoundary wraps React islands; 404 page uses BaseLayout |
| **Advanced patterns** | Error boundary pattern, Progressive enhancement |
| **Anti‑patterns** | Missing error handling, inconsistent layouts |

#### Subtasks for WEB‑012

- [ ] **WEB‑012‑1** – `src/layouts/BaseLayout.astro` – Page shell with `<Head />`, `<Navigation />`, `<Footer />`, CSP meta tags, view-transition meta (`<meta name="view-transition" content="same-origin" />`), and `<slot />` for page content
  `imports_from`: Head, Navigation, Footer
  `verification`: `pnpm build` succeeds and BaseLayout renders correctly
- [ ] **WEB‑012‑2** – `src/web/components/ErrorBoundary.tsx` – React error boundary for client islands
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit` compiles and unit tests pass
- [ ] **WEB‑012‑3** – `src/pages/404.astro` – Custom 404 page using BaseLayout
  `imports_from`: BaseLayout
  `verification`: page renders at `/404` with BaseLayout

---

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
- [ ] **WEB‑001‑5** – `features/company/company-browsing.feature` – Gherkin scenarios (visitor actor). Note: This feature is distinct from COMP-001-7's `company-evaluation.feature`, which covers admin-side evaluation workflows. This feature focuses on visitor-side browsing and filtering.
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
  **Error-state testing**: Include test cases for loading spinner, empty state message, and network error message
- [ ] **WEB‑002‑5** – `features/broker/broker-dashboard.feature` – Scenarios  
  `verification`: `pnpm test:bdd`

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
- [ ] **WEB-003-4** - `tests/unit/web/components/technology_showcase.test.ts` - Component tests  
  `imports_from`: TechnologyShowcase  
  `verification`: pass  
  **Error-state testing**: Include test cases for loading spinner, empty state message, and network error message
- [ ] **WEB‑003‑5** – `features/tech/technology-browsing.feature` – Gherkin scenarios  
  `verification`: `pnpm test:bdd` passes

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
- [ ] **WEB-004-4** - `tests/unit/web/components/violation_tracker.test.ts` - Component tests  
  `imports_from`: ViolationTracker  
  `verification`: pass  
  **Error-state testing**: Include test cases for loading spinner, empty state message, and network error message
- [ ] **WEB‑004‑5** – `features/violation/violation-tracking.feature` – Gherkin scenarios  
  `verification`: `pnpm test:bdd` passes

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

- [ ] **WEB‑005‑1** – `src/web/components/legal_guide.tsx` – Main component with inline AdequacyIndicator and LegalDisclaimer sub-components
  `imports_from`: LegalQueryService, LawDTO
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑005‑2** – `src/web/components/jurisdiction_selector.tsx` – Jurisdiction filter
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑005‑3** – `tests/unit/web/components/legal_guide.test.ts` – Component tests
  `imports_from`: LegalGuide
  `verification`: pass
  **Error-state testing**: Include test cases for loading spinner, empty state message, and network error message
- [ ] **WEB‑005‑4** – `features/law/legal-guide.feature` – Gherkin scenarios
  `verification`: `pnpm test:bdd` passes

---

### `WEB‑006A` Core feature pages | Status: `backlog`
`depends_on`: `WEB‑002`, `WEB‑003`, `WEB‑004`, `ARCH‑012`

- [ ] **Parent** `WEB‑006A` – Build core directory and listing pages.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | `Feature:` Visitor navigates site `Scenario 1:` Visit homepage → see overview `Scenario 2:` Visit companies page → see listing `Scenario 3:` Visit technologies page → see showcase `Scenario 4:` Visit data brokers page → see dashboard |
| **TDD** | `Test file(s):` tests/integration/pages.test.ts (partial) |
| **Deep module** | N/A – page wiring |
| **Definition of done** | Core pages created and accessible via `pnpm build` |
| **Out of scope** | Dynamic routing |
| **Rules** | Pages consume components; use BaseLayout; use `import.meta.env.SITE` instead of deprecated `Astro.site` API; add `<meta name="view-transition" content="same-origin" />` in head for progressive enhancement |
| **Advanced patterns** | Astro static routing |
| **Anti‑patterns** | Missing pages, broken navigation |

#### Subtasks for WEB‑006A

- [ ] **WEB‑006A‑1** – `src/pages/index.astro` – Homepage with site overview and navigation to main sections. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/` and grep finds no `Astro.site` usage
- [ ] **WEB‑006A‑2** – `src/pages/companies.astro` – Uses `<CompanyListing />` and `CompanyQueryService`. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: CompanyListing
  `verification`: page renders at `/companies` and grep finds no `Astro.site` usage
- [ ] **WEB‑006A‑3** – `src/pages/technologies.astro` – Uses `<TechnologyShowcase />` with filter for all technologies. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: TechnologyShowcase
  `verification`: page renders at `/technologies` and grep finds no `Astro.site` usage
- [ ] **WEB‑006A‑4** – `src/pages/data-brokers.astro` – Uses `<BrokerDashboard />`. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: BrokerDashboard
  `verification`: page renders at `/data-brokers` and grep finds no `Astro.site` usage

---

### `WEB‑006B` Practice content pages | Status: `backlog`
`depends_on`: `WEB‑005`, `ARCH‑012`

- [ ] **Parent** `WEB‑006B` – Build privacy practice educational pages.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | `Feature:` Visitor learns about practices `Scenario 1:` Visit data collection page → see practices `Scenario 2:` Visit data breaches page → see violation data `Scenario 3:` Visit regulatory actions page → see legal guide `Scenario 4:` Visit surveillance page → see tracking methods |
| **TDD** | `Test file(s):` tests/integration/pages.test.ts (partial) |
| **Deep module** | N/A – page wiring |
| **Definition of done** | Practice pages created and accessible via `pnpm build` |
| **Out of scope** | Dynamic routing |
| **Rules** | Pages consume components where applicable; use BaseLayout; use `import.meta.env.SITE` instead of deprecated `Astro.site` API; add `<meta name="view-transition" content="same-origin" />` in head |
| **Advanced patterns** | Astro static routing |
| **Anti‑patterns** | Missing pages, broken navigation |

#### Subtasks for WEB‑006B

- [ ] **WEB‑006B‑1** – `src/pages/practices/data-collection.astro` – Content page with fact‑based lists. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/practices/data-collection` and grep finds no `Astro.site` usage
- [ ] **WEB‑006B‑2** – `src/pages/practices/data-breaches.astro` – Displays violation data using `<ViolationTracker />`. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: ViolationTracker
  `verification`: page renders at `/practices/data-breaches` and grep finds no `Astro.site` usage
- [ ] **WEB‑006B‑3** – `src/pages/practices/regulatory-actions.astro` – Uses `<LegalGuide />` filtered for enforcement actions. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: LegalGuide
  `verification`: page renders at `/practices/regulatory-actions` and grep finds no `Astro.site` usage
- [ ] **WEB‑006B‑4** – `src/pages/practices/surveillance.astro` – Static content page on tracking methods. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/practices/surveillance` and grep finds no `Astro.site` usage

---

### `WEB‑006C` Resource & comparison pages | Status: `backlog`
`depends_on`: `ARCH‑012`

- [ ] **Parent** `WEB‑006C` – Build resource and comparison pages.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | `Feature:` Visitor explores resources `Scenario 1:` Visit smart devices page → see IoT risks `Scenario 2:` Visit software comparison page → see OS comparisons `Scenario 3:` Visit devices page → see hardware privacy `Scenario 4:` Visit platforms page → see platform comparisons |
| **TDD** | `Test file(s):` tests/integration/pages.test.ts (partial) |
| **Deep module** | N/A – page wiring |
| **Definition of done** | Resource pages created and accessible via `pnpm build` |
| **Out of scope** | Dynamic routing |
| **Rules** | Pages consume components where applicable; use BaseLayout; use `import.meta.env.SITE` instead of deprecated `Astro.site` API; add `<meta name="view-transition" content="same-origin" />` in head |
| **Advanced patterns** | Astro static routing |
| **Anti‑patterns** | Missing pages, broken navigation |

#### Subtasks for WEB‑006C

- [ ] **WEB‑006C‑1** – `src/pages/practices/smart-devices.astro` – IoT privacy risks. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/practices/smart-devices` and grep finds no `Astro.site` usage
- [ ] **WEB‑006C‑2** – `src/pages/practices/software-comparison.astro` – OS comparisons (renamed from platforms.astro). Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/practices/software-comparison` and grep finds no `Astro.site` usage
- [ ] **WEB‑006C‑3** – `src/pages/devices.astro` – Privacy-focused hardware page rendering device cards from seed data. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none (reads from seed data directly)
  `verification`: page renders at `/devices` with device information from `src/data/seed.json` and grep finds no `Astro.site` usage
- [ ] **WEB‑006C‑4** – `src/pages/practices/platforms.astro` – Platform comparisons. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/practices/platforms` and grep finds no `Astro.site` usage

---

### `WEB‑006D` Meta & informational pages | Status: `backlog`
`depends_on`: `ARCH‑012`

- [ ] **Parent** `WEB‑006D` – Build meta and informational pages.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | `Feature:` Visitor accesses site information `Scenario 1:` Visit take action page → see consumer guidance `Scenario 2:` Visit methodology page → see scoring explanation `Scenario 3:` Visit glossary page → see ubiquitous language `Scenario 4:` Visit about page → see site information `Scenario 5:` Visit sitemap → see XML sitemap |
| **TDD** | `Test file(s):` tests/integration/pages.test.ts (partial) |
| **Deep module** | N/A – page wiring |
| **Definition of done** | Meta pages created and accessible via `pnpm build` |
| **Out of scope** | Dynamic routing |
| **Rules** | Pages use BaseLayout; use `import.meta.env.SITE` instead of deprecated `Astro.site` API; add `<meta name="view-transition" content="same-origin" />` in head |
| **Advanced patterns** | Astro static routing |
| **Anti‑patterns** | Missing pages, broken navigation |

#### Subtasks for WEB‑006D

- [ ] **WEB‑006D‑1** – `src/pages/take-action.astro` – Consumer guidance page. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/take-action` and grep finds no `Astro.site` usage
- [ ] **WEB‑006D‑2** – `src/pages/methodology.astro` – Explains scoring and evaluation. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/methodology` and grep finds no `Astro.site` usage
- [ ] **WEB‑006D‑3** – `src/pages/glossary.astro` – Ubiquitous language terms. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/glossary` and grep finds no `Astro.site` usage
- [ ] **WEB‑006D‑4** – `src/pages/about.astro` – Standard about page. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/about` and grep finds no `Astro.site` usage
- [ ] **WEB‑006D‑5** – `src/pages/sitemap.astro` – XML sitemap for SEO and navigation. Uses `import.meta.env.SITE` instead of deprecated `Astro.site` API. Add `<meta name="view-transition" content="same-origin" />` in head.
  `imports_from`: none
  `verification`: page renders at `/sitemap.xml` and grep finds no `Astro.site` usage

---

### `WEB‑006E` Page integration verification | Status: `backlog`
`depends_on`: `WEB‑006A`, `WEB‑006B`, `WEB‑006C`, `WEB‑006D`

- [ ] **Parent** `WEB‑006E` – Verify all pages build and are accessible.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | N/A |
| **TDD** | `Test file(s):` tests/integration/pages.test.ts |
| **Deep module** | N/A – verification |
| **Definition of done** | All pages build successfully, are accessible, and Astro 6 API compliance verified |
| **Out of scope** | Dynamic routing |
| **Rules** | Verify no `Astro.site` or deprecated API usage across all pages |
| **Advanced patterns** | Integration testing |
| **Anti‑patterns** | Broken builds, missing pages |

#### Subtasks for WEB‑006E

- [ ] **WEB‑006E‑1** – `tests/integration/pages.test.ts` – Verify all pages build and are accessible, and verify Astro 6 API compliance across all pages
  `verification`: `pnpm build` succeeds, all pages accessible, and grep finds no `Astro.site` usage in any page

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

### `WEB‑009` Software & Applications page | Status: `backlog`
`depends_on`: `APP‑006`

- [ ] **Parent** `WEB‑009` – Build software applications page with category filtering and platform indicators.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | `Feature:` Visitor browses software `Scenario 1:` Filter by category → list updates `Scenario 2:` No results message `Scenario 3:** View platform compatibility |
| **TDD** | tests/unit/web/components/software_showcase.test.ts |
| **Deep module** | `Public interface:** `<SoftwareShowcase category={} platform={} />` `Hidden complexity:** category filtering, platform badges `Depth metric:** moderate – 2 props, ~140 lines |
| **Definition of done** | Component, tests, `.feature`, DTO consumption |
| **Rules** | Uses `SoftwareQueryService`; category badges visual; platform indicators |

#### Subtasks for WEB‑009

- [ ] **WEB‑009‑1** – `src/web/components/software_showcase.tsx` – Main component
  `imports_from`: SoftwareQueryService, SoftwareDTO
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑009‑2** – `src/web/components/software_card.tsx` – Card component with category and platform badges
  `imports_from`: SoftwareDTO
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑009‑3** – `src/web/components/category_badge.tsx` – Category indicator
  `imports_from`: none
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑009‑4** – `tests/unit/web/components/software_showcase.test.ts` – Component tests
  `imports_from`: SoftwareShowcase
  `verification`: pass
  **Error-state testing**: Include test cases for loading spinner, empty state message, and network error message
- [ ] **WEB‑009‑5** – `features/software/software-browsing.feature` – Gherkin scenarios
  `verification`: `pnpm test:bdd` passes

---

### `WEB‑010` Data Broker Education page | Status: `backlog`
`depends_on`: `APP‑002`

- [ ] **Parent** `WEB‑010` – Build educational page explaining data brokers, DROP, and deletion process.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | N/A – educational content |
| **TDD** | N/A – static content with component integration |
| **Deep module** | N/A – content page |
| **Definition of done** | Educational page with factual explanations; links to official resources; integrates broker search component |
| **Out of scope** | Legal advice |
| **Rules** | Neutral factual explanations; no "good/bad" language; links to official DROP resources |

#### Subtasks for WEB‑010

- [ ] **WEB‑010‑1** – `src/pages/data-broker-education.astro` – Educational page explaining: what data brokers are, how they collect data, DROP explained, step-by-step deletion instructions, the reality of suppression lists (data never truly deleted), links to official resources
  `imports_from`: BrokerDashboard (for search integration)
  `verification`: page renders at `/data-broker-education` with educational content
- [ ] **WEB‑010‑2** – Verify all claims have source citations
  `imports_from`: none
  `verification`: grep finds source URLs for all factual claims on the page

---

### `WEB‑011` Platforms & Websites comparison | Status: `backlog`
`depends_on`: `APP‑006`

- [ ] **Parent** `WEB‑011` – Build comparison page for platforms and websites with factual privacy data.

| Aspect | Detail |
| :--- | :--- |
| **DDD** | `Bounded context:` Web Adapter (presentation) `Aggregate root:` N/A |
| **BDD** | `Feature:` Visitor compares platforms `Scenario 1:** View search engines comparison `Scenario 2:** View cloud storage comparison |
| **TDD** | tests/unit/web/components/platform_comparison.test.ts |
| **Deep module** | `Public interface:** `<PlatformComparison category={} />` `Hidden complexity:** comparison table, fact display `Depth metric:** moderate – 1 prop, ~150 lines |
| **Definition of done** | Component, tests, `.feature`, factual comparison data |
| **Rules** | Uses `SoftwareQueryService` for platforms; factual comparisons only |

#### Subtasks for WEB‑011

- [ ] **WEB‑011‑1** – `src/web/components/platform_comparison.tsx` – Main comparison component
  `imports_from`: SoftwareQueryService, SoftwareDTO
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑011‑2** – `src/web/components/comparison_table.tsx` – Comparison table component
  `imports_from`: SoftwareDTO
  `verification`: `pnpm tsc --noEmit`
- [ ] **WEB‑011‑3** – `tests/unit/web/components/platform_comparison.test.ts` – Component tests
  `imports_from`: PlatformComparison
  `verification`: pass
- [ ] **WEB‑011‑4** – `features/software/platform-comparison.feature` – Gherkin scenarios
  `verification`: `pnpm test:bdd` passes

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
| **Out of scope** | Full ACID consistency. **⚠️ v1 Limitation**: The current in-memory EventBus (ARCH‑004) does not persist events across Workers requests. For v1 static output this limitation has zero runtime impact (all data is pre-built). For v2 SSR, note that Cloudflare KV is eventually consistent and the in-memory EventBus does not survive worker restarts, meaning cross-request event handling will be effectively broken without Cloudflare Queues. The `CompanyReference` snapshot will only be populated when the violation is created immediately after the company (same request). Full cross‑request event handling requires Cloudflare Queues, planned for v2. |
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

## Meta‑conventions recap

- Every subtask has a `verification` command; if verification fails, halt and fix before proceeding.
- `blocked_by` is used only when a real‑world decision is pending; currently no tasks are blocked.
- `depends_on` exactly mirrors the phase‑based ordering; Phase 5 tasks may have cross‑phase dependencies.
- `blocked_by` is reserved for external decisions; tasks that are internally blocked by other tasks should use `depends_on`. Ensure all tasks follow this guideline.
- Depth refactoring is repeated as the final subtask inside every module; there is no separate global REF‑001.

This `TODO.md` is now fully executable by an AI agent following the stated conventions.

---

## Task Count Summary

- **Total tasks**: 46 (38 original + 8 new)
- **Phase 0 (Architecture)**: 8 tasks (ARCH-001 through ARCH-008)
- **Phase 1 (Domain Models)**: 6 tasks (COMP-001, TECH-001, VIOL-001, LAW-001, BROK-001, SOFT-001)
- **Phase 2 (Repositories & Infrastructure)**: 2 tasks (COMP-002, BROK-002)
- **Phase 2.5 (Seed Data & Content)**: 2 tasks (DATA-001, DATA-002)
- **Phase 3 (Application Layer)**: 6 tasks (APP-001 through APP-006)
- **Phase 4 (Web Components)**: 11 tasks (WEB-001 through WEB-011)
- **Phase 5 (Cross-context Integration)**: 4 tasks (INT-001 through INT-004)
- **Meta**: WEB-006 (page wiring), WEB-007 (legal pages), WEB-008 (navigation)