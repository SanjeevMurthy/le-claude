# 02 — Antigravity Architecture

## Overview

Antigravity IDE is architecturally distinct from every other AI-enhanced editor. Its entire design — from the UI chrome to the internal execution model — is built around the concept of **agents as first-class development participants**. This chapter dissects every layer of the system.

## High-Level Architecture

```
+=========================================================================+
|                           ANTIGRAVITY IDE                                |
|                                                                          |
|  +---------------------------+    +----------------------------------+   |
|  |      EDITOR VIEW          |    |       MANAGER VIEW               |   |
|  |  (Synchronous Coding)     |    |   (Mission Control)              |   |
|  |                           |    |                                  |   |
|  |  +---------+ +--------+  |    |  +------+ +------+ +------+     |   |
|  |  | Editor  | | Agent  |  |    |  |Agent | |Agent | |Agent |     |   |
|  |  | Panes   | | Side-  |  |    |  |Task 1| |Task 2| |Task 3|     |   |
|  |  |         | | bar    |  |    |  |      | |      | |      |     |   |
|  |  +---------+ +--------+  |    |  +------+ +------+ +------+     |   |
|  +---------------------------+    +----------------------------------+   |
|                                                                          |
|  +-------------------------------------------------------------------+  |
|  |                      AGENT EXECUTION LAYER                         |  |
|  |                                                                    |  |
|  |  +----------+  +-----------+  +-----------+  +----------------+   |  |
|  |  | File     |  | Terminal  |  | Browser   |  | Artifact       |   |  |
|  |  | System   |  | Execution |  | Automation|  | Generation     |   |  |
|  |  | Access   |  |           |  |           |  |                |   |  |
|  |  +----------+  +-----------+  +-----------+  +----------------+   |  |
|  +-------------------------------------------------------------------+  |
|                                                                          |
|  +-------------------------------------------------------------------+  |
|  |                         MODEL LAYER                                |  |
|  |                                                                    |  |
|  |  +---------------+  +------------------+  +-------------------+   |  |
|  |  | Gemini 3.1    |  | Claude Sonnet/   |  | GPT-OSS-120B     |   |  |
|  |  | Pro / Flash   |  | Opus 4.6         |  | (Open-Source)     |   |  |
|  |  +---------------+  +------------------+  +-------------------+   |  |
|  +-------------------------------------------------------------------+  |
|                                                                          |
|  +-------------------------------------------------------------------+  |
|  |                    CONTEXT & INTEGRATION LAYER                     |  |
|  |                                                                    |  |
|  |  +----------+  +-----------+  +----------+  +----------------+   |  |
|  |  | Project  |  | MCP       |  | Rules /  |  | Knowledge /    |   |  |
|  |  | Index    |  | Servers   |  | Skills / |  | Memory         |   |  |
|  |  |          |  |           |  | Workflows|  |                |   |  |
|  |  +----------+  +-----------+  +----------+  +----------------+   |  |
|  +-------------------------------------------------------------------+  |
+=========================================================================+
```

## The Dual-Surface Design

Antigravity's UI is split into two distinct views, each designed for a different mode of work.

### Editor View (Synchronous Coding)

The Editor View is familiar to any VS Code user. It provides:

- **File explorer** with project tree navigation
- **Code editor** with syntax highlighting, IntelliSense, and extensions
- **Integrated terminal** for running commands
- **Agent sidebar** for conversational interaction with the AI

The key addition is the **Agent Sidebar**, which provides a chat-like interface for:
- Describing tasks in natural language
- Viewing agent plans and artifacts inline
- Providing feedback on agent work
- Reviewing diffs and implementation proposals

```
+------------------------------------------------------------------+
|  EDITOR VIEW                                                      |
|                                                                   |
|  +----------+  +---------------------+  +---------------------+  |
|  | Explorer |  | Code Editor         |  | Agent Sidebar       |  |
|  |          |  |                     |  |                     |  |
|  | src/     |  | import React from   |  | Agent: I'll create  |  |
|  |  App.tsx |  |   'react';         |  | the user dashboard  |  |
|  |  api/    |  |                     |  | component with the  |  |
|  |  hooks/  |  | function App() {    |  | following plan:     |  |
|  |  utils/  |  |   return (          |  |                     |  |
|  |          |  |     <Dashboard />   |  | [Implementation     |  |
|  |          |  |   );                |  |  Plan Artifact]     |  |
|  |          |  | }                   |  |                     |  |
|  +----------+  +---------------------+  +---------------------+  |
|                                                                   |
|  +--------------------------------------------------------------+ |
|  | Terminal                                                      | |
|  | $ npm run test -- --watch                                     | |
|  +--------------------------------------------------------------+ |
+------------------------------------------------------------------+
```

### Manager View (Mission Control)

The Manager View is Antigravity's most revolutionary feature. It acts as a **control center** for orchestrating multiple agents across multiple workspaces simultaneously.

Key capabilities:
- **Spawn agents** with specific tasks and contexts
- **Monitor agent progress** in real-time across multiple conversations
- **Review artifacts** produced by each agent
- **Provide feedback** that agents incorporate into their work
- **Parallelize development** — one agent refactors while another writes tests

```
+------------------------------------------------------------------+
|  MANAGER VIEW (MISSION CONTROL)                                   |
|                                                                   |
|  +---------------------+  +---------------------+                |
|  | Workspace: frontend |  | Workspace: backend  |                |
|  |                     |  |                     |                |
|  | Agent 1: Building   |  | Agent 3: Writing    |                |
|  | user dashboard      |  | API endpoints       |                |
|  | Status: Executing   |  | Status: Planning    |                |
|  | [View Artifacts]    |  | [View Artifacts]    |                |
|  |                     |  |                     |                |
|  | Agent 2: Writing    |  | Agent 4: Adding     |                |
|  | unit tests          |  | database migrations |                |
|  | Status: Complete    |  | Status: Testing     |                |
|  | [Review Results]    |  | [View Artifacts]    |                |
|  +---------------------+  +---------------------+                |
|                                                                   |
|  [+ Spawn New Agent]  [Global Settings]  [View All Artifacts]    |
+------------------------------------------------------------------+
```

## Agent Execution Model

Every agent in Antigravity follows a structured execution lifecycle:

```
+------------------------------------------------------------------+
|                    AGENT EXECUTION LIFECYCLE                       |
|                                                                   |
|    +---------+     +----------+     +---------+     +----------+ |
|    |         |     |          |     |         |     |          | |
|    | RECEIVE | --> | PLAN     | --> | EXECUTE | --> | VERIFY   | |
|    | TASK    |     |          |     |         |     |          | |
|    |         |     |          |     |         |     |          | |
|    +---------+     +----------+     +---------+     +----------+ |
|         |               |                |               |       |
|         v               v                v               v       |
|    +---------+     +----------+     +---------+     +----------+ |
|    |Parse    |     |Create    |     |Edit     |     |Run       | |
|    |intent   |     |impl plan |     |files    |     |tests     | |
|    |& scope  |     |& task    |     |Run cmds |     |Browse UI | |
|    |         |     |list      |     |Browse   |     |Validate  | |
|    +---------+     +----------+     +---------+     +----------+ |
|                                                                   |
|                    +----> ITERATE <----+                          |
|                    |   (on feedback)   |                          |
|                    +-------------------+                          |
+------------------------------------------------------------------+
```

### Phase 1: Receive Task

The agent receives a task description in natural language. It parses the intent, identifies the scope (which files, which systems), and loads relevant context:

- **Project structure** — directory tree, package manifests
- **Existing code** — imports, function signatures, patterns
- **Rules** — coding standards, conventions, constraints
- **Skills** — relevant skill definitions for the task type
- **MCP data** — database schemas, API specs, external service context

### Phase 2: Plan (Planning Mode)

In Planning Mode, the agent produces two artifacts before writing any code:

1. **Implementation Plan** (`implementation_plan.md`) — A detailed technical proposal:
   - Description of the problem and approach
   - Component-by-component breakdown of file changes
   - New files, modified files, deleted files
   - Verification plan (how changes will be tested)

2. **Task List** (`task.md`) — A checklist broken into subtasks:
   ```
   - [ ] Create database migration for user preferences
   - [ ] Add UserPreferences model with validation
   - [ ] Create API endpoint for GET/PUT preferences
   - [ ] Write unit tests for the model
   - [ ] Write integration tests for the API
   - [ ] Update API documentation
   ```

The developer can review, comment on, and modify these plans before the agent proceeds.

### Phase 3: Execute

The agent has access to three execution surfaces:

| Surface | Capabilities |
|---|---|
| **File System** | Read, create, edit, delete files across the project |
| **Terminal** | Run commands: tests, builds, linters, git operations |
| **Browser** | Navigate to URLs, interact with web pages, take screenshots |

### Phase 4: Verify

After execution, the agent produces verification artifacts:

- **Walkthrough** (`walkthrough.md`) — Summary of changes, test results, screenshots
- **Browser Recordings** — WebP videos of browser interactions
- **Screenshots** — Captured UI states for visual verification
- **Test Results** — Output from test suite runs

## The Artifact System

Artifacts are the backbone of Antigravity's transparency model. Every meaningful action produces a verifiable output.

```
+------------------------------------------------------------------+
|                      ARTIFACT TYPES                               |
|                                                                   |
|  +------------------+  +-------------------+  +----------------+ |
|  | PLANNING         |  | EXECUTION         |  | VERIFICATION   | |
|  |                  |  |                   |  |                | |
|  | implementation   |  | Code diffs        |  | walkthrough.md | |
|  | _plan.md         |  | (inline view)     |  |                | |
|  |                  |  |                   |  | Screenshots    | |
|  | task.md          |  | File create/      |  | (.png, .webp)  | |
|  | (checklist)      |  | delete ops        |  |                | |
|  |                  |  |                   |  | Browser        | |
|  |                  |  | Terminal output   |  | recordings     | |
|  |                  |  |                   |  | (.webp video)  | |
|  |                  |  |                   |  |                | |
|  |                  |  |                   |  | Test results   | |
|  +------------------+  +-------------------+  +----------------+ |
+------------------------------------------------------------------+
```

### Artifact Storage

Artifacts are stored in the Antigravity data directory:

```
~/.gemini/antigravity/brain/<conversation-id>/
├── task.md                    # Task checklist
├── implementation_plan.md     # Technical plan
├── walkthrough.md             # Post-execution summary
└── .system_generated/
    └── logs/
        └── overview.txt       # Conversation log
```

## Model Layer

Antigravity supports multiple AI models with different characteristics:

| Model | Provider | Strengths | Best For |
|---|---|---|---|
| **Gemini 3.1 Pro** | Google | Large context, multi-modal, strong reasoning | Complex planning, multi-file refactoring |
| **Gemini 3 Flash** | Google | Fast, lower latency | Quick tasks, rapid iteration |
| **Claude Sonnet 4.6** | Anthropic | Balanced speed/quality | General development tasks |
| **Claude Opus 4.6** | Anthropic | Highest quality reasoning | Architecture design, complex debugging |
| **GPT-OSS-120B** | Open-source | Open-source, customizable | Self-hosted scenarios |

### Model Selection Strategy

Power users switch models based on task complexity:

```
Fast/Simple Tasks ──────────> Gemini 3 Flash
                              Low latency, sufficient quality

General Development ────────> Claude Sonnet 4.6
                              Best balance of speed and quality

Complex Architecture ───────> Gemini 3.1 Pro / Claude Opus 4.6
                              Maximum reasoning capability

Rate-Limited ───────────────> Switch provider
                              Maintain workflow continuity
```

## Context & Integration Layer

### Project Indexing

Antigravity automatically indexes the entire project on workspace open:

- Builds a **file tree** representation
- Extracts **imports and dependencies**
- Identifies **function signatures and types**
- Creates **embeddings** for semantic search
- Respects `.gitignore` and exclusion patterns

### Model Context Protocol (MCP)

MCP is a standard protocol that connects agents to external data sources:

```
+------------------------------------------------------------------+
|                    MCP INTEGRATION                                 |
|                                                                   |
|  Agent                                                            |
|    |                                                              |
|    +---> MCP Client                                               |
|           |                                                       |
|           +---> PostgreSQL MCP Server  --> Live DB schemas         |
|           |                                                       |
|           +---> GitHub MCP Server      --> PRs, issues, branches  |
|           |                                                       |
|           +---> Cloud SQL MCP Server   --> Production data models |
|           |                                                       |
|           +---> Jira MCP Server        --> Tickets, requirements  |
|           |                                                       |
|           +---> Custom MCP Server      --> Any API / data source  |
+------------------------------------------------------------------+
```

MCP servers are configured in `mcp_config.json`, accessible from the MCP Store panel in the editor. See [Chapter 11 — Advanced Configurations](./11-advanced-configurations.md) for detailed setup.

### Knowledge & Memory

Antigravity maintains persistent knowledge across conversations:

- **Knowledge Items (KIs)** — Distilled insights from previous conversations stored in `~/.gemini/antigravity/knowledge/`
- **Conversation Logs** — Full transcripts stored per conversation ID
- **Artifacts** — Reusable plans and walkthroughs from past work

This memory system allows agents to:
- Reference prior architectural decisions
- Reuse established patterns
- Avoid repeating research already completed
- Maintain consistency across sessions

---

*Previous: [01 — Introduction](./01-introduction.md) | Next: [03 — Installation and Setup](./03-installation-and-setup.md)*
