# 06 — Skills System

## Overview

Skills are **reusable packages of knowledge** that extend an agent's capabilities for specific task types. They are Antigravity's primary mechanism for codifying expertise — think of them as "expert playbooks" that agents can consult when performing specialized work.

## How Skills Work

```
+------------------------------------------------------------------+
|                   SKILL LIFECYCLE                                  |
|                                                                   |
|  1. DISCOVERY                                                     |
|     Agent loads skill names + descriptions at conversation start  |
|                                                                   |
|  2. RELEVANCE CHECK                                               |
|     Agent evaluates: "Does this skill apply to the current task?" |
|                                                                   |
|  3. ACTIVATION                                                    |
|     Agent reads the full SKILL.md content                         |
|                                                                   |
|  4. EXECUTION                                                     |
|     Agent follows the skill's instructions + uses its resources   |
|                                                                   |
|  5. COMPLETION                                                    |
|     Agent may reference skill in the walkthrough                  |
+------------------------------------------------------------------+
```

> **Key insight:** Skills are loaded *lazily*. Only the name and description are loaded initially. The full content is read only when the agent determines the skill is relevant. This minimizes context consumption.

## Skill Structure

Every skill resides in a dedicated folder and must contain a `SKILL.md` file:

```
skills/
└── my-skill/
    ├── SKILL.md              # REQUIRED: Main instruction file
    ├── scripts/              # Optional: Helper scripts
    │   ├── validate.sh
    │   └── generate.py
    ├── examples/             # Optional: Reference implementations
    │   ├── good-example.ts
    │   └── bad-example.ts
    └── resources/            # Optional: Templates, assets
        ├── template.md
        └── checklist.json
```

## SKILL.md Format

```markdown
---
name: "TypeScript API Developer"
description: "Expert skill for building REST APIs with TypeScript, Express, and Prisma. Includes request validation, error handling, authentication patterns, and API documentation generation."
---

# TypeScript API Developer Skill

## When to Use This Skill
Use this skill when:
- Building new REST API endpoints
- Implementing CRUD operations
- Adding request validation
- Setting up authentication middleware
- Generating API documentation

## Architecture Patterns

### Project Structure
\```
src/
├── routes/           # Express route definitions
├── controllers/      # Request handlers
├── services/         # Business logic
├── models/           # Prisma models
├── middleware/        # Auth, validation, error handling
├── validators/       # Zod schema validators
└── utils/            # Helper utilities
\```

### Layer Responsibilities
- **Routes**: Define HTTP method + path → controller mapping
- **Controllers**: Parse request, call service, format response
- **Services**: Pure business logic, database operations via Prisma
- **Models**: Prisma schema definitions
- **Validators**: Zod schemas for request/response validation

## Implementation Checklist
1. Define Prisma model (if new entity)
2. Create Zod validation schema
3. Implement service functions
4. Create controller with error handling
5. Define routes with middleware
6. Write unit tests for service layer
7. Write integration tests for endpoints
8. Update API documentation

## Code Patterns

### Controller Pattern
\```typescript
import { Request, Response, NextFunction } from 'express';
import { UserService } from '../services/user.service';
import { CreateUserSchema } from '../validators/user.validator';

export async function createUser(
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> {
  try {
    const validated = CreateUserSchema.parse(req.body);
    const user = await UserService.create(validated);
    res.status(201).json({ data: user });
  } catch (error) {
    next(error);
  }
}
\```

### Error Handling Pattern
\```typescript
export class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public code: string
  ) {
    super(message);
  }
}

export function errorHandler(
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
): void {
  if (error instanceof AppError) {
    res.status(error.statusCode).json({
      error: { code: error.code, message: error.message }
    });
    return;
  }
  res.status(500).json({
    error: { code: 'INTERNAL_ERROR', message: 'An unexpected error occurred' }
  });
}
\```

## Anti-Patterns to Avoid
- ❌ Business logic in controllers
- ❌ Direct database queries in route handlers
- ❌ Unvalidated request bodies
- ❌ Generic error messages without error codes
- ❌ Missing async error boundaries
```

## Skill Locations

### Workspace Skills (Project-Specific)

```
<workspace-root>/.agents/skills/<skill-folder>/SKILL.md
```

Best for:
- Project-specific conventions
- Framework-specific patterns
- Team coding standards
- Domain-specific knowledge

### Global Skills (Cross-Project)

```
~/.gemini/antigravity/skills/<skill-folder>/SKILL.md
```

Best for:
- Personal development patterns
- Language-specific best practices
- General-purpose utilities
- Reusable across all projects

### Override Behavior

Workspace skills with the **same name** override global skills. This allows teams to customize global skills for specific projects.

## Designing Effective Skills

### Best Practices

1. **Write clear descriptions** — The description is the agent's only clue for relevance matching
2. **Be specific, not vague** — "Builds REST APIs with Express + Prisma" > "Backend development"
3. **Include examples** — Show correct and incorrect patterns
4. **Provide checklists** — Step-by-step instructions the agent can follow
5. **Keep it focused** — One skill = one domain of expertise
6. **Include anti-patterns** — Explicitly state what NOT to do
7. **Use scripts for complex tasks** — Shell/Python scripts for multi-step operations

### Common Mistakes

| Mistake | Problem | Fix |
|---|---|---|
| Vague description | Agent can't determine relevance | Be specific about technologies and use cases |
| Too broad scope | Skill tries to cover everything | Split into focused, composable skills |
| Missing examples | Agent lacks concrete patterns | Include real code examples |
| No anti-patterns | Agent may repeat common mistakes | Explicitly state forbidden patterns |
| Hardcoded paths | Breaks across projects | Use relative references |

## Advanced Skill Features

### Skills with Scripts

Skills can include executable scripts for complex operations:

```
skills/
└── database-migration/
    ├── SKILL.md
    └── scripts/
        ├── generate-migration.sh
        ├── validate-schema.py
        └── seed-data.ts
```

In `SKILL.md`:
```markdown
## Migration Generation
Run the migration generation script:
\```bash
bash .agents/skills/database-migration/scripts/generate-migration.sh
\```
```

The agent will read the script contents and execute them as part of the skill workflow.

### Skills with Resources

Include templates, checklists, and reference materials:

```
skills/
└── api-documentation/
    ├── SKILL.md
    ├── resources/
    │   ├── openapi-template.yaml
    │   ├── endpoint-doc-template.md
    │   └── response-codes.json
    └── examples/
        └── sample-api-doc.md
```

### Skill Chaining

Skills can reference other skills:

```markdown
# Full-Stack Feature Skill

## Step 1: Database Layer
Refer to the **Database Migration** skill for schema changes.

## Step 2: API Layer
Refer to the **TypeScript API Developer** skill for endpoint creation.

## Step 3: UI Layer
Refer to the **React Component** skill for frontend implementation.

## Step 4: Testing
Refer to the **Integration Testing** skill for end-to-end verification.
```

## Building a Skill Library

### Recommended Skill Categories

```
skills/
├── languages/
│   ├── typescript-strict/
│   ├── python-flask/
│   └── go-stdlib/
├── frameworks/
│   ├── react-nextjs/
│   ├── express-prisma/
│   └── django-rest/
├── practices/
│   ├── security-review/
│   ├── code-review/
│   ├── performance-audit/
│   └── accessibility-check/
├── devops/
│   ├── docker-compose/
│   ├── github-actions/
│   └── terraform/
└── testing/
    ├── unit-testing/
    ├── integration-testing/
    └── e2e-testing/
```

See [Chapter 16 — Agent Skill Library](./16-agent-skill-library.md) for a complete library of ready-to-use skill definitions.

---

*Previous: [05 — Agent System](./05-agent-system.md) | Next: [07 — Agent Team Configuration](./07-agent-team-configuration.md)*
