# docs-consolidator — Design Spec

**Date:** 2026-03-23
**Status:** Approved

## Problem

After extended development on a project, documentation artifacts accumulate across multiple tools and locations — codemaps, design specs, devlogs, backlogs, progress trackers, READMEs, ADRs. No single place provides a navigable entry point. Finding the right document at the right time becomes painful.

## Solution

A skill that **consolidates and curates** existing documentation rather than generating competing docs. It discovers all doc artifacts in a repo, analyzes their coverage and staleness, and produces a navigable hub at `docs/_hub/` with links, summaries, and gap analysis.

## What It Is NOT

- Not a replacement for `doc-updater` (codemap generation)
- Not a replacement for `dev-backlog` (backlog capture)
- Not a replacement for `dev-progress` (progress tracking)
- Not a content generator (except for infra-ops, which has no existing tool)

## Skill Identity

- **Name:** `docs-consolidator`
- **Invocation:** `/docs-consolidator` (single command, no subcommands)
- **Model:** `sonnet`
- **Skill files:** `SKILL.md` + `references/hub-templates.md`

## When to Use

- After extended development sessions when docs have sprawled
- When onboarding someone to a project and need a "start here"
- After running `doc-updater`, `dev-progress:init`, or `dev-backlog:add` multiple times
- Periodically as a documentation health check

---

## Workflow

### Phase 1 — Artifact Discovery

Scan the repo for all documentation artifacts:

| Artifact Type | Search Pattern | Maps To |
|---|---|---|
| Codemaps | `docs/CODEMAPS/*.md` | `architecture.md` |
| Design specs | `docs/superpowers/specs/*.md`, `docs/specs/*.md`, `**/ADR-*.md`, `**/adr-*.md` | `specs.md` |
| Devlog | `DEVLOG.md` | `roadmap.md` |
| Backlog | `dev-backlog/*.md` | `roadmap.md` |
| Progress tracker | `dev-progress/*.md` | `roadmap.md` |
| READMEs | `**/README.md` | `architecture.md` |
| CI/CD configs | `.github/workflows/*.yml`, `Dockerfile*`, `docker-compose*`, `.gitlab-ci.yml`, `Jenkinsfile` | `infra-ops.md` |
| Infra configs | `terraform/**`, `k8s/**`, `helm/**`, `.env.example` | `infra-ops.md` |
| Package manifests | `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml` | `architecture.md` |
| Existing docs | `docs/**/*.md` (excluding `docs/_hub/**`) | listed in "Other Docs" section of `architecture.md` |

**Exclusion rule:** All discovery patterns MUST exclude `docs/_hub/**` from scanning to prevent circular references on re-runs. This must be stated explicitly in SKILL.md instructions.

### Phase 2 — Analysis

For each discovered artifact:
- **Exists?** — record path and last commit date via `git log --follow --format=%ci -1 -- <path>`
- **Stale?** — an artifact is stale if `git log --since=<artifact_last_commit_date> -- <related_source_paths>` returns any commits. "Related source path" is determined by: (1) for codemaps, the source directory whose name matches the codemap filename stem (e.g., `docs/CODEMAPS/backend.md` → `src/backend/` or `backend/`). If neither `src/<stem>/` nor `<stem>/` exists on disk, fall back to the broad `src/` or `lib/` signal and note "approximate match" in GAP-ANALYSIS.md; (2) for design specs, filesystem paths mentioned within the spec (see dead-spec extraction rules in Phase 3); (3) for READMEs, the directory containing the README; (4) for all other artifacts, the repo root `src/` or `lib/` as a broad signal. Non-tracked files (not yet committed) are excluded from staleness checks and flagged as "uncommitted" instead.
- **Coverage** — identify major source directories with no documentation pointing at them
- **Git unavailable?** — if the directory is not a git repo or `git` is not on PATH, skip staleness analysis entirely and note "Git unavailable — staleness analysis skipped" in GAP-ANALYSIS.md

### Phase 3 — Gap Detection

Check for common gaps:
- Modules/services with no codemap entry
- No CI/CD documentation despite workflow files existing
- No deployment docs despite Dockerfile/k8s configs existing
- Dead specs: extract filesystem paths from design specs that begin with a known project root prefix (`src/`, `lib/`, `services/`, `packages/`, `apps/`, `modules/`). Ignore paths inside fenced code blocks, paths starting with `/` (URL routes), and paths containing `://` (URLs). Only flag a path as dead if its parent directory exists on disk but the path itself does not — this prevents false positives from paths in directories that were never part of this repo.
- Source directories with zero documentation coverage

No content generation in this phase — cataloguing only.

### Phase 4 — Content Generation (infra-ops only)

Generate `infra-ops.md` content by parsing detected infrastructure configs. This is the only phase that produces original prose. Scoping rules:
- Summarize up to 10 pipeline/workflow files and 10 infra config files
- For larger repos, summarize by directory grouping rather than per-file
- Log a warning in the output if caps are hit

---

## Output Structure

All output goes to `docs/_hub/`. Every run wipes and regenerates this directory.

**`docs/_hub/` is a fully managed directory.** Users must NOT manually edit files inside it — all content is regenerated on every run. Each generated file includes a warning comment: `<!-- Generated by docs-consolidator. Do not edit — will be overwritten on next run. -->`

```
docs/_hub/
├── README.md          # Entry point — project snapshot + navigation
├── architecture.md    # Links to codemaps + structural overview
├── specs.md           # Design specs organized by domain
├── infra-ops.md       # Generated CI/CD, deployment, infra docs
├── roadmap.md         # Links to backlog + progress + devlog
└── GAP-ANALYSIS.md    # Stale docs, missing docs, dead references
```

### README.md — Entry Point

```markdown
# Project Hub
> Auto-generated by docs-consolidator. Last run: YYYY-MM-DD

## Project Snapshot
- **Tech Stack:** (detected from manifests)
- **Repo Type:** Monorepo | Multi-repo (detected)
- **Services/Modules:** N detected (list)

## Quick Navigation
| Area | Doc | Status |
|---|---|---|
| Architecture | architecture.md | N codemaps found, N stale |
| Specs & Decisions | specs.md | N design specs, N ADRs |
| Infrastructure & Ops | infra-ops.md | N gaps identified |
| Roadmap & Progress | roadmap.md | N backlog items, Phase N in progress |

## Documentation Health
- **Coverage:** X% of source directories documented
- **Staleness:** N artifacts need refresh
- **Gaps:** N identified → GAP-ANALYSIS.md
```

### architecture.md — Links to Codemaps + Structural Overview

- Lists all CODEMAPS with one-line summary and link
- Lists all README files found across the repo
- If no codemaps exist, notes gap and suggests running `doc-updater`
- Generates lightweight module/service map from package manifests (names, dependencies, entry points — not a full codemap)

### specs.md — Design Specs Organized by Domain

- Group by domain when domain can be inferred from filename prefixes or path. Fall back to chronological (year-month) grouping when domain is unclear. Ungrouped specs go under "Other".
- Lists ADRs with status if a `Status:` field is present in the first 10 lines of the file (supports MADR frontmatter and Nygard `## Status` heading formats)
- Links to each spec with title and one-line summary extracted from the doc
- Flags specs referencing modules that no longer exist

### infra-ops.md — Generated Content (Exception)

Since no existing skill covers infra/ops, this file generates content:
- Documents detected CI/CD pipelines (parsed from workflow YAML — trigger, jobs, what they deploy)
- Documents detected containerization (Dockerfile stages, compose services)
- Documents detected infra-as-code (terraform modules, k8s manifests)
- Lists environment variables from `.env.example` if present
- Flags anything detected but undocumented

### roadmap.md — Links to Backlog + Progress + Devlog

- Links to `dev-backlog/` with summary stats (X open features, Y fixes, Z tech debt)
- Links to all files under `dev-progress/` (typically `MASTER-PLAN.md` and `PROGRESS.md`) with current phase and completion %
- Links to `DEVLOG.md` with last session date
- If none of these exist, notes the gap

### GAP-ANALYSIS.md — Actionable Output

Three sections:
1. **Stale Documentation** — artifact, last updated, commits since, suggested action
2. **Missing Documentation** — gap, evidence of need, suggested action
3. **Dead References** — specs/docs referencing files that no longer exist

---

## Constraints

- **Idempotent:** Every run wipes `docs/_hub/` and regenerates. No incremental updates. Users must NOT manually edit files inside `docs/_hub/` — it is a fully managed directory.
- **Read-only toward existing docs:** Never modifies artifacts outside `docs/_hub/`.
- **No auto-orchestration:** Does not invoke `doc-updater`, `dev-progress`, or other tools. Reports gaps and suggests commands.
- **Staleness is git-based:** Uses `git log` commit dates (not filesystem mtime) to determine staleness. Flagged when related source directory has git commits newer than the artifact's last commit date.
- **No auto-commit:** Generates files, user reviews, user decides to commit.
- **Generated marker:** Each file includes `<!-- Generated by docs-consolidator. Do not edit — will be overwritten on next run. -->` in the header.
- **Self-exclusion:** All discovery patterns must exclude `docs/_hub/**` to prevent circular references.
- **Mono vs multi-repo:**
  - Monorepo: `docs/_hub/` at repo root
  - Multi-repo workspace: detected by presence of multiple sibling directories each containing `.git/`. Places `docs/_hub/` at the parent workspace root. Links reference individual repo paths with relative paths. If the parent workspace root is not writable, fall back to the current repo's root. Note: `docs/_hub/` in a multi-repo workspace is intentionally outside any single repo's git tracking — the user must manage version control for this directory manually.
- **Infra-ops scope cap:** Summarize up to 10 pipeline files and 10 infra config files. For larger repos, summarize by directory grouping. Log a warning if caps are hit.
- **Git unavailable fallback:** If not a git repo, skip staleness analysis and note it in GAP-ANALYSIS.md.

## Skill File Structure

```
~/.claude/skills/docs-consolidator/
├── SKILL.md              # Main skill definition
└── references/
    └── hub-templates.md  # Markdown templates for each _hub/ file
```

`hub-templates.md` contains the skeleton templates for all six output files (README.md, architecture.md, specs.md, infra-ops.md, roadmap.md, GAP-ANALYSIS.md). Loaded explicitly via Read tool during the generation phase.
