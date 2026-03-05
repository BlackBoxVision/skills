# BlackBoxVision Claude Code Skills

A collection of [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) for BlackBoxVision teams — reusable, shareable prompts that extend Claude Code's capabilities for common workflows.

---

## What are Claude Code Skills?

Claude Code skills are `.skill` files that package a focused prompt (plus optional reference files) into a named capability that Claude Code can automatically trigger. Once registered, a skill appears in Claude Code's skill selector and can be invoked with a `/` command or triggered automatically when relevant keywords are detected.

Skills live in a ZIP archive structured as:

```
<skill-name>/
├── SKILL.md          # The main prompt and trigger rules (required)
└── references/       # Supporting reference files (optional)
    └── *.md
```

---

## Available Skills

| Skill | Description | Trigger keywords |
|-------|-------------|-----------------|
| [agent-readiness](./agent-readiness/) | Audits a project for AI-assisted development readiness. Scores 8 dimensions and produces a prioritized report. | "agent readiness", "AI readiness", "ready for Claude Code", "audit for automation" |
| [project-cost-estimator](./project-cost-estimator/) | Analyzes source code, git history, and architecture to estimate what a project cost to build — and what it would cost using a modern AI agent fleet. Produces a side-by-side human vs. AI cost report with gross margin delta. | "how much did this cost to build", "estimate project cost", "AI vs human development cost", "gross margin from using agents", "what's this codebase worth" |

---

## How to Use a Skill

### Install via Claude Code UI

1. Open Claude Code
2. Go to **Settings → Skills**
3. Click **Add skill** and point it to the `.skill` file (or the folder if using the unpacked format)

### Reference in a prompt

Once installed, you can invoke a skill directly:

```
/agent-readiness
```

Or reference a skill file from a prompt:

```
@agent-readiness audit this project
```

Claude Code will auto-trigger the skill when it detects relevant keywords (see trigger keywords per skill above).

---

## Contributing

Want to add a new skill?

1. Create a folder under the repo root: `<skill-name>/`
2. Add `SKILL.md` with at minimum a YAML front-matter block and the skill prompt:

```markdown
---
name: your-skill-name
description: |
  What this skill does and when to trigger it.
  Include trigger keyword examples.
---

# Your Skill Title

Skill prompt content here...
```

3. Optionally add reference files under `<skill-name>/references/`
4. Update this README with a row in the **Available Skills** table
5. Open a PR — small, focused additions are welcome

---

## License

MIT
