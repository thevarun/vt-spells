# Escalation Guide — Decision Gate Reference

> Used by the orchestrator to decide when to involve the user vs auto-resolve.

## Auto-Resolve Rules

These are resolved silently from preferences. **Never ask the user:**

| Decision | Resolution |
|----------|-----------|
| Tech stack | Apply from preferences (Next.js, Supabase, etc.) |
| DB schema naming | Always project-specific (`projectname.*`) |
| SaaS template features | Mark "provided by SaaS template", skip |
| Component library | shadcn/ui — settled |
| Hosting | Vercel + Supabase — settled |
| Auth approach | Supabase Auth — settled |
| Metrics complexity | Keep lean for MVP |
| Undecided tech/libraries | Keep generic, don't lock in |
| NFR boilerplate | Use standard values (99.9% uptime, <200ms P95, etc.) |

## Quick Confirm Triggers

Present a recommendation with 1-line rationale. Expect a quick yes/no:

| Trigger | Format |
|---------|--------|
| Project classification | "Classifying as [type] because [reason]. OK?" |
| MVP scope boundary | "Proposing [X] as MVP, [Y] as Future. OK?" |
| Simple tech choice | "Using [library] for [purpose] — [1-line why]. OK?" |
| Ambiguous requirement | "[Requirement] could mean [A] or [B]. Which?" |

## Full Gate Triggers

Present the multi-perspective decision table. These warrant real discussion:

| Trigger | Why it needs a gate |
|---------|-------------------|
| Team disagrees on approach | PM and Architect have different views |
| Genuine trade-off | Significant impact on user experience or cost |
| Missing information | Only the user has this context |
| Contradicts preferences | Something in the notes conflicts with standing preferences |
| Novel domain | No prior sessions to draw from |

## Decision Gate Format

```markdown
## Decision Gate: [Title]
**Context:** [1-2 sentences]

| Perspective | View |
|-------------|------|
| PM (John) | [position + reasoning] |
| Architect (Winston) | [position + reasoning] |

**Recommendation:** [option] — [reasoning]
**Alternatives:** [list other options briefly]
**Your call:** [specific question]
```

## Progress Update Format

Use when transitioning between phases:

```markdown
## Phase [N] Complete: [Name]

**Decisions made:** [count] ([auto-resolved], [quick confirms], [gates])
**Artifacts:** [files written/updated]
**Next:** Phase [N+1] — [Name] ([who] leads)
```
