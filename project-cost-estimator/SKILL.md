---
name: project-cost-estimator
description: Analyzes a software project's source code, git history, and architecture to estimate what it actually cost to build — and compares that against what it would cost using a modern AI agent fleet (Claude Code, OpenCode, Goose, Codex, Kimi Code) managed by a single engineer. Use this skill whenever the user wants to: audit a project's build cost, estimate software value, understand the ROI of AI-assisted development, calculate gross margins from AI vs human teams, evaluate outsourcing vs in-house costs, or benchmark a codebase's investment. Trigger on phrases like "how much did this cost to build", "estimate project cost", "AI vs human development cost", "gross margin from using agents", "what's this codebase worth", "value of the code", or "audit this project's effort".
---

# Project Cost Estimator

## Purpose

This skill estimates the **real engineering cost** of a software project and evaluates whether it was executed efficiently. It is designed for **internal agency use** — not for client-facing presentations.

The core question it answers: **Did we execute this project well, financially?**

Secondary questions:
- Was AI leverage maximized?
- Where was effort concentrated relative to complexity?
- What is our gross margin at market rate vs. our actual cost?
- What would it cost to rebuild this today, with a modern AI fleet?

The tone of all output is **analytical and neutral** — not consultative or judgmental.

---

## Step 1: Gather Project Signals

### 1a. Repository metrics

```bash
# Project root
ls -la <project-root>

# Total LOC by language (cloc is preferred)
cloc <project-root> --exclude-dir=node_modules,dist,.next,coverage,generated

# TypeScript/JS files only
find <project-root> -name "*.ts" -o -name "*.tsx" | xargs wc -l 2>/dev/null | tail -1

# Git commit history
git -C <project-root> log --oneline | wc -l
git -C <project-root> log --format="%ad" --date=short | sort -u | head -1
git -C <project-root> log --format="%ad" --date=short | sort -u | tail -1

# Commit size distribution (lines changed per commit)
git -C <project-root> log --shortstat --oneline | grep "files changed" | \
  awk '{sum+=$4+$6; count++} END {print "Total lines changed:", sum, "| Commits:", count, "| Avg:", sum/count}'

# Author contributions
git -C <project-root> shortlog -sn --no-merges | head -10

# High-churn files (modified most often)
git -C <project-root> log --name-only --format="" | sort | uniq -c | sort -rn | head -20

# Branch history
git -C <project-root> branch -a | wc -l
```

### 1b. Feature surface metrics

These are far stronger effort predictors than LOC for modern TypeScript stacks.

```bash
# API endpoints (NestJS controllers)
grep -r "@Get\|@Post\|@Put\|@Patch\|@Delete" <project-root>/src --include="*.ts" -l | wc -l
grep -r "@Get\|@Post\|@Put\|@Patch\|@Delete" <project-root>/src --include="*.ts" | wc -l

# Database models (Prisma)
grep -c "^model " <project-root>/prisma/schema.prisma 2>/dev/null

# Background jobs/queues
grep -r "@Processor\|@Queue\|BullModule\|@InjectQueue" <project-root>/src --include="*.ts" -l | wc -l

# External integrations (HTTP clients, SDKs)
grep -r "HttpModule\|axios\|fetch\|new.*Client\|SDK" <project-root>/src --include="*.ts" -l | sort -u | wc -l

# UI screens / pages (Refine/Next/React)
find <project-root> -name "*.tsx" -path "*/pages/*" -o -name "*.tsx" -path "*/views/*" 2>/dev/null | wc -l

# Role-based actions / guards
grep -r "@Roles\|@UseGuards\|RolesGuard\|Permissions" <project-root>/src --include="*.ts" | wc -l

# State machines / complex flows
grep -r "createMachine\|useMachine\|state.*machine\|FSM\|StateMachine" <project-root>/src --include="*.ts" -l | wc -l

# Tests
find <project-root> -name "*.spec.ts" -o -name "*.test.ts" | wc -l
```

### 1c. Module inventory

Manually identify and categorize all modules into three complexity tiers:

| Tier | Definition | Effort Range |
|------|-----------|--------------|
| Simple | CRUD module, single entity, no side effects | 20–40 hrs |
| Standard | Multi-entity, business logic, integrations | 40–80 hrs |
| Complex | State machines, external APIs, concurrency, auth flows | 80–160 hrs |

Use the feature surface metrics above to inform module classification. **Do not use LOC to classify modules.**

---

## Step 2: Build the System Scope Snapshot

Before any estimation, produce a single clear table that summarizes project scope. This is the anchor for all subsequent analysis.

```
System Scope
────────────────────────────────
Applications (apps/packages):  N
API modules:                   N
Database models:               N
Background queues:             N
External API integrations:     N
User roles:                    N
Primary workflows:             N
UI screens (estimated):        N
Browser extension flows:       N
────────────────────────────────
```

If a number cannot be determined with confidence, mark it `~N` (approximate) or `?` (unknown).

---

## Step 3: Estimate Development Effort

### 3a. Module-based estimation (primary method)

For each identified module, assign a complexity tier and estimate hours:

```
Module Inventory
──────────────────────────────────────────────────────────
Module Name           Tier        Hours (range)
──────────────────────────────────────────────────────────
[module 1]            Complex     80–120
[module 2]            Standard    50–70
[module 3]            Simple      25–35
...
──────────────────────────────────────────────────────────
Subtotal — Implementation                   XXX–XXX hrs
+ Infrastructure / DevOps (10–15%)          XX–XX hrs
+ Architecture & design (5–10%)             XX–XX hrs
+ Integration & QA (10–15%)                 XX–XX hrs
──────────────────────────────────────────────────────────
TOTAL ESTIMATED EFFORT                      XXX–XXX hrs
```

### 3b. Git-signal cross-check

Use git signals to validate or adjust the module estimate:

- **Commit volume**: N commits over X weeks = Y avg commits/week. High-frequency projects (>5/day) suggest intensive development. Low-frequency (<1/day) may indicate design-heavy or async work.
- **Avg lines changed per commit**: Very high (>500) suggests generated code, squash merges, or AI bursts. Very low (<20) suggests iterative polish. Neither directly indicates effort.
- **High-churn files**: Files modified in >10% of commits are complexity hotspots. If they align with a "simple" module classification, revise upward.
- **Author split**: If 80%+ of commits are from one author, solo velocity assumptions apply. Multi-author projects have coordination overhead (add 10–20%).

**Important**: Git signals are cross-checks, not primary drivers. A squash-merge repo will look artificially low-commit.

### 3c. Do NOT use COCOMO

Classic COCOMO is designed for large waterfall teams building from scratch with minimal framework leverage. It produces estimates 3–10x higher than reality for modern TypeScript stacks with NestJS, Prisma, Refine, and AI-assisted development. If COCOMO is referenced anywhere in prior reports, discard that number entirely.

---

## Step 4: Estimate AI-Assisted Rebuild Cost

### 4a. AI code generation split

Estimate the portion of the codebase that was or could be AI-generated:

```
Code Authorship Split (estimated)
──────────────────────────────────────
AI-generated / assisted:   ~X% of LOC
Human-written:             ~X% of LOC
──────────────────────────────────────
```

Typical ranges for AI-assisted projects:
- Boilerplate (CRUD, DTOs, migrations): 70–90% AI-generatable
- Business logic, integrations: 30–50% AI-generatable
- Custom algorithms, state machines: 10–30% AI-generatable

### 4b. Token consumption model

Do NOT estimate AI cost by mapping "AI hours" to tokens. Instead:

```
Token Estimation
──────────────────────────────────────────────────────────
Estimated AI-generated LOC: X lines
Code generation tokens:     X lines × 10 tokens/LOC = X tokens
Context + iteration (3–5×): X tokens
──────────────────────────────────────────────────────────
Estimated total tokens:     ~X M tokens
Cost @ Claude Sonnet:       ~$X (at $3/M input + $15/M output)
Cost @ GPT-4o:              ~$X
──────────────────────────────────────────────────────────
```

### 4c. AI-assisted rebuild timeline

Break delivery into phases and show human vs AI-assisted timelines:

```
Delivery Timeline by Phase
──────────────────────────────────────────────────────────
Phase               Human team      AI-assisted (1 eng)
──────────────────────────────────────────────────────────
Discovery & design  2–3 weeks       2–3 weeks (unchanged)
Architecture        2–3 weeks       1–2 weeks
Implementation      X months        X months
Integration         X weeks         X weeks
Testing & QA        X weeks         X weeks
Iteration           X weeks         X weeks
──────────────────────────────────────────────────────────
TOTAL               ~X months       ~X months
──────────────────────────────────────────────────────────
```

Note: AI accelerates implementation, not discovery, product iteration, or stakeholder feedback cycles. A 3–4× speedup claim must be scoped to implementation phases only.

---

## Step 5: Financial Assessment

This is the core output for the agency. The goal is to evaluate execution efficiency.

### 5a. Cost comparison table

```
Cost Analysis
──────────────────────────────────────────────────────────
                           Hours       Rate        Total
──────────────────────────────────────────────────────────
Actual cost (billed/spent) XXX hrs    $XX/hr      $XX,XXX
Market rate (human team)   XXX hrs    $XX/hr      $XX,XXX
AI-assisted rebuild        XXX hrs    $XX/hr      $XX,XXX
──────────────────────────────────────────────────────────
```

### 5b. Gross margin analysis

```
Gross Margin
──────────────────────────────────────────────────────────
Contract value (if known):         $XX,XXX
Actual cost (labor + tools):       $XX,XXX
Gross margin:                      XX%
──────────────────────────────────────────────────────────
If rebuilt with AI fleet today:
  Estimated cost:                  $XX,XXX
  Margin at same contract value:   XX%
──────────────────────────────────────────────────────────
```

### 5c. AI leverage delta

```
AI Leverage Assessment
──────────────────────────────────────────────────────────
Estimated AI usage in delivery:    Low / Medium / High
Potential AI usage for this scope: High
Unrealized leverage:               Significant / Moderate / Minimal
──────────────────────────────────────────────────────────
```

---

## Step 6: Risk & Complexity Drivers

List the top 4–8 complexity risks that drove cost. This explains why certain modules are expensive and adds credibility to the estimates.

```
Complexity Risk Table
──────────────────────────────────────────────────────────
Risk Factor                    Impact
──────────────────────────────────────────────────────────
[e.g. OEM portal automation]   Brittle UI scraping, high maintenance
[e.g. Queue-based sync]        Concurrency errors, idempotency
[e.g. Browser extension auth]  Security surface, cross-context state
[e.g. Claim state machine]     Business logic depth, edge cases
──────────────────────────────────────────────────────────
```

---

## Step 7: Estimator Confidence Model

Always include this section to communicate reliability of each estimate.

```
Estimator Confidence
──────────────────────────────────────────────────────────
Signal                    Confidence    Basis
──────────────────────────────────────────────────────────
LOC & file counts         High          Direct measurement
Feature surface metrics   High          Direct analysis
Module complexity tiers   Medium        Expert judgment
Git signal analysis       Medium        Repo history
Actual hours billed       Medium        Requires external data
AI usage intensity        Low           Inferred from patterns
Historical benchmarks     Low           Limited comparators
──────────────────────────────────────────────────────────
```

---

## Step 8: Produce the Report

Use the following report structure. Maintain an objective, analytical tone throughout. Avoid value judgments ("you did well", "you left money on the table"). Use neutral framing: "Cost Efficiency Assessment", "Delivery Efficiency", "Engineering Quality Indicators".

---

## Report Format

```
# Project Cost & Execution Audit
**Project**: [name]
**Date**: [date]
**Analyzed by**: BlackBox Vision — Internal

---

## System Scope

[Scope snapshot table from Step 2]

---

## Effort Estimation

[Module inventory and total from Step 3a]

### Git Signal Analysis

[Cross-check findings from Step 3b — commit volume, avg size, churn files, author split]

---

## AI-Assisted Rebuild

### Code Authorship Split
[From Step 4a]

### Token Consumption Model
[From Step 4b]

### Delivery Timeline
[Phase table from Step 4c]

---

## Financial Assessment

### Cost Comparison
[Table from Step 5a]

### Gross Margin Analysis
[From Step 5b]

### AI Leverage Assessment
[From Step 5c]

---

## Complexity & Risk Drivers

[Risk table from Step 6]

---

## Estimator Confidence

[Confidence table from Step 7]

---

## Key Findings

[3–5 bullet points — factual observations only, no prescriptions]
- Estimated effort: XXX–XXX hrs; actual delivery: XXX hrs (ΔX%)
- AI leverage: [low/medium/high relative to scope potential]
- Highest-complexity areas: [module names]
- Gross margin at contract price: XX%
- Estimated rebuild cost with AI fleet today: $XX,XXX
```

---

## Important Constraints

1. **Never use COCOMO.** It is not appropriate for NestJS/Prisma/Refine/Turborepo stacks.
2. **LOC is a secondary signal only.** Use it for context, not as a primary effort driver. Modern TypeScript stacks (Prisma, NestJS decorators, generated DTOs) are inherently verbose.
3. **Approximate clearly.** When a value is estimated rather than measured, prefix with `~` and note the basis.
4. **No prescriptions in the main report.** The report describes what happened. If the user asks for recommendations, provide them separately.
5. **Module tiers beat formulas.** An experienced engineer's judgment on module complexity is more reliable than any algorithmic estimate for projects of this size.
6. **This report is internal.** Write for a technical co-founder reviewing their own work, not for a client. Be direct about gaps, overestimates, and unrealized leverage.
