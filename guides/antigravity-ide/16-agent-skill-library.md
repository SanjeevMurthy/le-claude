# 16 — Agent Skill Library

## Overview

This chapter provides a complete library of production-ready skill definitions covering all major development roles. Each skill can be dropped into `.agents/skills/<skill-name>/SKILL.md` and used immediately.

---

## 1. Architect Skill

```markdown
---
name: "Software Architect"
description: "System architecture design, technology evaluation, component decomposition, and Architecture Decision Records (ADRs). Use for new system design, major refactoring, or architectural reviews."
---

# Software Architect

## Process
1. Gather requirements (functional + non-functional)
2. Analyze existing architecture and patterns
3. Design component decomposition with clear boundaries
4. Define interfaces between components
5. Document decisions as ADRs in docs/adr/

## Output Format
- Architecture Decision Record (ADR)
- Component diagram (ASCII)
- Interface definitions
- Risk assessment

## ADR Template
\```markdown
# ADR-NNN: [Title]
## Status: [Proposed | Accepted | Deprecated]
## Context
[What is the issue / decision needed?]
## Decision
[What is the decision and why?]
## Consequences
[What are the trade-offs?]
\```

## Principles
- Favor simplicity over cleverness
- Design for 10x scale
- Minimize coupling between components
- Make failure modes explicit
```

---

## 2. Frontend Developer Skill

```markdown
---
name: "React Frontend Developer"
description: "React/Next.js frontend development with TypeScript, CSS Modules, TanStack Query, and React Hook Form. Use for any UI component, page, or frontend feature."
---

# React Frontend Developer

## Stack: React 18+ / Next.js 14+ / TypeScript

## Component Template
\```typescript
interface Props {
  // All props fully typed
}

export function ComponentName({ prop1, prop2 }: Props): JSX.Element {
  return (
    // JSX with semantic HTML
  );
}
\```

## Checklist
- [ ] TypeScript props interface
- [ ] Loading state
- [ ] Error state
- [ ] Empty state
- [ ] Keyboard navigation
- [ ] Mobile-first responsive
- [ ] Unit test with React Testing Library
- [ ] No prop drilling (use context if > 2 levels)

## Forbidden
- No `any` types
- No inline styles
- No useEffect for data fetching (use TanStack Query)
- No components > 200 lines
```

---

## 3. Backend Developer Skill

```markdown
---
name: "Node.js Backend Developer"
description: "REST API development with Express/NestJS, Prisma ORM, and PostgreSQL. Covers controllers, services, repositories, validation, and error handling."
---

# Backend Developer

## Architecture: Routes → Controllers → Services → Repositories → Prisma

## Endpoint Template
\```typescript
// Route: router.post('/resource', validate(Schema), controller.create);
// Controller: parse → call service → format response
// Service: business logic → call repository
// Repository: data access → Prisma operations
\```

## Response Format
\```json
{ "data": {...}, "error": null, "meta": { "page": 1 } }
\```

## Error Format
\```json
{ "data": null, "error": { "code": "NOT_FOUND", "message": "..." } }
\```

## Rules
- Input validation with Zod at every endpoint
- Parameterized queries only
- Soft deletes (deleted_at column)
- Timestamps on all tables (created_at, updated_at)
- Correlation IDs in all log entries
```

---

## 4. Database Engineer Skill

```markdown
---
name: "Database Engineer"
description: "Database schema design, Prisma migrations, query optimization, and indexing strategies. Use for any data model changes or database performance work."
---

# Database Engineer

## Prisma Conventions
- Table names: PascalCase in schema, maps to snake_case
- Always include: id (UUID), createdAt, updatedAt
- Soft deletes: deletedAt DateTime?
- Foreign keys: explicit with cascade rules

## Model Template
\```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  posts     Post[]
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  deletedAt DateTime? @map("deleted_at")

  @@map("users")
  @@index([email])
}
\```

## Migration Workflow
1. Modify schema.prisma
2. Run: npx prisma migrate dev --name <name>
3. Review generated SQL
4. Run: npx prisma generate
5. Test with seed data

## Performance
- Index all foreign keys
- Index frequently queried columns
- Use composite indexes for multi-column queries
- Avoid N+1 queries (use include/select)
```

---

## 5. Security Reviewer Skill

```markdown
---
name: "Security Reviewer"
description: "Code security audit covering OWASP Top 10, input validation, authentication, authorization, data exposure, and dependency vulnerabilities."
---

# Security Reviewer

## Checklist
- [ ] Inputs validated and sanitized
- [ ] SQL injection protected (parameterized queries)
- [ ] XSS prevented (output encoding)
- [ ] Authentication on all sensitive routes
- [ ] Authorization at service layer
- [ ] Secrets in env vars only
- [ ] No sensitive data in logs
- [ ] CORS properly configured
- [ ] Rate limiting enabled
- [ ] Dependencies free of known CVEs

## Severity Levels
- 🔴 Critical: Must fix before merge
- 🟡 Warning: Should fix, creates risk
- 🔵 Info: Best practice recommendation

## Report Format
\```markdown
## Security Review: [Feature]
### Critical Findings: N
### Warning Findings: N
### [Finding details with file + line + fix]
\```
```

---

## 6. Code Reviewer Skill

```markdown
---
name: "Code Reviewer"
description: "Comprehensive code review evaluating correctness, architecture alignment, readability, test coverage, performance, and security."
---

# Code Reviewer

## Review Dimensions
1. Correctness — Does it work as intended?
2. Architecture — Follows established patterns?
3. Readability — Clear and maintainable?
4. Tests — Adequate coverage?
5. Performance — Obvious issues?
6. Security — Any vulnerabilities?

## Output
### Status: APPROVED | CHANGES REQUESTED
### Critical Findings (must fix)
### Suggestions (should fix)
### Positive Highlights
```

---

## 7. DevOps Skill

```markdown
---
name: "DevOps Engineer"
description: "CI/CD pipelines, Docker, Kubernetes, Terraform, and deployment automation. Use for infrastructure, builds, and deployment configuration."
---

# DevOps Engineer

## Docker: Multi-stage builds, non-root user, health checks
## CI/CD: GitHub Actions with test → security → build → deploy stages
## K8s: Deployment + Service + Ingress + HPA
## Secrets: Never in code — use sealed secrets or external vault

## Dockerfile Template
\```dockerfile
FROM node:20-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-slim
RUN addgroup --system app && adduser --system --group app
WORKDIR /app
COPY --from=builder --chown=app:app /app/dist ./dist
COPY --from=builder --chown=app:app /app/node_modules ./node_modules
USER app
EXPOSE 3000
HEALTHCHECK CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
\```
```

---

## 8. Test Engineer Skill

```markdown
---
name: "Test Engineer"
description: "Test creation and strategy with Jest, Vitest, React Testing Library, Playwright. Covers unit, integration, e2e tests, TDD, and coverage analysis."
---

# Test Engineer

## Test Pyramid: 70% Unit / 20% Integration / 10% E2E

## Unit Test Template
\```typescript
describe('ServiceName', () => {
  describe('method', () => {
    it('should [expected] when [condition]', async () => {
      // Arrange
      const input = createTestInput();
      // Act
      const result = await service.method(input);
      // Assert
      expect(result).toEqual(expected);
    });
  });
});
\```

## Coverage Targets
- New code: ≥ 90%
- Overall: ≥ 80%
- Critical paths: 100% branch

## Rules
- No test interdependence
- Clean up after each test (afterEach)
- Mock external services, not internal logic
- Test behaviors, not implementation details
```

---

## 9. Documentation Writer Skill

```markdown
---
name: "Technical Documentation Writer"
description: "Creates and maintains technical documentation including READMEs, API docs, architecture guides, and user documentation."
---

# Technical Documentation Writer

## Documentation Types
1. README.md — Project overview and getting started
2. API Documentation — Endpoint reference (OpenAPI or markdown)
3. Architecture Docs — System design and ADRs
4. User Guides — End-user documentation
5. Runbooks — Operational procedures

## README Template
\```markdown
# Project Name
One-line description.

## Features
- Feature 1
- Feature 2

## Quick Start
\\\`bash
npm install
npm run dev
\\\`

## Architecture
[Brief overview with diagram]

## API Reference
[Link to API docs]

## Contributing
[Guidelines]
\```

## Style Rules
- Active voice
- Present tense
- Code examples for all features
- Keep paragraphs short (3-4 sentences max)
```

---

## 10. Performance Engineer Skill

```markdown
---
name: "Performance Engineer"
description: "Performance analysis and optimization for frontend (Core Web Vitals, bundle size) and backend (query optimization, caching, load testing)."
---

# Performance Engineer

## Frontend Metrics
- LCP < 2.5s
- FID < 100ms
- CLS < 0.1
- Bundle size budget: < 200KB initial JS

## Backend Metrics
- API response time: p95 < 200ms
- Database query time: p95 < 50ms
- Throughput: defined per endpoint

## Optimization Techniques
### Frontend
- Code splitting with dynamic imports
- Image optimization (next/image, WebP, AVIF)
- Font optimization (next/font)
- Skeleton loading states
- Virtual scrolling for large lists

### Backend
- Database indexing
- Query optimization (explain analyze)
- Response caching (Redis)
- Connection pooling
- Pagination on all list endpoints

## Analysis Process
1. Profile current performance
2. Identify bottlenecks
3. Propose targeted optimizations
4. Implement and measure improvement
5. Document results
```

---

## How to Use This Library

1. **Copy** the skill definitions you need
2. **Create** folder structure: `.agents/skills/<skill-name>/SKILL.md`
3. **Customize** for your project's specific stack and patterns
4. **Test** by asking the agent to perform a task related to the skill
5. **Iterate** — refine based on agent output quality

---

*Previous: [15 — Example Configurations](./15-example-configurations.md) | Next: [17 — Troubleshooting](./17-troubleshooting.md)*
