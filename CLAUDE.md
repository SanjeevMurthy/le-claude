# CLAUDE.md

## Project overview

This is a knowledge-base and skill-development repo for Claude Code. It contains guides, research data (video transcripts, blog excerpts), and skill design specs. There is no application code to build or test — the primary outputs are Markdown documents.

## Repo structure

```
claude-tutorial/    # Finished comprehensive guides
data/               # Raw research material (transcripts, articles)
todo-skills/        # Skill specs in progress
initialize.md       # Project brief
prompt.txt          # Supplementary prompt notes
```

## Key conventions

- All documentation is written in Markdown.
- Guides live in `claude-tutorial/`. Do not scatter guide files elsewhere.
- Raw research data goes under `data/<topic>/`. One subfolder per topic.
- Skill design specs go in `todo-skills/` until the skill is built and deployed.
- Keep files focused — one guide per topic, not monolithic dumps.

## When working in this repo

- Read existing guides before editing them to preserve structure and tone.
- Do not auto-commit. Wait for explicit user instruction before committing.
- When researching online to update guides, cite sources where practical.
- Prefer concise, actionable writing over verbose explanations.
