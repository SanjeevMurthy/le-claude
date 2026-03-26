# 05 — Agent System

## Overview

The agent system is the heart of Antigravity IDE. Unlike traditional AI coding assistants that respond to individual prompts, Antigravity agents are **autonomous entities** that maintain state, follow structured workflows, and produce verifiable artifacts throughout their work.

## Agent Architecture

```
+------------------------------------------------------------------+
|                    AGENT INTERNAL ARCHITECTURE                     |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |                   AGENT CONTEXT                              |  |
|  |                                                              |  |
|  |  +-----------+  +----------+  +----------+  +----------+   |  |
|  |  | System    |  | Project  |  | Conv     |  | Knowledge|   |  |
|  |  | Prompt +  |  | Context  |  | History  |  | Items    |   |  |
|  |  | Rules     |  |          |  |          |  |          |   |  |
|  |  +-----------+  +----------+  +----------+  +----------+   |  |
|  +------------------------------------------------------------+  |
|                          |                                       |
|                          v                                       |
|  +------------------------------------------------------------+  |
|  |                   REASONING ENGINE                           |  |
|  |                                                              |  |
|  |  Task Decomposition --> Planning --> Execution --> Verify    |  |
|  +------------------------------------------------------------+  |
|                          |                                       |
|                          v                                       |
|  +------------------------------------------------------------+  |
|  |                      TOOLS                                   |  |
|  |                                                              |  |
|  |  +----------+  +----------+  +----------+  +----------+    |  |
|  |  | view_file|  | write_   |  | run_     |  | browser_ |    |  |
|  |  | find_by_ |  | to_file  |  | command  |  | subagent |    |  |
|  |  | name     |  | replace_ |  | send_    |  | read_url |    |  |
|  |  | grep_    |  | file_    |  | command_ |  | search_  |    |  |
|  |  | search   |  | content  |  | input    |  | web      |    |  |
|  |  +----------+  +----------+  +----------+  +----------+    |  |
|  +------------------------------------------------------------+  |
|                          |                                       |
|                          v                                       |
|  +------------------------------------------------------------+  |
|  |                   ARTIFACT OUTPUT                            |  |
|  |                                                              |  |
|  |  task.md | implementation_plan.md | walkthrough.md           |  |
|  |  screenshots | browser recordings | code diffs              |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

## Agent Modes

### Planning Mode (Recommended for Complex Tasks)

In Planning Mode, the agent follows a rigorous process:

1. **Research Phase** — Examines project structure, reads relevant files, checks KIs
2. **Planning Phase** — Creates `implementation_plan.md` and `task.md`
3. **Review Gate** — Pauses for developer approval
4. **Execution Phase** — Writes code according to the approved plan
5. **Verification Phase** — Runs tests, validates changes, creates `walkthrough.md`

```
Developer Request
       │
       v
┌──────────────┐
│  PLANNING    │──> implementation_plan.md
│  MODE        │──> task.md
└──────┬───────┘
       │
       v
┌──────────────┐
│  DEVELOPER   │──> Approve / Request Changes / Add Comments
│  REVIEW      │
└──────┬───────┘
       │
       v
┌──────────────┐
│  EXECUTION   │──> File edits, terminal commands, browser actions
│  MODE        │
└──────┬───────┘
       │
       v
┌──────────────┐
│ VERIFICATION │──> walkthrough.md, screenshots, test results
│  MODE        │
└──────────────┘
```

### Fast Mode (For Simple Tasks)

Fast Mode skips the planning phase and executes directly. Use for:
- Simple renames or formatting changes
- Quick bug fixes with obvious solutions
- Adding boilerplate code
- Running commands

### Agent-Assisted Mode

A balance where the agent decides when to plan and when to execute directly:
- Plans when the task is complex or ambiguous
- Executes directly when the task is straightforward
- Asks for clarification when uncertain

## Agent Tool System

Agents interact with the development environment through a comprehensive tool set:

### File System Tools

| Tool | Purpose |
|---|---|
| `view_file` | Read file contents (supports line ranges) |
| `list_dir` | List directory contents with metadata |
| `find_by_name` | Search for files using glob patterns |
| `grep_search` | Search for patterns within files (ripgrep) |
| `write_to_file` | Create new files |
| `replace_file_content` | Edit a contiguous block in existing files |
| `multi_replace_file_content` | Edit multiple non-contiguous blocks |

### Terminal Tools

| Tool | Purpose |
|---|---|
| `run_command` | Execute shell commands |
| `send_command_input` | Send input to running commands |
| `command_status` | Check output of background commands |
| `read_terminal` | Read terminal contents |

### Browser Tools

| Tool | Purpose |
|---|---|
| `browser_subagent` | Launch a browser automation subagent |
| `read_url_content` | Fetch and parse web page content |
| `search_web` | Perform web searches |
| `generate_image` | Generate images from text prompts |

### Communication Tools

| Tool | Purpose |
|---|---|
| `notify_user` | Communicate with user during tasks |
| `task_boundary` | Update task progress and status |

## Agent Communication Patterns

### The Task Boundary Pattern

Agents use `task_boundary` to communicate progress through structured updates:

```
TaskName: "Implementing User Authentication"
Mode: PLANNING → EXECUTION → VERIFICATION
TaskStatus: "Researching existing auth patterns"
TaskSummary: "Identified NextAuth.js as the authentication library..."
```

This creates a visible timeline in the Manager View for tracking agent progress.

### Artifact-Based Feedback Loop

```
Agent produces artifact
       │
       v
Developer reviews inline
       │
       v
Developer adds comments directly
       │
       v
Agent incorporates feedback
       │
       v
Agent updates artifact
       │
       v
Cycle repeats until satisfied
```

Developers can comment directly on implementation plans, similar to Google Docs commenting. The agent reads these comments and adjusts its work accordingly.

## Multi-Agent Collaboration

### Parallel Agent Pattern

In the Manager View, multiple agents can work simultaneously:

```
┌──────────────────────────────────────────────┐
│              PARALLEL AGENTS                  │
│                                              │
│  Agent A: "Implement user dashboard"         │
│  ├── Planning... → Executing...              │
│  └── Workspace: frontend/                    │
│                                              │
│  Agent B: "Write API endpoints for users"    │
│  ├── Planning... → Executing...              │
│  └── Workspace: backend/                     │
│                                              │
│  Agent C: "Create database migrations"       │
│  ├── Executing... → Complete                 │
│  └── Workspace: backend/                     │
│                                              │
│  Agent D: "Write integration tests"          │
│  ├── Waiting (depends on A, B)               │
│  └── Workspace: tests/                       │
└──────────────────────────────────────────────┘
```

### Sequential Agent Pipeline

For dependent tasks, agents can work in sequence:

```
Agent 1: Architecture Design
    │
    └──> Produces: architecture-doc.md
              │
              v
Agent 2: Backend Implementation
    │
    └──> Produces: API code + tests
              │
              v
Agent 3: Frontend Implementation
    │
    └──> Produces: UI components + tests
              │
              v
Agent 4: Integration Testing
    │
    └──> Produces: test results + walkthrough
```

## Agent Memory and Context Persistence

### Within a Conversation

- Full message history maintained
- All file reads and writes tracked
- Terminal output preserved
- Artifact state maintained

### Across Conversations

Antigravity maintains persistent memory through:

1. **Knowledge Items (KIs)** — Distilled insights stored in `~/.gemini/antigravity/knowledge/`
2. **Conversation Logs** — Searchable transcripts of past sessions
3. **Artifacts** — Previous plans, walkthroughs, and analysis

### Practical Impact

When starting a new conversation:
- The agent receives **summaries** of recent conversations
- It can **retrieve** full conversation logs when needed
- It can **read KI artifacts** for previously researched topics
- It avoids **redundant research** by consulting prior work first

```
New Task: "Add pagination to the user list API"
       │
       v
Agent checks KI summaries
       │
       ├── Found: "API Design Patterns" KI
       │   └── Reads: pagination_patterns.md
       │
       ├── Found: "User API Implementation" KI
       │   └── Reads: user_endpoints.md
       │
       └── No relevant KI for specific requirement
           └── Starts fresh research on project code
```

## Agent Orchestration Strategies

### Strategy 1: Divide and Conquer

Assign different aspects of a feature to different agents:
- Agent A handles data model changes
- Agent B handles API endpoints
- Agent C handles UI components
- Agent D handles tests

### Strategy 2: Specialist Agents

Assign agents with specialized skills:
- Architecture Agent (uses architecture skill)
- Security Agent (uses security review skill)
- Testing Agent (uses test generation skill)

### Strategy 3: Review Loop

One agent implements, another reviews:
- Agent A writes the code
- Agent B reviews for quality, security, and patterns
- Agent A addresses review feedback

---

*Previous: [04 — Core Configuration](./04-core-configuration.md) | Next: [06 — Skills System](./06-skills-system.md)*
