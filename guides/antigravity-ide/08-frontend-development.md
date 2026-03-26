# 08 — Frontend Development

## Overview

Antigravity excels at frontend development through its unique combination of **code generation, browser automation, and visual verification**. This chapter covers configuring Antigravity for React, Next.js, and modern frontend workflows.

## Frontend Development Architecture in Antigravity

```
+------------------------------------------------------------------+
|           FRONTEND DEVELOPMENT WORKFLOW                           |
|                                                                   |
|  1. DESIGN                                                        |
|     ├── Generate UI mockup with Nano Banana                      |
|     ├── Describe component hierarchy                              |
|     └── Define responsive breakpoints                             |
|                                                                   |
|  2. IMPLEMENT                                                     |
|     ├── Agent creates component files                             |
|     ├── Agent writes styles (CSS Modules / Tailwind)             |
|     ├── Agent implements state management                         |
|     └── Agent handles API integration                             |
|                                                                   |
|  3. VERIFY                                                        |
|     ├── Agent runs dev server                                     |
|     ├── Agent opens browser (browser_subagent)                   |
|     ├── Agent takes screenshots                                   |
|     ├── Agent records interactions                                |
|     └── Agent validates responsive behavior                       |
|                                                                   |
|  4. ITERATE                                                       |
|     ├── Developer reviews screenshots                             |
|     ├── Developer provides feedback                               |
|     └── Agent refines implementation                              |
+------------------------------------------------------------------+
```

## Configuring for React / Next.js

### Recommended Rules for React Projects

```markdown
# .agents/rules/react-standards.md
---
description: "React and Next.js development standards"
activation: always
globs: ["**/*.tsx", "**/*.ts", "**/*.css"]
---

# React Development Standards

## Component Architecture
- Use React Server Components by default
- Use 'use client' directive only when client interactivity is needed
- Prefer composition over prop drilling
- Maximum component length: 200 lines

## State Management
- Server state: TanStack Query (React Query)
- Client state: Zustand (avoid Redux for new code)
- Form state: React Hook Form + Zod
- URL state: Next.js searchParams

## Performance
- Use React.memo only when profiler shows re-render issues
- Lazy load routes and heavy components with dynamic imports
- Use next/image for all images
- Implement skeleton loading states

## Styling
- Use CSS Modules for component-specific styles
- Use CSS custom properties for theming
- Mobile-first responsive design
- Minimum touch target: 44x44px

## Accessibility
- All interactive elements must be keyboard navigable
- Use semantic HTML (button, nav, main, section)
- Include ARIA labels on icon-only buttons
- Test with screen reader in the walkthrough
```

### Next.js App Router Skill

```markdown
---
name: "Next.js App Router Developer"
description: "Skill for building Next.js 14+ applications with the App Router, Server Components, Server Actions, and modern data fetching patterns."
---

# Next.js App Router Skill

## Directory Convention
\```
app/
├── layout.tsx              # Root layout (Server Component)
├── page.tsx                # Home page
├── loading.tsx             # Loading UI
├── error.tsx               # Error boundary ('use client')
├── not-found.tsx           # 404 page
├── globals.css             # Global styles
├── (auth)/                 # Route group (no URL segment)
│   ├── login/
│   │   └── page.tsx
│   └── register/
│       └── page.tsx
├── dashboard/
│   ├── layout.tsx          # Dashboard layout
│   ├── page.tsx            # Dashboard home
│   └── settings/
│       └── page.tsx
└── api/
    └── [...route]/
        └── route.ts        # API route handler
\```

## Data Fetching Patterns

### Server Component (default)
\```typescript
// app/users/page.tsx — Server Component (no 'use client')
async function UsersPage() {
  const users = await db.user.findMany();
  return <UserList users={users} />;
}
\```

### Client Component
\```typescript
'use client';
// Only when you need interactivity, browser APIs, or hooks
import { useState } from 'react';

function SearchBar({ onSearch }: { onSearch: (query: string) => void }) {
  const [query, setQuery] = useState('');
  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      onKeyDown={(e) => e.key === 'Enter' && onSearch(query)}
    />
  );
}
\```

### Server Actions
\```typescript
'use server';
// app/actions/user.ts
import { revalidatePath } from 'next/cache';

export async function createUser(formData: FormData) {
  const name = formData.get('name') as string;
  await db.user.create({ data: { name } });
  revalidatePath('/users');
}
\```
```

## UI Generation Workflows

### Design-to-Code Pipeline

Antigravity provides a powerful design-to-code workflow:

```
Text Description ──> Agent generates component
                         │
Image/Mockup ──────> Agent matches visual design
                         │
                         v
                    Agent runs dev server
                         │
                         v
                    Agent captures screenshot
                         │
                         v
                    Developer compares
                         │
                    ┌────┴────┐
                    │ Match?  │
                    │ Yes? No?│
                    └────┬────┘
                         │
                    ┌────┴────┐
              ┌─────┤         ├─────┐
              │     │         │     │
              v     │         │     v
          Complete  │         │  Agent iterates
                    │         │  on feedback
                    └─────────┘
```

### Using generate_image for Mockups

Antigravity's `generate_image` tool lets agents create UI mockups:

```
Agent prompt: "Generate a modern dashboard UI with a sidebar navigation,
header with user avatar, and a main content area showing analytics charts
with a dark mode theme using blue and purple gradients."
```

The generated image can then be used as a visual reference for code generation.

### Component Design Agent Pattern

A powerful workflow for building UI components:

1. **Describe the component** in detail
2. **Agent generates code** based on description
3. **Agent starts dev server** (`npm run dev`)
4. **Agent opens browser** and navigates to the component
5. **Agent takes screenshot** of the rendered result
6. **Agent includes screenshot in walkthrough**
7. **Developer reviews** and provides feedback
8. **Agent iterates** until design is satisfactory

### Example Workflow for UI Component

```markdown
# .agents/workflows/build-component.md
---
description: "Build and verify a new UI component with visual verification"
---

## Steps

1. Read the component requirements from the task description
2. Check existing design patterns in the codebase
3. Create the component file with TypeScript + CSS Modules
4. Create a test file with React Testing Library
5. Start the dev server
// turbo
6. Open the browser and navigate to the component
7. Take a screenshot and include in the walkthrough
8. Run the test suite
// turbo
9. Report results to the developer
```

## Figma-Like Workflows

While Antigravity doesn't directly integrate with Figma, you can replicate Figma-like workflows:

### From Figma Export to Code

1. Export component screenshots from Figma
2. Save to project `design/` directory
3. Prompt: "Implement this component to match the design in `design/header-mockup.png`. Use CSS Modules. Match colors, spacing, and typography exactly."
4. Agent will:
   - Analyze the image
   - Generate matching HTML/CSS/React code
   - Verify in the browser
   - Compare screenshot to original

### UI/UX Critique Agent

Configure an agent to review UI implementations:

```markdown
---
name: "UI/UX Critic"
description: "UI/UX review skill that evaluates implemented interfaces against design principles, accessibility standards, and user experience best practices."
---

# UI/UX Critic Skill

## Review Criteria
1. **Visual Hierarchy** — Is information properly prioritized?
2. **Consistency** — Do similar elements look and behave the same?
3. **Accessibility** — Color contrast, keyboard nav, screen reader support
4. **Responsiveness** — Does it work on mobile, tablet, desktop?
5. **White Space** — Proper spacing and breathing room?
6. **Typography** — Readable font sizes, proper line heights?
7. **Interactive States** — Hover, focus, active, disabled states defined?
8. **Loading States** — Skeleton screens, spinners, progress indicators?
9. **Error States** — Clear error messages, recovery paths?
10. **Empty States** — Meaningful content when no data exists?

## Output
Produce a UI review report with screenshots annotated with findings.
```

## Responsive Design Verification

Agents can verify responsive designs across breakpoints:

```
1. Start dev server
2. Open browser at 1440px width (desktop)
3. Take screenshot → desktop_view.png
4. Resize to 768px (tablet)
5. Take screenshot → tablet_view.png
6. Resize to 375px (mobile)
7. Take screenshot → mobile_view.png
8. Include all screenshots in walkthrough
9. Report any layout issues detected
```

---

*Previous: [07 — Agent Team Configuration](./07-agent-team-configuration.md) | Next: [09 — Backend Development](./09-backend-development.md)*
