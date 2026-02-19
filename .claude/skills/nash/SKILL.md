---
name: nash
description: Workflow optimization loop that reviews Claude Code session transcripts to extract learnings and improve workflows. Run at end of sessions to create a continuous improvement cycle.
tools: Read, Write, Edit, Glob, Grep, Bash, Task, AskUserQuestion
---

# Nash: Workflow Optimization Loop

**Goal:** Review Claude Code session transcripts to extract learnings and improve workflows, creating a continuous improvement cycle.

---

## Configuration

### NPM Package Sources

When a workflow is from an NPM package, edits are made to the source directory.

Read config from `~/.claude/nash-sources.yaml`. If the file is missing, ask the user for their local source paths and offer to create the file (see `nash-sources.example.yaml` for the format).

### Workflow File Locations

| Type | Pattern |
|------|---------|
| Commands | `.claude/commands/{name}.md` |
| Skills | `.claude/skills/{name}/SKILL.md` |
| Agents | `.claude/agents/{name}.md` |
| BMAD | `.claude/commands/bmad/{module}/...` |

---

## How It Works

1. **Discovery** - Parse arguments, find sessions, detect workflows
2. **Selection** - Choose session(s) and workflow to optimize
3. **Analysis** - Prune transcript if needed, launch Opus subagent
4. **Review** - Present findings, get user approval on action list
5. **Execute** - Apply changes, record learnings

**Checkpoints:** 4 total (Discovery, Source Confirmation, Action Approval)

---

## STEP 1: Discovery & Selection

### 1.1 Parse Arguments & Identify Context

Parse the `/nash` invocation for optional workflow name argument:

```
/nash                        → No target workflow specified
/nash review /designer-founder → Target: designer-founder
/nash /implement-epic         → Target: implement-epic
```

**Argument parsing rules:**
- Strip leading `/` from workflow names
- Accept workflow names with or without the slash prefix
- Multiple words after the workflow name are treated as notes/context (ignore for matching)

Detect the current project:
```bash
PROJECT_PATH=$(pwd)
SESSIONS_DIR=~/.claude/projects
```

### 1.2 Load Prior Learnings

Check for existing learnings file:

```bash
# Check project-level learnings
LEARNINGS_FILE=".claude/skills/nash/nash-learnings.md"
```

The file ships with the skill (pre-created with a header comment). If it contains only the header and no entries, treat it as "no prior learnings." If it has entries, pass the file path to Opus as `{PRIOR_LEARNINGS_INSTRUCTION}`.

### 1.3 Discover Sessions

**Search scope depends on whether a workflow name was provided:**

#### No workflow argument → Current project only

Search recent sessions in the current project directory:
```bash
# Find sessions for current project
# The project path is encoded in the sessions directory name
ls -lt "$SESSIONS_DIR"/<current-project-encoded>/*.jsonl 2>/dev/null | head -20
```

For each session, extract metadata:
- **Date**: From file modification time
- **Size**: File size (human-readable)
- **Line count**: `wc -l`
- **Workflows detected**: Scan for `/command` patterns and Skill tool invocations

#### Workflow argument provided → Cross-project search

Search across **all** project directories:
```bash
# Scan all project session directories
for project_dir in "$SESSIONS_DIR"/*/; do
  ls -lt "$project_dir"/*.jsonl 2>/dev/null
done
```

For each session found, grep for the target workflow name. Group matching sessions by project:
```
Recent sessions with /designer-founder:

ContentFlow (~/Coding/contentflow):
  a) Feb 13 | 425 lines | 0.4MB
  b) Feb 10 | 63 lines  | 0.2MB

FamilyTree (~/Coding/familytree):
  c) Feb 10 | 49 lines  | 0.1MB
```

### 1.4 Present Options to User

Build the options menu based on discovery results:

#### When no workflow argument was provided:

```
=== NASH: Workflow Optimization ===

Current Project: [project-name]

Current Session:
- Session: [session-uuid]
- Workflows detected: /dev-story, /code-review

Options:
1. [Default] Optimize from current session
   - Workflow: [first detected workflow]

2. Recent sessions (this project):
   a) [time ago] /create-story | 320 lines | 0.3MB
   b) [time ago] /implement-epic | 1240 lines | 1.1MB

3. Manual selection
   - Specify workflow + session path
```

#### When workflow argument was provided:

```
=== NASH: Workflow Optimization ===

Target Workflow: /designer-founder (from args)
Found 4 sessions across 2 projects.

Options:
1. [Default] Multi-session analysis (all 4)
   - Recommended when 3+ sessions match

2. Current project only (2 sessions)

3. Specific session — pick from list:
   ContentFlow:
     a) Feb 13 | 425 lines | 0.4MB
     b) Feb 10 | 63 lines  | 0.2MB
   FamilyTree:
     c) Feb 10 | 49 lines  | 0.1MB

4. Manual — specify different workflow + session path
```

**Smart defaults:**
- 1 matching session → auto-select it, skip to confirmation
- 2 sessions → offer both individually or combined
- 3+ sessions → default to multi-session analysis

**Edge cases:**
- No workflows detected in current session → skip option 1, show recent sessions list and manual selection
- Multiple workflows in current session → list all detected, let user pick (prefer most recently invoked as default)

### CHECKPOINT 1

**Present to user via AskUserQuestion:**
- Sessions discovered (with metadata)
- Workflow detected or specified
- Options menu (tailored to argument/no-argument mode)

**Wait for user selection before proceeding.**

---

## STEP 2: Load Context & Determine Source

### 2.1 Resolve Workflow Files

Based on user selection, locate all relevant workflow files:

```bash
# For commands
WORKFLOW_FILE=".claude/commands/{name}.md"

# For skills
WORKFLOW_FILE=".claude/skills/{name}/SKILL.md"
# Also check for additional files in the skill directory

# For agents
WORKFLOW_FILE=".claude/agents/{name}.md"
```

### 2.2 Dynamic NPM Package Detection

Determine if the workflow comes from an NPM package by reading the config and `plugin.json` files:

```bash
# Read package source paths from config
# ~/.claude/nash-sources.yaml contains:
#   packages:
#     "@torka/claude-workflows": "~/Coding/vt-spells/npm-claude-workflows"
#     "@torka/claude-qol": "~/Coding/vt-spells/npm-claude-qol"

CONFIG_FILE="$HOME/.claude/nash-sources.yaml"
# Parse package paths from config (expand ~ to $HOME)

# For each package, read plugin.json to find workflow sources
for pkg in <parsed_package_paths>; do
  if [ -f "$pkg/.claude-plugin/plugin.json" ]; then
    cat "$pkg/.claude-plugin/plugin.json"
    # Parse the commands, agents, skills arrays
    # If workflow name matches, this is the source
  fi
done
```

**Detection logic:**
1. Read `~/.claude/nash-sources.yaml` to get package source paths
2. Read each package's `plugin.json`
3. Check if workflow name appears in `commands`, `agents`, or `skills` arrays
4. If match found → NPM package source (record package path from config)
5. If no match → Project-specific
6. If config file missing → Ask user for paths and offer to create it

### 2.3 Gather All Context Files

Collect:
1. **Session transcript(s)** - The JSONL file(s) selected
2. **Primary workflow file** - The main workflow/command/skill definition
3. **Step files** - Any included steps (for skills)
4. **Template files** - Referenced templates
5. **Project CLAUDE.md** - For project context

### CHECKPOINT 2

**Confirm with user via AskUserQuestion:**
- Workflow file path(s) to analyze
- Session file path(s) and count
- Workflow source: `project-specific` or `npm-package: @torka/{name}`
- Context files that will be included
- Analysis mode: single-session or multi-session

**Wait for user confirmation before proceeding to analysis.**

---

## STEP 3: Analysis (Opus Subagent)

### 3.0 Transcript Preparation

Before sending to Opus, prune large transcripts to fit within context limits. Pruned files are written to `.claude/skills/nash/tmp/` (already exists, shipped with the skill).

**First**, clean up any orphans from previous crashed runs:
```bash
find .claude/skills/nash/tmp -name "*.pruned" -mmin +60 -delete 2>/dev/null
```

Run the co-located pruning script:
```bash
python3 .claude/skills/nash/prune_transcript.py .claude/skills/nash/tmp SESSION_FILE_1 [SESSION_FILE_2 ...]
```

**Size tiers:**
- **Under 2MB**: Pass full transcript unchanged
- **2-10MB**: Prune — keep all user messages in full, truncate tool_use inputs >500 chars, tool_result content >1000 chars, text blocks >2000 chars (keep first 200-500 + `[truncated]`), preserve errors in full, drop metadata events
- **Over 10MB**: Aggressive — also remove consecutive exploratory Glob/Grep/Read events (keep final one in sequence), insert `[... N results pruned ...]` markers

After Opus has finished reading, **clean up immediately**:
```bash
rm -f .claude/skills/nash/tmp/*.pruned
```

### 3.1 Prepare Analysis Prompt

Read the Opus prompt template from `.claude/skills/nash/OPUS-ANALYSIS-PROMPT.md`.

**The key optimization:** Do NOT read transcript or workflow file contents into main agent context. Instead, pass **file paths** to the Opus subagent and let it read them directly. This keeps the main agent's context lean.

Populate only the lightweight placeholders:

| Placeholder | Value | How |
|-------------|-------|-----|
| `{WORKFLOW_NAME}` | Workflow name | String |
| `{WORKFLOW_TYPE}` | command / skill / agent | String |
| `{WORKFLOW_SOURCE}` | project-specific / npm-package | String |
| `{ANALYSIS_MODE}` | single-session / multi-session | String |
| `{PROJECT_CLAUDE_MD_PATH}` | Path to project's CLAUDE.md | File path |
| `{WORKFLOW_FILE_PATH}` | Primary workflow file path | File path |
| `{ADDITIONAL_WORKFLOW_FILE_PATHS}` | Extra file paths (steps, templates) | See below |
| `{PRIOR_LEARNINGS_INSTRUCTION}` | Learnings path or "none" message | See below |
| `{SESSION_FILES_INSTRUCTION}` | Transcript file path(s) | See below |

**Additional workflow files** — format as a bullet list of Read instructions:
```
Also read these supporting files:
- `{path_to_step_1}`
- `{path_to_template_1}`
```
If none, use: "No additional files."

**Prior learnings** — if `nash-learnings.md` exists:
```
Read: `.claude/skills/nash/nash-learnings.md`

Use prior learnings as helpful context, not gospel. Earlier decisions may need revision as workflows evolve. Weigh prior patterns but don't blindly repeat them — if new evidence contradicts an old learning, flag it explicitly.
```
If no file exists: "No prior learnings recorded for this project."

**Session files** — for single-session:
```
Read the pruned transcript: `.claude/skills/nash/tmp/{filename}.pruned`
```

For multi-session, enforce a **4MB total budget**:
- After pruning, sum the sizes of all `.pruned` files
- If total exceeds 4MB: keep only the most recent sessions that fit within budget, notify user which sessions were dropped
- If still over budget with a single session: re-run pruning with forced aggressive mode

Then list the files:
```
Read each pruned transcript in order:
1. `.claude/skills/nash/tmp/{session_1}.pruned` — {project_name} | {date} ({size})
2. `.claude/skills/nash/tmp/{session_2}.pruned` — {project_name} | {date} ({size})
```

### 3.2 Launch Opus Subagent

Use the Task tool to spawn an Opus subagent:

```
Task(
    subagent_type="general-purpose",
    model="opus",
    prompt=<populated OPUS-ANALYSIS-PROMPT>
)
```

### 3.3 Receive Structured Analysis

The Opus subagent returns findings in structured markdown:
- Design Intent summary
- What Went Well (with evidence)
- What Could Be Better (with evidence)
- Specific Improvement Suggestions (prioritized by hierarchy)
- Multi-Session Patterns (if applicable)


### CHECKPOINT 3

**Wait for Opus subagent to return with findings.**

**Proceed directly to Review — no user input needed here.**

---

## STEP 4: Review & Prioritize

### 4.1 Present Findings

Display the analysis results in an actionable format:

```
=== ANALYSIS COMPLETE ===

Design Intent: [Opus's summary of the workflow's philosophy]

What Went Well:
- [finding 1]
- [finding 2]

Improvement Opportunities:
1. [HIGH] Add pre-validation step before X
   - Evidence: User had to restart 3 times due to missing file
   - Effort: Simple

2. [MEDIUM] Clarify step 3 instructions
   - Evidence: User asked clarifying question
   - Effort: Simple

3. [LOW] Add parallel tool calls for search operations
   - Evidence: Sequential searches added latency
   - Effort: Moderate

Select items to action [1,2,3 or 'all' or 'none']:
```

### 4.2 Gather User Feedback

Using AskUserQuestion:
- Which improvements to implement
- Any custom improvements to add
- Any modifications to suggestions
- Final confirmation of action list

### CHECKPOINT 4

**Action list confirmed:**
- [X] Improvement 1
- [X] Improvement 3
- [ ] Improvement 2 (skipped)

**Wait for user approval of final action list before executing.**

---

## STEP 5: Execute Changes

### 5.1 Generate Edit Plan

For each approved improvement:
1. Identify exact file(s) to modify
2. Draft specific edits (old_string → new_string)
3. Note dependencies between edits

Present edit plan:
```
File: .claude/commands/dev-story.md
Edit 1: Add validation step
  - Location: After step 1
  - Change: [before → after preview]
```

### 5.2 Execute Based on Source Type

#### Path (a) - Project-Specific Workflow

```bash
# Make edits directly to project's .claude/ directory
Edit(
    file_path=".claude/commands/{name}.md",
    old_string="...",
    new_string="..."
)

# Validate changes
cat .claude/commands/{name}.md | head -50
```

**Done after validation.**

#### Path (b) - NPM-Packaged Workflow

**Step 1:** Read NPM package CLAUDE.md for package-specific instructions:
```bash
cat {source_path}/CLAUDE.md
```

**Step 2:** Make edits to NPM package source:
```bash
Edit(
    file_path="{source_path}/{source-path}",
    old_string="...",
    new_string="..."
)
```

**Step 3:** Bump version (patch for fixes, minor for features):
```bash
cd {source_path}
npm version patch --no-git-tag-version
```

**Step 4:** Commit changes:
```bash
cd {source_path}
git add -A
git commit -m "fix({workflow-name}): {brief description}"
```

**Step 5:** Push if configured:
```bash
cd {source_path}
git push origin main 2>/dev/null || echo "Push manually if needed"
```

**Step 6:** Prompt for publish:
```
Ready to publish @torka/{package-name}@{new-version}

If you have 2FA enabled on npmjs.com, run manually:
cd {source_path} && npm publish

Otherwise, I can run `npm publish` for you.
```

**Step 7:** Update project dependency:
```bash
npm update @torka/{package-name}
```

### 5.3 Validate Changes

```bash
# Verify file was modified correctly
cat {modified-file-path}

# For NPM packages, verify version bump
cat {source_path}/package.json | grep version
```

### 5.4 Record Learnings

Append an entry to `.claude/skills/nash/nash-learnings.md`. Create the file if it doesn't exist.

**Entry format:**

```markdown
## {YYYY-MM-DD} - {workflow_name}
**Session(s)**: {first 8 chars of session UUID} | {single-session | multi-session}

**Changes applied**:
- {change_title}: {one-line description of what was changed and why}
- {change_title}: {one-line description}

**Key insights**:
- {insight}: {what we learned about how this workflow behaves in practice}
- {insight}: {another learning, if applicable}

**Rejected suggestions**:
- {suggestion_title}: Rejected because {specific reason — e.g., "adds friction without safety benefit", "contradicts design intent of autonomous execution"}
- {suggestion_title}: Rejected because {reason}
```

**Writing rules:**
- Each applied change gets its own bullet with a title and one-line description
- Each insight gets its own bullet — capture what we learned, not just what we changed
- Each rejected suggestion gets its own bullet with the specific rejection reason (this helps Opus avoid suggesting the same thing again)
- If there are no rejections, omit the "Rejected suggestions" section
- If there's only one change/insight, still use the bullet format for consistency
- Keep entries concise — this is a reference log, not a narrative

**Size management:** No hard line limit. When the file grows large (~50+ entries), periodically consolidate the oldest entries into a "Historical Summary" section at the top, preserving key patterns while removing individual entry detail:

```markdown
# Nash Learnings

## Historical Summary (consolidated from {N} earlier entries)
- {workflow}: {recurring pattern or settled decision}
- {workflow}: {another consolidated insight}

---

## {recent entries continue below}
```

### 5.5 Completion Summary

Display a summary of everything done — no confirmation needed:

```
=== NASH COMPLETE ===

Workflow: {workflow_name}
Session(s) analyzed: {count}
Analysis mode: {single-session | multi-session}

Changes made:
- {file}: {change summary}
- {file}: {change summary}

Learnings recorded: .claude/skills/nash/nash-learnings.md
Source: {project-specific | npm-package}
Next steps: {if any, e.g., "run npm publish manually"}
```

---

## Quick Reference

### Session File Location
```
~/.claude/projects/{encoded-project-path}/{session-uuid}.jsonl
```

### Priority Levels
- **HIGH**: Caused workflow failures or significant friction
- **MEDIUM**: Caused confusion or required clarification
- **LOW**: Optimization opportunities (speed, clarity)

### Effort Levels
- **Simple**: Single file edit, clear change
- **Moderate**: Multiple files or requires careful consideration
- **Complex**: Architectural change or significant rewrite

### JSONL Event Types (for parsing)
- `user` - User message (look for /commands here)
- `assistant` - Assistant response with tool_calls
- `tool_result` - Result of a tool call
