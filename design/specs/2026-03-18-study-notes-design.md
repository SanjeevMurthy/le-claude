# Study Notes Skill — Design Spec

**Date:** 2026-03-18
**Status:** Approved
**Skill Name:** `study-notes`
**Pattern:** Sequential Workflow
**Scope:** Global (`~/.claude/skills/study-notes/`)

---

## Purpose

A skill that generates comprehensive, production-grade study notes for any topic. User provides a topic, the skill researches it, generates a structured Markdown document, and saves it to a nested folder in the current working directory.

Separate from the `interview-prep` skill, which handles Q&A, exotic questions, and scenario-based preparation.

**When NOT to activate:** Exam prep requests ("prepare me for CKA exam"), interview Q&A requests ("interview questions for Kubernetes"), or certification study guides. These belong to `interview-prep` or `cert-prep` skills.

---

## SKILL.md Frontmatter

This is the exact frontmatter that will appear in `SKILL.md`:

```yaml
---
name: study-notes
description: >
  Use when the user asks to generate study notes, learn a topic, create a study guide,
  or says "study notes for X", "teach me X", "notes on X", "learn X", or "deep dive into X".
  Do NOT use for interview prep, exam prep, or certification study guides.
---
```

---

## Structure on Disk

```
~/.claude/skills/study-notes/
  ├── SKILL.md                    # Entry point — workflow + instructions
  └── references/
      └── output-template.md      # Template with 9 sections + TOC (source of truth for output structure)
```

- `SKILL.md` kept under 200 lines
- Template in `references/` loaded explicitly via Read tool during Step 3 (not auto-loaded)
- The output template inlined in this spec is a preview; `references/output-template.md` is the authoritative source

---

## Workflow

### Step 1: Parse Topic & Determine Category

- Extract the topic from user input (e.g., "Kubernetes CRDs", "AWS VPC Peering", "Project Management")
- **Multi-topic requests:** If the user provides multiple topics (e.g., "study notes for Kubernetes, Helm, and ArgoCD"), generate separate documents for each — one file per topic
- Determine nested folder path using the folder taxonomy rules (see below)
- Determine file name slug (e.g., `vpc-peering.md`)
- For ambiguous topics, ask **one** clarifying question, then proceed

**Folder Taxonomy Rules:**

Use `<vendor-or-domain>/<subcategory>/` where the top-level folder is the primary technology or domain in lowercase:

| Topic Example | Folder Path |
|---|---|
| AWS VPC Peering | `aws/networking/vpc-peering.md` |
| Kubernetes CRDs | `kubernetes/custom-resources/crds.md` |
| Terraform Modules | `terraform/modules/modules.md` |
| React Hooks | `react/hooks/hooks.md` |
| Python Decorators | `python/language-features/decorators.md` |
| Project Management | `project-management/project-management.md` |
| System Design | `system-design/system-design.md` |

Rules:
- Vendor/cloud provider names: `aws/`, `gcp/`, `azure/`
- CNCF/container tools: `kubernetes/`, `docker/`, `helm/`
- Programming languages: `python/`, `go/`, `java/`
- Non-vendor domains: `<domain>/` (e.g., `networking/`, `databases/`, `project-management/`)
- Subcategory is optional — omit if the topic IS the top-level domain
- All lowercase, hyphens for spaces

**Ambiguity Examples:**

Topics that SHOULD trigger a clarifying question:
- "Lambda" (AWS Lambda vs lambda calculus vs Lambda architecture)
- "Patterns" (design patterns, cloud patterns, data patterns?)
- "Networking" (computer networking, cloud networking, Kubernetes networking?)

Topics that should NOT:
- "Kubernetes CRDs" — unambiguous
- "AWS VPC Peering" — unambiguous
- "React Server Components" — unambiguous

### Step 2: Research (3-5 Targeted Searches)

1. `"<topic> official documentation"` — primary source
2. `"<topic> architecture overview tutorial"` — core concepts
3. `"<topic> ecosystem tools integrations"` — related tools
4. `"<topic> production best practices real-world"` — production usage + case studies
5. *(Conditional)* — only if results from 1-4 are thin

- Claude supplements gaps from training knowledge
- No subagents — runs in main context
- No open-ended research loops
- If WebSearch/WebFetch are unavailable (no internet, tools disabled), skip research entirely and generate from Claude's knowledge with a disclaimer

### Step 3: Generate Study Notes

- Read `references/output-template.md` using the Read tool for the output structure
- Write complete Markdown with all 9 sections + TOC in a single file
- Include code blocks for commands/examples, tables for comparisons, bullet points for readability
- Cite sources inline where practical
- Adapt sections for non-technical topics using the mapping table (see below)

**Target document length:** 2,500-5,000 words. Per-section guidance:

| Section | Target Length |
|---|---|
| Overview | 150-300 words |
| Core Concepts | 400-800 words |
| Related Tools & Ecosystem | 300-600 words |
| Setup & Configuration | 300-500 words |
| Practical Examples | 400-800 words |
| Production Usage | 300-500 words |
| Real-World Use Cases | 200-400 words |
| Summary | 100-200 words |
| Sources & Further Reading | N/A (list format) |

### Step 4: Save & Report

- Create nested folder in CWD if it doesn't exist (e.g., `./aws/networking/`)
- Save as `<topic-name>.md` (e.g., `./aws/networking/vpc-peering.md`)
- If file already exists, overwrite (user is regenerating)
- Report back: file path, section summary, any topics where research was thin

---

## Output Template

**Note:** This is a preview. The authoritative source is `references/output-template.md`.

The output has **9 sections**: 8 content sections + Sources appendix.

```markdown
# <Topic Name> — Study Notes

> Generated: <date> | Source: Official docs + web research + Claude knowledge

---

## Table of Contents
1. [Overview](#1-overview)
2. [Core Concepts](#2-core-concepts)
3. [Related Tools & Ecosystem](#3-related-tools--ecosystem)
4. [Setup & Configuration](#4-setup--configuration)
5. [Practical Examples](#5-practical-examples)
6. [Production Usage](#6-production-usage)
7. [Real-World Use Cases](#7-real-world-use-cases)
8. [Summary](#8-summary)
9. [Sources & Further Reading](#9-sources--further-reading)

---

## 1. Overview
- What it is and what problem it solves
- Why it exists / historical context

## 2. Core Concepts
- Key components and architecture
- Important terminology
- How it works internally

## 3. Related Tools & Ecosystem

### Must Know
- Detailed coverage (~100-200 words each)
- What it does, why it matters, basic usage example

### Good to Know
- Brief mentions (1-2 sentences each)

## 4. Setup & Configuration
- Prerequisites
- Step-by-step setup
- Commands and config examples

## 5. Practical Examples
- Working examples with code/CLI
- Expected outputs

## 6. Production Usage
- Best practices
- Scalability, reliability, security considerations

## 7. Real-World Use Cases
- Companies/products using this
- Industry scenarios

## 8. Summary
- Key takeaways
- When to use vs not use

---

## 9. Sources & Further Reading
- [Source Title](url) — brief description
```

---

## Non-Technical Topic Adaptation

When the topic is non-technical, sections adapt:

| Technical Section | Non-Technical Adaptation |
|---|---|
| Setup & Configuration | Getting Started / Prerequisites |
| Practical Examples | Practical Scenarios / Exercises |
| Production Usage | Professional Application |
| Related Tools & Ecosystem | Related Frameworks / Methodologies |

The SKILL.md will include this mapping table so Claude applies it consistently.

---

## Token Efficiency

- **3-5 targeted web searches** per invocation (no open-ended loops)
- **No subagents** — single-context execution
- **~15-25K tokens** estimated per invocation
- Template loaded on-demand via explicit Read tool call (not in SKILL.md body)

---

## Error Handling

| Scenario | Behavior |
|---|---|
| Thin research results | Disclaimer at top of document; generate all sections from Claude knowledge |
| Ambiguous topic | Ask one clarifying question (see examples above), then proceed |
| Folder already exists | Reuse it |
| File already exists | Overwrite (user is regenerating) |
| Non-technical topic | Adapt section names per mapping table |
| WebSearch/WebFetch unavailable | Skip research; generate from Claude knowledge with disclaimer: "Note: Web research unavailable. Content based on Claude's training knowledge." |
| Multi-topic request | Generate separate files, one per topic |

---

## Design Decisions Log

| Decision | Choice | Rationale |
|---|---|---|
| Output location | CWD `<technology>/<subtopic>/` | Portable — works anywhere |
| File count | Single file with TOC | Simpler to share and reference |
| Research strategy | Web research first + Claude supplement | Accuracy with efficiency |
| Interaction mode | Fully autonomous | No checkpoints — fire and forget |
| Topic scope | Any topic (technical or not) | Maximum applicability |
| Architecture | Single-phase generator | Simplest design that meets all requirements |
| Ecosystem depth | Tiered (Must Know / Good to Know) | Best signal-to-noise ratio |
| Source citations | Always at bottom | Credibility + further reading |
| Folder structure | Nested with taxonomy rules | Organized, consistent, predictable |
| Separate from interview-prep | Yes | Keeps each skill focused |
| Document length | 2,500-5,000 words | Comprehensive but not bloated |
| Template authority | references/output-template.md | Single source of truth, spec preview is secondary |
