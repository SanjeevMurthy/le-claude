/plan
Act as an Expert Developer Experience (DX) Engineer and Claude Skill Creator. I need you to design and build a reusable "Skill" (a script, tool, or structured prompt workflow) that I can use to automatically generate comprehensive, expert-level interview preparation materials for any specific technology I provide.

The target audience for these materials is a Senior Engineer/Architect with 10+ years of hands-on experience (e.g., someone interviewing for Lead Cloud, DevOps, AI, or Principal Engineering roles).

Here are the strict requirements for the skill you need to build:

1. WORKFLOW & FILE STRUCTURE:
   For every technology topic I input (e.g., "Kubernetes", "AWS Networking", "System Design"), the skill must automatically:

- Create a parent folder named after the technology (e.g., `/interview-prep/{tech-topic}/`).
- Generate a series of structured Markdown (`.md`) files inside this folder. Do not put everything in one file.
- Example file structure to generate:
  - `01_Architecture_and_Internals.md`
  - `02_Real_World_Troubleshooting.md`
  - `03_Advanced_Interview_QA.md`
  - `04_System_Design_and_Scaling.md`

2. CONTENT QUALITY & DEPTH (10+ YOE Level):
   The generated content within these markdown files must go far beyond basic definitions. It must cover:

- Deep Internals: How the technology works under the hood.
- Subject Matter Expert (SME) Scenarios: High-traffic, highly available, and distributed system contexts.
- Battle-Tested Troubleshooting: Real-world outages, debugging complex issues, and memory/CPU profiling.
- The "Missing Pieces": Edge cases, undocumented limitations, and common pitfalls that only experienced engineers know.

3. INTERVIEW Q&A FORMAT:
   Inside the Q&A specific files, every question must follow this structure:

- The Question: A complex, scenario-based interview question.
- The Approach: How a senior candidate should structure their thought process before answering.
- The Ideal Answer: Deep, technically accurate, and concise.
- Cross-Questions / Follow-ups: 2-3 curveball questions the interviewer is likely to ask based on the candidate's answer to test the depth of their knowledge.
- Connecting Points: How this topic integrates with other enterprise ecosystem tools (e.g., how Kubernetes networking impacts AWS security groups).

Deliverables for this plan:

1. Outline the architecture of this skill (e.g., will it be a Python script, a Bash script using the Claude API, or a custom prompt template workflow?).
2. Draft the exact system prompt or code logic that will power the generation of these files.
3. Wait for my approval on the plan before proceeding to build the skill.
