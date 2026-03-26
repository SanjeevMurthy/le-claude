# CLAUDE.md

## Project overview

This is a knowledge-base and documentation repo covering AI-powered development tools — primarily Claude Code and Google Antigravity IDE. It contains comprehensive guides, research data (video transcripts, blog excerpts), skill design specs, and implementation plans. There is no application code to build or test — the primary outputs are Markdown documents.

## Repo structure

```
guides/
  claude-code/          # Claude Code guides (setup, web design, SOP)
  antigravity-ide/      # Antigravity IDE guide (17 chapters)
skills-in-progress/     # Skill specs under development
design/                 # Implementation plans and design specs
  plans/                # Detailed implementation plans
  specs/                # Technical design documents
research/               # Raw research data — git-ignored
prompts/                # Original research prompts — git-ignored
```

## Key conventions

- All documentation is written in Markdown.
- Guides live in `guides/<tool-name>/`. One subfolder per AI tool.
- Raw research data goes under `research/<topic>/`. One subfolder per topic.
- Skill design specs go in `skills-in-progress/` until the skill is built and deployed.
- Implementation plans go in `design/plans/`, design specs in `design/specs/`.
- Keep files focused — one guide per topic, not monolithic dumps.

## When working in this repo

- Read existing guides before editing them to preserve structure and tone.
- Do not auto-commit. Wait for explicit user instruction before committing.
- When researching online to update guides, cite sources where practical.
- Prefer concise, actionable writing over verbose explanations.
- When adding a new AI tool guide, create a new subfolder under `guides/`.
