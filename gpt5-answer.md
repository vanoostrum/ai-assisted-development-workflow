Awesome brief — you’re already 80% of the way there. Below is a battle-tested workflow you can drop into Cursor (or any LLM-assisted setup) that keeps *you* in control of architecture, libraries, and quality while letting the AI do the heavy lifting.

I’ve broken it into:

1. an end-to-end development workflow (backend + frontend),
2. recommendations for the three instruction aspects (Rules, Scope, Detail), and
3. extra instruction areas people often miss (tests, security, perf, etc.).

You’ll also get ready-to-paste templates: a minimal Rule file, task prompt templates, and requirement specs for endpoints/screens.

---

# 1) Optimal development workflow

## Guiding principles

* **Single source of truth:** Make the API contract (OpenAPI/JSON Schema) *authoritative*. Generate types from it on both sides.
* **Small, verifiable steps:** Each AI task must end with a runnable artifact: a passing test, a built page, a generated schema, etc.
* **No silent drifts:** Pin library versions, enforce lint/format, require self-review checklists, and run CI on every AI PR.
* **Work within a scaffolded architecture:** You define layers, boundaries, and conventions up front; the AI must stay inside them.

## Phase 0 — Repo scaffolding (once)

* **Backend:** Create project (FastAPI or NestJS). Add layers (`/api`, `/domain`, `/services`, `/repos`, `/infra`).
* **Frontend:** Create project (Next.js / React Native). Add `/app` or `/screens`, `/components`, `/lib`, `/styles`, `/api`.
* **Shared policies:** code style (Prettier/Black), lint (ESLint/ruff), tests (pytest/Vitest/Jest), commit lint, git hooks, CI pipeline.
* **Contracts:** `/contracts/openapi.yaml`, `/contracts/schemas/*.json`.
* **Tooling:** typegen (e.g., openapi-typescript / datamodel-codegen), Swagger UI in dev, Storybook (web) or Expo + story files (mobile).
* **Version pinning:** lock files checked in; Renovate/Dependabot for controlled upgrades.

## Phase 1 — Requirements & architecture docs (authoritative inputs)

Create 3 short docs (checked into `/docs`):

1. **System Brief** – purpose, non-functional reqs (security, perf, compliance), tech choices, “do not use” list.
2. **Architecture & Conventions** – layered diagram, naming, error model, DTO vs domain model, transaction boundaries, logging and tracing shapes, feature flags approach.
3. **API Contract** – OpenAPI + JSON Schemas for all request/response bodies; start with stubs and evolve by PR.

> These docs are what your Rule file points the AI to. They keep generations consistent and cheap.

## Phase 2 — Contract-first backbone

* Draft endpoints in **OpenAPI** (even placeholders).
* Generate **types** on both sides from the contract.
* Add **contract tests** (e.g., Dredd, Schemathesis or simple pytest hitting FastAPI with the spec) and **Pact** (optional) if you want consumer-driven contract checks.

Deliverable: CI that fails if server deviates from OpenAPI or if frontend types become stale.

## Phase 3 — Backend implementation (task slices)

Recommended task slices (each = one Cursor task + one PR):

1. **Endpoint skeleton** (routing, DTOs, validators).
2. **Service logic** (business rules, pure functions).
3. **Repo + persistence** (queries, migrations, seed).
4. **Cross-cutting** (errors → problem+json, logging, tracing, authz).
5. **Tests** (unit for services, integration for endpoint flow).
6. **Docs** (endpoint examples, error catalog).

> If your service is small, you can merge 1–3 for an endpoint into a single task. For core flows (auth, payments), keep them separate to force better tests.

## Phase 4 — Frontend implementation (task slices)

1. **Design tokens & theming** extracted from Figma (colors, spacing, radii, typography) → design-system primitives.
2. **Component inventory** (from Figma frames) → atom/molecule/organism list with Storybook/Stories or Expo stories.
3. **Screen shells** (layout, routing/navigation, placeholder data).
4. **Data wiring** (hooks/services generated from OpenAPI; optimistic updates rules; error states; skeletons).
5. **Interactions & accessibility** (keyboard traps, ARIA, focus management).
6. **E2E happy-paths** (Playwright/Detox).

> Use Figma link **plus** a terse screen spec (below). Let AI propose a component map first; then approve; then implement in small PRs.

## Phase 5 — Verification loop on every PR

* **AI self-review checklist** (included below) must pass.
* **Static checks:** lint, typecheck, unused deps, bundle size guardrails.
* **Tests:** unit + integration/e2e on touched areas.
* **Contract:** regenerated client types match spec; breaking change detector flags deltas.

---

# 2) Instruction aspects

## A) Rules (the minimal, high-leverage Rule file)

Keep this short (~150–300 lines). Link to your `/docs` so it doesn’t balloon token usage.

**Goals of the Rule file**

* Fix the **thinking process**: “plan → implement → self-review → produce diff”.
* Lock **architecture boundaries** & **library choices**.
* Lock **style** and **error model**.
* Provide **checklists** (security, perf, a11y) as *bullet lists*, not essays.

**Paste-ready Rule file (trim to taste):**

```
# Cursor Rule: Engineering Doctrine (v1)

## Role
You are a senior engineer contributing small, verifiable diffs to an existing repo. 
You MUST:
- Propose a brief plan first.
- Implement only the files you list in the plan.
- Explain design trade-offs.
- Run a self-review checklist before finalizing.

## Architecture boundaries
- Backend (FastAPI/Nest): controllers -> services -> repos. No controller ↔ repo calls. 
- Domain logic in services (pure where possible). 
- DTOs == OpenAPI schemas; domain models are separate.
- Errors: return RFC7807 problem+json shape with `type`, `title`, `detail`, `instance`, `code`.

## Libraries (pin versions; do NOT introduce alternatives)
- FastAPI + Pydantic v2, SQLAlchemy 2, Alembic, httpx, pytest.
- Next.js (App Router), React, React Query/TanStack Query, Zod, Zustand/Redux (pick one per repo), Playwright.
- Do NOT add state libs, design kits, or date libs beyond the approved list without an explicit TODO and rationale.

## Coding style
- TypeScript strict, ESLint + Prettier; Python ruff + black; 100% typed.
- No dead code, no wildcard imports, no default exports except React components.
- Small pure functions; pure business logic is framework-agnostic.

## Data contracts
- Server and client types are generated from /contracts/openapi.yaml.
- Any schema change requires updating openapi.yaml, re-gen, and contract tests.

## Security & privacy checklist
- Validate all inputs (Zod/Pydantic) at boundaries. 
- Parameterized queries only; no string SQL concatenation.
- Authz before action. Never trust client flags for privileged logic.
- No secrets in code; use env with typed loader.
- Log PII with hashing/redaction rules only.

## Performance checklist
- N+1 avoided (use `.options(joinedload)` / include relations).
- Paginate endpoints that return lists.
- Cache rules: memoize pure queries, set SWR on the client as per endpoint semantics.
- Measure: add basic timing logs to hot paths.

## Frontend accessibility checklist
- Keyboard nav, focus states, ARIA labels, color contrast ≥ 4.5:1, `prefers-reduced-motion`.

## Process
1) PLAN: list changed files + brief steps.
2) IMPLEMENT: keep diff < ~200 LOC when possible.
3) SELF-REVIEW:
   - [ ] Follows architecture boundaries
   - [ ] Types aligned with OpenAPI
   - [ ] Input validation & error mapping added
   - [ ] Logging/tracing at controller boundary, not inside pure logic
   - [ ] Tests updated/added; coverage for happy + sad paths
   - [ ] No new deps (or justified)
4) OUTPUT: show final diff and commands to run tests.
```

**Why this works**

* Short, enforceable, and token-cheap.
* Anchors the agent to your architecture and libraries.
* Converts “please be careful” into a concrete checklist (huge quality boost).

## B) Scope of tasks (how big should each ask be?)

**Default:** split into small, testable tasks. Aim for **≤200 LOC / PR** or **<5 files** touched.

**When to merge tasks**

* Trivial CRUD or prototype spikes: combine routing+service+repo for a single endpoint or screen.
* Design-system refactor: do an inventory plan task, then implement in 2–3 grouped PRs (tokens/theme, primitives, complex components).

**Decomposition patterns**

* **Backend per endpoint**:

  * Task A: controller + DTO + validators
  * Task B: service (+ pure unit tests)
  * Task C: repo + migration (+ integration test)
* **Cross-cutting** (logging, authz, metrics): separate tasks with repo-wide search-and-replace guarded by tests.
* **Frontend per screen**:

  * Task A: component map proposal from Figma + Stories
  * Task B: screen layout + mocked data
  * Task C: data hook wired to API + loading/empty/error states
  * Task D: interactions/a11y polish + e2e

**Pitfalls & mitigations**

* *Drift across many small tasks:* keep a living checklist and CI gates (typegen, contract tests).
* *Context loss for larger refactors:* provide a repo map (`tree -L 3`) and explicit file paths to touch.

## C) Level of detail (how much to specify?)

* **Backend:** bias to **high detail** (exact fields, invariants, error codes, pagination, auth model).
* **Frontend:** provide **medium detail** + Figma; specify data flows and interaction rules, but let the agent propose componentization and micro-copy you can review.

**Templates (fast to write, precise enough)**

**Endpoint Requirement (per endpoint)**

```
Name: POST /v1/projects
Purpose: Create a project owned by the authenticated user.
Auth: Bearer JWT; must have scope "projects:write".
Request body (JSON): { name: string(3..80), visibility: "private"|"public", tags?: string[] max 10, description?: string ≤ 1,000 }
Validation: name trimmed; tags unique, lowercase; description markdown allowed, sanitized.
Behavior:
- Slug is kebab-case of name; unique per owner. On collision, append "-{n}".
- Owner becomes "admin" role in project_members.
- Emit domain event "project.created".
Responses:
- 201 { id, slug, ... } with Location header.
- 400 validation errors (RFC7807).
- 409 on slug collision race.
- 401/403 standard errors.
Storage:
- Tables: projects(id pk uuid), project_members(project_id, user_id, role).
- Index: projects(owner_id, slug) unique.
Tests:
- Unit: slug generation, tag normalization.
- Integration: happy path, 409 race simulated.
Observability:
- Log info on create with project_id (no PII).
```

**Screen Requirement (per screen)**

```
Screen: Project Details
Purpose: Read and manage a project.
Breakdown:
- Header: name, visibility badge, actions (Edit, Archive).
- Tabs: Overview | Members | Activity
- Overview: description (markdown), tags (clickable chips), stats (3 KPIs).
Data:
- GET /v1/projects/{id}, /v1/projects/{id}/stats
- Mutations: PATCH /v1/projects/{id}, POST /archive
States:
- Loading: skeleton for header + tabs + card
- Empty: "No description yet" CTA
- Error: inline error banner with retry
Interactions:
- Edit opens modal with zod-validated form; optimistic update on success
A11y:
- Landmark roles, focus trap in modal, ESC closes modal, tab order documented
```

---

# 3) Frontend + Figma: what to give the AI

**Give both**: a Figma frame link **and** a short, structured screen spec (above).
Also provide:

* **Design tokens** (export as JSON or list) and spacing scale.
* **Component inventory request:** Ask the AI to output a *component map* first (atoms → molecules → organisms with props). Approve it, then implement in small tasks.
* **UI states list:** loading/empty/error/success; required skeletons and placeholders.
* **Data contract reference:** link to generated `@api/types` and `@api/client` usage pattern (e.g., TanStack Query hooks).

**Formulation example (task prompt)**

> “Using Figma frame X and the Screen Requirement doc, propose a component inventory (atoms/molecules/organisms) with prop types and states, then implement only the atoms with Storybook stories. Do not fetch real data yet. Keep the diff under 200 LOC.”

This yields better structure than “build the page from this Figma link”.

---

# 4) Concrete task prompt templates (Cursor)

**Backend – endpoint slice**

```
Goal: Implement controller + DTO + validation for POST /v1/projects as per /docs/api/requirements/projects.md.

Inputs:
- Codebase path: backend/
- Contract: /contracts/openapi.yaml (do not diverge)
- Architecture: controllers -> services -> repos (no skipping)
- Libraries: FastAPI, Pydantic v2, SQLAlchemy, pytest

Deliverable:
- Files to create/edit: list them first.
- New tests: pytest unit for validators and controller happy/sad paths.
- Update openapi.yaml and regenerate types if needed.
Constraints:
- Problem+json errors only; map pydantic errors to RFC7807.
- Keep diff ~≤200 LOC.

Process:
1) Show brief plan + files to touch.
2) Implement.
3) Run self-review checklist (from Rules) and show test commands.
```

**Backend – service + repo slice**

```
Goal: Add service logic and SQLAlchemy repo for creating a project (see /docs/api/requirements/projects.md).

Deliverable:
- Service with pure functions where possible (unit tests).
- Repo with transaction boundary in service.
- Migration for tables/indexes.
- Integration tests that hit the FastAPI app.

Constraints: No direct controller→repo calls; handle concurrency 409.
```

**Frontend – screen shell**

```
Goal: Implement Project Details screen shell from Figma frame <url> and screen spec /docs/ui/project-details.md.

Deliverable:
- Route/page (Next.js app router) with layout and placeholders.
- Components: Header, Tabs, Overview card (static), Actions bar.
- Stories for components.
- No real fetching yet; use mocked props.

Constraints:
- Stateless visual shell only.
- a11y: focus order, keyboard reachable tabs.
```

**Frontend – data wiring**

```
Goal: Wire Project Details screen to API using generated client.

Deliverable:
- React Query hooks for GET/patch endpoints.
- Loading/empty/error states with skeletons.
- Optimistic update for edit modal.
- Playwright test for happy path.

Constraints:
- Use @api/client generated from OpenAPI; no ad-hoc fetch.
```

---

# 5) What people miss (add these to your instructions)

* **CI gates as guardrails**

  * Typegen must be up to date (fail if diff).
  * Contract tests (server ↔ OpenAPI).
  * Lint, typecheck, unit + integration, bundle size budgets (frontend).
  * Secret scan (gitleaks/truffleHog).

* **Error model consistency**

  * Standardize one error envelope (RFC7807), map all exceptions to it, and test it.

* **Observability**

  * Correlation IDs, request/response logging (redacted), trace spans around controllers and external calls. Basic metrics counters.

* **Security posture**

  * Input validation everywhere, output encoding, strict CORS, rate limiting for public endpoints, permission checks in services, secure cookies/headers (Helmet/secure middleware), CSRF where relevant.

* **Data migration discipline**

  * “Migration PRs” are separate from “feature PRs” unless trivial; require rollback scripts for risky changes.

* **Feature flags**

  * Wrap risky UI/feature work behind flags; document kill-switch steps.

* **Accessibility & i18n**

  * Include a short a11y checklist in the Rule file; basic i18n plumbing even if only one locale at start.

* **Release tracks**

  * Staging env with seeded data; preview deployments per PR (Vercel/Fly/Render); smoke tests on preview.

* **Prompt rot control**

  * Treat Rule file like code: version it, keep it short, and link out to `/docs` for the details you’ll refine over time.

---

# 6) Pros/cons & pitfalls of the recommended approach

**Split tasks (recommended)**

* ✅ Easier review, reproducible, less hallucination, cheaper tokens per run.
* ❌ More orchestration overhead; risk of spec drift → mitigated by CI contract gates and typegen.

**High-detail backend requirements**

* ✅ Predictable output, fewer rewrites, better tests.
* ❌ More upfront writing; but pays off by enabling codegen + contract tests.

**Medium-detail frontend + Figma**

* ✅ Lets the AI propose reasonable componentization; you guide it with small corrections.
* ❌ Risk of UI mismatch → mitigated by component inventory step & stories.

**Short Rule file**

* ✅ Lower token cost, fewer contradictions, easier to maintain.
* ❌ Requires good `/docs`—but that’s exactly what you want versioned in the repo.

---

# 7) Quick-start checklist (copy this into `/docs/AI_CHECKLIST.md`)

* [ ] Rule file present and linked to System Brief, Architecture, OpenAPI.
* [ ] CI: lint, typecheck, tests, contract tests, typegen check, secret scan.
* [ ] OpenAPI is the only source for client/server types.
* [ ] Each AI task ≤200 LOC and ends with verifiable tests or stories.
* [ ] Error model unified (RFC7807).
* [ ] Logging/tracing minimal but present; PII redacted.
* [ ] Figma → component inventory → atoms/molecules → screen shells → data wiring → e2e.
* [ ] Migrations reviewed separately when risky; rollback documented.
* [ ] Feature flags for risky UI.

---

If you want, I can turn this into:

* a concrete FastAPI + Next.js starter with the Rule file,
* minimal OpenAPI + generated clients,
* CI config and example tasks as Markdown in `/tasks`.

Say the word and tell me your preferred stack (FastAPI or Nest; Zustand vs Redux; DB), and I’ll generate the starter layout + the initial Rule file and task docs.
