# Agent Readiness — Scoring Rubric

Each dimension is scored 0–10. Use the breakpoints as anchors; interpolate for partial implementations.

---

## Dimension 1: Architecture & Monorepo Structure

**What it measures**: Whether the project is organized so that agents can navigate boundaries, understand dependencies, and make localized changes without unintended side effects.

| Score | Condition |
|-------|-----------|
| 9–10 | Monorepo with explicit workspace tooling (Turborepo, Nx, pnpm workspaces). Clear package separation. Naming conventions are consistent and self-documenting. Dependency graph is clean (low coupling). |
| 7–8 | Monorepo structure present but tooling is minimal. Boundaries are mostly clear but a few packages are ambiguously scoped. |
| 5–6 | Multiple packages exist but without workspace tooling. Some shared code is duplicated. Naming is inconsistent in places. |
| 3–4 | Single large package with unclear internal structure. Modules are entangled. |
| 0–2 | No discernible structure. Everything in one directory. No separation of concerns. |

**Signals to look for**: `turbo.json`, `pnpm-workspace.yaml`, `nx.json`, `lerna.json`, clearly named `apps/` and `packages/` directories, consistent directory naming conventions.

---

## Dimension 2: Structural Documentation

**What it measures**: Whether an agent could reconstruct a working mental model of the system by reading only the available documentation — no tribal knowledge required.

| Score | Condition |
|-------|-----------|
| 9–10 | README covers: product context, problem solved, architecture overview, stack, key features, available commands, local setup, deployment process, internal conventions. Also has CLAUDE.md or AGENTS.md with agent-specific context (project conventions, constraints, decision rationale). |
| 7–8 | README is solid but missing 1-2 key sections (e.g., architecture or conventions). No agent-specific context file, but documentation is enough to work from. |
| 5–6 | README exists but is generic (just a name + install instructions). No architectural context. Setup is described but incomplete. |
| 3–4 | Minimal README. Key decisions and constraints are undocumented. |
| 0–2 | No README or only a placeholder. |

**Signals to look for**: `README.md` length and sections, presence of `CLAUDE.md` / `AGENTS.md` / `.claude/`, architecture diagrams, inline code comments explaining *why* not just *what*.

---

## Dimension 3: Script Standardization

**What it measures**: Whether the project exposes a consistent, executable interface that agents and pipelines can rely on — without needing to understand the underlying toolchain.

| Score | Condition |
|-------|-----------|
| 9–10 | Root and all sub-packages expose: `lint`, `typecheck`, `build`, `test`. Scripts are composable via pipeline tool (e.g., `turbo run test`). All scripts exit with correct codes (non-zero on failure). |
| 7–8 | All four scripts exist at root level. Sub-packages may be inconsistent. Turbo pipeline not yet configured. |
| 5–6 | Some scripts present (`build`, `test`) but `lint` or `typecheck` are missing or named differently across packages. |
| 3–4 | Scripts exist but are inconsistently named, combined (e.g., `lint-and-build`), or silently succeed on failure. |
| 0–2 | No standardized scripts. Developers run ad hoc commands. |

**Signals to look for**: `"scripts"` section in `package.json`, consistency across workspace packages, `turbo.json` pipeline definitions, CI steps that call standardized commands.

---

## Dimension 4: Commit & Versioning Conventions

**What it measures**: Whether the commit history is a reliable, machine-readable audit trail — allowing agents to analyze history, generate changelogs, and understand the semantic impact of past changes.

| Score | Condition |
|-------|-----------|
| 9–10 | commitlint (or equivalent) enforced via git hooks or CI. All recent commits follow Conventional Commits format. CHANGELOG is auto-generated or consistently maintained. Versioning is semantic. |
| 7–8 | Commit format is mostly consistent. No enforcement but developers follow the convention. Changelog may be manual. |
| 5–6 | Some commits follow a format; others are informal ("fix", "wip", "updates"). No enforcement. |
| 3–4 | No convention. Commit messages are random and non-descriptive. |
| 0–2 | Commit history is meaningless noise. |

**Signals to look for**: `commitlint.config.*`, `.husky/`, `.git/hooks/commit-msg`, `CHANGELOG.md`, git log output, `standard-version` or `release-it` config.

---

## Dimension 5: PR & Review Process

**What it measures**: Whether the code review process is proportional to impact — allowing low-risk changes to flow quickly while protecting architectural integrity on high-impact changes.

| Score | Condition |
|-------|-----------|
| 9–10 | Explicit PR classification system (or equivalent) that distinguishes operational vs. structural vs. strategic changes. Pipeline gates enforce quality before human review. High percentage of small, focused PRs. Auto-merge or expedited flow exists for low-risk changes. |
| 7–8 | PRs are generally small and focused. Pipeline must pass before merge. No formal classification but team applies judgment appropriately. |
| 5–6 | PRs exist and CI is required to pass, but no differentiation between change types. Review is uniform regardless of impact. |
| 3–4 | PRs are large and infrequent. Review is inconsistent. CI may exist but isn't strictly enforced. |
| 0–2 | No PR process. Direct pushes to main. No review. |

**Signals to look for**: Branch protection rules, PR templates, `.github/pull_request_template.md`, CI required status checks, PR size patterns in git history, auto-merge configuration.

---

## Dimension 6: Automated Validation Pipeline

**What it measures**: Whether the project has reliable, automated gates that catch problems before they reach production — giving agents a trustworthy feedback loop.

| Score | Condition |
|-------|-----------|
| 9–10 | CI/CD runs: lint, typecheck, build, test, static analysis (SonarQube or similar), dependency audit (Dependabot or similar), automated PR review (CodeRabbit or similar). All gates must pass for merge. |
| 7–8 | CI runs lint + typecheck + build + test. Security/static analysis may be missing. All required to pass. |
| 5–6 | CI exists but only runs build and/or test. Lint/typecheck may be skipped. Some checks are optional. |
| 3–4 | CI exists but is flaky, rarely passes, or is not enforced on merge. |
| 0–2 | No CI/CD. Validation is entirely manual. |

**Signals to look for**: `.github/workflows/`, `.gitlab-ci.yml`, `sonar-project.properties`, `.github/dependabot.yml`, CodeRabbit config (`.coderabbit.yaml`), required status checks.

---

## Dimension 7: Test Coverage Culture

**What it measures**: Whether the project has a genuine testing discipline — not just test files that exist, but tests that encode intent and enable safe refactoring.

| Score | Condition |
|-------|-----------|
| 9–10 | Tests exist alongside the code they cover. Tests are meaningful (not just smoke tests). Coverage thresholds enforced. Test-first or test-concurrent workflow evident from file dates and commit history. E2E tests present for critical flows. |
| 7–8 | Good unit/integration test coverage. Tests reflect real functionality. No coverage thresholds enforced but coverage is reasonable. |
| 5–6 | Tests exist but coverage is spotty. Many important modules lack tests. Tests may be outdated. |
| 3–4 | A handful of test files exist but they're superficial or rarely updated. |
| 0–2 | No tests. Or test files exist but are all empty/disabled. |

**Signals to look for**: `*.test.*` / `*.spec.*` files near source files, `jest.config.*`, `vitest.config.*`, `playwright.config.*`, coverage reports, `coverageThreshold` in config.

---

## Dimension 8: Specification & Intent Clarity

**What it measures**: Whether the project surfaces clear, structured intent — making it possible for agents to understand *what* is being built, *why* decisions were made, and *what success looks like* for any given task.

| Score | Condition |
|-------|-----------|
| 9–10 | Issues/tickets include clear problem statements, acceptance criteria, and context. ADRs (Architecture Decision Records) document major decisions. PRs reference specs. Task descriptions are decomposed and verifiable. |
| 7–8 | Most issues include enough context to work from. Major decisions have some documentation. PRs have meaningful descriptions. |
| 5–6 | Issues exist but are often vague ("fix the bug", "update X"). Some decisions are documented informally in PR comments. |
| 3–4 | Tasks are informal and undocumented. Intent lives in Slack or memory. |
| 0–2 | No structured task management. Work is entirely implicit. |

**Signals to look for**: Issue tracker quality (can be inferred from PR descriptions that reference issues), `docs/decisions/` or `docs/adr/` directories, PR templates with required fields, task descriptions in referenced ticket systems.

