# 09 — Backend Development

## Overview

Antigravity's backend development capabilities combine autonomous code generation with terminal-based verification — agents can write API code, run database migrations, execute test suites, and verify HTTP endpoints, all without human intervention.

## Backend Development Workflow

```
+------------------------------------------------------------------+
|           BACKEND DEVELOPMENT WORKFLOW                             |
|                                                                   |
|  1. DATA MODEL                                                    |
|     ├── Design database schema                                    |
|     ├── Write migration scripts                                   |
|     ├── Run migrations                                            |
|     └── Verify schema                                             |
|                                                                   |
|  2. BUSINESS LOGIC                                                |
|     ├── Define service interfaces                                 |
|     ├── Implement service layer                                   |
|     ├── Write validation schemas                                  |
|     └── Implement error handling                                  |
|                                                                   |
|  3. API LAYER                                                     |
|     ├── Define route handlers                                     |
|     ├── Implement controllers                                     |
|     ├── Add middleware (auth, validation)                         |
|     └── Generate OpenAPI documentation                            |
|                                                                   |
|  4. TESTING                                                       |
|     ├── Unit tests for services                                   |
|     ├── Integration tests for API endpoints                       |
|     ├── Run test suite via terminal                               |
|     └── Report coverage                                           |
+------------------------------------------------------------------+
```

## Configuring for Node.js / Express / NestJS

### Workspace Rules for Backend Projects

```markdown
# .agents/rules/backend-standards.md
---
description: "Backend development standards for Node.js services"
activation: always
globs: ["src/**/*.ts", "test/**/*.ts", "prisma/**"]
---

# Backend Development Standards

## Architecture
- Layered architecture: Routes → Controllers → Services → Repositories
- Business logic ONLY in the service layer
- Controllers handle HTTP concerns only (parsing, response formatting)
- Repositories abstract database operations

## Database
- Use Prisma as the ORM
- All schema changes via migrations (never manual SQL)
- Soft deletes with `deleted_at` column
- Created/updated timestamps on all tables
- UUID primary keys

## API Design
- RESTful conventions (proper HTTP methods and status codes)
- Versioned endpoints: `/api/v1/resource`
- Pagination on all list endpoints
- Consistent error response format
- Rate limiting on public endpoints

## Security
- Input validation with Zod on every endpoint
- Parameterized queries only (never string interpolation in SQL)
- No secrets in code — use environment variables
- CORS configured for specific origins
- Authentication middleware on all non-public routes

## Error Handling
- Custom AppError class with error codes
- Global error handler middleware
- Typed errors (ValidationError, NotFoundError, AuthError)
- No unhandled promise rejections
- Structured logging with correlation IDs
```

## Database Design Workflow

### Schema-First Development

```markdown
# .agents/workflows/database-migration.md
---
description: "Create and validate a new database migration"
---

## Steps

1. Review the requirements and existing schema in `prisma/schema.prisma`
2. Add the new model/fields to the Prisma schema
3. Generate the migration:
// turbo
4. Run: `npx prisma migrate dev --name <migration-name>`
5. Verify the generated SQL in `prisma/migrations/`
6. Generate the Prisma client:
// turbo
7. Run: `npx prisma generate`
8. Create seed data if applicable
9. Run the seed script:
// turbo
10. Run: `npx prisma db seed`
```

### Example: Full API Endpoint Development

When an agent builds a new API endpoint, it follows this complete process:

```
Agent receives: "Create a user preferences API"
       │
       v
Step 1: Research existing patterns
       ├── Read prisma/schema.prisma
       ├── Read src/routes/ for routing patterns
       ├── Read src/services/ for service patterns
       └── Check KIs for "API patterns"
       │
       v
Step 2: Create implementation plan
       ├── New model: UserPreference
       ├── Migration script
       ├── Zod validation schema
       ├── PreferencesService (CRUD)
       ├── PreferencesController
       ├── Route definitions
       └── Tests (unit + integration)
       │
       v
Step 3: Implement (after developer approval)
       ├── prisma/schema.prisma (add model)
       ├── src/validators/preferences.validator.ts
       ├── src/services/preferences.service.ts
       ├── src/controllers/preferences.controller.ts
       ├── src/routes/preferences.routes.ts
       ├── test/unit/preferences.service.test.ts
       └── test/integration/preferences.api.test.ts
       │
       v
Step 4: Verify
       ├── Run: npx prisma migrate dev
       ├── Run: npm run test
       ├── Run: npm run lint
       └── Report results in walkthrough
```

## API Development Patterns

### RESTful Endpoint Structure

```typescript
// src/routes/preferences.routes.ts
import { Router } from 'express';
import { authenticate } from '../middleware/auth';
import { validate } from '../middleware/validate';
import { 
  CreatePreferenceSchema, 
  UpdatePreferenceSchema 
} from '../validators/preferences.validator';
import * as controller from '../controllers/preferences.controller';

const router = Router();

router.use(authenticate); // All routes require auth

router.get('/',           controller.getPreferences);
router.get('/:key',       controller.getPreference);
router.put('/:key',       validate(UpdatePreferenceSchema), controller.updatePreference);
router.post('/',          validate(CreatePreferenceSchema), controller.createPreference);
router.delete('/:key',    controller.deletePreference);

export default router;
```

### Service Layer Pattern

```typescript
// src/services/preferences.service.ts
import { prisma } from '../database/client';
import { NotFoundError, ConflictError } from '../errors';
import type { CreatePreferenceDTO, UpdatePreferenceDTO } from '../validators/preferences.validator';

export class PreferencesService {
  static async getAll(userId: string) {
    return prisma.userPreference.findMany({
      where: { userId, deletedAt: null },
      orderBy: { key: 'asc' },
    });
  }

  static async getByKey(userId: string, key: string) {
    const preference = await prisma.userPreference.findFirst({
      where: { userId, key, deletedAt: null },
    });
    if (!preference) {
      throw new NotFoundError(`Preference '${key}' not found`);
    }
    return preference;
  }

  static async create(userId: string, dto: CreatePreferenceDTO) {
    const existing = await prisma.userPreference.findFirst({
      where: { userId, key: dto.key, deletedAt: null },
    });
    if (existing) {
      throw new ConflictError(`Preference '${dto.key}' already exists`);
    }
    return prisma.userPreference.create({
      data: { userId, ...dto },
    });
  }

  static async update(userId: string, key: string, dto: UpdatePreferenceDTO) {
    await this.getByKey(userId, key); // Verify exists
    return prisma.userPreference.update({
      where: { userId_key: { userId, key } },
      data: dto,
    });
  }

  static async delete(userId: string, key: string) {
    await this.getByKey(userId, key);
    return prisma.userPreference.update({
      where: { userId_key: { userId, key } },
      data: { deletedAt: new Date() },
    });
  }
}
```

## Backend Testing Workflow

### Automated Testing Pipeline

```markdown
# .agents/workflows/run-backend-tests.md
---
description: "Run the full backend test suite with coverage reporting"
---

## Steps

1. Clean any stale test data:
// turbo
2. Run: `npm run db:test:reset`
3. Run unit tests with coverage:
// turbo
4. Run: `npm run test:unit -- --coverage`
5. Run integration tests:
// turbo
6. Run: `npm run test:integration`
7. Check coverage thresholds:
// turbo
8. Run: `npm run test:coverage-check`
9. Report results in the walkthrough with pass/fail summary
```

## CI/CD Integration

### GitHub Actions for Backend

```yaml
# .github/workflows/backend-ci.yml
name: Backend CI
on:
  push:
    branches: [main, develop]
    paths: ['src/**', 'test/**', 'prisma/**']
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports: ['5432:5432']
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - run: npm ci
      - run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/test
      
      - run: npm run lint
      - run: npm run type-check
      - run: npm run test:unit -- --coverage
      - run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/test
```

---

*Previous: [08 — Frontend Development](./08-frontend-development.md) | Next: [10 — Full Autonomous Development](./10-full-autonomous-development.md)*
