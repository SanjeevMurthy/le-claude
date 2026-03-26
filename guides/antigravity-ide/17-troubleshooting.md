# 17 — Troubleshooting

## Overview

This chapter covers common issues encountered when using Antigravity IDE, their root causes, and proven solutions sourced from community discussions and documentation.

## Common Issues and Solutions

### 1. Agent Not Responding / Stuck

**Symptoms:**
- Agent stops producing output
- Progress indicator stalls
- No new messages appearing

**Solutions:**

| Solution | Steps |
|---|---|
| Move terminal to editor | If the agent is waiting for terminal input, move the terminal session to the IDE's terminal tab and provide input directly |
| Start new conversation | Long conversations can exhaust context; fresh start often resolves |
| Switch model | Rate limiting may be causing delays; switch to a different model provider |
| Restart Antigravity | Clear cached state: Cmd/Ctrl + Shift + P → "Reload Window" |

### 2. Agent Forgets Instructions

**Symptoms:**
- Agent stops following rules partitioned earlier in conversation
- Agent repeats previously addressed mistakes
- Quality degrades as conversation gets longer

**Root Cause:** Context window pressure — as conversation grows, earlier instructions may be displaced.

**Solutions:**

```
1. BREAK INTO SHORTER CONVERSATIONS
   One feature per conversation
   
2. USE RULES (NOT INLINE INSTRUCTIONS)
   Rules are always injected at the top of context
   Rules are never displaced by conversation growth
   
3. REFERENCE ARTIFACTS
   "As specified in implementation_plan.md..."
   This anchors the agent to a stable reference
   
4. RESTART WITH SUMMARY
   Start new conversation with:
   "Continue the work from conversation [ID].
   Current status: [summary]
   Next task: [specific task]"
```

### 3. Rate Limiting

**Symptoms:**
- "Rate limit exceeded" errors
- Slowdowns in agent response
- Temporary inability to send messages

**Solutions:**

| Strategy | Details |
|---|---|
| **Switch models** | If Gemini is rate-limited, switch to Claude (or vice versa) |
| **Wait and retry** | Rate limits reset periodically; wait 5-10 minutes |
| **Multiple accounts** | Some users authenticate multiple Google accounts via Open Code to load-balance requests |
| **Reduce frequency** | Batch smaller requests into fewer, larger requests |
| **Off-peak hours** | Usage is lower during non-US business hours |

### 4. Editor Becomes Sluggish/Laggy

**Symptoms:**
- Editor input lag
- Slow file switching
- High memory usage
- UI freezes during agent operations

**Root Cause:** Long sessions accumulate state; potential memory leaks in early versions.

**Solutions:**

```
1. RESTART ANTIGRAVITY
   The most reliable fix for accumulated state issues

2. REDUCE EXTENSIONS
   Disable extensions you're not actively using
   
3. OPTIMIZE INDEXING
   Add large directories to exclude patterns:
   "antigravity.indexing.exclude": [
     "**/node_modules/**",
     "**/dist/**",
     // etc.
   ]

4. CLOSE UNUSED TERMINALS
   Each terminal session consumes resources

5. REDUCE FILE WATCHERS
   "files.watcherExclude": {
     "**/node_modules/**": true,
     "**/dist/**": true
   }
```

### 5. Agent Generates Hallucinated Code

**Symptoms:**
- Agent references APIs/functions that don't exist
- Agent creates code that doesn't compile
- Agent invents configuration options
- Agent gets stuck in a fix-break loop

**Solutions:**

```
1. USE PLANNING MODE
   Forces research before code generation

2. PROVIDE REFERENCE FILES
   "Follow the exact pattern in src/services/existing.service.ts"

3. ADD ANTI-HALLUCINATION RULES
   (See Chapter 13 for the complete rule set)

4. CHECK MODEL
   Some models hallucinate more than others
   Claude Opus/Sonnet tend to be more reliable
   
5. BREAK DOWN THE TASK
   Smaller, focused tasks = fewer hallucinations

6. VERIFY WITH TOOLS
   Run lint/type-check/tests after each change
   Forces the agent to confront reality
```

### 6. Extensions Not Available

**Symptoms:**
- Favorite VS Code extension not found
- Extension marketplace showing limited results

**Root Cause:** Antigravity uses Open VSX Registry, not the official VS Code Marketplace.

**Solutions:**

| Solution | Details |
|---|---|
| **Search Open VSX** | Visit [open-vsx.org](https://open-vsx.org) to check availability |
| **Find alternatives** | Most popular extensions have Open VSX equivalents |
| **Manual install** | Download VSIX files and install manually: Extensions → "..." → "Install from VSIX" |

### 7. Workspace Not Recognized

**Symptoms:**
- Agent doesn't see project files
- Agent can't read the project structure
- "No workspace" errors

**Solutions:**

```
1. OPEN AS FOLDER
   Use File → Open Folder (not File → Open File)

2. CREATE .agents DIRECTORY
   mkdir -p .agents/rules .agents/skills .agents/workflows

3. VERIFY WORKSPACE
   Check that the bottom bar shows the workspace name

4. RESTART
   Close and reopen the workspace
```

### 8. Agent Performs Destructive Actions

**Symptoms:**
- Files deleted unexpectedly
- Environment variables overwritten
- Git history modified

**Prevention:**

```markdown
# .agents/rules/safety.md
---
description: "Safety constraints to prevent destructive actions"
activation: always
---

# Safety Rules

## NEVER (without explicit developer approval)
- Delete files or directories
- Force push to any branch
- Modify .env files
- Reset git history
- Run npm publish
- Modify CI/CD pipeline configs
- Change database schemas in production
- Remove tests that were passing

## ALWAYS
- Work on feature branches (never directly on main)
- Backup before large refactoring (git stash or new branch)
- Confirm destructive operations with the developer
```

### 9. MCP Server Connection Issues

**Symptoms:**
- MCP server fails to start
- "Connection refused" errors
- MCP data not available to agent

**Solutions:**

```
1. CHECK CONFIG SYNTAX
   Verify mcp_config.json is valid JSON
   
2. CHECK DEPENDENCIES
   Ensure the MCP server package is installed:
   npx -y @modelcontextprotocol/server-postgres --help

3. CHECK ENVIRONMENT VARIABLES
   MCP configs reference env vars — ensure they're set
   
4. CHECK PERMISSIONS
   Database MCP servers need correct credentials
   
5. TEST STANDALONE
   Run the MCP server command manually in terminal to debug

6. CHECK LOGS
   Look in Antigravity's output panel for MCP error messages
```

### 10. Agent Creates Wrong File Structure

**Symptoms:**
- Agent puts files in unexpected locations
- Agent doesn't follow project conventions
- Agent creates duplicate structures

**Solution — Use Architecture Rules:**

```markdown
# .agents/rules/file-structure.md
---
description: "Strict project file structure rules"
activation: always
---

# File Structure Rules

ALWAYS place files in these EXACT locations:
- React components: src/components/<feature>/ComponentName.tsx
- Pages: app/<route>/page.tsx
- API routes: app/api/<resource>/route.ts
- Services: src/services/<name>.service.ts
- Validators: src/validators/<name>.validator.ts
- Tests: Adjacent to source file (same directory, .test.tsx)
- Types: src/types/<name>.types.ts
- Utils: src/utils/<name>.ts

NEVER create files in:
- The project root (except config files)
- A separate test/ directory (co-locate tests)
- Nested src/src/ paths (common agent mistake)
```

## Diagnostic Checklist

When issues arise, work through this checklist:

```
□ Is Antigravity up to date? (Check for updates)
□ Is the workspace properly opened? (File → Open Folder)
□ Are rules properly formatted? (Valid markdown + frontmatter)
□ Is the network connection stable? (Required for model API)
□ Is the correct model selected? (Check model dropdown)
□ Are there relevant error messages? (Check Output panel)
□ Is the conversation too long? (Start a new one)
□ Are there conflicting rules? (Check for contradictions)
□ Is the terminal responsive? (Try executing a simple command)
□ Has the IDE been restarted recently? (Fixes many issues)
```

## Getting Help

| Resource | Where |
|---|---|
| Official Documentation | [antigravity.google](https://antigravity.google) |
| GitHub Issues | Search and file issues on the Antigravity GitHub repo |
| Reddit | r/AntigravityIDE, r/programming |
| Discord/Slack | Community channels (check official site for links) |
| YouTube | Official channel and community tutorials |

---

*Previous: [16 — Agent Skill Library](./16-agent-skill-library.md)*

---

**End of Antigravity IDE Research Guide**

*This guide was researched and compiled from official documentation, community discussions, developer blogs, YouTube tutorials, and security research. Last updated: March 2026.*
