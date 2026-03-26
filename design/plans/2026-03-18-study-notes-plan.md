# Study Notes Skill — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a global Claude Code skill that generates comprehensive, production-grade study notes for any topic as a structured Markdown document.

**Architecture:** Single-skill, sequential workflow. SKILL.md is the entry point (~150-180 lines) containing the full workflow instructions. A `references/output-template.md` file provides the output structure, loaded on-demand via Read tool. No scripts, no subagents — the skill orchestrates Claude's built-in tools (WebSearch, WebFetch, Read, Write).

**Tech Stack:** Claude Code skills system (SKILL.md frontmatter + Markdown instructions), WebSearch/WebFetch for research, Write for file output.

**Spec:** `docs/superpowers/specs/2026-03-18-study-notes-design.md`

---

## File Structure

```
~/.claude/skills/study-notes/
  ├── SKILL.md                    # Entry point — frontmatter + full workflow (150-180 lines)
  └── references/
      └── output-template.md      # Output template with 9 sections + TOC (~80 lines)
```

- `SKILL.md` — contains: frontmatter (name, description), When to Use, When NOT to Use, Workflow (4 steps), Folder Taxonomy Rules, Non-Technical Adaptation Table, Error Handling rules
- `references/output-template.md` — contains: the exact Markdown template that the skill fills in, with section headings, TOC anchors, and per-section content guidance

---

## Task 1: Create the output template

**Files:**
- Create: `~/.claude/skills/study-notes/references/output-template.md`

- [ ] **Step 1: Create the references directory**

```bash
mkdir -p ~/.claude/skills/study-notes/references
```

- [ ] **Step 2: Write the output template file**

Write the following to `~/.claude/skills/study-notes/references/output-template.md`:

```markdown
<!--
NON-TECHNICAL ADAPTATION:
- "Setup & Configuration" → "Getting Started / Prerequisites"
- "Practical Examples" → "Practical Scenarios / Exercises"
- "Production Usage" → "Professional Application"
- "Related Tools & Ecosystem" → "Related Frameworks / Methodologies"
-->

# {{TOPIC_NAME}} — Study Notes

> Generated: {{DATE}} | Source: Official docs + web research + Claude knowledge

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

<!-- What it is and what problem it solves. Why it exists / historical context. Target: 150-300 words. -->

## 2. Core Concepts

<!-- Key components and architecture. Important terminology. How it works internally. Target: 400-800 words. -->

## 3. Related Tools & Ecosystem

<!-- Target: 300-600 words total. -->

### Must Know
<!-- Detailed coverage of essential tools/frameworks (~100-200 words each). What it does, why it matters, basic usage example. -->

### Good to Know
<!-- Brief mentions of useful but non-essential tools (1-2 sentences each). -->

## 4. Setup & Configuration

<!-- Prerequisites. Step-by-step setup guide. Commands and configuration examples. Target: 300-500 words. -->

## 5. Practical Examples

<!-- Real working examples with code/CLI. Expected outputs. Target: 400-800 words. -->

## 6. Production Usage

<!-- Best practices. Scalability, reliability, security considerations. Target: 300-500 words. -->

## 7. Real-World Use Cases

<!-- Companies/products using this. Industry scenarios. Target: 200-400 words. -->

## 8. Summary

<!-- Key takeaways. When to use vs not use. Target: 100-200 words. -->

---

## 9. Sources & Further Reading

<!-- List all URLs consulted during research with brief descriptions.
Format: - [Source Title](url) — brief description -->
```

- [ ] **Step 3: Verify the file was created**

```bash
cat ~/.claude/skills/study-notes/references/output-template.md
```

Expected: The full template content is displayed with all 9 sections, TOC anchors, word count targets in comments, and the non-technical adaptation comment block.

- [ ] **Step 4: Done**

Version control for `~/.claude/skills/` is the user's responsibility. The skill files are ready to use immediately after writing.

---

## Task 2: Create the SKILL.md entry point

**Files:**
- Create: `~/.claude/skills/study-notes/SKILL.md`

This is the core of the skill. It must follow the conventions observed in the existing `cert-prep` and `interview-prep` skills: frontmatter with `name` and `description`, then structured Markdown instructions.

- [ ] **Step 1: Write SKILL.md with frontmatter**

Write the following to `~/.claude/skills/study-notes/SKILL.md`:

```markdown
---
name: study-notes
description: >
  Use when the user asks to generate study notes, learn a topic, create a study guide,
  or says "study notes for X", "teach me X", "notes on X", "learn X", or "deep dive into X".
  Do NOT use for interview prep, exam prep, or certification study guides.
---

# Study Notes Skill

Generate comprehensive, production-grade study notes for any topic. Produces a single structured Markdown document with 9 sections, saved to a nested folder in the user's current working directory.

## When to Use

- User asks for study notes on a topic
- User says "teach me X", "notes on X", "learn X", "deep dive into X"
- User says `/study-notes {topic}`

## When NOT to Use

- Interview prep, Q&A, or scenario-based questions → use `interview-prep`
- Certification exam preparation → use `cert-prep`
- Quick factual questions that don't need a full document

---

## Workflow

### Step 1: Parse Topic & Determine Category

Extract the topic from user input. Determine the folder path and file name.

**Folder Taxonomy Rules:**

Use `<vendor-or-domain>/<subcategory>/` in the current working directory:

| Topic Pattern | Folder Example |
|---|---|
| Cloud vendor topics | `aws/networking/`, `gcp/compute/`, `azure/identity/` |
| CNCF/container tools | `kubernetes/custom-resources/`, `docker/networking/`, `helm/charts/` |
| Programming languages | `python/language-features/`, `go/concurrency/`, `java/jvm/` |
| Non-vendor domains | `databases/`, `networking/`, `system-design/`, `project-management/` |

Rules:
- All lowercase, hyphens for spaces
- Subcategory is optional — omit if the topic IS the top-level domain
- File name: `<topic-slug>.md` (e.g., `vpc-peering.md`, `decorators.md`)

**Ambiguity check:** If the topic is genuinely ambiguous (could mean completely different things), ask ONE clarifying question, then proceed.

Examples that ARE ambiguous (ask): "Lambda", "Patterns", "Networking"
Examples that are NOT ambiguous (proceed): "Kubernetes CRDs", "AWS VPC Peering", "React Server Components"

**Multi-topic requests:** If the user provides multiple topics (e.g., "study notes for Kubernetes, Helm, and ArgoCD"), generate a separate document for each topic.

### Step 2: Research

Perform **3-5 targeted web searches** using WebSearch. Fetch promising pages with WebFetch.

1. `"{topic} official documentation"` — primary source
2. `"{topic} architecture overview tutorial"` — core concepts
3. `"{topic} ecosystem tools integrations"` — related tools for the ecosystem section
4. `"{topic} production best practices real-world"` — production usage + case studies
5. *(Conditional — only if results from 1-4 are thin)* — one more targeted search

After searching, supplement gaps with built-in knowledge. Do NOT launch subagents or loop back for more research.

**If WebSearch/WebFetch are unavailable:** Skip research entirely. Generate from built-in knowledge and add this disclaimer at the top of the document:
> Note: Web research unavailable. Content based on Claude's training knowledge.

### Step 3: Generate Study Notes

Read the template file at `~/.claude/skills/study-notes/references/output-template.md` using the Read tool.

Fill in every section following the template structure:

- Replace `{{TOPIC_NAME}}` with the actual topic name
- Replace `{{DATE}}` with today's date
- Follow word count targets in template comments
- Use code blocks for commands/examples
- Use tables for comparisons
- Use bullet points for readability
- Cite sources inline where practical (e.g., `[Source: K8s docs](url)`)

**For the "Related Tools & Ecosystem" section**, use tiered coverage:
- **Must Know:** essential tools — detailed coverage (~100-200 words each) with what it does, why it matters, and a basic usage example
- **Good to Know:** useful but non-essential tools — brief mentions (1-2 sentences each)

**For non-technical topics**, adapt section names:

| Technical Section | Non-Technical Adaptation |
|---|---|
| Setup & Configuration | Getting Started / Prerequisites |
| Practical Examples | Practical Scenarios / Exercises |
| Production Usage | Professional Application |
| Related Tools & Ecosystem | Related Frameworks / Methodologies |

**Target document length:** 2,500-5,000 words total.

### Step 4: Save & Report

1. Create the nested folder in the current working directory if it doesn't exist
2. Save the document as `<topic-slug>.md` in the folder
3. If the file already exists, overwrite it (user is regenerating)
4. Report back to the user:
   - File path where the document was saved
   - Brief summary of what each section covers
   - Any sections where research was thin

---

## Error Handling

| Scenario | Behavior |
|---|---|
| Thin research results | Add disclaimer at top; generate all sections from built-in knowledge |
| Ambiguous topic | Ask one clarifying question, then proceed |
| Folder already exists | Reuse it |
| File already exists | Overwrite |
| Non-technical topic | Adapt section names per table above |
| WebSearch/WebFetch unavailable | Skip research; add disclaimer; generate from knowledge |
| Multi-topic request | Generate separate files, one per topic |
```

- [ ] **Step 2: Verify SKILL.md line count**

```bash
wc -l ~/.claude/skills/study-notes/SKILL.md
```

Expected: approximately 120-160 lines (under the 200-line target).

- [ ] **Step 3: Verify frontmatter is valid YAML**

```bash
head -6 ~/.claude/skills/study-notes/SKILL.md
```

Expected: Shows `---`, `name: study-notes`, `description: >`, the description text, and closing `---`.

- [ ] **Step 4: Done**

Version control for `~/.claude/skills/` is the user's responsibility. The skill files are ready to use immediately after writing.

---

## Task 3: Smoke test — generate study notes for a known topic

**Files:**
- None created in the skill — this tests the skill end-to-end

- [ ] **Step 1: Create a test directory**

```bash
mkdir -p /tmp/study-notes-test && cd /tmp/study-notes-test
```

- [ ] **Step 2: Invoke the skill**

In Claude Code, from `/tmp/study-notes-test`, run:

```
/study-notes Kubernetes Custom Resource Definitions (CRDs)
```

Or type: "Generate study notes for Kubernetes Custom Resource Definitions (CRDs)"

- [ ] **Step 3: Verify output file exists**

```bash
ls -la /tmp/study-notes-test/kubernetes/custom-resources/
```

Expected: A file like `crds.md` or `custom-resource-definitions.md` exists.

- [ ] **Step 4: Verify output structure**

Check the generated file has all 9 sections:

```bash
grep "^## [0-9]" /tmp/study-notes-test/kubernetes/custom-resources/*.md
```

Expected output should show 9 numbered section headings:
```
## 1. Overview
## 2. Core Concepts
## 3. Related Tools & Ecosystem
## 4. Setup & Configuration
## 5. Practical Examples
## 6. Production Usage
## 7. Real-World Use Cases
## 8. Summary
## 9. Sources & Further Reading
```

- [ ] **Step 5: Verify TOC is present**

```bash
head -20 /tmp/study-notes-test/kubernetes/custom-resources/*.md
```

Expected: Title, generation metadata line, TOC with 9 entries.

- [ ] **Step 6: Verify word count is in target range**

```bash
wc -w /tmp/study-notes-test/kubernetes/custom-resources/*.md
```

Expected: approximately 2,500-5,000 words.

- [ ] **Step 7: Verify ecosystem section has tiered structure**

```bash
grep -A 2 "Must Know\|Good to Know" /tmp/study-notes-test/kubernetes/custom-resources/*.md
```

Expected: Both "Must Know" and "Good to Know" subsections exist under the ecosystem section.

- [ ] **Step 8: Verify sources section is populated**

```bash
tail -20 /tmp/study-notes-test/kubernetes/custom-resources/*.md
```

Expected: "Sources & Further Reading" section with at least 3-4 URL references.

- [ ] **Step 9: Clean up test directory**

```bash
rm -rf /tmp/study-notes-test
```

---

## Task 4: Smoke test — non-technical topic adaptation

**Files:**
- None created in the skill — tests section adaptation

- [ ] **Step 1: Create test directory**

```bash
mkdir -p /tmp/study-notes-test && cd /tmp/study-notes-test
```

- [ ] **Step 2: Invoke the skill with a non-technical topic**

```
/study-notes Agile Project Management
```

Or type: "Generate study notes for Agile Project Management"

- [ ] **Step 3: Verify adapted section names**

```bash
grep "^## " /tmp/study-notes-test/project-management/*.md
```

Expected: Should show adapted section names like "Getting Started / Prerequisites" instead of "Setup & Configuration", and "Professional Application" instead of "Production Usage".

- [ ] **Step 4: Clean up**

```bash
rm -rf /tmp/study-notes-test
```

---

## Done

After all 4 tasks pass:
- The skill is installed at `~/.claude/skills/study-notes/`
- It generates structured study notes for any topic
- It correctly adapts sections for non-technical topics
- Output follows the 9-section template with TOC, tiered ecosystem, and sources
