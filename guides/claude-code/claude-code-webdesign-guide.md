# Claude Code Web Design Guide

> A comprehensive guide to building beautiful, premium websites using Claude Code's skills, agents, MCP tools, and design workflows.

---

## Table of Contents

1. [Overview — Why Claude Code for Web Design](#1-overview--why-claude-code-for-web-design)
2. [Essential Setup](#2-essential-setup)
3. [The Frontend Design Skill](#3-the-frontend-design-skill)
4. [Design Skills Marketplace & Installation](#4-design-skills-marketplace--installation)
5. [The 4-Step Framework: Inspire → Build → Refine → Deploy](#5-the-4-step-framework-inspire--build--refine--deploy)
6. [Visual Iteration with Playwright MCP](#6-visual-iteration-with-playwright-mcp)
7. [Sub-Agent Pipelines for Design](#7-sub-agent-pipelines-for-design)
8. [Design Systems, Component Libraries & Tools](#8-design-systems-component-libraries--tools)
9. [Advanced Techniques](#9-advanced-techniques)
10. [Quick Reference — Prompts, Commands & Patterns](#10-quick-reference--prompts-commands--patterns)

---

## 1. Overview — Why Claude Code for Web Design

### The "AI Slop" Problem

If you've been building websites with Claude Code, Cursor, or any coding agent, you've likely encountered **distributional convergence** — every AI-generated site looks the same:

- Generic purple/blue ShadCN gradients
- Identical card layouts and hero sections
- The same "trusted by leading companies" ticker
- Cookie-cutter responsive grids

This happens because AI models default to the **most statistically common** design patterns from their training data. The result is "AI slop" — sites that are functional but unmistakably machine-generated.

### The Solution: Context Engineering for Design

The problem isn't the model's capability — it's the **environment** you place it in. Claude Code has deep design knowledge locked inside its multimodal training, but in default text-only mode, it can only access the code side of its intelligence, not the visual side.

The unlock comes from three pillars:

| Pillar         | What It Provides                            | How                                                       |
| -------------- | ------------------------------------------- | --------------------------------------------------------- |
| **Context**    | Design principles, style guides, references | CLAUDE.md, design principles files, screenshot references |
| **Tools**      | Visual feedback, browser control            | Playwright MCP, Figma MCP, browser screenshots            |
| **Validation** | Iterative self-correction loop              | Sub-agents, design review agents, agentic iteration       |

When you combine rich design context + visual tools + validation loops, Claude Code transforms from a generic builder into a **precision design instrument**.

### What Changed with Opus 4 / Sonnet 4

With the latest model generations, Claude's one-shot design quality has dramatically improved:

- **Style-aware generation**: Understands design vocabularies (brutalist, editorial luxury, organic, tech SaaS)
- **Animation-first**: Produces Framer Motion, GSAP, and CSS animations without explicit requests
- **Typography awareness**: Selects and pairs fonts appropriately for different aesthetics
- **Layout sophistication**: Creates asymmetric grids, diagonal layouts, and non-standard compositions

> **Key Insight**: The models are now incredibly capable designers — they just need the right prompts, visual feedback, and design context to unlock their potential.

---

## 2. Essential Setup

### 2.1 Installation

If you haven't already, install Claude Code:

```bash
# Install Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Or use the VS Code extension
# Extensions → Search "Claude Code" → Install
```

### 2.2 Configuring CLAUDE.md for Design

Your `CLAUDE.md` file is **memory for the agent** — everything here is loaded into the system prompt at the start of every session. A design-optimized `CLAUDE.md` should include:

```markdown
# Project: [Your Website Name]

## Tech Stack

- Framework: React / Next.js / Vite
- Styling: Tailwind CSS / Vanilla CSS
- Animations: Framer Motion / GSAP
- 3D: Three.js / React Three Fiber (if applicable)

## Design Principles

- Reference: ./context/design-principles.md
- Style Guide: ./context/style-guide.md

## Visual Development

### Playwright Browser Verification

When making ANY frontend changes:

1. Navigate to the affected pages using Playwright
2. Take screenshots at desktop viewport (1440x900)
3. Compare against acceptance criteria from the prompt
4. Check browser console for errors
5. Fix any visual discrepancies before completing

### Comprehensive Design Review

Trigger the @agent design-reviewer for:

- Pull requests with UI changes
- Significant UI/UX refactors
- Before marking any frontend task as complete

## Brand Guidelines

- Primary Color: #[hex]
- Secondary Color: #[hex]
- Font Family: [font name]
- Design Aesthetic: [e.g., "minimal brutalist with bold typography"]

## Rules

- Do NOT introduce new CSS frameworks or UI libraries without approval
- Always use the project's design tokens from style-tokens.ts
- Follow the component patterns in ./components/
```

### 2.3 The Design Principles File

Create a `./context/design-principles.md` file with curated design principles. A powerful approach is to use **deep research** (Gemini Deep Research, Perplexity, etc.) to compile best practices for your target aesthetic:

```markdown
# Design Principles

## Typography

- Use a maximum of 2 typeface families
- Establish clear hierarchy: Display → Heading → Body → Caption
- Line height: 1.2 for headings, 1.5–1.6 for body text
- Letter spacing: Tight (-0.02em) for large text, normal for body

## Color Theory

- **Primary**: Used for CTAs, buttons, key interactive elements (stands out most)
- **Secondary**: Complements primary; used for subtle action elements
- **Neutral**: Black, white, gray — makes up the majority of the UI
- **Semantic**: Green (success), Yellow (warning), Blue (info), Red (error)

## Spatial Design

- Use 8px grid system for all spacing
- Maintain generous whitespace — let elements breathe
- Create depth through layered backgrounds (3–4 shades of base color)

## Motion

- Entrance animations: 200–400ms ease-out
- Exits: 150–250ms ease-in
- Stagger delays: 50–100ms between elements
- Use motion to guide attention, not distract

## Shadows & Depth

- Implement two-layer shadow system (light shadow above, dark below)
- Three depth levels: surface, raised, floating
- Use subtle gradients on dark-mode components
- Eliminate borders by using background color layering instead
```

> **Pro Tip**: Use Gemini Deep Research to generate a comprehensive design principles document for your specific aesthetic. Then have Claude Code refine it into a concise markdown file.

### 2.4 Project Context Folder

Create a `./context/` folder containing reference materials Claude can access:

```
./context/
├── design-principles.md      # Design rules and guidelines
├── style-guide.md             # Colors, fonts, spacing tokens
├── brand-story.md             # Company description, messaging, voice
├── reference-screenshots/     # Screenshots of inspiring designs
└── component-patterns.md      # Preferred component structures
```

Reference this folder in your `CLAUDE.md` so Claude always knows where to find design context.

---

## 3. The Frontend Design Skill

### 3.1 What Is It?

Anthropic maintains an **official frontend design skill** in their `anthropics/skills` GitHub repository. This skill teaches Claude Code to generate **distinctive, production-grade interfaces** that break away from generic AI aesthetics.

The skill focuses on:

- **Unique typography** — custom font pairing in non-standard compositions
- **Intentional motion** — animations that guide the eye, not distract
- **Non-traditional layouts** — asymmetric grids, overlapping elements, creative spacing
- **Premium polish** — glass morphism, micro-interactions, edge tracers

### 3.2 Installation

**Method 1: Using skills.sh CLI** (Recommended)

```bash
# Install the skills CLI
npm install -g skills.sh

# Browse and install from the marketplace
npx skills add anthropics/skills

# Select "frontend-design" from the list
```

**Method 2: Manual Installation**

```bash
# Clone or download the skill
git clone https://github.com/anthropics/skills.git

# Copy the frontend-design skill to your project
cp -r skills/frontend-design/ .claude/skills/designer/

# Or install globally
cp -r skills/frontend-design/ ~/.claude/skills/designer/
```

The skill structure looks like this:

```
.claude/skills/designer/
├── skill.md              # Main skill instructions (always loaded)
├── typography.md          # Typography guidelines
├── color-theme.md         # Color palette rules
├── motion.md              # Animation principles
└── general.md             # Overall design philosophy
```

### 3.3 How Skills Work (Progressive Disclosure)

Skills use a **three-level loading system**:

| Level       | What Loads                                             | When                                             |
| ----------- | ------------------------------------------------------ | ------------------------------------------------ |
| **Level 1** | YAML front matter (name, description, triggers)        | Always — part of system prompt                   |
| **Level 2** | Core instructions from `skill.md` body                 | When trigger words match user's prompt           |
| **Level 3** | Linked files (`typography.md`, `color-theme.md`, etc.) | When the skill file references them with `@file` |

This means the skill is **automatically invoked** when you use trigger phrases like:

- "Improve the design of this page"
- "Make this look more premium"
- "Apply professional frontend design"
- "Redesign the landing page"

### 3.4 Usage Examples

**Basic invocation**:

```
Use the frontend design skill to improve the design of the homepage.
Focus on typography, spacing, and micro-interactions.
```

**With reference screenshots**:

```
@screenshot-of-stripe.png @screenshot-of-linear.png

Using the frontend design skill, redesign my landing page
to match the premium aesthetic shown in these references.
Keep my brand colors (#1a1a2e, #16213e, #0f3460) but
adopt their typography scale and spacing system.
```

**Custom slash command** (create `.claude/commands/frontend-design.md`):

```markdown
---
description: Apply professional frontend design improvements
---

Use the frontend-design skill to analyze and improve the current page.

Focus areas:

1. Typography hierarchy and font pairing
2. Color system and contrast ratios
3. Spacing and layout rhythm
4. Micro-interactions and hover states
5. Visual depth (shadows, gradients, layering)

Take a Playwright screenshot before and after changes.
Compare against the design principles in ./context/design-principles.md
```

### 3.5 Before vs. After

Without the skill, Claude Code typically produces:

- System font stacks (sans-serif defaults)
- Uniform spacing based on simple multiples
- Flat color schemes with one accent
- No animations or interactions
- Standard symmetric grid layouts

**With the skill**, you get:

- Curated font pairs (e.g., Inter + Playfair Display)
- Variable spacing that creates visual rhythm
- Multi-tone color palettes with intentional accent usage
- Smooth entrance/exit animations
- Asymmetric and magazine-style layouts

---

## 4. Design Skills Marketplace & Installation

### 4.1 Skills.sh Ecosystem

The `skills.sh` CLI is the primary way to discover, install, and manage Claude Code skills:

```bash
# Install the CLI
npm install -g skills.sh

# List available skills
npx skills list

# Install a specific skill
npx skills add <creator>/<skill-name>

# Install globally (available in all projects)
npx skills add --global <creator>/<skill-name>

# View installed skills
npx skills installed
```

### 4.2 Key Web Design Skills

| Skill                   | Description                        | Install Command                                |
| ----------------------- | ---------------------------------- | ---------------------------------------------- |
| **frontend-design**     | Anthropic's official design skill  | `npx skills add anthropics/skills`             |
| **landing-page**        | Landing page optimization patterns | `npx skills add community/landing-page`        |
| **web-design-reviewer** | Automated design review agent      | `npx skills add community/web-design-reviewer` |

### 4.3 Creating Custom Design Skills

You can create your own skills that encode specific design philosophies. Here's an example of a **design auditor skill** built from designer references:

**Step 1: Research designer philosophies**

Pick 3–5 designers whose work you admire and study their patterns:

- **Stripe**: Clean data visualization, gradient mastery, subtle depth
- **Linear**: Monochrome elegance, keyboard-first interactions, crisp typography
- **Vercel**: Dark mode perfection, code-inspired aesthetics, minimal color
- **Emil Kowalski**: Micro-interaction mastery, edge tracers, corner animations

**Step 2: Create the skill structure**

```
.claude/skills/design-auditor/
├── skill.md              # Core auditing workflow
├── references/
│   ├── stripe-patterns.md
│   ├── linear-patterns.md
│   └── vercel-patterns.md
└── scripts/
    └── audit-report.md    # Report template
```

**Step 3: Write skill.md**

```markdown
---
name: design-auditor
description: >
  Audits UI designs against premium design standards. 
  Use when a user says "audit my design", "review the UI",
  "check design quality", or "is this design good enough?"
---

# Design Auditor

You are a principal-level UI designer who has worked at Stripe,
Linear, and Vercel. Your aesthetic standards are extremely high.

## Audit Process

1. **Take screenshots** of the target pages using Playwright MCP
2. **Analyze** against each reference pattern file:
   - @references/stripe-patterns.md
   - @references/linear-patterns.md
   - @references/vercel-patterns.md
3. **Score** each dimension (typography, color, spacing, motion, depth)
4. **Generate** a prioritized list of improvements
5. **Implement** the top 3 improvements automatically

## Output Format

Produce a design audit report with:

- Overall score (A+ to F)
- Strengths (what's working well)
- High-priority issues (must fix)
- Nice-to-haves (polish items)
- Before/after screenshots
```

### 4.4 Iterating on Skills

Skills are living documents. Improve them over time:

1. **Add new designer references** — study a new designer's CodePen, extract patterns
2. **Refine trigger words** — add phrases your team commonly uses
3. **Include example outputs** — show Claude what "good" looks like
4. **Add validation steps** — include Playwright screenshot comparisons

```bash
# Check skill effectiveness with a test prompt
claude "Use the design-auditor skill to review the homepage"

# Compare outputs before/after skill updates
# Use git worktrees to A/B test skill variations
```

---

## 5. The 4-Step Framework: Inspire → Build → Refine → Deploy

This is the most practical and repeatable workflow for creating premium websites with Claude Code. Instead of trying to prompt your way to a beautiful design from scratch, you **start with inspiration and iterate**.

### 5.1 Step 1: Inspire

Find a website whose style, vibe, and layout you want to emulate. You're not copying — you're establishing a **design direction**.

**Inspiration Sources**:

| Resource      | Description                                 | URL           |
| ------------- | ------------------------------------------- | ------------- |
| **Dribbble**  | Design portfolio showcase, search by style  | dribbble.com  |
| **Godly**     | Curated collection of premium websites      | godly.website |
| **Awwwards**  | Award-winning web design gallery            | awwwards.com  |
| **Superhero** | Hero section inspiration specifically       | superhero.so  |
| **21st.dev**  | Component-level inspiration                 | 21st.dev      |
| **CodePen**   | CSS effects, animations, micro-interactions | codepen.io    |

**What to Capture**:

1. **Screenshots** — Take 5–10 screenshots covering every section of the page
2. **HTML/CSS** — Right-click → Inspect → Elements → Styles → Copy all
3. **URL** — The direct URL for Claude to potentially scrape

```
Capture Checklist:
□ Hero section (desktop + mobile)
□ Navigation bar
□ Feature/benefit sections
□ Social proof / testimonials
□ Pricing or CTA sections
□ Footer
□ Any unique animations or interactions (describe them)
```

### 5.2 Step 2: Build

Feed everything to Claude Code for replication:

```
I want to create a React/Vite project and copy this web page.

The following are screenshots of the page as well as the HTML code.
I want a pixel-perfect replication.

Here is the URL: [paste URL]

@screenshot1.png @screenshot2.png @screenshot3.png

[Paste HTML/CSS code below]
```

**What to expect**:

- First pass typically achieves **~80% match**
- Colors and typography are usually accurate
- Complex animations (particle effects, 3D backgrounds) may need separate attention
- Placeholder images will differ

**Tips for better first-pass results**:

- More screenshots = better accuracy
- Include mobile views if responsive design matters
- Explicitly mention animation libraries you want (Framer Motion, GSAP)
- Specify the tech stack: "Use React, Vite, and Tailwind CSS"

### 5.3 Step 3: Refine

This is where you spend the most time. The refinement phase has two parts:

#### Part A: Customize Messaging

Use **voice-to-text** (Whisper) to stream-of-consciousness describe your brand, then paste it into Claude:

```
I want to replace all the messaging on this website. Here's my product:

It's called [Name], and it's a [description]. Our target audience is
[audience]. The primary call to action is [CTA]. Our brand tone is
[tone - e.g., "bold and energetic" or "calm and trustworthy"].

What questions do you have before making these changes?
```

> **Pro Tip**: Ask Claude "What questions do you have?" — it will prompt you for missing details like colors, testimonials, feature lists, etc.

#### Part B: Add Premium Components

This is the **secret weapon** — component libraries that provide the "wow factor":

**21st.dev Workflow**:

1. Browse 21st.dev for components (backgrounds, buttons, cards, etc.)
2. Click **"Copy Prompt"** on the component you like
3. Paste the prompt into Claude Code
4. Claude implements the component in your project

```
Can we implement this as our background with a color scheme
that matches our brand?

[Paste 21st.dev component prompt here]
```

**Common Refinement Prompts**:

```
# Add glass morphism to cards
Make the feature cards have a glass morphism effect so the
background animation shows through. Use backdrop-filter: blur(10px)
with semi-transparent backgrounds.

# Extend animation across entire page
I want the background animation to span the entire page, not just
the hero. Make it visible behind all sections as I scroll.

# Add hover effects
Add subtle hover animations to all interactive elements:
- Buttons: slight scale + shadow increase
- Cards: lift effect with enhanced shadow
- Links: underline slide-in from left

# Improve a specific section
I hate how this section looks. Redesign it with a [style] approach.
Take Playwright screenshots before and after.
```

### 5.4 Step 4: Deploy

**GitHub + Vercel** (Most Common):

```bash
# Step 1: Create GitHub repo on github.com
# Step 2: In Claude Code:
Push this code to GitHub at [repo URL]

# Step 3: Go to vercel.com
# → New Project → Import from GitHub → Deploy

# Step 4: If errors occur:
[Paste Vercel error logs back into Claude Code]
```

**Vercel CLI** (Direct, skip GitHub):

```bash
# Install Vercel CLI
npm install -g vercel

# In Claude Code:
Deploy this project directly to Vercel using the CLI.
Here is my Vercel token: [token]
```

**Other Options**:

- **Netlify**: Similar to Vercel, great for static sites
- **Cloudflare Pages**: Fast edge deployment
- **GitHub Pages**: Free for static HTML/CSS/JS

---

## 6. Visual Iteration with Playwright MCP

### 6.1 The Missing Piece

Claude Code models are **multimodal** — they understand both code and images. But in default mode, they can only see code, not the visual output. This is like asking a designer to work blindfolded.

**Playwright MCP gives Claude eyes** by enabling it to:

- Capture screenshots of its own output
- Read browser console logs and network errors
- Navigate pages and click elements
- Emulate different devices and viewport sizes
- Scrape reference URLs for design inspiration

This unlocks the **vision modality** of the model — all the neurons trained on visual design data that were previously dormant.

### 6.2 Installation

```bash
# Quick install for Claude Code
claude mcp add playwright -- npx @anthropic-ai/playwright-mcp@latest

# Verify installation
# In Claude Code, type:
/mcp
# You should see Playwright listed with its tools
```

### 6.3 Configuration Options

Playwright MCP supports several configuration options via the MCP config JSON:

| Option       | Description                     | Example                            |
| ------------ | ------------------------------- | ---------------------------------- |
| `--browser`  | Browser engine                  | `chromium`, `firefox`, `webkit`    |
| `--device`   | Device emulation                | `"iPhone 15"`, `"iPad Pro"`        |
| `--headless` | Run without visible browser     | `true` / `false` (default: headed) |
| `--vision`   | Use coordinate-based operations | For pixel-precise interactions     |

**Config example** (in `.claude/mcp.json`):

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/playwright-mcp@latest", "--browser", "chromium"]
    }
  }
}
```

### 6.4 The Agentic Iteration Loop

This is the **single biggest unlock for frontend development**. The loop works like this:

```
┌──────────────────────────────────────────┐
│                                          │
│  1. Read spec/prompt (design reference)  │
│              ↓                           │
│  2. Write/modify code                    │
│              ↓                           │
│  3. Take Playwright screenshot           │
│              ↓                           │
│  4. Compare screenshot to spec           │
│              ↓                           │
│  5. Identify discrepancies               │
│              ↓                           │
│  6. Fix issues → Go to step 2            │
│                                          │
│  ✓ Exit when spec is satisfied           │
└──────────────────────────────────────────┘
```

**How to trigger it**:

```
Build this landing page based on the attached mockup.

After implementing, use Playwright to:
1. Navigate to localhost:3000
2. Take a full-page screenshot
3. Compare it against the mockup
4. Fix any discrepancies
5. Repeat until the page matches the mockup within 95% accuracy

Pay special attention to:
- Font sizes and weights
- Spacing between elements
- Color accuracy
- Animation timing
```

### 6.5 CLAUDE.md Auto-Verification

Add this to your CLAUDE.md to make Playwright verification automatic:

```markdown
## Visual Development — Playwright Verification

### After Every Frontend Change

1. Navigate to the affected page(s) at the running dev server
2. Set viewport to desktop size (1440x900)
3. Take a screenshot of each changed section
4. Compare against:
   - The original prompt/spec requirements
   - The design principles in ./context/design-principles.md
   - Any reference screenshots provided
5. Check browser console for errors or warnings
6. Fix visual issues before marking the task complete

### Mobile Responsive Check

For significant layout changes, also test:

- Tablet: 768x1024
- Mobile: 375x812 (iPhone viewport)
```

### 6.6 Key Workflows

**Design review via screenshots**:

```
Take screenshots of every page on our site at desktop, tablet,
and mobile viewports. Create a visual audit report noting:
- Responsive layout issues
- Typography inconsistencies
- Color contrast problems
- Alignment errors
```

**Scraping reference sites**:

```
Navigate to [URL] using Playwright. Take screenshots of:
1. The hero section
2. The pricing section
3. The footer

Analyze the design patterns used (typography, spacing, color palette)
and apply similar patterns to our homepage.
```

**Automated accessibility audit**:

```
Navigate to every page on our site using Playwright.
For each page:
1. Check color contrast ratios (WCAG AA minimum)
2. Verify all images have alt text
3. Test keyboard navigation flow
4. Check heading hierarchy (single h1, proper h2-h6 nesting)
5. Generate a prioritized list of accessibility fixes
```

---

## 7. Sub-Agent Pipelines for Design

### 7.1 The Three-Agent Pipeline

For complex design tasks, use a **pipeline of specialized sub-agents**, each with their own context window and model:

```
┌─────────────────────┐
│  Feature Architect   │ ← Opus (plans the design)
│  (Planning Agent)    │
└────────┬────────────┘
         │ Architecture document
         ▼
┌─────────────────────┐
│  Tool Implementer    │ ← Sonnet (writes the code)
│  (Building Agent)    │
└────────┬────────────┘
         │ Implemented code
         ▼
┌─────────────────────┐
│  QA Agent            │ ← Haiku/Sonnet (reviews quality)
│  (Review Agent)      │
└─────────────────────┘
```

**Why sub-agents for design?**

- Each agent gets a **fresh context window** — no context pollution from previous steps
- You can assign **different models** to different roles (Opus for creative, Sonnet for code, Haiku for review)
- The main thread's context stays clean — only the summary is returned
- Sub-agents can **call other sub-agents** (e.g., mobile reviewer → accessibility reviewer)

### 7.2 The Design Reviewer Agent

Create a dedicated design review agent at `.claude/agents/design-reviewer.md`:

```markdown
---
name: design-reviewer
description: >
  Reviews UI changes with the eye of a principal designer.
  Analyzes typography, color, spacing, motion, and accessibility.
model: sonnet
tools:
  - playwright
  - context7
  - Read
  - Grep
  - Glob
---

# Design Review Agent

You are a principal-level UI designer who has led design at companies like
Stripe, Airbnb, and Linear. You review pull requests and UI changes with
extremely high standards.

## Review Methodology

### Step 1: Gather Context

- Read the git diff for changed files
- Identify all affected UI components and pages

### Step 2: Visual Inspection

Using Playwright MCP:

1. Navigate to each affected page
2. Take screenshots at desktop (1440x900), tablet (768x1024), mobile (375x812)
3. Check browser console for errors

### Step 3: Design Analysis

Evaluate each dimension on a 1-10 scale:

| Dimension     | What to Check                                                |
| ------------- | ------------------------------------------------------------ |
| Typography    | Font hierarchy, sizes, weights, line heights, letter spacing |
| Color         | Palette consistency, contrast ratios, semantic usage         |
| Spacing       | Grid alignment, whitespace rhythm, padding consistency       |
| Motion        | Animation purpose, timing, easing curves                     |
| Depth         | Shadow consistency, layering, visual hierarchy               |
| Accessibility | Contrast, focus states, screen reader compatibility          |

### Step 4: Generate Report

Format:
```

## Design Review Report

**Overall Grade**: [A+ to F]
**Pages Reviewed**: [list]

### Strengths

- [what's working well]

### High Priority Issues

1. [issue] — [specific fix]
2. [issue] — [specific fix]

### Nice-to-Haves

- [polish item]

### Screenshots

[Before/after comparisons]

```

## Rules
- Do NOT make code changes — only report findings
- Focus on visual quality, not code quality
- Reference specific CSS properties when suggesting fixes
- Include Playwright screenshots in the report
```

**Invoke the agent**:

```
@agent design-reviewer Please review the last 3 commits
that affected frontend components.
```

### 7.3 Agent Chaining

Agents can invoke other agents, creating powerful review chains:

```
Design Reviewer
  ├── @agent mobile-reviewer    (checks responsive layouts)
  ├── @agent accessibility-checker (WCAG compliance)
  └── @agent performance-auditor  (Lighthouse scores)
```

### 7.4 Git Worktrees for A/B Design Testing

**Git worktrees** let you run multiple versions of your project simultaneously — perfect for comparing design approaches:

```bash
# Create worktrees for parallel design experiments
git worktree add ../project-design-a feature/design-a
git worktree add ../project-design-b feature/design-b
git worktree add ../project-design-c feature/design-c
```

Then open **three separate Claude Code sessions**, one in each worktree, with different prompts:

```
# Session A: "Make the hero section bold and brutalist"
# Session B: "Make the hero section minimal and editorial"
# Session C: "Make the hero section tech-forward with gradients"
```

Compare the three outputs and cherry-pick the winner.

> **Advanced**: One practitioner runs three headless Claude Code instances in GitHub workers, then uses a fourth Opus instance to judge which output is best.

---

## 8. Design Systems, Component Libraries & Tools

### 8.1 21st.dev — AI-Powered Component Registry

**21st.dev** is the most impactful tool for breaking out of generic AI designs. It's a community-driven registry of high-quality React components.

**Workflow**:

1. Browse categories: backgrounds, buttons, cards, carousels, scroll animations
2. Preview the component live
3. Click **"Copy Prompt"** — this copies installation/usage instructions
4. Paste into Claude Code — it implements the component in your project

```
# Example: Adding an animated background
Can we implement this as our background with a color scheme
that matches our brand?

[Paste 21st.dev prompt here]
```

**Why it works**: Instead of trying to describe "I want those flowing line things in the background," you show Claude exactly what you mean with production-ready code.

**Best categories for design improvement**:

- **Backgrounds**: Animated meshes, particle systems, flowing paths
- **Buttons**: Glow effects, shimmer, magnetic hover
- **Cards**: 3D tilt, glass morphism, spotlight effects
- **Text**: Typewriter, gradient text, split animations
- **Scroll**: Parallax, reveal on scroll, sticky sections

### 8.2 ShadCN UI with Presets

ShadCN provides a **presets system** that lets you customize the entire design system:

```bash
# Install ShadCN with a preset
npx shadcn init

# Add specific components
npx shadcn add button card dialog

# Apply a custom theme
# Edit components.json and tailwind.config to match your brand
```

> **Warning**: Raw ShadCN is the #1 cause of "AI slop" — every AI tool defaults to it. Use it as a foundation but always customize with your own design tokens.

### 8.3 Color Tools

| Tool                | Use Case                                        | URL                |
| ------------------- | ----------------------------------------------- | ------------------ |
| **Coolors.co**      | Generate harmonious color palettes              | coolors.co         |
| **Realtime Colors** | Preview palettes on a live mockup               | realtimecolors.com |
| **Happy Hues**      | Curated palette collections with usage examples | happyhues.co       |
| **Color Hunt**      | Browse trendy palettes                          | colorhunt.co       |

**Workflow**: Generate a palette → export hex codes → paste into your CLAUDE.md or design tokens file.

### 8.4 Design Feedback Tools

**Drawbridge**: A visual annotation tool that lets you draw directly on your browser and export precise feedback.

```
# Drawbridge workflow:
1. Open your site in the browser
2. Activate Drawbridge
3. Draw circles, arrows, and notes on problem areas
4. Export annotations as screenshots
5. Drag annotations into Claude Code as context
```

**Leva / Tweakpane**: Real-time parameter control panels that let you tune design values live:

```javascript
// Leva: Attach a control panel to any React component
import { useControls } from "leva";

function HeroSection() {
  const { blur, opacity, speed } = useControls({
    blur: { value: 10, min: 0, max: 50 },
    opacity: { value: 0.8, min: 0, max: 1 },
    speed: { value: 1, min: 0.1, max: 5 },
  });

  return <Background blur={blur} opacity={opacity} speed={speed} />;
}
```

This lets you find the perfect values for animations, blurs, opacities, colors — then hardcode them once you're satisfied.

### 8.5 Figma MCP Integration

If you have existing Figma designs, the **Figma MCP** bridges the gap between design and code:

**Setup**:

```bash
# Add Figma MCP to Claude Code
claude mcp add figma -- npx @anthropic-ai/figma-mcp@latest

# You'll need a Figma access token
```

**Design System Workflow**:

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│   Figma      │────▶│  Claude Code  │────▶│  Component   │
│   Design     │     │  + Figma MCP  │     │  Library     │
│   System     │     │              │     │  in Code     │
└─────────────┘     └──────────────┘     └──────────────┘
```

**Prompt for extracting design tokens**:

```
Using the Figma MCP, analyze our design system file and create:

1. style-tokens.ts — All color variables, typography scales,
   spacing values, border radii, and shadow definitions
2. A style guide page that renders all tokens visually
3. Component documentation (.txt) for each component with:
   - Visual specs (sizes, colors, spacing)
   - Interaction states (hover, active, disabled, focus)
   - Usage guidelines
   - TypeScript interface
```

**Component building workflow** (from Figma):

```
Following this workflow for each component:

1. Initialize: Connect to Figma file via MCP
2. Analyze: Extract component data (variants, states, interactions)
3. Implement: Build React component matching exact Figma specs
4. Apply tokens: Use design tokens from style-tokens.ts
5. Preview: Create interactive preview page
6. Document: Generate component documentation
7. Verify: Compare screenshot to Figma original
```

### 8.6 Google Stitch

Google Stitch is an AI design tool that generates UI designs from descriptions. Use it as a **design-first step** before coding:

1. Describe your desired page in Google Stitch
2. Export the design as an image or HTML
3. Feed the export to Claude Code as a reference
4. Claude builds the production version

---

## 9. Advanced Techniques

### 9.1 3D Websites with Three.js

Build immersive 3D landing pages using **Three.js** and **React Three Fiber**:

**Asset Sources**:

- **SketchFab**: Free 3D models (search for low-poly versions)
- **Poly Pizza**: Stylized low-poly assets
- **Quaternius**: Free game-ready 3D models

**Critical: 3D Optimization**

Heavy 3D models will destroy your site's performance. Always optimize:

```bash
# Install gltf-transform CLI
npm install -g @gltf-transform/cli

# Compress and optimize a 3D model
gltf-transform optimize input.glb output.glb --compress draco

# Simplify (reduce polygon count)
gltf-transform simplify input.glb output.glb --ratio 0.5
```

**Guidelines**:

- Keep draw count under **1000** for smooth interaction
- Use **Draco compression** on all glTF models
- Test on mid-range devices, not just your development machine
- Implement loading states — 3D models take time to download

**Prompt for 3D landing page**:

```
Create a premium 3D landing page using React Three Fiber.

Tech stack: Next.js + React Three Fiber + @react-three/drei

Requirements:
1. Hero section with an interactive 3D model (./assets/model.glb)
   - Mouse-controlled orbit rotation
   - Smooth auto-rotation when idle
   - Proper lighting (ambient + directional + spot)
2. Smooth scroll transitions between sections
3. Performance optimization:
   - Lazy load the 3D model
   - Use Suspense with a loading spinner
   - Enable Draco loader for compressed models
```

### 9.2 Animation Libraries

| Library           | Best For                                              | Complexity |
| ----------------- | ----------------------------------------------------- | ---------- |
| **CSS keyframes** | Simple hover effects, transitions                     | Low        |
| **Framer Motion** | React components, page transitions, scroll animations | Medium     |
| **GSAP**          | Complex timelines, scroll-triggered sequences         | High       |
| **Lottie**        | Pre-made animations from After Effects                | Low        |
| **Three.js**      | 3D scenes, WebGL effects                              | High       |

**Framer Motion with Next.js** (most common premium stack):

```
Create a Next.js landing page with the following Framer Motion animations:

1. Hero text: Staggered letter-by-letter entrance (50ms delay each)
2. Feature cards: Fade-in + slide-up on scroll (IntersectionObserver)
3. Page transitions: Smooth crossfade between routes
4. Buttons: Scale to 1.05 on hover, with spring physics
5. Navbar: Blur + shrink on scroll past hero section

Use the 'spring' transition type with damping: 25, stiffness: 300
for a premium, physics-based feel.
```

### 9.3 Style-Specific Prompting

Different aesthetics require different prompt patterns. Here are five proven style templates:

**1. Bold & Brutalist**

```
Style: Bold brutalist design with heavy typography, high-contrast blocks,
raw textures, asymmetric grids, and intentional visual disruption.
Use monospace fonts for body, ultra-bold sans-serif for headings.
Black/white with one accent color used sparingly.
```

**2. Editorial Luxury**

```
Style: Editorial luxury with elegant serif typography, generous whitespace,
muted color palette, subtle gold/cream accents, and refined micro-animations.
Think high-end fashion magazine or luxury real estate.
Photographic imagery with tasteful overlays.
```

**3. Organic & Natural**

```
Style: Organic web design with soft rounded shapes, flowing curves,
calm spaciousness, natural textures (grain, paper, linen),
gentle transitions, and earthy color palette.
Use rounded sans-serif fonts with generous line-height.
```

**4. Tech SaaS**

```
Style: Modern tech SaaS with dark theme, multi-gradient backgrounds,
animated particle/mesh backgrounds, glassmorphism cards,
neon accent colors, and futuristic typography.
Include animated code snippets and dashboard previews.
```

**5. High-Performance Nike/Apple**

```
Style: High-performance minimalist — diagonal layouts, ultra-clean typography,
large-scale product photography, bold scroll-triggered animations,
black/white with one dominant brand color, Nike/Apple aesthetic.
```

### 9.4 Design Principles as Prompts

Three fundamental principles that dramatically improve any AI-generated UI:

#### Shadows & Depth

```
Apply these shadow and depth improvements:

1. Color Layering: Create 3-4 shades of the base color. Use darker
   shades for backgrounds, lighter for foreground elements.
   ELIMINATE all borders — use color difference instead.

2. Two-Layer Shadows: Apply dual shadows to all elevated elements:
   - Light shadow (above): rgba(255,255,255,0.1) 0px -1px 2px
   - Dark shadow (below): rgba(0,0,0,0.15) 0px 4px 12px

3. Three Depth Levels:
   - Surface (cards, sections): subtle shadow
   - Raised (buttons, modals): medium shadow
   - Floating (dropdowns, tooltips): strong shadow + blur

4. Subtle Gradients: Apply light gradient overlays to dark-mode
   components for a premium feel.
```

#### Responsiveness

```
Make this layout fully responsive following these principles:

1. Flexible Box Model: Every section is a system of nested flex/grid
   containers. Each box should have a parent-child relationship that
   adapts to available space.

2. Priority-Based Rearrangement: Responsiveness isn't about shrinking —
   it's about rearranging based on priorities:
   - Desktop: Full layout with all elements visible
   - Tablet: Collapse secondary elements, stack grids
   - Mobile: Show only essential content, use carousels for overflow

3. Cards: Never squish cards on mobile. Maintain their proper shape
   and use horizontal carousel for overflow.
```

#### Color System

```
Implement a proper color system:

1. Primary Color: [hex] — Used for CTAs, buttons, and key interactive
   elements. This is the color that STANDS OUT most, not the most common.

2. Secondary Color: [hex] — Complements primary. Used for subtle actions,
   highlights, new feature badges.

3. Neutral Colors: Grays, whites, blacks — makes up 80%+ of the UI.
   Use 5-7 shades of neutral for backgrounds, text, and borders.

4. Semantic Colors:
   - Success: #22c55e (green)
   - Warning: #eab308 (amber)
   - Info: #3b82f6 (blue)
   - Error: #ef4444 (red)
```

### 9.5 Building Design Playgrounds

Create experimental spaces where you can freely explore components:

```
Create a design playground at /playground that includes:

1. A component showcase with every UI element we use
2. Interactive controls (using Leva or Tweakpane) for:
   - Typography scale
   - Color palette swatches
   - Spacing grid overlay
   - Animation speed/easing controls
3. A "mood board" section where I can drag in reference images
4. Dark/light mode toggle
5. Viewport size preset buttons (desktop, tablet, mobile)
```

---

## 10. Quick Reference — Prompts, Commands & Patterns

### Essential Installation Commands

```bash
# Claude Code
npm install -g @anthropic-ai/claude-code

# Playwright MCP
claude mcp add playwright -- npx @anthropic-ai/playwright-mcp@latest

# Frontend Design Skill
npx skills add anthropics/skills   # select "frontend-design"

# 3D Optimization
npm install -g @gltf-transform/cli

# Deployment
npm install -g vercel
```

### Design Skill Invocation

```
# Basic skill use
Use the frontend design skill to improve this page.

# With references
@reference1.png @reference2.png
Using the frontend design skill, redesign the homepage
to match these reference screenshots.

# Specific focus
Apply the frontend design skill focusing specifically on
typography hierarchy, micro-interactions, and visual depth.
```

### Agent Commands

```
# Design review
@agent design-reviewer Review the last 3 commits

# Trigger via slash command
/design-review

# Background agent
Ctrl+B to send agent to background while continuing work
```

### Playwright Workflows

```
# Quick visual check
Take a screenshot of localhost:3000 and identify
any obvious visual issues.

# Full responsive audit
Screenshot localhost:3000 at 1440px, 768px, and 375px width.
Report any responsive layout issues.

# Reference comparison
Navigate to [URL], take screenshots.
Compare their design patterns to ours and suggest improvements.
```

### Style Prompt Quick Reference

| Style                | Key Phrases                                                              |
| -------------------- | ------------------------------------------------------------------------ |
| **Brutalist**        | heavy typography, high contrast, raw textures, asymmetric, monospace     |
| **Editorial Luxury** | serif typography, generous whitespace, muted palette, gold accents       |
| **Organic**          | rounded shapes, flowing curves, natural textures, earthy colors          |
| **Tech SaaS**        | dark theme, gradients, glassmorphism, neon accents, animated backgrounds |
| **High-Performance** | diagonal layouts, ultra-clean typography, bold animations, minimalist    |

### Common Debugging

| Problem              | Fix                                                                      |
| -------------------- | ------------------------------------------------------------------------ |
| CSS not rendering    | "Check if Tailwind CSS is properly configured and PostCSS is processing" |
| Animations janky     | "Use `will-change: transform` and GPU-accelerated properties only"       |
| 3D model laggy       | "Run gltf-transform to compress and simplify the model"                  |
| Fonts not loading    | "Check if Google Fonts import is in the HTML head or globals.css"        |
| Colors look washed   | "Verify color space — use oklch() for perceptually uniform gradients"    |
| Mobile layout broken | "Add proper viewport meta tag and check for fixed-width elements"        |

### Project Structure Template

```
my-website/
├── .claude/
│   ├── agents/
│   │   └── design-reviewer.md
│   ├── commands/
│   │   ├── design-review.md
│   │   └── frontend-design.md
│   ├── skills/
│   │   └── designer/
│   │       ├── skill.md
│   │       ├── typography.md
│   │       ├── color-theme.md
│   │       └── motion.md
│   └── mcp.json              # Playwright MCP config
├── context/
│   ├── design-principles.md
│   ├── style-guide.md
│   ├── brand-story.md
│   └── reference-screenshots/
├── src/
│   ├── components/
│   ├── pages/
│   ├── styles/
│   │   └── design-tokens.ts
│   └── assets/
├── CLAUDE.md
├── package.json
└── README.md
```

---

## Summary

The key to building beautiful websites with Claude Code is **not better prompts** — it's **better environment setup**:

1. **Context**: Design principles, style guides, reference screenshots in CLAUDE.md
2. **Skills**: Install and customize the frontend-design skill for your aesthetic
3. **Tools**: Playwright MCP for visual iteration; Figma MCP for design systems
4. **Validation**: Sub-agents for design review; agentic loops for self-correction
5. **Components**: Use 21st.dev, ShadCN presets, and CodePen for the "wow factor"
6. **Process**: Follow Inspire → Build → Refine → Deploy for repeatable results

The gap between "AI slop" and premium design isn't the model — it's the orchestration around it.

---

_This guide was synthesized from community workflows, official Anthropic resources, and practitioner insights documented across 12+ design-focused Claude Code tutorials._
