---
name: product-architect
description: Agent team that creates PRD + Architecture from product notes. PM and Architect collaborate autonomously, escalating only for key decisions.
---

# Product Architect

**Goal:** Take product notes and produce a complete PRD + Architecture document with ~3-5 user interactions (down from 20+). A PM and Architect agent collaborate autonomously, applying BMAD methodology and your standing preferences, escalating only for genuine decisions.

---

## Prerequisites

1. **Agent Teams enabled** — requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` in settings
2. **BMAD Method installed** — `_bmad/bmm/workflows/` must exist in the project
3. **Output directory** — `_bmad-output/planning-artifacts/` (created automatically)

---

## Initialization

### Step 1: Verify Environment

Check these in order. Stop on first failure:

**1a. Agent Teams**
Check if Agent Teams is available by verifying the environment. If not available, stop immediately:
> This skill requires Agent Teams. Enable by adding to your settings.json:
> ```json
> { "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
> ```
> Then restart Claude Code and run `/product-architect` again.

**1b. BMAD Installation**
Verify these paths exist (all relative to project root):
- `_bmad/bmm/workflows/2-plan-workflows/create-prd/steps-c/` — PRD methodology
- `_bmad/bmm/workflows/2-plan-workflows/create-prd/templates/prd-template.md` — PRD template
- `_bmad/bmm/workflows/2-plan-workflows/create-prd/data/` — classification data
- `_bmad/bmm/workflows/3-solutioning/create-architecture/steps/` — Architecture methodology
- `_bmad/bmm/workflows/3-solutioning/create-architecture/architecture-decision-template.md` — Architecture template

If any path is missing, list which ones are missing and stop:
> BMAD Method is required but some paths are missing:
> - [list missing paths]
> Install BMAD Method first, then re-run `/product-architect`.

**1c. Config**
Read `_bmad/bmm/config.yaml` to extract:
- `project_name` — used for DB schema naming, file headers
- `output_folder` — typically `_bmad-output`
- `user` — for attribution

### Step 2: Load Preferences

Read `.claude/skills/product-architect/vt-preferences.md` and apply all Standing Preferences as constraints for the session.

### Step 3: Detect Input

The user provides product notes in one of these forms:
- **Notion link** → Use Notion MCP to fetch content (if available), otherwise ask user to paste
- **File path** → Read the file
- **Inline text** → Use directly
- **No input** → Ask: "What product are we building? Paste notes, a Notion link, or describe the idea."

### Step 4: Check for Existing Artifacts (Recovery)

Check `_bmad-output/planning-artifacts/` for existing files:

| Found | Meaning | Action |
|-------|---------|--------|
| Nothing | Fresh start | Proceed to Phase 1 |
| `prd.md` exists, incomplete | Previous session crashed during PRD | Offer: "Found an in-progress PRD. Continue from it or start fresh?" |
| `prd.md` exists, complete | PRD done, architecture not started | Offer: "Found a completed PRD. Skip to Architecture phase?" |
| Both exist, arch incomplete | Previous session crashed during Architecture | Offer: "Found PRD + in-progress Architecture. Continue Architecture?" |
| Both exist, both complete | Everything done | "Both PRD and Architecture already exist. Want to revise, or start fresh?" |

**Completeness heuristic:** A document is "complete" if it contains a "## Document Status" section with "Complete" or "Final" in it, OR if it's longer than 200 lines. Otherwise treat as incomplete.

---

## Team Creation

### Step 5: Create the Agent Team

Create an Agent Team with **delegate mode** — you (the lead) coordinate only, never implement.

Enable delegate mode (Shift+Tab after team creation) so you focus on orchestration.

**Spawn two teammates:**

#### PM (John)
Spawn with this prompt:
> Read your role definition at `.claude/skills/product-architect/agents/pm-agent.md`.
>
> **User Preferences:** [paste full Standing Preferences section from vt-preferences.md]
>
> **BMAD Paths (all relative to project root):**
> - PRD steps: `_bmad/bmm/workflows/2-plan-workflows/create-prd/steps-c/`
> - PRD data: `_bmad/bmm/workflows/2-plan-workflows/create-prd/data/`
> - PRD template: `_bmad/bmm/workflows/2-plan-workflows/create-prd/templates/prd-template.md`
>
> **Project:** [project_name from config]
> **Output file:** `_bmad-output/planning-artifacts/prd.md`
>
> **Product Notes:**
> [paste the user's product notes]
>
> Wait for phase instructions from the lead before starting work.

#### Architect (Winston)
Spawn with this prompt:
> Read your role definition at `.claude/skills/product-architect/agents/architect-agent.md`.
>
> **User Preferences:** [paste full Standing Preferences section from vt-preferences.md]
>
> **BMAD Paths (all relative to project root):**
> - Architecture steps: `_bmad/bmm/workflows/3-solutioning/create-architecture/steps/`
> - Architecture template: `_bmad/bmm/workflows/3-solutioning/create-architecture/architecture-decision-template.md`
>
> **Project:** [project_name from config]
> **Output file:** `_bmad-output/planning-artifacts/architecture.md`
>
> Wait for phase instructions from the lead before starting work.

### Step 6: Create Task List

Create these tasks with dependencies:

```
Task 1: [Phase 1] Discovery & Classification (PM + Architect parallel)
Task 2: [Phase 2] Product Definition — PRD creation (PM leads)
  - blocked by: Task 1
Task 3: [Phase 3] Architecture Design (Architect leads)
  - blocked by: Task 2
Task 4: [Phase 4] Validation & Polish (both parallel)
  - blocked by: Task 3
Task 5: Memory update proposal (Lead)
  - blocked by: Task 4
```

If recovering from a crash (Step 4), skip completed tasks and start from the appropriate phase.

---

## Phase Execution

Read `.claude/skills/product-architect/references/escalation-guide.md` for decision gate rules.

### Phase 1: Discovery (Task 1)

**Both agents work in parallel:**

Message PM:
> **Phase 1 — Discovery.** Read BMAD step `step-02-discovery.md`. Analyze the product notes:
> 1. Classify the project using `data/project-types.csv`
> 2. Identify key personas, domains, and complexity signals
> 3. Message Architect with your classification and any tech signals you spotted
> 4. When done, report findings back to me (the lead)

Message Architect:
> **Phase 1 — Discovery.** Read BMAD step `step-02-context.md`. Analyze the product notes:
> 1. Identify technology signals, constraints, and integration needs
> 2. Note anything that affects architecture decisions
> 3. Message PM with your tech assessment and any questions about the product
> 4. When done, report findings back to me (the lead)

**Lead actions after both report:**
- Review findings for alignment
- If classification is ambiguous → Quick Confirm gate to user
- If aligned → announce Phase 1 complete, move to Phase 2
- Mark Task 1 complete

### Phase 2: Product Definition (Task 2)

**PM leads. Architect challenges.**

Message PM:
> **Phase 2 — Product Definition.** Drive PRD creation using BMAD steps 3-9:
> 1. Read each step file as you reach it (step-03-success through step-09-functional)
> 2. Start from the PRD template. Write sections incrementally to your output file.
> 3. Apply MVP-first scoping. Mark anything beyond-MVP as "Future Consideration."
> 4. SaaS template features (auth, billing, subscriptions) → "Provided by SaaS template"
> 5. Message Architect for feasibility checks on complex features
> 6. When PRD is complete, message me and Architect

Message Architect:
> **Phase 2 — Support role.** PM is creating the PRD. Your job:
> 1. When PM messages you with feasibility questions, evaluate and respond
> 2. If you see something that will cause architecture problems, message PM proactively
> 3. Wait for PM to announce PRD completion before starting Phase 3

**Lead actions:**
- Monitor messages between PM and Architect
- If PM escalates a scope question → present to user as Quick Confirm or Full Gate
- When PM announces PRD complete → read `_bmad-output/planning-artifacts/prd.md` to verify it exists and has substance
- Mark Task 2 complete

### Phase 3: Architecture Design (Task 3)

**Architect leads. PM validates.**

Message Architect:
> **Phase 3 — Architecture Design.** The PRD is complete at `_bmad-output/planning-artifacts/prd.md`. Read it first.
> Then drive architecture creation using BMAD steps 2-6:
> 1. Read each step file as you reach it (step-02-context through step-06-structure)
> 2. Start from the architecture template. Write sections incrementally.
> 3. Apply standing tech conventions (DB schema, hosting, UI library — all settled).
> 4. Verify library versions via web search where appropriate.
> 5. Map every FR from the PRD to a location in the architecture.
> 6. Message PM to validate requirements coverage when data model and structure are drafted.
> 7. When architecture is complete, message me and PM.

Message PM:
> **Phase 3 — Support role.** Architect is creating the Architecture document.
> 1. When Architect asks you to validate requirements coverage, check the PRD against what they've designed
> 2. Flag any FRs that aren't addressed in the architecture
> 3. Wait for Architect to announce completion

**Lead actions:**
- Monitor messages
- If Architect escalates a tech decision → present to user per escalation guide
- When Architect announces complete → read `_bmad-output/planning-artifacts/architecture.md` to verify
- Mark Task 3 complete

### Phase 4: Validation & Polish (Task 4)

**Both agents work in parallel:**

Message PM:
> **Phase 4 — Validation.** Read BMAD steps 10-12 (if they exist). Polish the PRD:
> 1. Add NFRs (keep lean for MVP — standard values for uptime, latency, etc.)
> 2. Final document review — check for gaps, inconsistencies, orphan FRs
> 3. Add `## Document Status` section with "Complete" and today's date
> 4. Save final version. Message me when done.

Message Architect:
> **Phase 4 — Validation.** Read BMAD step `step-07-validation.md`. Validate:
> 1. Every FR maps to the architecture
> 2. Data model is complete and consistent
> 3. No missing integration points
> 4. Directory tree has no placeholders
> 5. Add `## Document Status` section with "Complete" and today's date
> 6. Save final version. Message me when done.

**Lead actions:**
- Wait for both to complete
- If critical gaps found → present to user
- Mark Task 4 complete
- Present completion summary to user:

```markdown
## Product Architect — Complete

**Documents created:**
- PRD: `_bmad-output/planning-artifacts/prd.md`
- Architecture: `_bmad-output/planning-artifacts/architecture.md`

**Decisions made:** [X] total ([auto-resolved], [quick confirms], [full gates])
**Escalations to user:** [count]
```

---

## Memory Update (Task 5)

After Phase 4, review all decisions made during the session. Identify new learnings — preferences, tech choices, corrections, patterns.

Present to the user:

```markdown
## Proposed Memory Updates

Based on this session, I'd like to remember for future sessions:

1. "[learning]" — Save? [Y/N]
2. "[learning]" — Save? [Y/N]
3. "[learning]" — Save? [Y/N]
```

For each approved item, append to the `## Learned Preferences` section of `.claude/skills/product-architect/vt-preferences.md`.

For significant decisions, also append to `## Decision Log` with format:
```
- [YYYY-MM-DD] [project_name]: [decision summary]
```

Mark Task 5 complete.

---

## Team Cleanup

After memory updates:
1. Ask PM and Architect to shut down
2. Wait for both to confirm shutdown
3. Clean up the team

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Teammate stops responding | Check their status. If stuck, send a nudge. If crashed, spawn replacement with context of what was completed. |
| Task appears stuck | Check if work is actually done but task wasn't marked complete. Update manually if needed. |
| Teammate starts implementing instead of waiting | Message them to stop and wait for phase instructions. |
| Lead tries to implement | You are in delegate mode. If you catch yourself implementing, stop and delegate. |
| BMAD step file missing | Skip that step, note it in the completion summary. The core methodology is in the agent role files. |
