# AI Agent Cost Models

> Last updated: Q1 2025. Token pricing changes frequently.
> Always verify against official provider pricing pages before quoting to a client.
> Claude pricing: anthropic.com/pricing | OpenAI: openai.com/pricing | Moonshot: platform.moonshot.cn

---

## Token cost assumptions for project estimation

For a typical software task completed by an AI coding agent:
- **Input tokens per task**: 5,000–50,000 (context: code, instructions, prior output)
- **Output tokens per task**: 1,000–8,000 (generated code, explanations)
- **Tasks per feature**: 3–15 back-and-forth turns
- **Features per KLOC**: roughly 8–20 (varies widely by domain)

Use these to derive a per-KLOC token cost for each agent.

---

## Agent Profiles

### 1. Claude Code (Anthropic)

**Backend models:**
| Model | Input ($/M tokens) | Output ($/M tokens) | Notes |
|-------|---------------------|----------------------|-------|
| claude-opus-4 | $15 | $75 | Most capable, complex reasoning |
| claude-sonnet-4.5 | $3 | $15 | Best cost/quality for most tasks |
| claude-haiku-4.5 | $0.80 | $4 | Fast iteration, simple tasks |

**Typical usage pattern:**
- Architecture decisions, complex debugging → Opus
- Most coding tasks → Sonnet (default)
- Boilerplate generation, fast edits → Haiku

**Estimated cost per effective KLOC produced:**
- Low complexity project: $8–25/KLOC
- Medium complexity: $20–60/KLOC
- High complexity: $50–150/KLOC

**Strengths:** Deep reasoning, long context (200K), tool use, autonomous multi-step
**Weaknesses:** Higher cost at Opus tier, slower on very large parallel tasks

---

### 2. OpenCode (OpenCode.ai)

**Backend:** User-configurable (Claude, GPT-4o, Gemini, local models)

**Cost model:** Pass-through token cost of underlying model + platform fee (~$0 for OSS)

**Typical setup for cost optimization:**
- Route simple tasks → Gemini Flash or GPT-4o mini (~$0.15/M input)
- Route complex tasks → Claude Sonnet or GPT-4o (~$3–5/M input)

**Estimated cost per effective KLOC produced:**
- Low: $5–15/KLOC (optimized routing)
- Medium: $15–40/KLOC
- High: $40–100/KLOC

**Strengths:** Model routing flexibility, open source, can use local models (zero token cost)
**Weaknesses:** Requires more setup, quality depends on model selection

---

### 3. Goose (Block/Square OSS)

**Backend:** Configurable — commonly Claude or GPT-4o

**Cost model:** Token cost of underlying model; Goose itself is free/OSS

**Key differentiator:** Excellent agentic tool use, file system operations, shell commands.
Particularly efficient for DevOps and infrastructure tasks.

**Estimated cost per effective KLOC produced:**
- Low: $8–20/KLOC (using mid-tier model)
- Medium: $20–55/KLOC
- High: $50–130/KLOC

**Strengths:** Agentic file/shell ops, extensible plugins, DevOps tasks
**Weaknesses:** Younger ecosystem, fewer integrations than Claude Code

---

### 4. Codex / OpenAI Codex CLI (OpenAI)

**Backend models:**
| Model | Input ($/M tokens) | Output ($/M tokens) |
|-------|---------------------|----------------------|
| gpt-4o | $2.50 | $10 |
| gpt-4o mini | $0.15 | $0.60 |
| o3 | $10 | $40 |
| o4-mini | $1.10 | $4.40 |

**Typical usage pattern:**
- Complex logic → o3 or o4-mini (reasoning models)
- Standard coding → gpt-4o
- High-volume simple tasks → gpt-4o mini

**Estimated cost per effective KLOC produced:**
- Low: $6–18/KLOC
- Medium: $18–50/KLOC
- High: $45–120/KLOC

**Strengths:** Strong JS/Python/TypeScript, large ecosystem, well-tested
**Weaknesses:** Shorter context window on cheaper tiers, less long-horizon planning

---

### 5. Kimi Code (Moonshot AI)

**Backend:** Kimi k1.5 / k2 models

**Cost model:**
| Model | Input ($/M tokens) | Output ($/M tokens) | Notes |
|-------|---------------------|----------------------|-------|
| kimi-k2 | $0.60 | $2.50 | Current flagship for coding |
| kimi-k1.5 | $0.15 | $2.50 (long CoT) | Reasoning tasks |

**Estimated cost per effective KLOC produced:**
- Low: $2–8/KLOC
- Medium: $8–25/KLOC
- High: $20–60/KLOC

**Strengths:** Very cost-effective, strong on agentic tasks, 128K context, competitive coding benchmarks
**Weaknesses:** Newer, less battle-tested in production, primarily China-market focus (latency may vary)

---

## Recommended fleet compositions by project type

### SaaS Web App (typical full-stack)
| Role | Agent | Rationale |
|------|-------|-----------|
| Architecture + complex features | Claude Code (Sonnet) | Best reasoning |
| Standard CRUD + API routes | Kimi Code or Codex gpt-4o | Cost-effective |
| DevOps / CI/CD / infra | Goose | Best at shell/file ops |
| Frontend boilerplate | Codex gpt-4o mini | Fast, cheap |

### Data / ML Platform
| Role | Agent | Rationale |
|------|-------|-----------|
| Data pipeline logic | Claude Code (Opus/Sonnet) | Complex reasoning |
| Feature engineering | Codex o4-mini | Reasoning tasks |
| ETL boilerplate | Kimi Code | Cost-effective |

### Mobile App (React Native / Flutter)
| Role | Agent | Rationale |
|------|-------|-----------|
| Core logic + state | Claude Code (Sonnet) | Best context handling |
| UI components | OpenCode (routed to GPT-4o) | Flexible |
| Native integrations | Claude Code | Tool use depth |

### Enterprise Backend / Microservices
| Role | Agent | Rationale |
|------|-------|-----------|
| Domain logic | Claude Code (Opus) | Complex reasoning |
| Service scaffolding | Kimi Code | High volume, cheap |
| Integration testing | Goose | Shell + file ops |
| Documentation | Claude Code (Haiku) | Fast, accurate |

---

## Manager overhead model

A single human manages the AI fleet:

| Manager role | Hours as % of traditional team hours | Notes |
|-------------|--------------------------------------|-------|
| Prompt engineering + task decomposition | 8–12% | Defining tasks clearly |
| Output review + integration | 10–15% | Reviewing PRs, merging |
| Debugging AI mistakes | 5–10% | Catching hallucinations |
| Architecture decisions | 5–8% | Humans still own the big picture |
| **Total** | **28–45%** | Effective oversight ratio |

**Rule of thumb:** 1 manager can oversee 3–6 concurrent agents on a well-structured project.

---

## Cost comparison template

```
Project: <Name>
Complexity tier: <Low / Medium / High>

HUMAN BUILD
────────────────────────────────
Person-hours:          X hrs
Blended rate:          $Y/hr
Human build cost:      $Z

AI AGENT FLEET
────────────────────────────────
Agent compute cost:    $A   (token costs across fleet)
Manager hours:         B hrs
Manager rate:          $C/hr
Manager cost:          $D
─────────────────
AI fleet total cost:   $E

COMPARISON
────────────────────────────────
Cost reduction:        $(Z-E)  = F%
If client-billed at $Z:
  Traditional margin:  ~20–35%  (agency overhead)
  AI fleet margin:     ~G%      (Z - E - overhead) / Z
  Margin uplift:       ~H pp

Time compression:      ~Xh faster (calendar time)
```

---

## Important caveats

1. **Quality isn't free.** AI agents make mistakes. Insufficient review overhead
   is the #1 cause of AI fleet failures. Never underestimate the manager cost.

2. **Token costs evolve fast.** Kimi k2 launched at $0.60/M input in mid-2025
   and is already undercutting the field. Expect continued price compression.

3. **Context window limits matter.** Large monolithic codebases stress 128K–200K
   context limits. Projects requiring full-codebase awareness benefit more from
   Claude Code's architecture.

4. **Not everything is delegatable.** Client communication, architectural vision,
   and ambiguity resolution still require human judgment. Factor this in.

5. **Regulatory / compliance domains** require more review time (+20–40% manager overhead).
