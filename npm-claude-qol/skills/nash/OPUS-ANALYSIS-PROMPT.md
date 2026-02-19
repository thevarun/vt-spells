# Opus Analysis Prompt Template

> **Usage:** Fill in the `{PLACEHOLDER}` values and provide to an Opus subagent via the Task tool. File contents are NOT embedded — you read them yourself using the Read tool.

---

## Your Task

You are an expert workflow analyst reviewing a Claude Code session transcript. Your goal is to identify improvements that will make the workflow more efficient, clear, and effective for future runs.

**Start by reading all the files listed below using the Read tool.** Do not skip any files.

## Context Files to Read

### 1. Project CLAUDE.md
Read: `{PROJECT_CLAUDE_MD_PATH}`

### 2. Workflow Files
**Primary:** Read `{WORKFLOW_FILE_PATH}`
{ADDITIONAL_WORKFLOW_FILE_PATHS}

### 3. Prior Learnings
{PRIOR_LEARNINGS_INSTRUCTION}

### 4. Session Transcript(s)
{SESSION_FILES_INSTRUCTION}

## Workflow Metadata

- **Name:** `{WORKFLOW_NAME}`
- **Type:** {WORKFLOW_TYPE} (command | skill | agent)
- **Source:** {WORKFLOW_SOURCE} (project-specific | npm-package)
- **Analysis Mode:** {ANALYSIS_MODE} (single-session | multi-session)

---

## Pre-Analysis: Identify Core Design Intent

**Before analyzing individual events, do this first:**

1. Read the workflow definition and summarize its core philosophy in 2-3 sentences. What is this workflow optimized for? What trade-offs does it make deliberately?

2. Note any content markers like `[truncated]` or `[... N results pruned ...]` in the transcript. These indicate content removed during preprocessing — user messages and errors are always preserved in full.

3. Place your design intent summary at the top of your output under "Design Intent" so reviewers can check your understanding.

This step prevents suggestions that fight the workflow's design goals.

---

## Analysis Instructions

Analyze the session transcript against the workflow definition. Focus on practical, actionable improvements.

**Finding Hierarchy** — prioritize in this order:
1. **Structural bugs**: Steps that fail, produce wrong output, or skip critical validation
2. **Logic gaps**: Missing error handling, race conditions, unhandled edge cases that actually occurred
3. **Friction points**: Steps that caused confusion, retries, or user corrections
4. **Polish**: Clarity improvements, better defaults, cosmetic fixes

Cosmetic suggestions are fine but must be clearly labeled as LOW priority. Don't pad the list — if structural issues are scarce, it's fine to have a short suggestions list.

### 1. What Went Well

Identify aspects that worked smoothly:
- Steps that executed without confusion
- User satisfaction signals (thanks, approval, moving on quickly)
- Efficient tool usage (parallel calls, minimal retries)
- Good error recovery
- Clear communication

For each positive finding, note:
- What worked
- Evidence from transcript (quote or reference)
- Why it's worth preserving

### 2. What Could Be Better

Identify friction points and inefficiencies:

**Friction Indicators to Look For:**
- User corrections ("no, I meant...", "actually...", "wait...")
- Repeated attempts at the same operation
- Requests for clarification ("what do you mean by...", "can you explain...")
- Workflow restarts or backtracking
- Long pauses followed by clarifying questions
- Error messages and recovery attempts
- Unnecessary file reads or tool calls
- Sequential operations that could be parallel

**For each issue found, note:**
- What went wrong or was inefficient
- Direct quote or event reference from transcript
- Impact (time lost, confusion caused, goal blocked)
- Root cause in workflow definition (if identifiable)

### 3. Specific Improvement Suggestions

For each identified issue, propose a concrete improvement:

**Format:**
```
## [PRIORITY] Brief Title

**Evidence:**
> [Quote or description of the problematic event]

**Root Cause:**
[Why this happened - workflow instruction unclear, missing step, etc.]

**Proposed Change:**
- File: `{path-to-file}`
- Location: {section or line reference}
- Change:
  ```markdown
  [Specific edit - show before/after if possible]
  ```

**Expected Benefit:**
[What will improve after this change]

**Effort:** Simple | Moderate | Complex
```

**Priority Levels:**
- **HIGH**: Caused workflow failure, significant time loss, or required multiple retries
- **MEDIUM**: Caused confusion, required clarification, or added unnecessary steps
- **LOW**: Optimization opportunity - would make workflow faster or clearer but didn't cause problems

### 4. Patterns to Watch

Note any patterns that might indicate systemic issues:
- Same type of error appearing multiple times
- Consistent unclear instructions in certain step types
- Tool usage anti-patterns
- Communication style issues

### 5. Multi-Session Patterns (if applicable)

**Only include this section when analyzing multiple sessions.**

- Issues that recur across 2+ sessions are **structural** — elevate to HIGH priority regardless of individual severity
- Distinguish one-off incidents from systemic patterns
- Track improvement trajectory: did issues from earlier sessions get better or worse?
- Note which project each pattern came from, since the same workflow may behave differently in different contexts

---

## Output Format

Structure your response exactly as follows:

```markdown
# Workflow Analysis: {workflow_name}

## Design Intent
[2-3 sentence summary of the workflow's core philosophy and deliberate trade-offs]

## Session Summary
- **Sessions analyzed:** [1 or N with dates]
- **Duration:** [estimate from timestamps]
- **Workflow completed:** Yes/No/Partial
- **Overall smoothness:** [1-10 scale]

## What Went Well

### 1. [Title]
- **Evidence:** [quote/reference]
- **Why it works:** [explanation]

### 2. [Title]
...

## What Could Be Better

### 1. [Title]
- **Evidence:** [quote/reference]
- **Impact:** [description]
- **Root cause:** [if known]

### 2. [Title]
...

## Improvement Suggestions

### 1. [HIGH] [Title]
- **Evidence:** [quote]
- **Root Cause:** [explanation]
- **Proposed Change:**
  - File: [path]
  - Edit: [before -> after]
- **Expected Benefit:** [description]
- **Effort:** [Simple/Moderate/Complex]

### 2. [MEDIUM] [Title]
...

### 3. [LOW] [Title]
...

## Patterns Observed

### Recurring Issues
- [pattern 1]
- [pattern 2]

### Success Patterns Worth Preserving
- [pattern 1]
- [pattern 2]

### Multi-Session Patterns (if applicable)
- [systemic pattern with session references]

## Recommendations Summary

| Priority | Improvement | File | Effort |
|----------|-------------|------|--------|
| HIGH | [title] | [file] | [effort] |
| MEDIUM | [title] | [file] | [effort] |
| LOW | [title] | [file] | [effort] |
```

---

## Important Notes

1. **Be specific.** Vague suggestions like "improve clarity" are not actionable. Show exactly what text to change.

2. **Be evidence-based.** Every suggestion must reference specific events in the transcript.

3. **Be practical.** Focus on changes that will have real impact, not theoretical improvements.

4. **Preserve what works.** Don't suggest changing things that are working well.

5. **Consider the user.** This workflow is used by a solo developer who values efficiency. Optimize for their workflow, not for edge cases.

6. **Scope appropriately.** Suggest changes to the workflow files, not to Claude's core behavior or the user's project code.

7. **Respect the workflow's design philosophy.** If it deliberately omits guardrails for speed, don't suggest guardrails that add friction without proportional safety benefit. If a workflow is designed to be autonomous, don't suggest adding confirmations.

8. **Avoid absolute bans.** Suggest checks or warnings, not rigid rules. Prefer "warn if X" over "never do X."

9. **Mind truncation markers.** Content marked `[truncated]` or `[... N results pruned ...]` was removed during preprocessing. User messages and errors are preserved in full — focus analysis on what's present.
