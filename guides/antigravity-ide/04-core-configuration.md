# 04 — Core Configuration

## Overview

Antigravity's configuration operates at three levels: **global** (all projects), **workspace** (per-project), and **conversation** (per-session). Understanding this hierarchy is critical for effective agent management.

```
+------------------------------------------------------------------+
|                  CONFIGURATION HIERARCHY                          |
|                                                                   |
|  GLOBAL (~/.gemini/)                                              |
|  ├── GEMINI.md                    Always-active rules             |
|  ├── antigravity/settings.json    IDE preferences                 |
|  ├── antigravity/skills/          Available everywhere            |
|  └── antigravity/global_workflows Available everywhere            |
|       |                                                           |
|       v                                                           |
|  WORKSPACE (.agents/)                                             |
|  ├── rules/                       Project-specific rules          |
|  ├── skills/                      Project-specific skills         |
|  └── workflows/                   Project-specific workflows      |
|       |                                                           |
|       v                                                           |
|  CONVERSATION (in-session)                                        |
|  ├── Agent mode (Planning/Fast)                                   |
|  ├── Model selection                                              |
|  └── Inline instructions                                         |
+------------------------------------------------------------------+
```

## Model Selection and Configuration

### Available Models

| Model | Token Limit | Speed | Quality | Cost |
|---|---|---|---|---|
| Gemini 3.1 Pro | ~2M tokens | Moderate | Highest (Google) | Free tier generous |
| Gemini 3 Flash | ~1M tokens | Fast | Good | Free tier generous |
| Claude Sonnet 4.6 | ~200K tokens | Fast | Very High | API-based |
| Claude Opus 4.6 | ~200K tokens | Slower | Highest | API-based |
| GPT-OSS-120B | ~128K tokens | Moderate | Good | Self-hosted |

### Model Switching Strategy

```
┌─────────────────────────────────────────────────────────────┐
│                   MODEL SELECTION FLOW                       │
│                                                             │
│   Task Complexity Assessment                                │
│        │                                                    │
│        ├── Simple (rename, format, small fix)                │
│        │   └── Gemini 3 Flash                               │
│        │                                                    │
│        ├── Medium (new component, unit tests, refactor)      │
│        │   └── Claude Sonnet 4.6 or Gemini 3.1 Pro          │
│        │                                                    │
│        ├── Complex (architecture, multi-file, debugging)     │
│        │   └── Claude Opus 4.6 or Gemini 3.1 Pro            │
│        │                                                    │
│        └── Rate-limited on current provider?                 │
│            └── Switch to alternate provider                  │
└─────────────────────────────────────────────────────────────┘
```

## Rules Configuration

Rules are the most powerful mechanism for controlling agent behavior. They are **persistent guidelines** injected into the agent's system prompt.

### Rule File Format

```markdown
---
description: "Enforce TypeScript strict mode and functional patterns"
activation: always  # Options: always, manual, model_decision, glob_pattern
globs: ["**/*.ts", "**/*.tsx"]  # Only for glob_pattern activation
---

# TypeScript Coding Standards

## Required Patterns
- Always use strict TypeScript (`strict: true` in tsconfig.json)
- Prefer `const` over `let`; never use `var`
- Use functional components with hooks, not class components
- All functions must have explicit return types
- All exported functions must have JSDoc documentation

## Forbidden Patterns
- No `any` type — use `unknown` and type guards
- No default exports — use named exports only
- No inline styles — use CSS modules or styled-components
- No console.log in production code — use the logger utility

## Error Handling
- All async functions must have try/catch with typed errors
- Use the `Result<T, E>` pattern for fallible operations
- Never swallow errors silently

## Testing
- Every new function must have at least one unit test
- Test file must be co-located: `Component.tsx` → `Component.test.tsx`
- Use `describe`/`it` blocks with descriptive test names
```

### Rule Types

| Activation Type | Behavior | Example Use Case |
|---|---|---|
| `always` | Injected into every conversation | Core coding standards |
| `manual` | Developer triggers explicitly | Special review processes |
| `model_decision` | Agent decides if relevant | Context-specific guidelines |
| `glob_pattern` | Active only for matching files | Language-specific rules |

### Global vs. Workspace Rules

| Location | Scope | Example |
|---|---|---|
| `~/.gemini/GEMINI.md` | All projects | Personal coding style, preferred patterns |
| `.agents/rules/*.md` | Current project only | Project-specific architecture constraints |

> **Important:** Workspace rules with the same name override global rules.

### Character Limit

Rules are limited to **12,000 characters** per file. For complex rule sets, split into multiple files:

```
.agents/rules/
├── coding-standards.md      # Language and style rules
├── architecture.md          # Architectural constraints
├── testing.md               # Testing requirements
├── security.md              # Security policies
└── documentation.md         # Documentation standards
```

## Context Window Management

### How Context is Loaded

When a conversation starts, the agent's context is populated in this order:

1. **System prompt** — Antigravity's core instructions
2. **Rules** — All active rules injected
3. **Skill summaries** — Names and descriptions of available skills (lightweight)
4. **Project structure** — File tree, dependencies
5. **Open files** — Content of files visible in editor
6. **Conversation history** — Prior messages in this session
7. **MCP data** — External context from connected servers
8. **Knowledge Items** — Relevant KIs from past conversations

### Context Budget Strategy

```
+------------------------------------------------------------------+
|              CONTEXT WINDOW BUDGET (Example: 2M tokens)           |
|                                                                   |
|  ┌────────────────────────────────────┐                           |
|  │ System Prompt + Rules   (~5K)     │ Fixed overhead             |
|  ├────────────────────────────────────┤                           |
|  │ Skill Summaries         (~2K)     │ Scales with # skills       |
|  ├────────────────────────────────────┤                           |
|  │ Project Structure       (~10K)    │ Scales with project size   |
|  ├────────────────────────────────────┤                           |
|  │ Open Files + Diffs      (~50K)    │ Variable                   |
|  ├────────────────────────────────────┤                           |
|  │ Conversation History    (~100K+)  │ Grows per message          |
|  ├────────────────────────────────────┤                           |
|  │ MCP Data                (~20K)    │ Per-query                   |
|  ├────────────────────────────────────┤                           |
|  │ Knowledge Items         (~10K)    │ On-demand                   |
|  ├────────────────────────────────────┤                           |
|  │ >>>  AVAILABLE FOR TASK  <<<      │ ~1.8M remaining            |
|  └────────────────────────────────────┘                           |
+------------------------------------------------------------------+
```

### Optimization Tips

1. **Keep rules concise** — Every character counts against the context budget
2. **Use glob-activated rules** — Only load rules relevant to the current file type
3. **Start new conversations** for unrelated tasks — prevents context bloat
4. **Use skills over inline instructions** — Skills are loaded only when needed
5. **Limit MCP server connections** — Only connect servers relevant to the current task

## Terminal Integration

### Command Execution Configuration

```json
// .vscode/settings.json
{
  "antigravity.terminal.autoExecute": "auto",
  "antigravity.terminal.allowList": [
    "npm run *",
    "npx *",
    "git status",
    "git diff *",
    "git log *",
    "python -m pytest *",
    "make *"
  ],
  "antigravity.terminal.denyList": [
    "rm -rf *",
    "git push --force*",
    "npm publish",
    "docker rm *",
    "kubectl delete *"
  ]
}
```

### Turbo Mode Configuration

For trusted environments, Turbo Mode allows full autonomous command execution with safety rails:

```markdown
// In workflow files, annotate with turbo:

2. Run the test suite
// turbo
3. Run the linter
// turbo
4. Deploy to staging (requires manual approval)
5. Run smoke tests
// turbo
```

The `// turbo` annotation auto-approves the immediately following step. Use `// turbo-all` in a workflow to auto-approve every command-execution step.

## File System Permissions

### Project Indexing Configuration

```json
// .vscode/settings.json
{
  "antigravity.indexing.exclude": [
    "**/node_modules/**",
    "**/dist/**",
    "**/build/**",
    "**/.git/**",
    "**/coverage/**",
    "**/*.min.js",
    "**/*.map"
  ],
  "antigravity.indexing.maxFileSize": "1MB",
  "antigravity.indexing.include": [
    "**/*.ts",
    "**/*.tsx",
    "**/*.py",
    "**/*.go",
    "**/*.md"
  ]
}
```

---

*Previous: [03 — Installation and Setup](./03-installation-and-setup.md) | Next: [05 — Agent System](./05-agent-system.md)*
