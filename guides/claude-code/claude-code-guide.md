# The Complete Claude Code Guide

> **A comprehensive, step-by-step reference** for mastering Claude Code — from installation to fully autonomous multi-agent pipelines.

---

## Table of Contents

1. [Introduction & Overview](#1-introduction--overview)
2. [Installation & Setup](#2-installation--setup)
3. [The .claude Directory](#3-the-claude-directory)
4. [CLAUDE.md & the Memory System](#4-claudemd--the-memory-system)
5. [Context Management](#5-context-management)
6. [Tools, Permissions & Modes](#6-tools-permissions--modes)
7. [Slash Commands & Custom Commands](#7-slash-commands--custom-commands)
8. [Skills System](#8-skills-system)
9. [Sub-Agents](#9-sub-agents)
10. [Agent Teams](#10-agent-teams)
11. [Hooks & MCP Servers](#11-hooks--mcp-servers)
12. [Plugins](#12-plugins)
13. [Git Worktrees](#13-git-worktrees)
14. [Deployment](#14-deployment)
15. [Best Practices & Workflows](#15-best-practices--workflows)
16. [Advanced Patterns & Automation](#16-advanced-patterns--automation)
17. [Official Documentation Links](#17-official-documentation-links)

---

## 1. Introduction & Overview

### What is Claude Code?

Claude Code is Anthropic's **agentic coding tool** that lives inside your terminal. Unlike traditional AI chatbots, Claude Code can:

- **Read** your entire codebase to understand project architecture
- **Edit** files directly with precise, targeted changes
- **Run** shell commands (git, npm, tests, linting, etc.)
- **Search** the web for documentation and solutions
- **Orchestrate** teams of specialised AI agents working in parallel

Think of it as an extremely capable junior developer sitting inside your terminal — one that reads your code, understands your conventions, and can execute multi-step tasks end-to-end.

### Where It Runs

| Surface         | Description                                                |
| --------------- | ---------------------------------------------------------- |
| **Terminal**    | The core experience — `claude` command in any shell        |
| **VS Code**     | Extension with inline integration                          |
| **JetBrains**   | Plugin for IntelliJ, WebStorm, etc.                        |
| **Desktop App** | Standalone app with visual diff review (`/desktop`)        |
| **Web**         | Browser-based at claude.ai/code                            |
| **iOS App**     | Start tasks on mobile, pull into terminal with `/teleport` |
| **Slack**       | Mention `@Claude` with a bug report, get a PR back         |
| **CI/CD**       | GitHub Actions & GitLab CI/CD integrations                 |

### Three Modes of Operation

Understanding these three modes is fundamental to getting real leverage from Claude Code:

| Mode           | Context Windows          | Communication                   | Best For                             |
| -------------- | ------------------------ | ------------------------------- | ------------------------------------ |
| **Default**    | 1 (shared)               | N/A                             | Simple tasks, admin, small changes   |
| **Sub-Agent**  | 1 main + N isolated      | Summary only (→ main)           | Research, exploration, focused tasks |
| **Agent Team** | 1 lead + N collaborative | Peer-to-peer + shared task list | Implementation, complex builds       |

### Pricing Plans

| Plan    | Price         | Usage         | Notes                               |
| ------- | ------------- | ------------- | ----------------------------------- |
| **Pro** | $20/month     | Base usage    | Good for getting started            |
| **Max** | $100/month    | 20× Pro usage | Recommended for serious development |
| **API** | Pay-per-token | Unlimited     | Full control, usage-based billing   |

> **Tip:** The Max plan is heavily subsidised by Anthropic and is the best value for active Claude Code users. If you're not running Claude in an autonomous loop, you'll rarely hit the limit.

---

## 2. Installation & Setup

### Prerequisites

- **Node.js** v18+ (for npm install) — or use the native installer (no Node required)
- A Claude account (Pro, Max, or API key)

### Installation

**Option A — npm (requires Node.js):**

```bash
npm install -g @anthropic-ai/claude-code
```

**Option B — Native installer (no Node.js needed):**

```bash
# macOS / Linux
curl -fsSL https://claude.ai/install.sh | sh
```

### Updating

```bash
claude update
```

This checks for and installs the latest version. Run it regularly — features and models are updated frequently.

### First Run

```bash
# Navigate to your project
cd /path/to/your/project

# Launch Claude Code
claude
```

On first run, Claude Code will ask you to authenticate. Follow the browser-based OAuth flow to connect your account.

### VS Code Integration

1. Open VS Code
2. Install the **Claude Code** extension from the marketplace
3. Open a terminal in VS Code → type `claude`
4. Claude Code runs inside the VS Code terminal with full access to your project

### Key CLI Flags

| Flag                                    | Purpose                                        |
| --------------------------------------- | ---------------------------------------------- |
| `claude`                                | Launch interactive session                     |
| `claude "prompt"`                       | One-shot: execute a prompt and exit            |
| `claude -p "prompt"`                    | Print mode: output result to stdout (pipeable) |
| `claude --model opus`                   | Select a specific model                        |
| `claude --dangerously-skip-permissions` | Skip all permission prompts (use with caution) |
| `claude --resume`                       | Resume the most recent conversation            |

### One-Shot Examples

```bash
# Commit with a descriptive message
claude "commit my changes with a descriptive message"

# Run tests and fix failures
claude "write tests for the auth module, run them, and fix any failures"

# Pipe-based workflows
git diff main --name-only | claude -p "review these changed files for security issues"
tail -f app.log | claude -p "Slack me if you see any anomalies"
```

---

## 3. The .claude Directory

The `.claude` directory is the **configuration hub** for Claude Code in your project. It lives at the root of your workspace and can contain rules, agents, skills, and settings.

### Directory Structure

```
.claude/
├── CLAUDE.md          # Project-level memory (alternative to root CLAUDE.md)
├── settings.json      # Local settings (e.g., enable experimental features)
├── rules/             # Modular rules (replaces monolithic CLAUDE.md rules)
│   ├── code-style.md
│   ├── testing.md
│   └── security.md
├── skills/            # Custom skills (reusable workflows)
│   ├── deploy/
│   │   └── SKILL.md
│   └── scrape-leads/
│       ├── SKILL.md
│       └── scripts/
│           └── scraper.py
└── agents/            # Custom subagents
    ├── code-reviewer.md
    ├── qa.md
    └── researcher.md
```

### Modular Rules with `.claude/rules/`

Instead of cramming everything into `CLAUDE.md`, you can create focused rule files:

```markdown
<!-- .claude/rules/testing.md -->

## Testing Rules

- Always write tests for new functions
- Use Jest for unit tests, Playwright for E2E
- Test files go next to the source file with `.test.ts` suffix
- Aim for 80%+ code coverage on new code
```

📖 **Official Docs:** [Modular Rules](https://docs.anthropic.com/en/docs/claude-code/memory#modular-rules-with-claude/rules/)

**Path-specific rules:** You can scope rules to specific file patterns using frontmatter:

```markdown
---
globs: ["*.tsx", "*.jsx"]
---

Always use functional components with hooks. Never use class components.
```

### Settings

`.claude/settings.json` stores project-specific configuration:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

---

## 4. CLAUDE.md & the Memory System

### What is CLAUDE.md?

`CLAUDE.md` is a markdown file that acts as **persistent memory** for Claude Code. Think of it as the onboarding document you'd give a new team member — it tells Claude:

- What this project is
- How things are done
- What patterns to follow
- What mistakes to avoid

Claude reads this file **every time you start a session**. It's the single most important file for getting consistent, high-quality results.

### Setting It Up

**Auto-generate with `/init`:**

```bash
claude
> /init
```

The `/init` command scans your codebase and generates a `CLAUDE.md` with:

- Architecture overview
- Tech stack details
- Project structure
- Development commands (`npm run dev`, `npm test`, etc.)
- Key naming conventions

> **Important:** `/init` gives you a starting point. You should review and refine it. Remove anything Claude could figure out on its own and add your team-specific rules.

**Manual creation:**

Create a file called `CLAUDE.md` in your project root.

### The Three Memory Levels

| Level             | File Location               | Scope             | Version Controlled?  |
| ----------------- | --------------------------- | ----------------- | -------------------- |
| **Project**       | `./CLAUDE.md`               | Shared with team  | ✅ Yes (recommended) |
| **Local Project** | `./.claude/CLAUDE.local.md` | Your machine only | ❌ No (gitignored)   |
| **Global**        | `~/.claude/CLAUDE.md`       | All your projects | N/A                  |

Claude loads all three in order: Global → Project → Local. Later rules override earlier ones.

### How to Write a Great CLAUDE.md

Follow this five-question framework:

#### 1. What is this? (1–2 lines)

```markdown
This project is a Next.js e-commerce app using React, Tailwind CSS, and Supabase.
```

#### 2. How do we run things?

```markdown
## Dev Commands

- `npm run dev` — start dev server
- `npm test` — run Jest tests
- `npm run lint` — ESLint check
- `npm run build` — production build
```

#### 3. What patterns do I follow?

```markdown
## Patterns

- Use server components by default; client components only when needed
- All API routes go in `app/api/`
- Use Zod for validation in all API routes
- For full brand voice guide, see `docs/brand-voice.md`
```

#### 4. What trips you up? (Edge cases & mistakes)

```markdown
## Common Mistakes

- IMPORTANT: The `status` column (not `post_status`) is the canonical field
- Never use `any` type in TypeScript — always define explicit types
- The auth middleware must be called before database queries
```

#### 5. How do we work?

```markdown
## Conventions

- File naming: kebab-case for all files
- Commit messages: conventional commits (feat:, fix:, chore:)
- Never auto-publish or schedule content
```

### Golden Rules

| Rule                               | Why                                                                 |
| ---------------------------------- | ------------------------------------------------------------------- |
| **Keep it short**                  | 200–500 lines max. Readable in 60 seconds.                          |
| **20–30 instructions max**         | Claude's attention degrades with too many rules.                    |
| **Reference, don't dump**          | Complex guidelines → separate files. CLAUDE.md points to them.      |
| **Version control it**             | `CLAUDE.md` should be in your Git repo for the whole team.          |
| **Update in real-time**            | When Claude makes a mistake, tell it: "Add this rule to CLAUDE.md." |
| **Only include non-obvious rules** | Don't tell Claude how JavaScript works. Tell it YOUR conventions.   |

### Adding Memory During a Session

You can add memories without editing the file manually:

```
> Remember: always use the `useOptimistic` hook for form submissions
```

Claude will add this to the appropriate memory file.

---

## 5. Context Management

### Understanding the Context Window

Claude Code operates within a **context window** — the total amount of information (tokens) it can "see" at once.

| Model  | Standard Window | Extended Window  |
| ------ | --------------- | ---------------- |
| Sonnet | 200,000 tokens  | —                |
| Opus   | 200,000 tokens  | 1,000,000 tokens |

**Key insight:** As the context fills up, response quality **degrades**. This is called **context rot**. Research shows that at ~10,000 tokens of input, you lose up to 50% of contextual accuracy.

### What Fills the Context

Everything accumulates:

- Your prompts
- Claude's responses
- Tool calls and their outputs (file reads, bash output, web searches)
- `CLAUDE.md` contents
- Skill descriptions

### Checking Context Usage

```
> /context
```

This shows your current token usage against the maximum. The bottom bar in the terminal also shows a percentage.

### Context Management Commands

| Command    | What It Does                                         | When to Use                                           |
| ---------- | ---------------------------------------------------- | ----------------------------------------------------- |
| `/clear`   | Wipes the entire session history (keeps CLAUDE.md)   | Starting a new task in the same session               |
| `/compact` | Summarises the conversation into a dense summary     | When context is getting large but you need continuity |
| `Esc Esc`  | Rewinds to the previous turn, removing later history | When Claude went down the wrong path                  |
| `/exit`    | Exits Claude Code entirely                           | When you're done or want a completely fresh start     |

### Adding Context to Your Prompts

**Using the `@` symbol:**

```
> @src/components/Button.tsx Can you refactor this component?
```

**Dragging and dropping** files into the terminal also works.

**Cursor position in IDE:** When using Claude Code in VS Code, the file you have open and your cursor position provide implicit context.

**Images:** You can drag and drop screenshots, mockups, or error screenshots.

### Auto-Compaction

When you reach ~95% of the context window, Claude Code automatically **compacts** the conversation — summarising earlier messages to make room. This means:

- Early instructions may be lost or summarised
- The quality of responses for earlier parts of the conversation degrades
- This is the primary reason to use sub-agents (Section 8)

### Context Conservation Strategy

1. **Use sub-agents** for research-heavy tasks (their context is isolated)
2. **Clear between tasks** — don't carry unnecessary context forward
3. **Compact proactively** when at ~60–70% before it auto-compacts
4. **Keep CLAUDE.md lean** — it's loaded every time
5. **Break large tasks into phases** — complete one, clear, start the next

---

## 6. Tools, Permissions & Modes

### Built-in Tools

Claude Code comes with several built-in tools it can invoke:

| Tool                     | Description                    |
| ------------------------ | ------------------------------ |
| `Read`                   | Read file contents             |
| `Edit`                   | Make targeted edits to files   |
| `Write`                  | Create new files               |
| `Bash`                   | Execute shell commands         |
| `WebSearch`              | Search the web for information |
| `TodoRead` / `TodoWrite` | Manage internal task lists     |
| `SubAgent`               | Launch sub-agents              |

### Permission System

Claude Code asks for permission before taking sensitive actions. You'll see prompts like:

```
Claude wants to edit src/auth.ts
Allow? [y/n/always]
```

**Permission options:**

| Response | Effect                                      |
| -------- | ------------------------------------------- |
| `y`      | Allow this one time                         |
| `n`      | Deny this action                            |
| `always` | Allow this tool for the rest of the session |

### Configuring Permanent Permissions

Create or edit `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Edit",
      "Write",
      "Bash(git *)",
      "Bash(npm test)",
      "Bash(npm run lint)"
    ],
    "deny": ["Bash(rm -rf *)", "Bash(sudo *)"]
  }
}
```

This lets you whitelist safe commands (like git operations and tests) while blocking dangerous ones.

### The Three Interaction Modes

Cycle between modes using **`Shift + Tab`**:

#### 1. Accept Edits Mode (Default)

- Claude reads code, proposes changes, and applies them after your approval
- Standard interactive workflow

#### 2. Plan Mode

- **Read-only** — Claude cannot make any changes
- Claude researches, analyses, and proposes a plan
- Uses the built-in "Ask User Questions" tool to clarify assumptions
- You review and approve the plan before switching to Accept Edits

**When to use:** Always start complex tasks in Plan Mode. Boris Cherny, Claude Code's creator, says: _"I use plan mode. And I'll go back and forth with Claude until I like its plan."_

**Pro tips for Plan Mode:**

- Don't blindly accept the plan — **read it critically**
- Edit and refine the plan before approving
- It saves tokens compared to letting Claude implement something wrong, then fixing it

#### 3. Auto-Accept Mode

- Claude applies all changes without asking permission
- Equivalent to the `Alt+M` shortcut
- Use only when you trust the current task is scoped correctly

### Thinking Mode

Thinking mode activates deeper reasoning at the cost of more tokens. Trigger it with keywords in your prompt:

| Keyword                      | Thinking Depth                  | Token Cost |
| ---------------------------- | ------------------------------- | ---------- |
| `"think"`                    | Standard extended thinking      | Moderate   |
| `"think hard"`               | Deep reasoning                  | Higher     |
| `"ultrathink"`               | Maximum reasoning depth         | Highest    |
| `"think about it carefully"` | Also triggers extended thinking | Moderate   |

**When to use:** Complex logic, architectural decisions, debugging subtle issues, or when Claude's first attempt wasn't good enough.

### Effort Toggles (Opus 4.6)

When using Opus 4.6, you can adjust the effort level:

| Effort     | Effect                                            |
| ---------- | ------------------------------------------------- |
| **Low**    | Fastest, cheapest, slight capability reduction    |
| **Medium** | Balanced approach, moderate token savings         |
| **High**   | Best for complex reasoning and difficult problems |

**Recommendation:** Start at Low effort. Increase only when you hit quality issues.

---

## 7. Slash Commands & Custom Commands

### Built-in Slash Commands

| Command        | Description                                   |
| -------------- | --------------------------------------------- |
| `/init`        | Scan codebase and generate `CLAUDE.md`        |
| `/clear`       | Clear conversation history                    |
| `/compact`     | Compress conversation into a summary          |
| `/model`       | Switch the active model                       |
| `/permissions` | Review and manage tool permissions            |
| `/agents`      | List and create sub-agents                    |
| `/context`     | View current context usage                    |
| `/mcp`         | Manage MCP server connections                 |
| `/skills`      | List available skills                         |
| `/teleport`    | Pull a web/mobile session into terminal       |
| `/desktop`     | Hand off to the Desktop app for visual review |

### Creating Custom Commands

Custom commands are **saved prompts** that you trigger with `/command-name`. They live as markdown files in `.claude/commands/`.

**Step 1: Create the directory structure:**

```
.claude/
  commands/
    review-pr.md
    create-component.md
    deploy-staging.md
```

**Step 2: Write the command file:**

```markdown
## <!-- .claude/commands/create-component.md -->

description: Create a new UI component with tests
argument_hint: "ComponentName | brief description"

---

# Context

@src/styles/theme.css
@src/components/index.ts

# Instructions

Create a new React component named $ARGUMENTS.

## Requirements:

1. Place it in `src/components/` directory
2. Follow the project's component pattern (see existing components)
3. Include TypeScript types
4. Add variants and sizes consistent with the design system
5. Create a test file at `src/components/__tests__/`
6. Export the component from `src/components/index.ts`

## Style rules:

- Use CSS modules, not inline styles
- Follow the design tokens in `theme.css`
```

**Step 3: Restart Claude Code** (new commands aren't picked up until restart):

```bash
# Exit and restart
ctrl+c
claude
```

**Step 4: Use the command:**

```
> /create-component Button | A reusable button with primary, secondary, and ghost variants
```

### Command Features

| Feature           | Syntax               | Example                                     |
| ----------------- | -------------------- | ------------------------------------------- |
| **Arguments**     | `$ARGUMENTS`         | User input after the command name           |
| **Description**   | YAML `description`   | Shown in the slash command menu             |
| **Argument hint** | YAML `argument_hint` | Placeholder text in the input               |
| **Context refs**  | `@file/path`         | Files loaded into context when command runs |

### Command Design Tips

- **One command, one purpose** — don't create mega-commands
- **Include context references** — point to relevant files with `@`
- **Be explicit about output location** — tell Claude where to save files
- **Add constraints** — "under 200 words", "no hashtags", "kebab-case filenames"

---

## 8. Skills System

### What Are Skills?

Skills are **reusable, auto-discovered knowledge packs** that give Claude Code domain expertise. They're more powerful than slash commands because:

- They're **automatically invoked** when Claude's task matches the skill description
- They can include **supporting files** (scripts, references, templates, examples)
- They use **progressive disclosure** — only loading full content when needed

Think of skills as onboarding guides for specific workflows: "Here's how we do X in this project."

### Skill Anatomy

```
.claude/skills/
  csv-pipeline/
    skill.md          ← Entry point (required)
    scripts/
      validate.py     ← Helper scripts
      transform.py
    references/
      schema.json      ← Reference data
    templates/
      output.md        ← Output templates
    examples/
      good-output.csv  ← Example inputs/outputs
```

The only required file is `skill.md`. Everything else is optional and loaded on demand.

### The skill.md File

```markdown
---
name: csv-data-pipeline
description: >
  Cleans, validates, and transforms CSV files for analysis.
  Use when a user says "Clean the CSV", "Fix my data",
  "Prepare the spreadsheet for analysis", or "Process this CSV".
---

# CSV Data Pipeline

## Step 1: Validate Input

- Check file exists and is valid CSV
- Report column count, row count, and data types
- Flag any encoding issues

## Step 2: Clean Data

- Remove duplicate rows
- Standardise date formats to ISO 8601
- Trim whitespace from all string fields
- Run `scripts/validate.py` for schema validation

## Step 3: Transform

- Apply transformations per `references/schema.json`
- Generate summary statistics
- Output to `output/` directory

## Step 4: Verify

- Run integrity checks
- Compare row counts pre/post transformation
- Generate a diff report
```

### Progressive Disclosure (3 Levels)

This is the key performance optimisation of skills:

| Level                          | What Loads                     | When                              | Token Cost |
| ------------------------------ | ------------------------------ | --------------------------------- | ---------- |
| **Level 1: YAML Front Matter** | `name` + `description` only    | Always (in system prompt)         | Minimal    |
| **Level 2: Core Instructions** | Full `skill.md` body           | When description matches the task | Moderate   |
| **Level 3: Linked Files**      | Scripts, references, templates | Only when explicitly needed       | Full       |

This "just-in-time" loading means you can have many skills installed without bloating your context — only the relevant skill's content gets loaded.

### Writing Effective Skill Descriptions

The description is the **most important part**. It determines whether Claude ever uses the skill.

**Rules:**

- Keep under **1024 characters**
- Include **3–5 exact trigger phrases** users might say
- Be specific about the task domain

**Good:**

```yaml
description: >
  Cleans, validates, and transforms CSV files for analysis.
  Use when a user says "Clean the CSV", "Fix my data",
  "Prepare the spreadsheet for analysis", or "Process this CSV".
```

**Bad:**

```yaml
description: A data processing skill.
```

### Five Skill Design Patterns

| Pattern                          | Description                                      | Example                                              |
| -------------------------------- | ------------------------------------------------ | ---------------------------------------------------- |
| **Sequential Workflow**          | Linear step-by-step execution                    | CSV pipeline: validate → clean → transform → verify  |
| **Multi-MCP Coordination**       | Orchestrate multiple MCP servers in order        | Pull from Airtable → process → push to Slack         |
| **Iterative Refinement**         | Multiple cycles of generate → evaluate → improve | Writing: draft → review → humanise → final check     |
| **Conditional Logic**            | Different paths based on input type/conditions   | If CSV → CSV processor; If DOCX → document processor |
| **Domain-Specific Intelligence** | Embed enterprise rules and SOPs                  | Company coding standards, deployment procedures      |

### Installing Skills

#### Using skills.sh CLI (by Vercel)

```bash
# Install a skill from a GitHub repo
npx skills add anthropics/skills    # → select from list

# Install specific skill
npx skills add owner/repo

# List installed skills
npx skills list

# Initialise a new skill
npx skills init my-skill-name
```

#### Manual Installation

```bash
# Clone into project skills
git clone https://github.com/owner/repo.git .claude/skills/skill-name

# Or into global skills
git clone https://github.com/owner/repo.git ~/.claude/skills/skill-name
```

### Skill Scopes

| Scope       | Location            | Available To      |
| ----------- | ------------------- | ----------------- |
| **Project** | `.claude/skills/`   | This project only |
| **Global**  | `~/.claude/skills/` | All your projects |

### Community Skill Sources

| Source                  | URL                          | Notes                                                               |
| ----------------------- | ---------------------------- | ------------------------------------------------------------------- |
| **skills.sh**           | skills.sh                    | Vercel's skill marketplace — browse, search, install                |
| **Anthropic Skills**    | github.com/anthropics/skills | Official skills by Anthropic (skill-creator, frontend-design, etc.) |
| **Hugging Face Skills** | huggingface.co               | Model training, dataset processing, fine-tuning                     |

### Creating Your Own Skills

**Recommended approach — build the workflow first:**

1. Complete the task manually in Claude Code (e.g., process a video, create content)
2. Refine the process over several iterations
3. Ask Claude: _"Turn what we just did into a skill"_
4. Claude generates the `skill.md` and supporting files
5. Iterate: run the skill → find edge cases → update the skill

**Shortcut — use the skill-creator skill:**

```bash
npx skills add anthropics/skills   # Select "skill-creator"
```

Then ask Claude to create a skill and it will follow Anthropic's own best practices.

### Skill Best Practices

- **Start with existing skills** — browse skills.sh before building from scratch
- **Iterate aggressively** — skills rarely work perfectly on the first try
- **Include edge case handling** — teach Claude the gotchas
- **Validate before committing** — run the skill multiple times before sharing
- **Keep skills focused** — one skill = one workflow
- **Use global install** for tools you use across projects (e.g., skill-creator, frontend-design)
- **Use project install** for project-specific workflows
- **Prefer local scripts over MCPs** — scripts are faster and use fewer tokens

---

## 9. Sub-Agents

### What Are Sub-Agents?

Sub-agents are **specialised Claude instances with isolated context windows**. When the main agent delegates a task to a sub-agent:

1. The sub-agent gets its own fresh context
2. It completes the task independently
3. It returns **only a summary** to the main agent
4. Its full context (tool calls, intermediate steps) is discarded

**The key benefit:** Sub-agent token usage does **not** affect the main context window. This protects your main conversation from context rot.

### Built-in Sub-Agents

| Agent                 | Model        | Purpose                                           |
| --------------------- | ------------ | ------------------------------------------------- |
| **Bash Agent**        | —            | Shell commands, git operations, terminal tasks    |
| **General Purpose**   | —            | Complex research, multi-step tasks                |
| **Explore Agent**     | Haiku        | Fast, cheap codebase scanning and pattern finding |
| **Planning Agent**    | Same as main | Investigation and solution planning               |
| **Status Line Setup** | —            | Configure the terminal status bar                 |
| **Claude Code Guide** | —            | Answer questions about Claude Code features       |

### Invoking Sub-Agents

**Direct invocation (using `@`):**

```
> @explore Analyse the codebase and summarise the tech stack
> @plan Help me plan the authentication refactor
> @claude-code-guide What are hooks in Claude Code?
```

**Delegated invocation (ask the main agent):**

```
> Please use two explore agents to investigate the tech stack and features in parallel
```

**Background execution (`Ctrl+B`):**

Press `Ctrl+B` after launching a sub-agent to send it to the background. This frees up the main agent for other work.

**Viewing background agents:**

Press the down arrow key → Enter to see running background tasks. Click any task to see its live output.

### Creating Custom Sub-Agents

**Option 1: Using the `/agents` command:**

```
> /agents
```

Choose:

- Create at **project level** (`.claude/agents/`) or **personal level** (`~/.claude/agents/`)
- Let Claude generate the agent or configure manually

**Option 2: Creating the file directly:**

```markdown
## <!-- .claude/agents/ui-expert.md -->

name: ui-expert
description: UI/UX expert specialising in neo-brutalism design
model: opus
tools:

- Read
- Edit
- Write
- Bash

---

You are an experienced UI/UX expert with 20+ years of experience.
For this project, ensure the application uses a neo-brutalism design:

- Bright colors with hard shadows
- Vibrant, bold color palette
- Minimalist layouts
- Responsive on desktop, tablet, and mobile

When reviewing or creating UI:

1. Check responsiveness across breakpoints
2. Ensure consistent spacing and typography
3. Follow the project's design tokens
4. Test hover states and interactions
```

### Choosing Models for Sub-Agents

| Agent Type            | Recommended Model | Reasoning                                              |
| --------------------- | ----------------- | ------------------------------------------------------ |
| **Coding Agent**      | Opus or Sonnet    | Needs high capability for code generation              |
| **Code Review Agent** | Haiku or Sonnet   | Pattern matching is fast; doesn't need deep reasoning  |
| **Research Agent**    | Sonnet            | Good balance of capability and cost                    |
| **Planning Agent**    | Opus              | Needs deepest understanding for architecture decisions |
| **Explore Agent**     | Haiku             | Speed is priority; scanning is a simpler task          |

### The Practical Sub-Agent Workflow

Here's the workflow demonstrated by power users for complex projects:

```
Step 1: PLAN (Plan Mode)
├── Use 3 planning agents (Opus) to investigate the task in parallel
├── Each agent focuses on a different aspect
├── Main context stays at ~14% instead of ~60%
└── Generate a detailed implementation plan → save to spec/ folder

Step 2: IMPLEMENT (Accept Edits Mode)
├── Clear the conversation (/clear)
├── Load the implementation plan (@spec/plan.md)
├── Tell the main agent: "Don't write code yourself. Orchestrate."
├── Coder agents (Sonnet/Opus) implement each phase
├── Code review agents (Haiku) review after each phase
└── Iteration loop: coder → reviewer → coder (until approved)

Step 3: FIX (post-implementation)
├── Review agent identifies issues
├── Spin up coder agents to fix issues in parallel
└── Final review by 3 code review agents from different perspectives
```

**Real-world result:** One user built a full to-do app with Kanban board, calendar view, PostgreSQL database, and authentication — using only 68% of the context window. Without sub-agents, this would have required multiple sessions and compactions.

### Key Principles

> **Sub-agents don't reduce total token usage. They protect the main conversation.**

- Only the **summary** comes back to the main thread
- Detailed tool calls, intermediate reasoning, and dead-ends stay in the sub-agent
- This keeps the main context clean and high-quality longer

### Subagent Reliability: The Probability Problem

When spawning multiple subagents, failure probabilities multiply:

```
Single agent success rate: 95%

3 subagents:  0.95³ = 85.7% all succeed
10 subagents: 0.95¹⁰ = 59.9% all succeed
50 subagents: 0.95⁵⁰ = 7.7% all succeed
```

**Mitigation strategies:**

- Keep subagent tasks simple and well-defined
- Don't overload subagents with complex multi-step workflows
- Let the parent agent handle synthesis and decision-making
- Use subagents for leaf tasks (classification, review, search) not orchestration

### Foreground vs Background Subagents

| Type           | Behavior                                                                                           |
| -------------- | -------------------------------------------------------------------------------------------------- |
| **Foreground** | Blocks main conversation; passes permission prompts through                                        |
| **Background** | Runs concurrently; auto-denies unapproved permissions; press `Ctrl+B` to background a running task |

---

## 10. Agent Teams

### What Are Agent Teams?

Agent teams are the newest Claude Code feature — **multiple agents with a shared task list that communicate with each other in real time**. Unlike sub-agents (isolation), agent teams provide **collaboration**.

### Agent Teams vs. Sub-Agents

| Aspect            | Sub-Agents                    | Agent Teams                          |
| ----------------- | ----------------------------- | ------------------------------------ |
| **Communication** | None (isolated)               | Peer-to-peer + shared task list      |
| **Context**       | Fully isolated                | Shared awareness of team progress    |
| **Token cost**    | Lower                         | 2–4× higher                          |
| **Best for**      | Research, exploration         | Implementation, complex builds       |
| **Coordination**  | None — may step on each other | Managed — agents update shared state |
| **Maturity**      | Stable                        | Experimental                         |

**Rule of thumb:**

- **Sub-agents** for research → produce a plan
- **Agent teams** for implementation → execute the plan

### Why Agent Teams Matter

With sub-agents, a real problem emerges: when one agent changes an API endpoint, another agent building the frontend doesn't know about the change. They step on each other's toes.

Agent teams solve this. A backend agent can tell the frontend agent: _"I changed the user API response shape — update your component."_

### Setup

**Step 1: Enable agent teams**

```bash
# Option A: Environment variable
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# Option B: Global settings
cat >> ~/.claude/settings.json << 'EOF'
{
  "claude_code_experimental_agent_teams": 1,
  "tmux_split_panes": true
}
EOF
```

**Step 2: Install TMUX** (optional but highly recommended for visual monitoring)

```bash
# macOS
brew install tmux

# Ubuntu/Debian
sudo apt install tmux

# Windows (requires WSL)
sudo apt install tmux  # inside WSL
```

> **Note:** TMUX does NOT work inside VS Code or Cursor terminals. Use your system terminal.

**Step 3: Start a TMUX session**

```bash
tmux new-session -s claude
```

**Step 4: Launch Claude Code inside TMUX**

```bash
claude
# or
claude --dangerously-skip-permissions
```

### Using Agent Teams

**Triggering a team:**

Include the phrase "create an agent team" in your prompt:

```
> Create an agent team to review my codebase.
  Have one agent focus on security, one on code quality,
  and one on documentation.
```

**Specifying team details:**

```
> Create an agent team with 3 members to implement this feature.
  Use Sonnet for all agents.
  Agent 1: Backend (database + API)
  Agent 2: Frontend (React components)
  Agent 3: Tests (unit + integration)
```

### What Happens Behind the Scenes

1. **Team leader** (your main Claude instance) reads your request
2. It creates a **shared task list** with all required tasks
3. It **spawns agents** — each in their own TMUX pane
4. Each agent receives:
   - Their specific role and prompt
   - Access to the shared task list
   - Instructions for communicating with other agents
5. Agents work in parallel, updating the shared task list
6. **Race condition protection** prevents two agents from grabbing the same task
7. When all tasks complete, the team leader **shuts down** the team and brings everything back to one session

### Navigating TMUX Panes

| Shortcut                | Action                                 |
| ----------------------- | -------------------------------------- |
| `Ctrl+B` then arrow key | Switch between agent panes             |
| `Ctrl+B` then `[`       | Scroll mode (use arrow keys to scroll) |
| Type in any pane        | Chat with that specific agent          |

### Contract-First Spawning

A key strategy for reliable agent teams:

**Problem:** If you start a database agent and a backend agent at the same time, the backend agent might finish before the database schema is defined — wasting tokens on incorrect code.

**Solution:** Contract-first spawning:

1. Start the **upstream agent** first (e.g., database)
2. It defines the schema and sends its "contract" to the lead agent
3. Only then does the lead agent start the **downstream agent** (e.g., backend)
4. The backend agent starts with correct schema knowledge
5. Database agent **continues working** in parallel

This gives you the benefits of parallelism while ensuring dependencies are respected.

### Team Lifecycle

```
1. SPAWN → Team leader creates agents and task list
2. WORK  → Agents work in parallel, communicate, update tasks
3. IDLE  → Agents finish but stay alive for follow-up feedback
4. FIX   → You tell team leader: "Send backend agent back to fix X"
            (Agent retains its full context — no re-learning)
5. SHUTDOWN → Team leader closes all sessions, cleans up
```

### Persistent Memory Between Sessions

For coding projects, create a shared memory file:

```
> Have all agents log their bugs, fixes, and decisions to a shared file
  called `team-memory.md` in the project root.
```

This file persists after the team shuts down, so future sessions or teams can reference it.

### Cost Considerations

Agent teams are **token-heavy** — typically 2–4× the cost of sub-agents or default mode.

**Cost-saving strategies:**

- Define specific models per agent (Haiku for QA, Sonnet for coding)
- Use contract-first spawning to avoid wasted work
- Monitor costs after each session via the cost tab
- Start with low effort and increase only if needed

### The Debate Pattern

One powerful pattern is using **adversarial agents**:

```
> Spin up four agents to document all security issues, plus a fifth and
> sixth "debate agent" that play devil's advocate. One argues the
> findings are not real issues; the other argues they are real and need
> fixing.
```

This leverages the principle behind **Generative Adversarial Networks** — adversarial pressure improves output quality.

### Practical Example: Website Design Exploration

```
> I'm designing a personal website. Generate three agents using a team
> and create three fundamentally different designs. Open all three once
> done and I'll compare, contrast, and give feedback.
```

Claude's team lead:

1. Spawns a **research agent** to gather information about the user
2. Spawns three **design agents** (Design Minimalist, Design Dark, Design Warm)
3. All three work in parallel on different designs
4. Team lead collects results and presents them

### Converting Team Workflows to Skills

After a successful agent team run:

```
> Based on this conversation, create a skill that I can reuse.
  It should accept a parameter for the research topic or Jira ticket.
  Include the agent team structure we just used.
```

This turns your one-off workflow into a repeatable `/command`.

---

## 11. Hooks & MCP Servers

### Hooks

Hooks are **automatic triggers** that fire when Claude Code performs specific actions — without using any LLM tokens. They're purely programmatic.

📖 **Official Docs:** [Hooks Reference](https://docs.anthropic.com/en/docs/claude-code/hooks)

**Use cases:**

- Check for banned words after generating content
- Validate word count limits
- Auto-format files after editing
- Run linting after code changes
- Log all actions to an audit file

### Hook Events

| Event                            | When It Fires                                           |
| -------------------------------- | ------------------------------------------------------- |
| `SessionStart`                   | When Claude Code starts a new session                   |
| `UserPromptSubmit`               | When you press Enter on a prompt                        |
| `PreToolUse`                     | Before Claude executes a tool (read, write, bash, etc.) |
| `PostToolUse`                    | After a tool finishes executing                         |
| `PostToolUseFailure`             | After a tool fails                                      |
| `PermissionRequest`              | When Claude asks for permission                         |
| `SubagentStart` / `SubagentStop` | When subagents are created/destroyed                    |
| `PreCompact`                     | Before context compaction occurs                        |
| `Stop`                           | When Claude finishes a response                         |
| `SessionEnd`                     | When the session ends                                   |

### Hook Configuration

Hooks are configured in `.claude/settings.json` (project-level) or `~/.claude/settings.json` (global):

```json
{
  "hooks": {
    "afterWrite": [
      {
        "command": "python scripts/check_banned_words.py $FILE",
        "description": "Check for banned words in output"
      }
    ],
    "afterEdit": [
      {
        "command": "npx prettier --write $FILE",
        "description": "Auto-format edited files"
      }
    ]
  }
}
```

**Key point:** Hooks are not AI-powered. They're simple automation scripts. This means:

- ✅ Zero token cost
- ✅ 100% deterministic
- ✅ Run every time (no "hoping" Claude remembers)
- ❌ Can't handle complex reasoning tasks

**Pro tip:** Don't write hook JSON manually. Ask Claude: _"Create a hook that runs ESLint after every file edit"_ and it will generate the settings for you.

### MCP Servers (Model Context Protocol)

MCP servers are **bridges** between Claude Code and external applications. They let Claude read from and write to your day-to-day tools.

**Setup:**

```bash
# Option A: Use the built-in command
claude
> /mcp add airtable

# Option B: Create .mcp.json manually
```

**Example `.mcp.json`:**

```json
{
  "mcpServers": {
    "airtable": {
      "command": "npx",
      "args": ["-y", "@airtable/mcp-server"],
      "env": {
        "AIRTABLE_API_KEY": "your-api-key-here"
      }
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@context7/mcp-server"]
    }
  }
}
```

**Useful MCP servers:**

| Server         | Purpose                                    |
| -------------- | ------------------------------------------ |
| **Airtable**   | Read/write content calendars, spreadsheets |
| **Context7**   | Access library/framework documentation     |
| **Playwright** | Browser automation and visual testing      |
| **Jira**       | Read tickets and update stories            |
| **Slack**      | Send messages and notifications            |
| **GitHub**     | Enhanced GitHub integration                |

**Token considerations:**

> MCPs can be **token-heavy**. Claude isn't always efficient at using MCP tools — it may make redundant calls or retrieve more data than needed. Prefer **local scripts** and **built-in tools** when possible. Reserve MCPs for cases where external app access is truly necessary.

**Recommendation from experienced users:**

- Context7 MCP is excellent (efficient documentation access)
- Be selective — don't install 14 MCPs "just in case"
- Prefer built-in tools for most tasks

### The Memory Hierarchy

Here's how all the configuration layers fit together:

```
┌─────────────────────────────────────────┐
│               LLM Context               │
│  ┌───────────────────────────────────┐  │
│  │  CLAUDE.md (always loaded)        │  │
│  │  Skills (loaded on demand)        │  │
│  │  Commands (loaded when invoked)   │  │
│  │  Conversation history             │  │
│  └───────────────────────────────────┘  │
├─────────────────────────────────────────┤
│           Non-LLM Automation            │
│  ┌───────────────────────────────────┐  │
│  │  Hooks (automatic, deterministic) │  │
│  │  MCP Servers (external bridges)   │  │
│  │  settings.json (permissions)      │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

---

## 12. Plugins

### What Are Plugins?

Plugins are community or official extensions that add capabilities to Claude Code. They're simpler than MCPs and typically focused on improving specific workflows.

### Managing Plugins

```
> /plugins
```

This opens the plugin management interface where you can browse, install, and remove plugins.

### Notable Plugins

| Plugin               | What It Does                                                                                              |
| -------------------- | --------------------------------------------------------------------------------------------------------- |
| **Context7**         | Searches and compresses API documentation from any source, feeds it to Claude in a token-efficient format |
| **Front-end Design** | Anthropic's own plugin claiming to improve UI/UX output quality                                           |
| **Cloud-mem**        | Adds persistent memory across Claude instances by logging all messages                                    |
| **Code Review**      | Enhanced code review capabilities                                                                         |

### Plugin Marketplaces

1. **Official:** [claude-plugins-official](https://github.com/anthropics/claude-plugins-official) — Anthropic's curated directory
2. **Community:** Third-party marketplaces with user-contributed plugins

> **💡 Tip:** Plugins will likely be **deprecated and absorbed into skills** over time. Vanilla Claude Code without extensions already performs very well, and skills provide a more standardized, token-efficient extension mechanism.

---

## 13. Git Worktrees

### What Are Git Worktrees?

Git worktrees allow you to have **multiple working directories** from the same Git repository, each on a different branch. When combined with Claude Code, this enables parallel development without merge conflicts.

### Why Use Worktrees with Claude Code?

When using agent teams or multiple Claude instances, two agents might try to edit the same file simultaneously, causing conflicts. Worktrees solve this by giving each agent its own isolated working directory.

### The Workflow

```
Main Branch (don't touch during development)
    │
    ├── worktree-about/     ← Agent 1 works on about page
    ├── worktree-contact/   ← Agent 2 works on contact page
    └── worktree-services/  ← Agent 3 works on services page
```

### Setting Up a Worktree Workflow

```bash
# Create worktrees for each feature
git worktree add ../my-project-about feature/about
git worktree add ../my-project-contact feature/contact
git worktree add ../my-project-services feature/services
```

Then instruct Claude:

```
> I want to develop three new pages (about, contact, services) in
> parallel. Use git worktrees so each agent works in its own directory.
> Don't modify the main branch. When all pages are done, merge
> everything back.
```

### Advantages Over Agent Teams Alone

| Concern        | Agent Teams Only                         | Agent Teams + Worktrees                       |
| -------------- | ---------------------------------------- | --------------------------------------------- |
| File conflicts | Possible — two agents may edit same file | **Eliminated** — each agent has own directory |
| Rollback       | Difficult — changes are interleaved      | **Easy** — discard the worktree branch        |
| Code review    | Hard to isolate per-feature changes      | **Simple** — one PR per branch                |

### Merging Results

After all agents complete their work:

```bash
# Merge each feature branch back to main
git checkout main
git merge feature/about
git merge feature/contact
git merge feature/services

# Claude can also handle merge conflicts
> There are merge conflicts from the three feature branches. Resolve them.
```

> **💡 Tip:** Add worktree instructions to your CLAUDE.md so Claude Code automatically uses this pattern for parallel development tasks.

---

## 14. Deployment

### Static Sites: Netlify

For static sites (HTML/CSS/JS), **Netlify** is the simplest deployment option:

```
> Deploy this site to Netlify
```

Claude will:

1. Install the Netlify CLI (if needed)
2. Build the project
3. Push to Netlify
4. Return the live URL

### Serverless APIs: Modal

For backend functions, scripts, and API endpoints, **Modal** provides a simple way to create publicly accessible URLs:

```
> Deploy a simple API endpoint that returns "Hello World" and deploy
> it to Modal at a public URL.
```

**Setting up Modal:**

1. Sign up at [modal.com](https://modal.com) (free $5 credits)
2. Create an API token
3. Configure in your project

### The Skill-to-URL Pattern

One of the most powerful deployment patterns:

```
1. Build a skill  (e.g., scrape-leads)
2. Deploy it to a URL via Modal
3. Now anyone can trigger the workflow by visiting the URL
```

```
> Take the scrape-leads skill and put it online. I want a URL that
> shows a form asking what to scrape, then executes the workflow and
> returns a CSV download.
```

Result: A publicly accessible lead scraping tool at a URL like `yourname-scrape-leads.modal.run`

### Other Deployment Options

| Platform             | Best For                       |
| -------------------- | ------------------------------ |
| **Vercel**           | Next.js and React apps         |
| **Netlify**          | Static sites, JAMstack         |
| **Cloudflare Pages** | Fast edge deployment           |
| **GitHub Pages**     | Free static HTML/CSS/JS        |
| **Modal**            | Serverless APIs and functions  |
| **Railway**          | Full-stack apps with databases |

### Deployment Safety

> **⚠️ Warning:** Never deploy AI-generated applications with:
>
> - User authentication
> - Payment processing
> - Sensitive data handling
>
> ...without a thorough **human security review**. AI-generated code frequently has subtle security vulnerabilities that are not obvious to non-engineers.

---

## 15. Best Practices & Workflows

### The Six Core Principles

These principles come from extensive real-world usage by power users and Claude Code's own creator:

#### 1. Stay in Control — Don't Loop Blindly

While autonomous loops (like ROLF) exist, most experienced developers recommend **staying in the driver's seat**:

- Review every plan before execution
- Don't let Claude run indefinitely without oversight
- The output is only as good as your supervision

#### 2. Plan Before Building

**Always start in Plan Mode** for any non-trivial task (`Shift+Tab` to switch):

1. Write your prompt in Plan Mode
2. Let Claude ask clarifying questions
3. Review the proposed plan critically
4. Edit/refine the plan
5. Switch to Accept Edits mode
6. Claude often "one-shots" the implementation from a good plan

> _"If my goal is to write a pull request, I will use plan mode. And I'll go back and forth with Claude until I like its plan. From there, I switch into auto-accept mode. And Claude can usually one-shot it."_ — Boris Cherny, Claude Code creator

#### 3. Use Skills and Agents

Don't rely on the default agent for everything:

- **Skills** give Claude domain expertise (loaded lazily, token-efficient)
- **Sub-agents** isolate heavy tasks from your main context
- **Custom agents** encode your team's standards (UI conventions, coding standards, review criteria)

#### 4. Explicit Over Implicit

Don't hope Claude does the right thing — **tell it explicitly**:

```
# Bad
> Add Google authentication

# Good
> Add Google OAuth authentication using the BetterAuth library.
  Use the docs-explorer agent to read the BetterAuth documentation
  before implementing. Follow the pattern in src/auth/email.ts
  for the implementation structure.
```

#### 5. Trust but Verify

- **Review the code** — don't assume it's correct
- **Give Claude self-verification tools**: tests, linting, type-checking
- **Be aware**: Claude may write tests that match its code rather than your spec
- **Run the app yourself** — visual verification catches issues tests don't

#### 6. You Can Still Write Code

Not everything needs AI:

- Trivial changes (changing a margin, fixing a typo) → do it yourself
- Understanding the codebase → read it yourself
- AI is a tool, not a replacement for your expertise

### Daily Workflow Template

```
1. Start session → /clear (fresh context)
2. Plan mode → Describe the task, review the plan
3. Accept edits → Let Claude implement the plan
4. Verify → Run tests, lint, check the app
5. If context is large → /compact or /clear + start a new task
6. Commit → claude "commit with a descriptive message"
7. Repeat
```

### Token Conservation Strategies

| Strategy                              | Savings                     | Effort |
| ------------------------------------- | --------------------------- | ------ |
| Use Plan Mode                         | 20–40% fewer wasted tokens  | Low    |
| Use sub-agents for research           | 40–60% main context savings | Medium |
| Keep CLAUDE.md lean                   | 5–10% per session           | Low    |
| `/clear` between tasks                | Resets to 0%                | Low    |
| `/compact` at 60%                     | Extends session life        | Low    |
| Choose lower models for simple agents | 50–70% cost reduction       | Low    |
| Avoid unnecessary MCPs                | 10–30% savings              | Low    |

### Safety Checklist

- [ ] Never deploy AI-generated auth/payment code without human review
- [ ] Always test in staging before production
- [ ] Set API key spending limits
- [ ] Use permission modes appropriate to the task
- [ ] Review AI-generated code for security vulnerabilities before sharing
- [ ] Back up your codebase before using Bypass Permissions
- [ ] Understand what MCPs can access before installing them

---

## 16. Advanced Patterns & Automation

### Multi-Agent Orchestration Workflow

The most powerful pattern for complex features:

```
Phase 1: RESEARCH (Sub-agents)
├── 3× Planning agents (Opus) research the codebase in parallel
├── Each focuses on a different aspect (architecture, dependencies, patterns)
├── Main context stays at ~14%
└── Output: detailed implementation plan saved to spec/ folder

Phase 2: PLAN (Main agent)
├── /clear the conversation
├── Load the plan: @spec/implementation-plan.md
├── Review and refine with the main agent
└── Output: approved, phased implementation plan

Phase 3: BUILD (Agent team or sub-agents)
├── Main agent orchestrates — it writes NO code itself
├── Identifies independent tracks (can run in parallel)
├── For each track:
│   ├── Coder agent (Sonnet/Opus) implements
│   ├── Code reviewer agent (Haiku) reviews
│   └── Iteration loop until approved
└── Final: 3× code review agents for comprehensive review

Phase 4: FIX (Sub-agents)
├── Address issues from the final review
├── Parallel coder agents fix issues independently
└── Main context ends at ~60–70% (vs. 100%+ without agents)
```

### Parallel Terminal Approach

For tasks that don't need agent team coordination, simply run multiple Claude instances:

```bash
# Terminal 1
claude "Write LinkedIn posts from the first 3 ideas in the content calendar"

# Terminal 2
claude "Write Instagram content from ideas 4-6"

# Terminal 3
claude "Create email newsletter from ideas 7-9"
```

Each terminal is independent — no shared state, but maximum parallelism.

**Boris Cherny's approach:** Run 5 Claude instances in terminal + 5–10 instances on claude.ai/code, all working on different tasks simultaneously.

### The ROLF Loop (Autonomous Execution)

ROLF is a pattern for running Claude in a completion loop:

**Components:**

1. **`prd.json`** — Product Requirements Document with user stories and acceptance criteria
2. **`loop.sh`** — Bash script that restarts Claude until all tasks are complete
3. **Max iterations** — Safety limit to prevent runaway token usage

**Example `prd.json`:**

```json
{
  "project": "weekly-content-batch",
  "tasks": [
    {
      "id": 1,
      "description": "Write 3 LinkedIn posts about AI coding tools",
      "acceptanceCriteria": "Under 200 words each, no banned words, includes CTA",
      "status": "pending"
    },
    {
      "id": 2,
      "description": "Write 2 Twitter threads about developer productivity",
      "acceptanceCriteria": "5-8 tweets per thread, conversational tone",
      "status": "pending"
    }
  ]
}
```

**How it works:**

1. Claude reads the PRD, picks the next pending task
2. Completes the task, runs self-verification
3. Updates the task status to "done"
4. Session ends; loop.sh starts a fresh Claude session
5. New session reads PRD, sees remaining tasks, picks the next one
6. Repeat until all tasks are done or max iterations reached

**When to use ROLF:**

- ✅ Well-defined, repetitive tasks with clear acceptance criteria
- ✅ Batch processing where each task is independent
- ❌ Complex projects needing planning or creative decisions
- ❌ Tasks with inter-dependencies

### The GSD Framework (Plan-Execute-Verify)

For larger projects, GSD adds a planning layer on top of ROLF:

```
.planning/
  roadmap.md          ← Project phases overview
  state.md            ← Current progress tracking
  requirements.md     ← Full requirements doc

  phase-1-auth/
    plan.md           ← Detailed plan for this phase
    summary.md        ← Execution summary
    uat.md            ← User acceptance testing criteria

  phase-2-database/
    plan.md
    summary.md
    uat.md
```

**Cycle per phase:**

1. **Plan** → Claude creates a detailed plan with tasks broken into waves
2. **Execute** → Claude implements each wave, marking tasks complete
3. **Verify** → UAT file defines what "done" means; Claude tests against it

**Context management benefit:** Each phase has its own context files. When moving to a new phase, Claude reads only that phase's documents — avoiding context rot from earlier phases.

### Converting Workflows to Skills

After completing any successful workflow:

```
> Turn what we just did into a skill.
  Accept a parameter for the topic.
  Let me choose the model for the agents.
  Save it in .claude/skills/
```

Claude will create the skill, which you can then invoke with `/skill-name` in future sessions.

### The Complete Configuration Reference

```
project-root/
├── CLAUDE.md                        ← Project memory (shared, version-controlled)
├── .mcp.json                        ← MCP server configurations
├── .claude/
│   ├── CLAUDE.local.md              ← Personal project memory (gitignored)
│   ├── settings.json                ← Project settings (hooks, permissions)
│   ├── settings.local.json          ← Personal settings (gitignored)
│   ├── commands/
│   │   ├── review-pr.md             ← Custom slash command
│   │   ├── create-component.md
│   │   └── deploy-staging.md
│   ├── agents/
│   │   ├── ui-expert.md             ← Custom sub-agent
│   │   ├── coder.md
│   │   └── code-reviewer.md
│   └── skills/
│       ├── humanizer/
│       │   └── skill.md             ← Installed or custom skill
│       ├── brand-voice/
│       │   ├── skill.md
│       │   └── examples/
│       └── csv-pipeline/
│           ├── skill.md
│           └── scripts/
├── spec/                            ← Implementation plans (optional)
│   └── implementation-plan.md
└── .planning/                       ← GSD framework docs (optional)
    ├── roadmap.md
    ├── state.md
    └── phase-1/
        ├── plan.md
        ├── summary.md
        └── uat.md

~/.claude/                           ← Global configuration
├── CLAUDE.md                        ← Global memory (all projects)
├── settings.json                    ← Global settings
├── agents/                          ← Global agents
│   └── docs-explorer.md
└── skills/                          ← Global skills
    └── skill-creator/
        └── skill.md
```

---

## Quick Reference Card

### Essential Keyboard Shortcuts

| Shortcut         | Action                                |
| ---------------- | ------------------------------------- |
| `Shift + Tab`    | Cycle modes: Accept Edits → Plan Mode |
| `Alt + M`        | Toggle auto-accept edits              |
| `Ctrl + B`       | Send sub-agent to background          |
| `Ctrl + C`       | Cancel current action / exit Claude   |
| `Esc Esc`        | Rewind to previous turn               |
| `↓` (Down arrow) | View background tasks                 |

### Essential Commands

| Command        | Purpose                               |
| -------------- | ------------------------------------- |
| `/init`        | Generate CLAUDE.md from codebase      |
| `/clear`       | Clear conversation history            |
| `/compact`     | Compress conversation to save context |
| `/context`     | View current token usage              |
| `/model`       | Switch AI model                       |
| `/agents`      | Manage sub-agents                     |
| `/skills`      | List available skills                 |
| `/mcp`         | Manage MCP connections                |
| `/permissions` | View/edit tool permissions            |

### Token-Saving Decision Tree

```
Is this a simple, single-file change?
├── YES → Default mode, do it yourself or single prompt
└── NO → Does it require deep research?
    ├── YES → Use SUB-AGENTS (context isolation)
    │         └── Summary returns to main thread
    └── NO → Does it involve multiple inter-dependent files?
        ├── YES → Use AGENT TEAMS (collaboration)
        │         └── Agents communicate via shared task list
        └── NO → Use PLAN MODE → ACCEPT EDITS
                  └── One-shot execution from a good plan
```

---

## 17. Official Documentation Links

| Topic              | URL                                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------------------ |
| Overview           | [docs.anthropic.com/.../overview](https://docs.anthropic.com/en/docs/claude-code/overview)             |
| CLI Reference      | [docs.anthropic.com/.../cli-usage](https://docs.anthropic.com/en/docs/claude-code/cli-usage)           |
| Memory & CLAUDE.md | [docs.anthropic.com/.../memory](https://docs.anthropic.com/en/docs/claude-code/memory)                 |
| Skills             | [docs.anthropic.com/.../skills](https://docs.anthropic.com/en/docs/claude-code/skills)                 |
| Subagents          | [docs.anthropic.com/.../sub-agents](https://docs.anthropic.com/en/docs/claude-code/sub-agents)         |
| MCP                | [docs.anthropic.com/.../mcp](https://docs.anthropic.com/en/docs/claude-code/mcp)                       |
| Hooks              | [docs.anthropic.com/.../hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)                   |
| VS Code            | [docs.anthropic.com/.../vs-code](https://docs.anthropic.com/en/docs/claude-code/vs-code)               |
| GitHub Actions     | [docs.anthropic.com/.../github-actions](https://docs.anthropic.com/en/docs/claude-code/github-actions) |
| Agent Teams        | [docs.anthropic.com/.../agent-teams](https://docs.anthropic.com/en/docs/claude-code/agent-teams)       |

---

_Last updated: February 2026_

> **📦 Source:** Based on the 4-hour Claude Code masterclass by NixRyf, cross-referenced with official Anthropic documentation and multiple community tutorials.
