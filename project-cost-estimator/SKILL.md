---
name: project-cost-estimator
description: >
  Analyzes a software project's source code, git history, and architecture to
  estimate what it actually cost to build — and compares that against what it
  would cost using a modern AI agent fleet (Claude Code, OpenCode, Goose, Codex,
  Kimi Code) managed by a single engineer. Use this skill whenever the user wants
  to: audit a project's build cost, estimate software value, understand the ROI of
  AI-assisted development, calculate gross margins from AI vs human teams, evaluate
  outsourcing vs in-house costs, or benchmark a codebase's investment. Trigger on
  phrases like "how much did this cost to build", "estimate project cost", "AI vs
  human development cost", "gross margin from using agents", "what's this codebase
  worth", "value of the code", or "audit this project's effort".
---

# Project Cost Estimator

Performs forensic analysis of a software project — source code, git history,
architecture — and produces two estimates side by side:

1. **Human build cost** — what the project actually cost (or would cost) with
   the real team profile (agency, in-house, freelancers, nearshore, offshore, etc.)
2. **AI agent fleet cost** — what the same project would cost using a managed
   fleet of AI coding agents, and the **gross margin delta** that reveals.

---

## Phase 0 — Gather inputs before touching the code

Before running any analysis, collect two things from the user:

### A. Project path

Ask for the full path to the project directory if not already provided. If the
user is in Claude Code, it may be the current working directory.

### B. Team profile interview

Ask the following questions (you can batch them in one message):

```
To calibrate the cost model I need to understand the team that built this:

1. **Team type** — which best describes who built it?
   - [ ] Internal / in-house engineering team
   - [ ] Agency (full-service software agency)
   - [ ] Nearshore agency or contractor team
   - [ ] Offshore agency or contractor team (e.g., India, Eastern Europe, LATAM)
   - [ ] Freelancers (assembled team or solo)
   - [ ] Mixed (describe)

2. **Geography / market** — where was the primary team based?
   (This drives the hourly rate range. E.g., "NYC", "Buenos Aires", "Kyiv", "Bangalore")

3. **Do you know the approximate hourly rate?**
   - If yes → provide it (we'll use it directly)
   - If no → I'll estimate from team type + geography using market benchmarks

4. **Known actuals (optional but improves accuracy)**
   - Rough calendar duration of active development (e.g., "8 months")
   - Approximate team size during peak (e.g., "3 devs + 1 designer")
   - Any known invoiced amount or budget (kept private, only used for calibration)
```

Store answers in a `team_profile` object before proceeding.

---

## Phase 1 — Source Code Analysis

Run these commands against the project directory. Adapt paths as needed.

### 1.1 Basic inventory

```bash
# File count and types
find <PROJECT_DIR> -type f | grep -v node_modules | grep -v .git | grep -v __pycache__ \
  | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -30

# Lines of code by language (use cloc if available, else wc fallback)
cloc <PROJECT_DIR> --exclude-dir=node_modules,.git,dist,build,vendor \
  --json 2>/dev/null || \
  find <PROJECT_DIR> -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" \
    -o -name "*.py" -o -name "*.go" -o -name "*.java" -o -name "*.rs" \
    -o -name "*.cs" -o -name "*.rb" -o -name "*.php" \) \
  | grep -v node_modules | grep -v .git \
  | xargs wc -l 2>/dev/null | tail -1
```

### 1.2 Architecture fingerprint

```bash
# Directory structure (top 2 levels)
find <PROJECT_DIR> -maxdepth 3 -type d | grep -v node_modules | grep -v .git \
  | grep -v __pycache__ | sort

# Key config files (detect frameworks/stack)
find <PROJECT_DIR> -maxdepth 2 -name "package.json" -o -name "requirements.txt" \
  -o -name "Cargo.toml" -o -name "go.mod" -o -name "pom.xml" \
  -o -name "build.gradle" -o -name "Gemfile" -o -name "composer.json" \
  -o -name "*.csproj" -o -name "Dockerfile" -o -name "docker-compose*.yml" \
  -o -name "terraform.tf" -o -name "*.tf" \
  | grep -v node_modules | head -20

# Read dependency manifest(s) for tech stack identification
# (read package.json, requirements.txt, etc. — pick the relevant one)
```

### 1.3 Complexity signals

```bash
# Largest files (often most complex)
find <PROJECT_DIR> -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.py" \
  -o -name "*.go" -o -name "*.java" \) \
  | grep -v node_modules | grep -v .git \
  | xargs wc -l 2>/dev/null | sort -rn | head -20

# Number of test files
find <PROJECT_DIR> -type f \( -name "*.test.*" -o -name "*.spec.*" -o -name "*_test.*" \
  -o -path "*/tests/*" -o -path "*/__tests__/*" \) \
  | grep -v node_modules | wc -l
```

Read the following files if they exist to understand architecture intent:
`README.md`, `ARCHITECTURE.md`, `docs/`, `ADR/` (Architecture Decision Records),
`CLAUDE.md`, `.claude/`.

---

## Phase 2 — Git History Analysis

```bash
# Contributor count and commit volume
git -C <PROJECT_DIR> log --pretty=format:"%ae" | sort | uniq -c | sort -rn

# Total commits
git -C <PROJECT_DIR> rev-list --count HEAD

# Date range of development
git -C <PROJECT_DIR> log --pretty=format:"%ad" --date=short | tail -1   # first commit
git -C <PROJECT_DIR> log --pretty=format:"%ad" --date=short | head -1   # last commit

# Commit frequency over time (weekly buckets)
git -C <PROJECT_DIR> log --pretty=format:"%ad" --date=format:"%Y-%W" | \
  sort | uniq -c

# Average commits per week (active period)
git -C <PROJECT_DIR> log --pretty=format:"%ad" --date=format:"%Y-%W" | \
  sort -u | wc -l

# Lines added/removed (effort signal)
git -C <PROJECT_DIR> log --pretty=tformat: --numstat | \
  awk 'NF==3 {added+=$1; removed+=$2} END {print "added:"added, "removed:"removed}'

# Branches (indicates parallel workstreams)
git -C <PROJECT_DIR> branch -r | wc -l
```

Parse this data to extract:
- **Active development window** (first to last commit with activity > 2 commits/week)
- **Contributor count** (unique email addresses)
- **Commit cadence** (indicates whether this was full-time or part-time work)
- **Net new code** (added - removed, after normalizing for refactors)

---

## Phase 3 — Architecture Assessment

Based on the collected data, classify the project across these dimensions.
Use references/complexity-matrix.md for scoring guidance.

| Dimension | Score (1–5) | Notes |
|-----------|-------------|-------|
| **Integration complexity** | | APIs, third-party services, webhooks |
| **Data model complexity** | | Schema depth, relations, migrations |
| **Frontend complexity** | | UI depth, state management, responsiveness |
| **Infrastructure complexity** | | Cloud, IaC, CI/CD, multi-env |
| **Business logic density** | | Domain rules, workflows, edge cases |
| **Security surface** | | Auth, permissions, compliance |
| **Test coverage maturity** | | Unit, integration, e2e |
| **Technical debt** | | Code smells, TODOs, deprecated patterns |

**Complexity multiplier** = average score across dimensions (feeds into cost model).

---

## Phase 4 — Human Cost Estimation

### 4.1 Effort estimation (bottom-up)

Use the COCOMO-inspired model adjusted for modern stacks:

```
Effective KLOC = (net new lines of code) / 1000
                 adjusted for language (see references/loc-weights.md)

Base effort (person-months) = 2.4 × (KLOC ^ 1.05)   ← organic mode baseline
Adjusted effort = base × complexity_multiplier × stack_multiplier
```

Cross-validate against git signals:
- Active weeks × contributor count × estimated hours/week (40 for full-time,
  20 for part-time based on commit cadence)

Take the **average of the two estimates** as the working effort figure.
Report both with a ± confidence range.

### 4.2 Rate lookup

Read `references/rate-table.md` for market hourly rate ranges by team type
and geography. Use the midpoint unless the user gave an actual rate.

### 4.3 Human cost output

```
Total effort:       X person-months  (Y person-hours)
Blended hourly rate: $Z/hr
────────────────────────────────────
Estimated build cost:  $A  (low: $B, high: $C)
```

---

## Phase 5 — AI Agent Fleet Cost Estimation

Read `references/agent-cost-models.md` for current pricing data.

### 5.1 Task decomposition

Based on the architecture assessment, break the project into task categories:

| Category | % of effort | AI suitability |
|----------|-------------|----------------|
| Boilerplate / scaffolding | ~15% | Very high (95%) |
| CRUD / standard patterns | ~20% | Very high (90%) |
| Integration work | ~20% | High (75%) |
| Business logic | ~25% | Medium (60%) |
| UI / UX implementation | ~10% | Medium-High (70%) |
| Testing | ~5% | High (80%) |
| DevOps / infra | ~5% | Medium (65%) |

**AI coverage** = weighted average suitability across categories.

### 5.2 Cost per agent

For each agent in the standard fleet, calculate:
- Token cost = estimated tokens per task × token price
- Tool call cost (if applicable)
- Assumed human oversight ratio (how many hours of manager time per agent-hour)

Agents to model (see `references/agent-cost-models.md` for current rates):

| Agent | Model backend | Cost model | Best for |
|-------|--------------|------------|----------|
| **Claude Code** | claude-sonnet/opus | Per token (input/output) | Complex reasoning, architecture |
| **OpenCode** | configurable | Per token | Multi-model routing, flexibility |
| **Goose** | configurable | Per token + tool calls | Agentic workflows, file ops |
| **Codex (OpenAI)** | gpt-4o/o3 | Per token | General coding, JS/Python |
| **Kimi Code** | Kimi k1.5/k2 | Per token (cheaper tier) | High volume, cost-sensitive tasks |

### 5.3 Fleet composition recommendation

Suggest an optimal agent mix for this specific project type.

### 5.4 Manager cost

One senior engineer manages the fleet:
- Geography: ask user, or default to same geography as original team
- Time allocation: typically 20–35% of original dev hours
  (prompting, reviewing, correcting, integrating)

### 5.5 AI fleet cost output

```
AI agent compute cost:      $A
Human manager cost:         $B  (X hrs × $Y/hr)
─────────────────────────────
Total AI fleet cost:        $C

vs. Human build cost:       $D
─────────────────────────────
Cost reduction:             $E  (F%)
Gross margin captured:      G%  (if this is client work)
Speed multiplier:           ~Xh (estimated calendar time compression)
```

---

## Phase 6 — Final Report

Produce a structured report with the following sections. Save to
`project-cost-report.md` (or `.pdf` if the pdf skill is available).

```markdown
# Project Cost Analysis: <Project Name>
Generated: <date>

## Executive Summary
[3-sentence summary of key findings]

## Project Profile
- Stack: ...
- Architecture type: ...
- Active development: <start> → <end> (<N> months)
- Team: <type>, <location>
- Codebase: <KLOC> KLOC across <N> files (<languages>)

## Effort Analysis
[Table: estimation method | person-hours | confidence]

## Human Build Cost
[Cost breakdown with assumptions]

## AI Agent Fleet Estimate
[Per-agent breakdown, manager cost, total]

## Comparative Analysis
[Side-by-side table, margin delta, speed comparison]

## Recommended Agent Fleet for This Project
[Specific recommendation with rationale]

## Assumptions & Caveats
[What could shift these numbers]
```

---

## Key principles

- **Always show your work.** Every number should trace back to a specific
  data source (git log output, LOC count, rate table).
- **Confidence intervals matter.** Never give a single number without a range.
- **Be honest about AI limits.** Not everything is automatable. The model
  explicitly tracks what AI agents can't do well.
- **Rates drift.** Agent pricing changes frequently — note the reference date
  in `agent-cost-models.md` and flag if data may be stale.
- **Context is king.** A 50K LOC codebase from a fintech startup costs very
  differently than 50K LOC of a CRUD admin panel.
