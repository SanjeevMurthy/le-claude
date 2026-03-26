# 15 — Example Configurations

## Complete Project Configuration: Full-Stack SaaS Application

This chapter provides complete, copy-paste-ready configuration files for a production full-stack SaaS application.

## Project Directory Structure

```
my-saas-app/
├── .agents/
│   ├── rules/
│   │   ├── coding-standards.md
│   │   ├── architecture.md
│   │   ├── security.md
│   │   ├── testing.md
│   │   └── documentation.md
│   ├── skills/
│   │   ├── nextjs-fullstack/
│   │   │   └── SKILL.md
│   │   ├── prisma-database/
│   │   │   └── SKILL.md
│   │   ├── api-development/
│   │   │   └── SKILL.md
│   │   ├── security-review/
│   │   │   └── SKILL.md
│   │   └── test-engineer/
│   │       └── SKILL.md
│   └── workflows/
│       ├── new-feature.md
│       ├── bug-fix.md
│       ├── security-scan.md
│       ├── deploy.md
│       └── quality-check.md
├── .vscode/
│   ├── settings.json
│   └── mcp_config.json
├── docs/
│   └── adr/
├── specs/
├── src/
├── app/
├── prisma/
├── test/
└── package.json
```

## Complete Rules Configuration

### `coding-standards.md`

```markdown
---
description: "Core coding standards for the project"
activation: always
---

# Coding Standards

## Language: TypeScript
- Strict mode enabled (`strict: true` in tsconfig.json)
- Explicit return types on all exported functions
- No `any` — use `unknown` with type guards
- Named exports only (no default exports)
- Prefer `const` over `let`; never use `var`

## Formatting
- Prettier with project config (tabWidth: 2, singleQuote: true)
- Max line length: 100 characters
- Trailing commas in multi-line

## Naming
- Files: kebab-case (`user-profile.service.ts`)
- Components: PascalCase (`UserProfile.tsx`)
- Functions/variables: camelCase (`getUserProfile`)
- Constants: SCREAMING_SNAKE_CASE (`MAX_RETRIES`)
- Types/interfaces: PascalCase (`UserProfile`)
- Database tables: snake_case (`user_profiles`)

## Git
- Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `test:`)
- One logical change per commit
- Branch naming: `feature/`, `fix/`, `chore/`
```

### `architecture.md`

```markdown
---
description: "Architecture constraints and patterns"
activation: always
---

# Architecture Rules

## Directory Structure
\```
app/                    # Next.js App Router pages
├── (auth)/            # Auth route group
├── (dashboard)/       # Dashboard route group
├── api/               # API routes
└── layout.tsx         # Root layout

src/
├── components/        # Shared UI components
│   ├── ui/           # Primitive components (Button, Input)
│   └── features/     # Feature-specific components
├── services/          # Business logic layer
├── repositories/      # Data access layer
├── validators/        # Zod schemas
├── middleware/        # Middleware functions
├── hooks/            # Custom React hooks
├── utils/            # Pure utility functions
├── types/            # Shared TypeScript types
└── config/           # Configuration
\```

## Layer Dependencies
Routes → Controllers → Services → Repositories → Prisma
Never skip layers.

## Component Guidelines
- Server Components by default
- `use client` only when necessary
- Max 200 lines per component
- Co-locate styles (CSS Modules)
- Co-locate tests (`Component.test.tsx`)
```

### `security.md`

```markdown
---
description: "Security rules for all changes"
activation: always
---

# Security Rules

- Validate ALL inputs with Zod at API boundaries
- Parameterized queries only (Prisma handles this, never use raw SQL)
- Never log passwords, tokens, PII, or financial data
- Secrets in environment variables only
- CORS: specific origins only (never wildcard in production)
- Authentication required on all non-public routes
- Rate limiting: 100 req/min per user on API, 10 req/min on auth
- Content Security Policy headers on all pages
- HttpOnly + Secure + SameSite cookies for sessions
```

### `testing.md`

```markdown
---
description: "Testing requirements"
activation: always
---

# Testing Rules

## Coverage
- Minimum: 80% line coverage
- Critical paths (auth, payments): 100% branch coverage
- New code: must include tests

## Test Types
- Unit tests: `test/unit/` — Pure functions, service logic
- Integration tests: `test/integration/` — API endpoints + database
- E2E tests: `test/e2e/` — Full user flows (Playwright)

## Test Patterns
\```typescript
describe('ServiceName', () => {
  describe('methodName', () => {
    it('should [expected behavior] when [condition]', () => {
      // Arrange → Act → Assert
    });
  });
});
\```

## Test Requirements
- Each service method: at least 1 happy path + 1 error path test
- Each API endpoint: at least 1 success + 1 auth failure + 1 validation failure test
- Each UI component: at least 1 render test + 1 interaction test
```

## Complete Workflow Configuration

### `new-feature.md`

```markdown
---
description: "End-to-end workflow for implementing a new feature"
---

## Steps

1. Read the feature specification from the specs/ directory
2. Research the existing codebase for related patterns
3. Create an Architecture Decision Record if the feature introduces new patterns
4. Create implementation_plan.md with detailed file changes
5. Create task.md with step-by-step checklist

6. Implement in order:
   a. Database schema changes (Prisma)
   b. Validation schemas (Zod)
   c. Repository layer
   d. Service layer  
   e. API routes/controllers
   f. Frontend components and pages

7. Run quality checks:
// turbo
8. Run: `npm run lint`
// turbo
9. Run: `npx tsc --noEmit`
// turbo
10. Run: `npm run test`

11. For frontend changes — visual verification:
    a. Start dev server
    b. Open browser
    c. Take screenshots

12. Create walkthrough with results

13. Git:
// turbo
14. Run: `git checkout -b feature/$(date +%Y%m%d)-new-feature`
// turbo
15. Run: `git add -A && git commit -m "feat: [description]"`

16. Notify developer for review
```

### `bug-fix.md`

```markdown
---
description: "Workflow for diagnosing and fixing bugs"
---

## Steps

1. Reproduce the bug:
   - Read the bug report
   - Identify the affected code
   - Write a failing test that demonstrates the bug

2. Diagnose:
   - Trace the code path
   - Identify root cause (not just symptoms)

3. Fix:
   - Apply minimal, targeted fix
   - Ensure fix doesn't break other functionality

4. Verify:
// turbo
5. Run: `npm run test`
6. Verify the failing test now passes
7. Run full test suite to check for regressions

8. Document:
   - Add comment explaining why the fix was needed
   - Update walkthrough with root cause analysis
```

## VS Code / Antigravity Settings

### `settings.json`

```json
{
  "typescript.preferences.importModuleSpecifier": "non-relative",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "antigravity.reviewPolicy": "agentDecides",
  "antigravity.terminal.autoExecute": "auto",
  "antigravity.terminal.allowList": [
    "npm run *",
    "npx *",
    "git status",
    "git diff *",
    "git log *",
    "git branch *",
    "git checkout *",
    "git add *"
  ],
  "antigravity.terminal.denyList": [
    "rm -rf *",
    "git push --force *",
    "npm publish",
    "git reset --hard *"
  ],
  "antigravity.indexing.exclude": [
    "**/node_modules/**",
    "**/dist/**",
    "**/build/**",
    "**/.next/**",
    "**/coverage/**",
    "**/.git/**"
  ]
}
```

### `mcp_config.json`

```json
{
  "servers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

## Global Configuration

### `~/.gemini/GEMINI.md`

```markdown
# Personal Agent Rules

## Communication
- Be concise — bullet points over paragraphs
- When uncertain, state assumptions explicitly
- Provide alternatives when multiple approaches exist

## Quality Bar
- Code must compile and lint on first attempt
- Never use `any`, `@ts-ignore`, or `eslint-disable` without justification
- Every change must include tests
- Error handling required on all async operations

## Behavior
- Always read existing code before modifying
- Match the patterns and style of the existing codebase
- Research before implementing — check KIs and prior conversations
- When a task is complex, create an implementation plan first
```

---

*Previous: [14 — Real-World Workflows](./14-real-world-workflows.md) | Next: [16 — Agent Skill Library](./16-agent-skill-library.md)*
