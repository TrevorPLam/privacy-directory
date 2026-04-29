# TODO.md — Comprehensive AI-Execution Analysis

> **Purpose**: This document catalogues every gap, ambiguity, defect, and missing task
> found in `TODO.md` so that an AI agent (Windsurf) can execute the task list to
> completion without stalling. Findings are graded **CRITICAL / HIGH / MEDIUM / LOW**.
> Each finding includes the exact location, the problem statement, and a concrete
> recommended fix.

---

## Executive Summary

The task list is architecturally coherent and impressively detailed. The DDD/BDD/TDD
discipline is consistent, the `depends_on` graph is mostly accurate, and the
`verification` commands are concrete and executable. However, **5 critical defects**
would cause the AI agent to deadlock or produce a structurally broken project, and
**~25 additional gaps** range from missing tasks to naming conflicts that will produce
silent errors at build time. Every finding is documented below with a recommended fix.

---

## CRITICAL Defects

These must be resolved before execution begins; they will cause hard stops or
architectural corruption.

---

### C-001 — Circular Dependency: DATA-001 ↔ DATA-002

**Location**: Phase 2.5 — `DATA-001` header, `DATA-002` header

**Problem**: Both tasks list each other as their `depends_on` target:

- `DATA-001` → `depends_on: DATA-002`
- `DATA-002` → `depends_on: DATA-001`

This is a true graph cycle. Any dependency resolver (human or AI) will deadlock when
it reaches this pair because neither task can be started until the other completes.

**Root cause**: `DATA-002` (content brief) is intended to *specify* the data, and
`DATA-001` (seed JSON) is intended to *implement* it. The brief must exist before the
data. The `depends_on` arrows are reversed on `DATA-001`.

**Fix**: Break the cycle by setting:

```
DATA-002  depends_on: ARCH-001A   (just needs the project to exist)
DATA-001  depends_on: DATA-002    (seed data is written after the brief)
DATA-003  depends_on: DATA-001    (audit runs after data is written)
```

Also update `APP-003`, `APP-004`, `APP-005`, `APP-006`, and `LAW-002` dependency
lists to include `DATA-001` (most already do; verify each).

---

### C-002 — CompanyRepository Interface Lives in Two Conflicting Locations

**Location**: `COMP-001` metadata table vs. `COMP-002-1` subtask

**Problem**:

- `COMP-001` metadata explicitly states:
  > "Repository interface location: Domain layer (`src/companies/domain/company_repository.ts`)"
- `COMP-002-1` instructs creating:
  > `src/companies/infrastructure/company_repository.ts` — "Repository interface"

These are two different file paths for what the document intends to be the same
interface. If Windsurf creates both, the implementations in `COMP-002-2` and
`COMP-002-3` will import from the wrong location, breaking DDD's Dependency
Inversion Principle and causing `dependency-cruiser` to raise violations (the
interface would live in infrastructure, which domain/application layers cannot
import).

**Fix**: Resolve to the domain-layer location stated in `COMP-001`. Rewrite `COMP-002`:

- **COMP-002-0 (new)** — `src/companies/domain/company_repository.ts` — domain interface
  (move out of COMP-002-1; this subtask should belong to `COMP-001` as `COMP-001-9`)
- **COMP-002-1** — `src/companies/infrastructure/memory_company_repository.ts` — in-memory impl
- **COMP-002-2** — `src/companies/infrastructure/cloudflare_kv_company_repository.ts` — KV impl

Update all `imports_from` references in APP-001 and INT-001 to import the interface
from `src/companies/domain/company_repository.ts`.

---

### C-003 — ARCH-007-* Tasks Are Phase 0 but Depend on Phase 1 Outputs

**Location**: `ARCH-007-company`, `ARCH-007-broker`, `ARCH-007-tech`,
`ARCH-007-violation`, `ARCH-007-law`

**Problem**: All five `ARCH-007-*` tasks are grouped under Phase 0 but each lists a
Phase 1 domain task in `depends_on`:

| ARCH-007 task | Depends on (Phase 1) |
|---|---|
| ARCH-007-company | COMP-001-7 |
| ARCH-007-broker | BROK-001-6 |
| ARCH-007-tech | TECH-001-7 |
| ARCH-007-violation | VIOL-001-7 |
| ARCH-007-law | LAW-001-7 |

Phase 0 is designated as the foundation that Phase 1 depends on — not the reverse.
Placing these tasks in Phase 0 implies they execute before Phase 1, which is
impossible given their dependencies. An AI agent respecting phase ordering will
either execute them out of order (incorrect) or stall.

**Fix**: Move all five `ARCH-007-*` tasks into **Phase 1**, appended after the domain
model they depend on. They should execute *after* each domain's feature file is
created, not before.

---

### C-004 — Missing Footer Component; WEB-012-1 Imports Undefined Symbol

**Location**: `WEB-012-1` subtask

**Problem**: `WEB-012-1` creates `src/layouts/BaseLayout.astro` with the directive:
> `imports_from: Head, Navigation, Footer`

`Head.astro` is created in `ARCH-009-1`. `Navigation` (as `.astro`) is created in
`ARCH-006-1`. But **no task anywhere creates a Footer component**. The import will
fail at build time and `pnpm astro check` will error.

**Fix**: Add a new subtask to `WEB-012`:

```
WEB-012-0 (new) — src/web/components/footer.astro — Global footer component
  imports_from: none
  verification: pnpm astro check compiles
```

Add `WEB-012-0` to the `depends_on` of `WEB-012-1`.

---

### C-005 — WEB-006C-3 (devices.astro) Reads seed.json Directly, Violating ARCH-013-3

**Location**: `WEB-006C-3` subtask

**Problem**: The subtask specifies:
> `imports_from: none (reads from seed data directly)`

This is explicitly disallowed by `ARCH-013-3`:
> "Script that checks that no file directly reads `seed.json` from the domain or
> application layers"

The architectural rule extends to the web (presentation) layer as well, since the
web adapter should consume DTOs from query services, not raw seed JSON. The
`scripts/check-seed-data-access.sh` script created in ARCH-013-3 will detect this
violation and fail. Additionally, no `DeviceQueryService` or `DeviceDTO` exists,
meaning the page has no properly typed data contract.

**Fix**: Add the following missing tasks (place in Phase 2.5 / Phase 3 respectively):

```
DEVICE-001 — Define Device domain model
  src/software/domain/device.ts (or src/devices/domain/device.ts)
  Aggregate fields: id, name, os, privacyFeatures, sourceUrl

APP-007 (new) — Device query service and DTOs
  depends_on: DATA-001
  src/devices/application/device_query_service.ts
  src/devices/application/device_dto.ts
  tests/unit/devices/application/device_query_service.test.ts
```

Update `WEB-006C-3`:
```
imports_from: DeviceQueryService, DeviceDTO
```

---

## HIGH Severity Gaps

These will produce incomplete or broken outputs that require backtracking.

---

### H-001 — Missing ARCH-007-software Step Definitions Task

**Location**: Phase 0 / Phase 1 ARCH-007 series

**Problem**: `SOFT-001-7` creates `features/software/software-evaluation.feature`,
and there are analogous `ARCH-007-*` step definition scaffold tasks for every other
domain (company, broker, tech, violation, law). No `ARCH-007-software` task exists.
`pnpm test:bdd` will report undefined steps when the software feature file is run.

**Fix**: Add:

```
ARCH-007-software — Create Software Cucumber step definitions
  depends_on: ARCH-002, SOFT-001-7
  tests/software/software.steps.ts
  verification: pnpm test:bdd --dry-run features/software/software-evaluation.feature
```

---

### H-002 — WEB-008 Listed in Task Count Summary but Not Defined

**Location**: Task Count Summary (line ~1871)

**Problem**: The summary states:
> "Meta: WEB-006 (page wiring), WEB-007 (legal pages), **WEB-008 (navigation)**"

No `WEB-008` task appears anywhere in the document body. Navigation is handled by
`ARCH-006` in Phase 0 (skeleton) and expanded in `WEB-012-1`. The summary count
of 11 WEB tasks (WEB-001 through WEB-011) excludes WEB-008 from the numbered set,
suggesting it was removed from the body but not from the summary — or it is an
intentionally separate navigation-integration task that was never written.

**Fix** (two options — pick one):

- **Option A (remove)**: Delete "WEB-008 (navigation)" from the summary. Confirm
  ARCH-006 and WEB-012-1 fully satisfy navigation requirements.
- **Option B (add)**: Write a `WEB-008` task that wires the navigation component
  into all pages, confirms all nav links resolve, and runs an integration accessibility
  test on navigation.

---

### H-003 — Missing `/violations` Listing Page

**Location**: `WEB-006A` subtasks

**Problem**: `WEB-006A` wires pages for companies, technologies, and data-brokers, but
there is no `/violations` page that mounts `<ViolationTracker />`. The component is
built (WEB-004), the query service exists (APP-004), and the navigation component
(ARCH-006) almost certainly links to a violations URL — but no `.astro` page exists
to serve that route.

`WEB-006B-2` creates `/practices/data-breaches.astro` which uses ViolationTracker,
but that is a *practice* page, not a primary violations directory page. Users navigating
to `/violations` will get a 404.

**Fix**: Add to `WEB-006A`:

```
WEB-006A-5 (new) — src/pages/violations.astro
  imports_from: ViolationTracker
  verification: page renders at /violations
```

---

### H-004 — APP-005 Missing Dependency on LAW-002

**Location**: `APP-005` parent task header

**Problem**: `APP-005` depends on `LAW-001` and `DATA-001` but not `LAW-002`. The
legal query service (`src/law/application/legal_query_service.ts`) must instantiate
or consume a `LawRepository`. `LAW-002` is the only task that defines a
`MemoryLawRepository` implementation. Without this dependency, Windsurf may execute
`APP-005` before `LAW-002`, and the service will import from a file that doesn't yet
exist.

**Fix**:

```
APP-005  depends_on: LAW-001, LAW-002, DATA-001
```

---

### H-005 — Missing Legal Freshness Disclaimer Banner Component

**Location**: `LAW-001-1`, `DATA-001-2`, and downstream law pages

**Problem**: Three separate subtasks reference a "visible banner on legal pages" stating
"Legal information last verified: [date]. This is not legal advice." — but no task
creates the UI component that renders this banner. The note appears in:

- `LAW-001-1`: "The site should display a visible banner..."
- `DATA-001-2`: "The site should display a visible banner..."

`WEB-007-3` only checks that the disclaimer *text* exists via grep, but there is no
task to build a reusable `<LegalDisclaimer />` or `<LegalFreshnessBanner />` component
backed by the `lastVerifiedDate` field.

**Fix**: Add to `WEB-005` (or as a standalone task before it):

```
WEB-005-0 (new) — src/web/components/legal_disclaimer_banner.tsx
  Props: lastVerifiedDate: string
  imports_from: none
  verification: pnpm tsc --noEmit
```

Update `WEB-005-1` to import and render this component. Update `WEB-007-3` grep to
check for the component tag, not just the text.

---

### H-006 — Missing `src/pages/software.astro` (Primary Software Listing Page)

**Location**: `WEB-006A`, `WEB-009`

**Problem**: `WEB-009` builds the `<SoftwareShowcase />` component, but no page in
`WEB-006A` or elsewhere mounts it as a primary `/software` route. `WEB-006C-2`
creates `/practices/software-comparison.astro` (OS comparisons — a different concept),
and `WEB-011` creates a platform comparison page, but there is no `/software` page
listing Signal, Brave, Proton Mail, etc. from the seed data.

**Fix**: Add to `WEB-006A` (or `WEB-006C`):

```
WEB-006A-5 or WEB-006C-5 (new) — src/pages/software.astro
  imports_from: SoftwareShowcase
  verification: page renders at /software
```

---

### H-007 — `pnpm lint` Used in Verification Commands but No ESLint Task Exists

**Location**: Every `depth refactor check` subtask; `ARCH-013-2`

**Problem**: The command `pnpm lint` appears in dozens of `verification` steps (e.g.,
`COMP-001-8`, `BROK-001-8`, `TECH-001-8`, `VIOL-001-8`, `LAW-001-8`, `SOFT-001-8`),
yet there is no task that:

1. Installs ESLint and a TypeScript-compatible config
2. Creates `.eslintrc.cjs` or `eslint.config.mjs`
3. Adds a `"lint"` script to `package.json`

Without this, `pnpm lint` will fail with "missing script" immediately, causing every
depth-refactor verification to be un-runnable.

**Fix**: Add a subtask to `ARCH-001A` or create a new `ARCH-001C`:

```
ARCH-001C (new) — Configure ESLint + TypeScript rules
  Subtasks:
  - Install: eslint, @typescript-eslint/eslint-plugin, @typescript-eslint/parser,
    eslint-plugin-astro, eslint-plugin-react
  - Create: eslint.config.mjs (flat config)
  - Add "lint": "eslint src tests" to package.json scripts
  verification: pnpm lint exits 0 on empty codebase
```

---

### H-008 — Missing `tsconfig.json` Setup Task

**Location**: `ARCH-001A`, `ARCH-001A-3`

**Problem**: `ARCH-001A-3` verification runs `pnpm tsc --noEmit && pnpm astro check`,
and virtually every subsequent subtask also runs `pnpm tsc --noEmit`. However, no
task creates `tsconfig.json`. Astro generates a starter `tsconfig.json` on `pnpm
create astro`, but the project scaffolding in `ARCH-001A-1` only covers
`package.json` creation — `astro create` is never explicitly invoked.

**Fix**: Either:

- Add to `ARCH-001A-1`: "Run `pnpm create astro@latest` or explicitly create
  `tsconfig.json` with strict mode, `moduleResolution: bundler`, `jsx: react-jsx`
  and path aliases for `src/`."
- Or add `ARCH-001A-6 (new)`: `tsconfig.json` — TypeScript project configuration
  with path aliases matching the domain-driven directory structure.

---

### H-009 — ARCH-002-5 Duplicates ARCH-001A-1 for `test:bdd` Script

**Location**: `ARCH-001A-1`, `ARCH-002-5`

**Problem**: `ARCH-001A-1` already instructs adding
`"test:bdd": "cucumber-js --config cucumber.config.ts"` to `package.json`. `ARCH-002-5`
then instructs adding the same script again. If Windsurf executes both sequentially
on the same file, the second write will either duplicate the script (invalid JSON)
or silently overwrite it (no-op but confusing).

**Fix**: Remove `ARCH-002-5` entirely. Mark the canonical location as `ARCH-001A-1`,
and update `ARCH-002-2`'s verification to reference the script already in place.

---

### H-010 — Missing `pnpm test` and `pnpm build` Script Definitions

**Location**: `ARCH-001A-1`

**Problem**: Verification commands throughout the document use `pnpm test` and
`pnpm build`, but `ARCH-001A-1` only explicitly adds `test:bdd`. The `pnpm build`
script comes from Astro (`astro build`), and `pnpm test` from Vitest
(`vitest run`). If `pnpm create astro` is not run (see H-008), these scripts won't
exist. Even if they do, the document should be explicit.

**Fix**: Expand `ARCH-001A-1` to enumerate all required `package.json` scripts:

```json
"scripts": {
  "dev": "astro dev",
  "build": "astro build",
  "preview": "astro preview",
  "astro": "astro",
  "test": "vitest run",
  "test:watch": "vitest",
  "test:bdd": "cucumber-js --config cucumber.config.ts",
  "lint": "eslint src tests",
  "tsc": "tsc --noEmit",
  "depcruise": "depcruise --validate"
}
```

---

## MEDIUM Severity Gaps

These produce incomplete implementations or misleading test coverage but won't
immediately crash the build.

---

### M-001 — Task Count Summary Is Substantially Incorrect

**Location**: Task Count Summary section (end of document)

**Problem**: The summary states "Phase 0 (Architecture): 8 tasks (ARCH-001 through
ARCH-008)". The actual Phase 0 tasks found in the document body are:

`ARCH-001A`, `ARCH-001B`, `ARCH-002`, `ARCH-007-company`, `ARCH-007-broker`,
`ARCH-007-tech`, `ARCH-007-violation`, `ARCH-007-law`, `ARCH-003`, `ARCH-004`,
`ARCH-005`, `ARCH-006`, `ARCH-008`, `ARCH-009`, `ARCH-010`, `ARCH-012`, `ARCH-013`

That is **17 parent tasks** in Phase 0, not 8. The total count of 46 tasks is also
likely wrong given the missing tasks identified in this analysis. An AI agent relying
on this count for progress tracking will incorrectly assess completion state.

**Fix**: Recount all phases after applying all fixes in this document, then update the
summary. The true total is approximately 55–60 parent tasks.

---

### M-002 — ARCH-011 Is Missing (Numbering Gap)

**Location**: Phase 0 architecture sequence

**Problem**: Task IDs jump from `ARCH-010` to `ARCH-012`. No `ARCH-011` is defined.
This suggests a task was removed during editing but the surrounding IDs were not
renumbered. An AI agent may search for ARCH-011 (e.g., as a dependency) and fail.

**Fix**: Either:
- Renumber `ARCH-012` to `ARCH-011`, or
- Document explicitly: "`ARCH-011` was removed; gap is intentional."

---

### M-003 — COMP-002 In-Memory Repository Doesn't Specify Seed Data Loading

**Location**: `COMP-002-2`

**Problem**: `COMP-002-2` creates `memory_company_repository.ts` and says "in-memory
implementation" but does not specify that it should load from `src/data/seed.json`.
Every other in-memory stub (APP-003, APP-004, APP-005, APP-006) explicitly states
"loads from seed.json" or "uses shared seed data from DATA-001."

If Windsurf implements an empty in-memory repo (no data), the company listing page
will render with zero results, and integration tests that expect seed data will fail.

**Fix**: Update `COMP-002-2`:
> "In-memory implementation that loads initial data from `src/data/seed.json`
> companies array. Same pattern as `memory_law_repository.ts` (LAW-002-2)."

---

### M-004 — Missing Repository Tasks for TECH, VIOL, and SOFT Contexts

**Location**: Phase 2 – Repositories & Infrastructure

**Problem**: Phase 2 defines `COMP-002` and `BROK-002` as infrastructure tasks, and
`LAW-002` appears later. But there are no repository tasks for:

- `TECH-002` — Technology repository
- `VIOL-002` — Violation repository
- `SOFT-002` — Software repository

Instead, their APP tasks say "uses in-memory stub" without an explicit infrastructure
task. This means:

1. There is no `TechnologyRepository` domain interface defined anywhere
2. There is no `ViolationRepository` domain interface defined anywhere
3. There is no `SoftwareRepository` domain interface defined anywhere

When v2 adds Cloudflare KV persistence, there will be no interface to implement
against, requiring retroactive DDD refactoring.

**Fix**: Add three tasks to Phase 2 (or note them explicitly as deferred v2 stubs):

```
TECH-002 — Define TechnologyRepository interface (domain layer)
  src/tech/domain/technology_repository.ts
  src/tech/infrastructure/memory_technology_repository.ts (loads from seed.json)

VIOL-002 — Define ViolationRepository interface (domain layer)
  src/violation/domain/violation_repository.ts
  src/violation/infrastructure/memory_violation_repository.ts

SOFT-002 — Define SoftwareRepository interface (domain layer)
  src/software/domain/software_repository.ts
  src/software/infrastructure/memory_software_repository.ts
```

Alternatively, add explicit "v2 deferred" notes with the interface stubs in the
respective domain tasks (TECH-001, VIOL-001, SOFT-001), keeping v1 APP services using
the stubs directly.

---

### M-005 — ARCH-002-1 Vitest Verification Command Is Likely Wrong

**Location**: `ARCH-002-1` verification

**Problem**: The verification states:
> `pnpm test -- --list` shows both project suites

The Vitest CLI flag to list test files is `--reporter=verbose` or `vitest list`, not
`--list`. The double-dash `--` passes args to the npm script, but `--list` is not a
valid Vitest flag in v1.x or v2.x. This command will exit non-zero or produce
unexpected output.

**Fix**: Replace with:
```
pnpm exec vitest list
```
or verify the exact Vitest version in use and confirm the flag name.

---

### M-006 — WEB-004-5 and ARCH-007-violation-1 Both Reference the Same Feature File

**Location**: `WEB-004-5`, `ARCH-007-violation-1`

**Problem**:
- `WEB-004-5` creates `features/violation/violation-tracking.feature` (Gherkin scenarios)
- `ARCH-007-violation-1` creates step definitions for `features/violation/violation-tracking.feature`
- But `ARCH-007-violation-1` depends on `VIOL-001-7`, which also creates
  `features/violation/violation-tracking.feature`

Both `VIOL-001-7` and `WEB-004-5` claim to create the same feature file. The former
covers domain-side scenarios (admin records violation), the latter covers web-side
scenarios (visitor tracks violations). If Windsurf creates both, the second write
will overwrite the first, losing domain scenarios.

**Fix**: Differentiate the two feature files:
- `VIOL-001-7` → `features/violation/violation-recording.feature` (admin actor)
- `WEB-004-5` → `features/violation/violation-browsing.feature` (visitor actor)

Update ARCH-007-violation-1 to dry-run against `violation-recording.feature`
specifically.

---

### M-007 — `src/layouts/PageLayout.astro` vs `src/layouts/BaseLayout.astro` Naming Confusion

**Location**: `ARCH-012-1`, `WEB-012-1`

**Problem**:
- `ARCH-012-1` creates `src/layouts/PageLayout.astro`
- `WEB-012-1` creates `src/layouts/BaseLayout.astro`

These appear to be two different files for the same concept. `WEB-012-1` says
BaseLayout wraps `<Head />`, `<Navigation />`, and `<Footer />` — which is what
a page scaffold does. `ARCH-012-1` also describes a static page scaffold with
`<html>`, `<head>`, `<body>`, and `<slot />`.

All page subtasks in WEB-006A/B/C/D reference `BaseLayout` in their "rules"
(`imports_from: BaseLayout`). None reference `PageLayout`. This means `PageLayout.astro`
is created but never used, and Windsurf may implement both as separate incomplete files.

**Fix**: Consolidate to a single layout file. Remove `ARCH-012` and fold its
requirements into `WEB-012-1` as the canonical `BaseLayout.astro`. Or, if both are
needed, explicitly document the relationship:
- `PageLayout.astro` = bare shell (html/head/body/slot) used by non-standard pages
- `BaseLayout.astro` = extends PageLayout, adds Header/Nav/Footer

---

### M-008 — Head.astro Created in Phase 0 (ARCH-009) but WEB-012 Depends on It Later

**Location**: `ARCH-009-1`, `WEB-012-1`

**Problem**: `ARCH-009-1` creates `src/web/components/Head.astro`. `WEB-012-1` creates
`BaseLayout.astro` which `imports_from: Head`. The `depends_on` for `WEB-012` lists
`ARCH-009` as a dependency — so the dependency is technically correct.

However, the `Head.astro` created in Phase 0 is described as containing:
> "title, description, OG tags, JSON-LD slot"

While `WEB-012-1` says BaseLayout should include:
> "CSP meta tags, view-transition meta"

Neither `Head.astro` task mentions CSP meta or view-transition meta. These will be
missing from `Head.astro` unless explicitly added. `ARCH-001A-2` handles CSP via
Astro's `security.csp` in `astro.config.mjs`, not a `<meta>` tag — but view-transition
meta must be in the HTML head.

**Fix**: Add to `ARCH-009-1`:
> "Also include `<meta name='view-transition' content='same-origin' />` in the
> component for Astro View Transitions progressive enhancement."

---

### M-009 — Duplicate/Conflicting Feature Files Between Domain Tasks and Web Tasks

**Location**: Multiple Gherkin subtasks

**Problem**: There is a structural ambiguity about where feature files live and who
"owns" them. The pattern across the document is:

- Domain feature file (e.g., `COMP-001-7` creates `features/company/company-evaluation.feature`) — admin actor
- Web browsing feature file (e.g., `WEB-001-5` creates `features/company/company-browsing.feature`) — visitor actor

This is correct and well-noted in WEB-001-5. However, the same distinction is NOT
made explicit for:

- **Violations**: `VIOL-001-7` creates `violation-tracking.feature`, and `WEB-004-5`
  also creates `violation-tracking.feature` (same filename). See M-006.
- **Technology**: `TECH-001-7` creates `technology-categorization.feature` and
  `WEB-003-5` creates `technology-browsing.feature` — these *are* distinct. ✓
- **Legal**: `LAW-001-7` creates `law-management.feature` and `WEB-005-4` creates
  `legal-guide.feature` — distinct. ✓
- **Software**: `SOFT-001-7` creates `software-evaluation.feature` and `WEB-009-5`
  creates `software-browsing.feature` — distinct. ✓
- **Brokers**: `BROK-001-6` creates `broker-management.feature` and `WEB-002-5`
  creates `broker-dashboard.feature` — distinct. ✓

The only collision is violations (M-006 above). No other action needed here beyond
fixing M-006.

---

### M-010 — BROK-002-3 CSV Parser Is Provisional; No Re-visit Task Exists

**Location**: `BROK-002-3`

**Problem**: The subtask explicitly states:
> "⚠️ Provisional: This parser implementation is provisional and must be revisited
> once the CA DROP API specification is available (expected August 2026)."

However, there is no follow-up task, no `blocked_by` note, and no reminder task to
revisit this in v2. The `json_parser.ts` stub is created as a fallback, but both
parsers use the same interface — so the interface itself may need to change when the
actual API spec arrives.

**Fix**: Add a `BROK-003` task (v2 placeholder):
```
BROK-003 — Revisit DROP adapter for CA DROP API (v2)
  depends_on: external CA DROP API specification (August 2026)
  blocked_by: CA DROP API spec not yet released
  Status: backlog (v2)
```

And add `blocked_by: "CA DROP API spec (expected Aug 2026)"` to `BROK-002-3`.

---

### M-011 — Missing `ARCH-008` Navigation to Main Document (Misplaced in Phase 0)

**Location**: `ARCH-008` — Configure accessibility testing

**Problem**: `ARCH-008` appears *after* `ARCH-013` in the document body, even though
it carries a lower number. While task IDs don't have to appear in document order,
the jump from `ARCH-013` back to `ARCH-008` at line ~539 is confusing and indicates
an editing artifact. More importantly, `ARCH-008` depends on `ARCH-002` (testing
infrastructure) which is correct, but its placement near `ARCH-013` (which depends
on `ARCH-001B` and `ARCH-002`) may confuse a linear document reader.

**Fix**: Reorder the document to place tasks in numeric order within each phase:
`ARCH-001A → ARCH-001B → ARCH-002 → ARCH-003 → ARCH-004 → ARCH-005 → ARCH-006 →
ARCH-007-* → ARCH-008 → ARCH-009 → ARCH-010 → ARCH-012 → ARCH-013`

---

### M-012 — Missing `docs/` Directory Creation Task

**Location**: Multiple documentation subtasks

**Problem**: Dozens of subtasks write to `docs/`:
- `docs/context-map.md` (ARCH-003-1)
- `docs/ubiquitous-language.md` (ARCH-003-2)
- `docs/event-subscriptions.md` (ARCH-004-3)
- `docs/data-flow.md` (ARCH-005-1)
- `docs/navigation-structure.md` (ARCH-006-2)
- `docs/neutral-language-guide.md` (DATA-001-5)
- `docs/content-brief.md` (DATA-002-1)
- `docs/content-audit-checklist.md` (DATA-003-1)
- `docs/qabage-checklist.md` (ARCH-002-3-1)

No task creates the `docs/` directory, and no `ARCH-001A-3` barrel file covers it.
While `mkdir` is usually implicit, some AI runners fail if the parent directory
doesn't exist before writing.

**Fix**: Add to `ARCH-001A-3`:
> "Also create `docs/` directory with a `README.md` placeholder."

---

### M-013 — Missing Legal Guide Page Route

**Location**: `WEB-006A`, `WEB-005`

**Problem**: `WEB-005` builds the `<LegalGuide />` component. `WEB-006B-3` mounts it
on `/practices/regulatory-actions`. But there is no `/legal` or `/legal-guide` primary
route. The navigation component (ARCH-006) almost certainly includes a "Legal Guide"
link, but no page serves that route. Users clicking "Legal Guide" in the nav will
get a 404.

**Fix**: Add to `WEB-006A` or `WEB-006B`:
```
WEB-006A-6 (new) — src/pages/legal-guide.astro
  imports_from: LegalGuide
  verification: page renders at /legal-guide
```

---

## LOW Severity Gaps

Cosmetic, consistency, or edge-case issues that won't break the build but reduce
quality.

---

### L-001 — `ARCH-002-3-1` Is Oddly Numbered (Sub-subtask)

The task `ARCH-002-3-1` creates `docs/qabage-checklist.md` as a sub-subtask of
`ARCH-002-3`. This creates a 4-level ID (phase-task-subtask-sub-subtask). The
document convention uses 3-level IDs (`TASK-NNN-N`). This inconsistency could
confuse ID parsing in downstream tooling.

**Fix**: Promote to `ARCH-002-4` (or renumber downstream tasks) and reference it
as a dependency of `ARCH-002-3`.

---

### L-002 — `ARCH-003-1` Verification May Fail Due to Grep Format

`ARCH-003-1` verification: `grep -c "Context Pair:" docs/context-map.md` must equal 15.
This requires the exact string `Context Pair:` in the document. If Windsurf uses
`## Context Pair:` or `**Context Pair:**` or any Markdown formatting, the grep
count will be wrong. The `grep` also counts occurrences, not sections, so if "Context
Pair:" appears twice in one section (heading + table), the count will be 30.

**Fix**: Change verification to:
```
grep -c "^## Context Pair:" docs/context-map.md` equals 15
```
Or use: `grep -cE "^#{1,3} Context Pair" docs/context-map.md`

---

### L-003 — `ARCH-001B-4` `_headers` File Has an Unclosed Rule Block

**Location**: `ARCH-001B-4` content block

**Problem**: The `public/_headers` content shown ends with:
```
Permissions-Policy: camera=(), microphone=(), geolocation=()
```
There is no closing marker (Cloudflare `_headers` syntax doesn't need one), but
the `Content-Security-Policy` header defined in `ARCH-001A-2` (via Astro's
`security.csp`) and the `X-Frame-Options: DENY` in `_headers` will conflict.
`X-Frame-Options` is superseded by `frame-ancestors` in CSP. Having both set will
produce duplicate conflicting headers.

**Fix**: Remove `X-Frame-Options: DENY` from `_headers` (Astro's CSP already sets
`frame-ancestors: 'none'` per `ARCH-001A-2` default policy).

---

### L-004 — Zod Added in DATA-001-4 but Not in ARCH-001A-1 Package Dependencies

**Location**: `DATA-001-4`

**Problem**: `DATA-001-4` states: "Use Zod from local import (add to package.json if
not already present)." But `ARCH-001A-1` lists all production dependencies and does
not include `zod`. Windsurf will need to run `pnpm add zod` mid-execution in Phase 2.5.
This is not inherently wrong, but it's inconsistent with the upfront dependency
declaration pattern and could cause issues if the install step doesn't run before
the import.

**Fix**: Add `zod` to the dependencies list in `ARCH-001A-1`.

---

### L-005 — Missing `src/infrastructure/event_subscriptions.ts` in Composition Root Documentation

**Location**: `ARCH-004-3`, `INT-001-2`

**Problem**: `ARCH-004-3` documents where subscriptions are wired (composition root),
and `INT-001-2` creates `src/infrastructure/event_subscriptions.ts`. However, no
task actually *imports and invokes* this subscriptions file. In a static Astro site,
the composition root is typically called during build. There is no task that calls
`registerSubscriptions()` or similar in the Astro build pipeline.

**Fix**: Add a note to `INT-001-2` (or a new INT-001-4 subtask):
> "Wire `event_subscriptions.ts` into the Astro build entry point or a build-time
> init script so subscriptions are registered during static generation."

---

### L-006 — Barrel Index Files Not Updated After Domain Tasks

**Location**: `ARCH-001A-3` and all domain subtasks

**Problem**: `ARCH-001A-3` creates barrel `index.ts` files per directory for clean
imports. However, no subsequent domain subtask includes "update the barrel index"
as a step. If Windsurf doesn't automatically maintain barrel files, imports like
`import { Company } from 'src/companies/domain'` (used extensively in test files)
will fail because the barrel won't export the new symbol.

**Fix**: Add a standing convention in the document header:
> "Convention: After every source file is created, update the nearest `index.ts`
> barrel to re-export the new public symbol."

Or add an explicit last subtask to each phase task: `"Update src/<context>/domain/index.ts with new exports."`

---

### L-007 — Missing `src/pages/violations.astro` in Navigation Structure

This partially duplicates H-003 but from the navigation perspective: `ARCH-006-1`
creates the navigation component and `ARCH-006-2` documents the navigation structure.
The document `docs/navigation-structure.md` will presumably include a `/violations`
link. If H-003 is fixed (page created), this is resolved. If it isn't, the nav
has a dead link.

---

### L-008 — `vi-axe` Package Existence Should Be Verified

**Location**: `ARCH-001A-1`, `ARCH-008-1`

**Problem**: `vi-axe` is listed as a dependency. As of early 2026, `vi-axe` was a
community package with intermittent maintenance. Its API and package name should be
verified against current npm. If it has been deprecated, `@axe-core/vitest` may be
the current alternative. Windsurf executing `pnpm install` may get a deprecation
warning or install failure.

**Fix**: Add a note: "Verify `vi-axe` current version/availability on npm before
installing; fallback to `@axe-core/vitest` if deprecated."

---

### L-009 — `neverthrow` Result Objects and Cloudflare Workers Serialization Warning Only Partially Addressed

**Location**: `ARCH-001B-1` note

**Problem**: The note on `neverthrow` serialization issues (prototype chain lost via
`structuredClone`) is acknowledged for v2 but `verdict-ts` is suggested as an
alternative without a migration path or a deferred task. This means the v2 migration
will require finding and updating every `Result` usage across the entire codebase.

**Fix**: Add a `// TODO(v2): migrate to verdict-ts for SSR serialization safety`
comment convention note to `ARCH-001B-1`, and log a `ARCH-v2-001` placeholder task:
```
ARCH-v2-001 — Migrate neverthrow → verdict-ts for Cloudflare Workers SSR
  depends_on: v2 SSR output mode decision
  blocked_by: v2 scope
```

---

### L-010 — `SECURE Data Act` Monitoring Item Has No Task

**Location**: `ARCH-003-2` note

**Problem**: The note states:
> "Add SECURE Data Act (discussion draft released April 22, 2026) to monitoring list
> — federal bill that would preempt state privacy laws if passed"

There is no `monitoring-list.md` task, no `blocked_by` reference, and no mechanism
to track this legislative risk. If passed, it would invalidate 20+ seed data law
entries.

**Fix**: Add to `DATA-001-5` (neutral-language guide) or create:
```
DATA-004 (new) — docs/legislative-monitoring.md
  Track: SECURE Data Act (April 2026 draft), Oklahoma ODPA (Jan 2027 effective)
  Convention: Review before each content refresh cycle
```

---

## Dependency Graph Corrections Summary

For Windsurf's dependency resolver, the following `depends_on` values should be
corrected or added:

| Task | Current `depends_on` | Corrected `depends_on` |
|---|---|---|
| DATA-001 | DATA-002 | DATA-002 |
| DATA-002 | DATA-001 | ARCH-001A |
| DATA-003 | DATA-002 | DATA-001 |
| APP-005 | LAW-001, DATA-001 | LAW-001, LAW-002, DATA-001 |
| WEB-012 | ARCH-006, ARCH-009, ARCH-010 | ARCH-006, ARCH-009, ARCH-010 (add WEB-012-0 footer task) |
| ARCH-007-company | ARCH-002, COMP-001-7 | Move to Phase 1 |
| ARCH-007-broker | ARCH-002, BROK-001-6 | Move to Phase 1 |
| ARCH-007-tech | ARCH-002, TECH-001-7 | Move to Phase 1 |
| ARCH-007-violation | ARCH-002, VIOL-001-7 | Move to Phase 1 |
| ARCH-007-law | ARCH-002, LAW-001-7 | Move to Phase 1 |
| WEB-006A | (current) | Add WEB-006A-5 (violations page) |
| WEB-006A | (current) | Add WEB-006A-6 (legal-guide page) |

---

## Missing Tasks: Complete List

The following tasks do not exist in the document but are required for a complete,
buildable project:

| New Task ID | File | Reason | Severity |
|---|---|---|---|
| ARCH-001C | ESLint configuration | `pnpm lint` used everywhere | HIGH |
| ARCH-001A-6 | `tsconfig.json` | No TypeScript config task | HIGH |
| ARCH-007-software | Software step definitions | Matches pattern for other domains | HIGH |
| WEB-012-0 | `footer.astro` | BaseLayout imports undefined Footer | CRITICAL |
| DEVICE-001 | Device domain model | devices.astro bypasses architecture | CRITICAL |
| APP-007 | Device query service + DTO | Same reason as DEVICE-001 | CRITICAL |
| WEB-006A-5 | `src/pages/violations.astro` | No primary violations route | HIGH |
| WEB-006A-6 | `src/pages/legal-guide.astro` | No primary legal guide route | HIGH |
| WEB-006A-7 | `src/pages/software.astro` | No primary software listing route | HIGH |
| WEB-005-0 | `legal_disclaimer_banner.tsx` | Referenced in 3 tasks, never built | HIGH |
| TECH-002 | Technology repository interface | Domain interface missing | MEDIUM |
| VIOL-002 | Violation repository interface | Domain interface missing | MEDIUM |
| SOFT-002 | Software repository interface | Domain interface missing | MEDIUM |
| BROK-003 | DROP API revisit (v2) | Provisional code needs tracking | MEDIUM |
| DATA-004 | Legislative monitoring doc | SECURE Data Act, ODPA | LOW |
| ARCH-v2-001 | neverthrow → verdict-ts migration | v2 SSR serialization | LOW |

---

## Verified Strengths

The following aspects of the document are well-designed and need no changes:

- **DDD structure** is coherent and consistent. Aggregate roots, value objects, domain
  events, and repository interfaces are properly separated. The distinction between
  admin-side and visitor-side feature files (noted explicitly in WEB-001-5) is good.
- **BDD coverage** is strong. Every domain aggregate has at least 2–3 BDD scenarios,
  and visitor-facing components have matching browsing features.
- **Error path testing** is explicitly called out in every APP query service (error
  test files) and web component test. This is exemplary TDD discipline.
- **Neutral framing convention** is well-enforced: dedicated audit script, style
  guide, banned adjective list, and grep verification.
- **Security posture** is thorough: CSP via Astro stable API, Cloudflare `_headers`,
  local font loading, no third-party tracking, privacy-first design.
- **Phase 2.5 content strategy** with explicit `DATA-002` content brief to prevent
  AI execution ambiguity is an excellent pattern.
- **`neverthrow` Result pattern** is consistently applied. `imports_from` on every
  subtask forces explicit dependency tracking.
- **v1/v2 delineation** is mostly clear: static output v1, SSR + KV v2. Most tasks
  correctly scope what is deferred.

---

## Recommended Execution Order (After Fixes)

```
Phase 0:  ARCH-001A → ARCH-001B → ARCH-001C(new) → ARCH-002 →
          ARCH-003 → ARCH-004 → ARCH-005 → ARCH-006 →
          ARCH-008 → ARCH-009 → ARCH-010 → ARCH-012 → ARCH-013

Phase 1:  [parallel] COMP-001, BROK-001, TECH-001, VIOL-001, LAW-001, SOFT-001, DEVICE-001(new)
          then: ARCH-007-company, ARCH-007-broker, ARCH-007-tech,
                ARCH-007-violation, ARCH-007-law, ARCH-007-software(new)

Phase 2:  DATA-002 → DATA-001 → DATA-003
          [parallel] COMP-002, BROK-002
          LAW-002
          TECH-002(new), VIOL-002(new), SOFT-002(new)

Phase 3:  [parallel] APP-001, APP-002, APP-003, APP-004, APP-005, APP-006, APP-007(new)

Phase 4:  WEB-012 (with footer) → WEB-001 → WEB-002 → WEB-003 → WEB-004 → WEB-005
          → WEB-006A (with violations + legal-guide + software pages)
          → WEB-006B → WEB-006C (devices via APP-007) → WEB-006D → WEB-006E
          → WEB-007 → WEB-009 → WEB-010 → WEB-011

Phase 5:  INT-001 → INT-002
```

---

*Analysis complete. Total findings: 5 Critical, 10 High, 13 Medium, 10 Low.*
*Recommended new tasks: 16.*