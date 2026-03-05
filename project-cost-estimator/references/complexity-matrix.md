# Complexity Matrix & Scoring Guide

Used in Phase 3 of the project cost estimator to score projects across 8 dimensions.

---

## Scoring Scale: 1–5

| Score | Label | Meaning |
|-------|-------|---------|
| 1 | Trivial | Minimal, standard patterns only |
| 2 | Simple | Some custom logic, standard integrations |
| 3 | Moderate | Multiple integrations, domain-specific logic |
| 4 | Complex | Non-trivial architecture, many moving parts |
| 5 | Very Complex | Distributed systems, novel solutions, high-stakes |

---

## Dimension Scoring Guide

### 1. Integration Complexity
Signals from code: API client files, webhook handlers, OAuth flows, event streams.

| Score | Description | Examples |
|-------|-------------|---------|
| 1 | No external integrations | Static site, CLI tool |
| 2 | 1–2 simple REST APIs | Single payment gateway, one SaaS API |
| 3 | 3–5 APIs or 1 complex integration | Multiple payment methods, Stripe + auth + email |
| 4 | 5–10 APIs, real-time data, webhooks | ERP integration, multi-platform sync |
| 5 | 10+ APIs, streaming, EDI, legacy systems | Enterprise middleware, marketplace platform |

### 2. Data Model Complexity
Signals: Schema files, migration count, number of tables/collections, join depth.

| Score | Description | Examples |
|-------|-------------|---------|
| 1 | <10 entities, flat structure | Simple CRUD app |
| 2 | 10–20 entities, basic relations | Blog, simple e-commerce |
| 3 | 20–40 entities, polymorphism, soft deletes | Multi-tenant SaaS |
| 4 | 40–80 entities, complex inheritance, versioning | Healthcare records, financial ledger |
| 5 | 80+ entities, event sourcing, temporal data, graph | Banking core, ERP, marketplace |

### 3. Frontend Complexity
Signals: Component count, state management libs, animation libs, responsive breakpoints.

| Score | Description | Examples |
|-------|-------------|---------|
| 1 | Server-rendered, minimal JS | Marketing site, simple form |
| 2 | Basic SPA, standard components | Admin panel, dashboard |
| 3 | Complex state, multiple views, responsive | SaaS product, e-commerce storefront |
| 4 | Real-time updates, drag-drop, data viz | Analytics platform, project management |
| 5 | Canvas/WebGL, complex animations, PWA offline | Design tool, trading terminal, game |

### 4. Infrastructure Complexity
Signals: Terraform/CloudFormation files, docker-compose services, CI/CD pipelines.

| Score | Description | Examples |
|-------|-------------|---------|
| 1 | Single server or PaaS (Heroku/Railway/Vercel) | Basic web app |
| 2 | Basic cloud (1–2 services, single env) | EC2 + RDS, single region |
| 3 | Multi-env (dev/staging/prod), basic IaC | Containerized app with CI/CD |
| 4 | Multi-region, auto-scaling, caching layer, queues | Production SaaS infrastructure |
| 5 | Multi-cloud, Kubernetes, zero-downtime deployments | Enterprise-grade platform |

### 5. Business Logic Density
Signals: Number of service/domain files vs total files, test count, condition depth.

| Score | Description | Examples |
|-------|-------------|---------|
| 1 | CRUD only, no domain rules | Simple form → email app |
| 2 | Light business rules, some calculations | Basic pricing, simple workflows |
| 3 | Multi-step workflows, approval chains | Order management, claims processing |
| 4 | Complex state machines, pricing engines, rule engines | Insurance, fintech calculations |
| 5 | Regulatory compliance embedded, actuarial logic | Core banking, healthcare billing |

### 6. Security Surface
Signals: Auth files, permission middleware, encryption utils, audit log tables.

| Score | Description | Examples |
|-------|-------------|---------|
| 1 | Public app, no auth | Static content site |
| 2 | Basic auth (username/password), single role | Simple SaaS with one user type |
| 3 | Role-based access, OAuth/SSO, session management | Multi-role SaaS |
| 4 | Fine-grained permissions, audit logging, MFA | Enterprise SaaS, financial app |
| 5 | Zero-trust, compliance (SOC2/HIPAA/PCI), encryption at rest | HealthTech, FinTech, GovTech |

### 7. Test Coverage Maturity
Signals: Test file count / total file ratio, test framework presence, CI test stages.

| Score | Description | Test file ratio |
|-------|-------------|----------------|
| 1 | No tests or <5% | <0.05 |
| 2 | Some unit tests, no integration | 0.05–0.15 |
| 3 | Unit + integration tests, CI | 0.15–0.30 |
| 4 | High coverage, e2e, contract tests | 0.30–0.50 |
| 5 | TDD discipline, mutation testing, perf tests | >0.50 |

### 8. Technical Debt
Signals: TODO/FIXME count, deprecated package versions, commented-out code blocks.

| Score | Description | Signs |
|-------|-------------|-------|
| 1 | Clean codebase, up-to-date deps | <10 TODOs, modern stack |
| 2 | Minor debt, some old deps | 10–50 TODOs, 1–2 major versions behind |
| 3 | Moderate debt, some workarounds | 50–150 TODOs, significant refactor backlog |
| 4 | High debt, fragile areas, mixed patterns | 150–400 TODOs, critical patches pending |
| 5 | Critical debt, rewrites needed | 400+ TODOs, end-of-life dependencies |

---

## Complexity multipliers

| Average Score | Label | Effort Multiplier |
|---------------|-------|-------------------|
| 1.0–1.5 | Very simple | × 0.7 |
| 1.5–2.0 | Simple | × 0.85 |
| 2.0–2.8 | Moderate | × 1.0 (baseline) |
| 2.8–3.5 | Above average | × 1.25 |
| 3.5–4.2 | Complex | × 1.6 |
| 4.2–4.7 | Very complex | × 2.0 |
| 4.7–5.0 | Extreme | × 2.5+ |

---

## Stack complexity multipliers

Apply on top of COCOMO base estimate to account for language/stack productivity:

| Stack | Multiplier | Notes |
|-------|-----------|-------|
| Python (Django/FastAPI) | × 0.85 | Highly productive |
| Ruby on Rails | × 0.80 | Convention over config |
| Node.js / TypeScript | × 0.90 | Strong ecosystem |
| React / Next.js | × 0.90 | |
| Go | × 0.95 | Fast but verbose |
| Java (Spring Boot) | × 1.10 | More boilerplate |
| Kotlin (Spring) | × 1.00 | Better than Java |
| PHP (Laravel) | × 0.90 | |
| C# (.NET) | × 1.05 | |
| Rust | × 1.40 | High productivity cost |
| C / C++ | × 1.50 | |
| COBOL / Legacy | × 2.00+ | |
| React Native | × 1.15 | Cross-platform penalty |
| Flutter | × 1.10 | |

For mixed stacks: use weighted average by LOC proportion.

---

## LOC productivity benchmarks (lines per person-day)

For the cross-validation model. These are **net new production code** (not tests or config):

| Domain | LOC/person-day | Notes |
|--------|----------------|-------|
| Business apps (CRUD heavy) | 150–400 | High reuse, standard patterns |
| API / backend services | 100–250 | Integration work slows things |
| Frontend UI | 80–200 | Design review cycles add time |
| DevOps / IaC | 60–150 | Testing infra takes longer |
| ML / data pipelines | 50–120 | Experimental cycles |
| Security / auth systems | 40–100 | Review-heavy |
| High-complexity domain logic | 30–80 | Fintech, healthcare |

Use 200 LOC/person-day as the default for mixed modern stacks if unsure.
Divide by 8 to get LOC per hour.
