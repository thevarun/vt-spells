# Product Architect — User Preferences & Memory

> Loaded by /product-architect at session start.
> **Standing Preferences**: edit freely to match your defaults.
> **Learned Preferences**: managed by the system at session end (propose-then-confirm).
> This file is never overwritten on package updates.

## Standing Preferences

### Developer Profile
- Solo developer building multiple SaaS products
- Values efficiency, lean MVP, quality over speed
- Mid-level technical familiarity, learns fast

### Philosophy
- MVP-first: focus exclusively on MVP scope. Beyond-MVP is "Future Consideration."
- Simplicity: keep architecture straightforward. Don't overcomplicate.
- Quality > Speed: correctness matters more than turnaround time.
- Generic when uncertain: don't lock in providers/tech that haven't been decided.
- No scope creep: don't suggest expanding beyond what was requested.

### SaaS Template
Auth, billing, subscriptions, user management are handled by a shared SaaS template.
Do NOT include these as features to build. Reference as "provided by SaaS template."

### Default Tech Stack
- Frontend: TypeScript, React, Next.js (App Router), Tailwind CSS
- Backend: TypeScript preferred (Python for data-heavy/ML projects)
- Database: PostgreSQL via Supabase (shared instance)
- DB Convention: ALWAYS use project-specific schema (e.g., `projectname.*`) not `public`
- Auth: Supabase Auth
- Hosting: Vercel (frontend) + Supabase (backend/DB)
- UI Components: shadcn/ui

### Data Sources
- Notion is primary planning tool. Expect Notion links as input for PRDs.
- PRD output: `_bmad-output/planning-artifacts/prd.md`
- Architecture output: `_bmad-output/planning-artifacts/architecture.md`

## Learned Preferences
<!-- System-managed. Entries added after user approval at session end. -->

## Decision Log
<!-- Key decisions across sessions. Format: - [date] [project]: [decision] -->
