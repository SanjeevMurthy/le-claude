# 13 — Performance Optimization

## Overview

Performance in Antigravity IDE spans two domains: **IDE performance** (responsiveness of the editor and agent) and **agent output quality** (accuracy and reliability of generated code). Optimizing both is critical for productive development.

## Reducing Hallucinations

AI hallucinations — where the agent generates incorrect, non-functional, or fabricated code — are the single biggest threat to productivity. Here are proven strategies to minimize them.

### Strategy 1: Use Planning Mode

```
WITHOUT Planning Mode:
  Request → Agent generates code immediately
           → Higher chance of architectural mistakes
           → Harder to catch errors early
           → Compounds errors across files

WITH Planning Mode:
  Request → Agent researches codebase
          → Agent creates implementation plan
          → Developer reviews plan
          → Agent implements approved plan
          → Errors caught before any code is written
```

**Impact:** Community reports indicate Planning Mode reduces hallucinations by approximately 60-70% for complex tasks.

### Strategy 2: Break Down Tasks

```
BAD: "Build a complete user management system"
(Too broad — agent tries to hold entire system in context)

GOOD:
1. "Design the database schema for user management"
2. "Implement the User model with validation"
3. "Create the authentication service"
4. "Build the user CRUD API endpoints"
5. "Write tests for the user service"
```

**Rule of thumb:** If the task would result in changes to more than 5 files, break it into subtasks.

### Strategy 3: Provide Explicit Constraints

```markdown
# .agents/rules/reduce-hallucinations.md
---
description: "Rules to minimize agent hallucinations"
activation: always
---

# Anti-Hallucination Rules

## Before Writing Code
- ALWAYS read existing files related to the change first
- NEVER assume an API, function, or import exists — verify by reading the source
- Check package.json / requirements.txt for available dependencies
- Read existing tests to understand expected behavior

## While Writing Code
- Use ONLY libraries that are already in the project's dependencies
- Match the coding patterns visible in existing code
- Use the exact function signatures from existing source files
- Never invent configuration keys — check existing configs first

## After Writing Code
- Run the linter to catch syntax errors
- Run the type checker to catch type errors
- Run affected tests to catch logic errors
- If any check fails, fix it before proceeding
```

### Strategy 4: Choose the Right Model

| Scenario | Best Model | Why |
|---|---|---|
| Complex architecture | Claude Opus 4.6 | Best reasoning, fewest hallucinations |
| Multi-file refactoring | Gemini 3.1 Pro | Largest context window |
| Quick fixes | Gemini 3 Flash | Speed over quality |
| Code review | Claude Sonnet 4.6 | Good balance |

### Strategy 5: Use Reference Material

Provide the agent with examples of correct implementation:

```
"Implement a new API endpoint for /api/v1/orders following
the exact same pattern used in src/routes/users.routes.ts
and src/controllers/users.controller.ts. Reference those
files for structure, error handling, and response format."
```

## Improving Code Quality

### Enforcing Architecture Consistency

```markdown
# .agents/rules/architecture-consistency.md
---
description: "Enforce consistent architecture patterns across the codebase"
activation: always
---

# Architecture Consistency Rules

## Project Structure (MUST Follow)
\```
src/
├── routes/        # Express route definitions ONLY
├── controllers/   # HTTP request/response handling ONLY
├── services/      # Business logic ONLY
├── repositories/  # Data access ONLY
├── models/        # Data models and schemas
├── middleware/     # Express middleware
├── validators/    # Request validation schemas
├── utils/         # Pure utility functions
└── config/        # Configuration loading
\```

## Layer Rules
- Routes → ONLY call controllers
- Controllers → ONLY call services + validators
- Services → ONLY call repositories + other services
- Repositories → ONLY call database/ORM
- NEVER skip layers (e.g., route directly calling repository)

## Naming Conventions
- Files: kebab-case (`user-profile.service.ts`)
- Classes: PascalCase (`UserProfileService`)
- Functions: camelCase (`getUserProfile`)
- Constants: SCREAMING_SNAKE_CASE (`MAX_RETRY_COUNT`)
- Database tables: snake_case (`user_profiles`)
```

### Code Quality Workflow

```markdown
# .agents/workflows/quality-check.md
---
description: "Run comprehensive code quality checks"
---

## Steps

1. Run ESLint:
// turbo
2. Run: `npm run lint`
3. If lint errors found, fix them automatically:
// turbo
4. Run: `npm run lint:fix`

5. Run TypeScript type checker:
// turbo
6. Run: `npx tsc --noEmit`

7. Run tests with coverage:
// turbo
8. Run: `npm run test -- --coverage`

9. Check for code complexity:
// turbo
10. Run: `npx eslint --rule '{"complexity": ["error", 10]}' src/ || true`

11. Report results with pass/fail for each gate
```

## Reducing Token Waste

### Context Efficiency Techniques

| Technique | Token Savings | How |
|---|---|---|
| **Glob-activated rules** | ~30% | Only load rules for relevant file types |
| **Focused conversations** | ~50% | One topic per conversation |
| **Specific file reading** | ~40% | Read line ranges, not entire files |
| **Skill lazy loading** | ~20% | Skills loaded only when needed |
| **MCP on-demand** | ~15% | Connect only needed servers |

### Prompt Optimization

```
WASTEFUL: "Can you please look at the entire codebase and
understand everything about it and then make some improvements
to the user authentication flow?"

EFFICIENT: "In src/services/auth.service.ts, the login function
(line 45-80) doesn't handle expired JWT tokens. Add a token
refresh flow using the existing refreshToken function in
src/utils/jwt.ts (line 20-35)."
```

### Anti-Token-Waste Rules

```markdown
# .agents/rules/efficient-operation.md
---
description: "Rules for token-efficient agent operation"
activation: always
---

# Token Efficiency Rules

## File Reading
- Read specific line ranges when you know what you need
- Use grep_search to locate relevant code before reading full files
- Never read entire node_modules or generated directories

## Output
- Keep implementation plans concise but complete
- Don't repeat obvious information in walkthroughs
- Use bullet points over paragraphs where possible
- Skip boilerplate explanations for standard patterns

## Task Execution
- Don't re-read files you've already read in this conversation
- Reference prior artifacts instead of recreating them
- Use KIs from past conversations when available
```

## Improving Long-Context Reasoning

### When Context Gets Large

Symptoms of context window pressure:
- Agent "forgets" earlier instructions
- Agent produces code that contradicts earlier changes
- Agent repeats itself
- Quality degrades later in long conversations

### Mitigation Strategies

1. **Start new conversations** when switching topics
2. **Summarize before continuing** — ask the agent to summarize the state
3. **Use artifacts as anchors** — reference `task.md` to maintain focus
4. **Break into sub-conversations** — one for planning, one for implementing

### The "Fresh Start" Pattern

```
Conversation 1: "Plan the new billing feature"
  → Produces: implementation_plan.md, task.md

Conversation 2: "Implement the database changes from the billing
  plan in implementation_plan.md (steps 1-3)"
  → Fresh context, focused on database work

Conversation 3: "Implement the API endpoints from the billing
  plan in implementation_plan.md (steps 4-7)"
  → Fresh context, focused on API work
```

## IDE Performance Optimization

### Addressing Editor Lag

Reported causes and fixes:

| Issue | Cause | Fix |
|---|---|---|
| Editor sluggish | Long session with accumulated history | Restart Antigravity |
| Slow indexing | Large project with many files | Configure exclusion patterns |
| Agent response delay | Rate limiting | Switch model provider |
| Memory usage growth | Memory leaks in long sessions | Periodic IDE restart |
| Extension conflicts | Incompatible VS Code extensions | Disable unnecessary extensions |

### Performance-Optimized Settings

```json
{
  "antigravity.indexing.exclude": [
    "**/node_modules/**",
    "**/dist/**",
    "**/build/**",
    "**/coverage/**",
    "**/.git/**",
    "**/*.min.js",
    "**/*.map",
    "**/__pycache__/**"
  ],
  "files.watcherExclude": {
    "**/node_modules/**": true,
    "**/dist/**": true,
    "**/build/**": true
  },
  "editor.minimap.enabled": false,
  "editor.renderWhitespace": "none"
}
```

## Best Model Configurations by Task

| Task Type | Recommended Model | Temperature | Planning Mode |
|---|---|---|---|
| Architecture design | Claude Opus 4.6 | Low | Yes |
| New feature | Gemini 3.1 Pro | Medium | Yes |
| Bug fix | Claude Sonnet 4.6 | Low | No |
| Refactoring | Gemini 3.1 Pro | Low | Yes |
| Code review | Claude Sonnet 4.6 | Low | No |
| Documentation | Gemini 3 Flash | Medium | No |
| Test generation | Claude Sonnet 4.6 | Low | No |
| Rapid prototyping | Gemini 3 Flash | High | No |

---

*Previous: [12 — Security Best Practices](./12-security-best-practices.md) | Next: [14 — Real-World Workflows](./14-real-world-workflows.md)*
