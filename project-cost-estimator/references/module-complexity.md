# Module Complexity Reference

This reference provides calibrated effort estimates for the technology stacks commonly used by BlackBox Vision projects (NestJS, Prisma, Refine, React Native, WXT).

---

## Complexity Tiers

### Simple (20–40 hrs)
Single-domain CRUD with minimal business logic.

Examples:
- NestJS CRUD module with Prisma (1 entity, standard DTOs, no side effects)
- Refine list/create/edit screens for a single resource
- Prisma schema additions with migration
- Static admin panel section (read-only, no mutations)
- Simple notification sender (no templates, single channel)

---

### Standard (40–80 hrs)
Multi-entity logic, integrations, or non-trivial frontend flows.

Examples:
- NestJS module with role-based guards, relations, and filtering
- BullMQ queue processor with retry logic and error handling
- External REST API integration with auth, pagination, and mapping
- Refine dashboard with charts, filters, and export
- React Native screen with form validation and async state
- Auth flow (JWT + refresh tokens, role enum, guards)
- Stripe webhook handler with idempotency
- Multi-step wizard UI with validation

---

### Complex (80–160 hrs)
Concurrency, external state, security surfaces, or intricate business logic.

Examples:
- OEM portal scraper / browser automation (brittle, maintenance-heavy)
- DMS sync engine with queue deduplication and `SKIP LOCKED`
- Browser extension with cross-context auth state (WXT)
- Claim processing state machine with external callbacks
- Multi-tenant permission system with resource-level ACLs
- Real-time features (WebSockets, event sourcing)
- OAuth + SSO integration (multiple providers)
- React Native BLE communication module
- Terraform multi-environment AWS architecture

---

## Stack-Specific Notes

### NestJS
- Decorator-heavy code inflates LOC significantly. A 500-line controller may represent 20–30 actual endpoints, not 500 lines of logic.
- Generated code (OpenAPI → DTOs, Prisma client) can represent 30–50% of total LOC with near-zero development effort.
- Count `@Get`, `@Post`, etc. decorators as the reliable endpoint metric.

### Prisma
- Each model in schema.prisma represents significant generated code (client methods, types) but minimal human effort to define.
- Count `model` blocks, not generated lines.
- Migrations are effort-light unless they include complex data transformations.

### Refine (with MUI/Ant)
- CRUD screens are very fast to scaffold (5–10 hrs per resource with Refine CLI).
- Custom layouts, dashboards, and non-CRUD workflows are where real effort lives.
- Count unique screen flows, not individual components.

### WXT (Browser Extension)
- Content scripts + background service worker + popup = 3 separate execution contexts with async messaging.
- Auth state synchronization across contexts is consistently complex.
- Estimate browser extension work at 1.5–2× equivalent web complexity.

### React Native
- Platform-specific behavior (iOS vs Android) adds ~20% overhead per native-touching feature.
- BLE, camera, file system, notifications: each is Standard–Complex regardless of apparent simplicity.
- Expo managed workflow reduces infra overhead but limits native modules.

---

## Common Estimation Errors

| Error | Correction |
|-------|-----------|
| Counting generated Prisma client LOC as effort | Count model definitions only |
| Treating NestJS boilerplate as logic | Count endpoints and business rules |
| Underestimating browser extension scope | Apply 1.5–2× multiplier vs web |
| Underestimating queue/async complexity | Add concurrency risk to tier |
| Treating AI-generated boilerplate as human effort | Split AI vs human contribution |
| Using COCOMO for modern stacks | Use module-based estimation |

---

## Benchmark Reference (BlackBox Vision projects)

| Project | Stack | LOC | API endpoints | DB models | Est. effort | Actual |
|---------|-------|-----|---------------|-----------|-------------|--------|
| DealerEdge | NestJS+Prisma+Refine | ~72k | ~85 | ~16 | 800–1,200 hrs | TBD |
| Vixon | NestJS+Prisma+React Native | ~? | ~? | ~? | TBD | TBD |
| Blessbay/Blessviva | Java/TomEE+React | ~? | ~? | ~? | TBD | TBD |

*Update this table as estimates are validated against actual hours.*
