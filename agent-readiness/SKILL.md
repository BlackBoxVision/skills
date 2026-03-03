---
name: agent-readiness
description: Audits a software project to evaluate how well-structured it is for AI-assisted development. Use this skill whenever someone asks about "agent readiness", "AI readiness", whether a project is "ready for AI agents", "optimized for LLMs", "Claude-friendly", or wants to know how well their codebase supports autonomous AI workflows, automation, or agent-based development. Also trigger when the user says things like "analyze my project structure for AI", "is my repo ready for Claude Code?", "audit my project for automation", or "how well can an AI work on this codebase?". This skill produces a structured report with scores across 8 dimensions and actionable recommendations.
---

# Agent Readiness Auditor

This skill evaluates how well a software project is structured for AI-assisted development — meaning: how effectively an LLM, autonomous agent, or automation tool can navigate, understand, modify, and validate the codebase without requiring implicit human knowledge.

The core principle: **AI amplifies existing structure. It doesn't repair structural deficits — it magnifies them.**

A project that scores high on agent readiness enables:
- Agents to operate with minimal ambiguity
- Automated pipelines to validate changes reliably
- Context to be reconstructed from artifacts alone
- Incremental changes to be made locally without broad systemic impact

---

## How to Run the Audit

### Step 1: Gather project information

If the user provides a local path, use bash_tool and view to explore the filesystem. If they paste a repo URL or describe the project, ask clarifying questions before scoring.

Explore these signals:
```bash
# Root structure
ls -la <project-root>

# Monorepo tooling
cat <project-root>/turbo.json 2>/dev/null
cat <project-root>/pnpm-workspace.yaml 2>/dev/null

# Documentation files
ls <project-root>/*.md 2>/dev/null
cat <project-root>/README.md 2>/dev/null
cat <project-root>/CLAUDE.md 2>/dev/null
cat <project-root>/AGENTS.md 2>/dev/null

# Commit history (last 15 commits)
git -C <project-root> log --oneline -15 2>/dev/null

# CI/CD pipelines
ls <project-root>/.github/workflows/ 2>/dev/null
cat <project-root>/.gitlab-ci.yml 2>/dev/null

# Linting / commit config
ls <project-root>/commitlint.config* <project-root>/.eslintrc* 2>/dev/null

# Scripts
cat <project-root>/package.json | python3 -c "import sys,json; d=json.load(sys.stdin); print(json.dumps(d.get('scripts',{}), indent=2))" 2>/dev/null

# Test files (signals TDD culture)
find <project-root>/src -name "*.test.*" -o -name "*.spec.*" 2>/dev/null | head -20

# Security / dependency tooling
cat <project-root>/.github/dependabot.yml 2>/dev/null
ls <project-root>/.github/workflows/ 2>/dev/null | grep -i "sonar\|security\|coderabbit\|audit"
```

If filesystem access is not available, ask the user to describe or paste the relevant files.

### Step 2: Score each dimension

Score the project across the 8 dimensions defined in references/dimensions.md. Each dimension is scored 0–10. Apply the rubric carefully — partial implementations earn partial credit.

### Step 3: Produce the report

Use the report format below. Always include per-dimension justification, an overall score (simple average), the health band, and top 3 priorities.

---

## Report Format

```
# Agent Readiness Audit
**Project**: [name]
**Date**: [date]
**Overall Score**: [X.X/10] — [Health Band]

---

## Dimension Scores

| Dimension | Score | Status |
|-----------|-------|--------|
| 1. Architecture & Monorepo Structure | X/10 | 🟢/🟡/🔴 |
| 2. Structural Documentation | X/10 | 🟢/🟡/🔴 |
| 3. Script Standardization | X/10 | 🟢/🟡/🔴 |
| 4. Commit & Versioning Conventions | X/10 | 🟢/🟡/🔴 |
| 5. PR & Review Process | X/10 | 🟢/🟡/🔴 |
| 6. Automated Validation Pipeline | X/10 | 🟢/🟡/🔴 |
| 7. Test Coverage Culture | X/10 | 🟢/🟡/🔴 |
| 8. Specification & Intent Clarity | X/10 | 🟢/🟡/🔴 |

---

## Dimension Details

[For each dimension: 2-4 sentences describing what was found, what works, what is missing]

---

## What This Score Means

[Explain in practical terms: what kinds of AI workflows are currently possible, and what is being blocked]

---

## Top 3 Priorities

1. **[Most impactful improvement]**: [Why + specific next step]
2. **[Second priority]**: [Why + specific next step]
3. **[Third priority]**: [Why + specific next step]

---

## Quick Wins (under 2 hours)

- [Concrete action that improves score fast]
- [Another quick win]
```

---

## Health Bands

| Score | Band | Practical Meaning |
|-------|------|-------------------|
| 8.5–10 | 🟢 AI-Native | Agents can operate autonomously with high reliability |
| 7–8.4 | 🟢 AI-Ready | Strong foundation; minor gaps won't block agents |
| 5–6.9 | 🟡 Transitional | Agents can assist but need more human oversight |
| 3–4.9 | 🟠 Fragile | Structural gaps introduce friction and agent errors |
| 0–2.9 | 🔴 Not Ready | Agents here will amplify problems, not solve them |

---

## Scoring Status Colors

- 🟢 7–10: Healthy
- 🟡 4–6: Partial / Needs work
- 🔴 0–3: Missing or broken

---

## Important Nuances

- **Scale appropriately**: A solo side project doesn't need full Turborepo. A 10-person team product does. Adjust expectations to context.
- **Behavior over config**: A project with commitlint installed but 50 non-conformant commits in a row scores low on conventions. Judge what actually happens, not just what's configured.
- **Ask before guessing**: If you can't read a file or a dimension is ambiguous from what's available, ask the user for clarification rather than assume.
- **Impact-ordered recommendations**: The goal is to unblock AI workflows, not to achieve checklist compliance. Prioritize what creates the most leverage.

Read references/dimensions.md for the full scoring rubric.
