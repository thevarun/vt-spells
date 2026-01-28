# Stitch Tool Execution

## Overview

Stitch is Google's AI design tool that generates web UI screens from text prompts.
This tool requires both Stitch MCP and Google's stitch-skills to be installed.

**Output Location:** `.stitch/screens/`
**Output Format:** HTML + CSS files + screenshots
**MCP Tools Used:** `generate_screen_from_text`, `get_screen`, `list_projects`, `create_project`
**Dependencies:** Google's stitch-skills (design-md skill at minimum)

---

## Prerequisites

Check availability before proceeding:

```
Stitch MCP: [check if mcp__stitch* or stitch* tools available]
Stitch Skills: [check if design-md skill installed]
```

### If Stitch MCP not available:

```
STITCH MCP NOT CONFIGURED

Stitch MCP is required for this design tool.

To use Stitch:
1. Configure Stitch MCP server (see stitch.withgoogle.com/docs/mcp/guide/)
2. Restart Claude Code

Alternative options:
[S] SuperDesign - HTML/CSS prototype
[M] MagicPatterns - AI-generated React components
[B] Back to tool selection
```

### If Stitch Skills not installed:

```
STITCH SKILLS REQUIRED

Google's stitch-skills are required for Stitch integration.

Install with:
npx skills add google-labs-code/stitch-skills --skill design-md -g -a claude-code -y
npx skills add google-labs-code/stitch-skills --skill react-components -g -a claude-code -y

Then restart Claude Code and try again.

[B] Back to tool selection
```

---

## Execution Flow

### 1. Check for DESIGN.md

```
STITCH PROJECT SETUP

Checking for design system documentation...
```

**If DESIGN.md exists in project root:**

```
Found: DESIGN.md
Design system will be used for prompt consistency.

[C] Continue with existing design system
[U] Update design system first (uses design-md skill)
```

**If DESIGN.md doesn't exist:**

```
No DESIGN.md found.

DESIGN.md helps Stitch generate consistent designs by documenting
your color palette, typography, and component styles.

Options:
[G] Generate DESIGN.md (Recommended)
    → Uses design-md skill to create semantic design documentation
    → Best for: New projects, establishing design consistency

[S] Skip - proceed without design system
    → Stitch will use default styling
    → Fine for: Quick prototypes, one-off designs
```

**If G (Generate):**
- Invoke the design-md skill: `/design-md`
- Wait for user to complete the skill workflow
- Continue to design generation after DESIGN.md is created

---

### 2. Prepare Design Prompt

Frame the design request from context:

```
DESIGN REQUEST FOR STITCH

Feature: {user_intent}

Context:
- {scope_summary}
- {inspiration_references}

Requirements:
- Responsive design (desktop-first)
- Tailwind CSS classes
- {additional_context_from_scope}

{if DESIGN.md exists}
DESIGN SYSTEM:
[Include relevant sections from DESIGN.md - colors, typography, component styles]
{/if}

Style Direction:
- {visual_direction_notes}
- {inspiration_sources}
```

---

### 3. Create or Select Project

**If `stitch.json` exists in project root with projectId:**

```
EXISTING STITCH PROJECT FOUND

Project: {project_title}
Project ID: {projectId}

[C] Continue with this project
[N] Create new project
```

**If no existing project or user selects [N]:**

```
Creating new Stitch project...
```

```
Tool: create_project
Parameters:
  title: "{feature_name} Design"
  description: "{user_intent summary}"
```

Save projectId to `stitch.json` for future sessions:

```json
{
  "projectId": "{created_project_id}",
  "projectTitle": "{project_title}",
  "createdAt": "{timestamp}"
}
```

---

### 4. Generate Screen

```
GENERATING DESIGN

Feature: {user_intent}
Project: {project_name}

Please wait while Stitch generates your design...
```

```
Tool: generate_screen_from_text
Parameters:
  projectId: "{projectId}"
  prompt: "{constructed_prompt}"
  deviceType: "DESKTOP"
```

---

### 5. Retrieve and Save Assets

After generation completes:

```
Tool: get_screen
Parameters:
  projectId: "{projectId}"
  screenId: "{screenId}"
```

Create output directory if needed:
```
mkdir -p .stitch/screens/
```

Download and save assets:
- **HTML:** `htmlCode.downloadUrl` → save to `.stitch/screens/{page}.html`
- **Screenshot:** `screenshot.downloadUrl` → save to `.stitch/screens/{page}.png`

---

### 6. Present Result

```
DESIGN CREATED

Project: {project_name}
Screen: {screen_title}

Files saved:
- HTML: .stitch/screens/{page}.html
- Preview: .stitch/screens/{page}.png

Options:
[V] View - Display screenshot
[P] Playwright - Capture live rendering (desktop + mobile)
[U] Update - Request changes via Stitch
[C] Continue - Design approved, proceed to next step
```

### Handle User Selection

**If V (View):**
- Display the screenshot image from `.stitch/screens/{page}.png`
- Return to options

**If P (Playwright) and Playwright available:**
- Navigate to `file://{absolute_path}/.stitch/screens/{page}.html`
- Capture desktop screenshot (1280px width)
- Resize viewport (375px width) and capture mobile screenshot
- Present both screenshots for comparison
- Return to options

**If U (Update):**
```
What changes would you like to make?

Describe the changes and I'll generate an updated design.
```
- Append change request to prompt
- Generate new screen with updated prompt
- Return to step 5

**If C (Continue):**
- Store design state
- Return control to parent step (step-03-design.md)

---

## Output State

After completion, set:

```yaml
design:
  tool_used: stitch
  output_location: ".stitch/screens/{page}.html"
  output_format: html
  needs_conversion: true  # HTML needs React conversion in Step 4
  screenshot: ".stitch/screens/{page}.png"
  stitch_project_id: "{projectId}"
  stitch_screen_id: "{screenId}"
  design_md_used: true/false
```

---

## Design Registry Pattern

When creating multiple screens (scope has multiple items), track progress:

```yaml
designs:
  - name: "{scope_item_name}"
    screen_id: "{id_or_pending}"
    status: [pending | created | approved]
    html_path: ".stitch/screens/{name}.html"
    screenshot_path: ".stitch/screens/{name}.png"
```

### Batch Generation Flow

If scope has multiple items:

```
SCOPE ITEMS TO DESIGN

You have {count} items to design:
{foreach scope_item}
[ ] {item_name}
{/foreach}

Options:
[A] All at once - Generate all designs in sequence
[O] One by one - Generate and approve each individually

Which approach? [A/O]
```

**If A (All at once):**
- Generate each design in sequence
- Track status in design registry
- Present all for review at end

**If O (One by one):**
- Generate first design
- Present for approval
- On approval, move to next
- Repeat until all complete

---

## Conversion Notes

Stitch outputs HTML/CSS with Tailwind classes. The conversion step (Step 4) offers two paths:

1. **Quick shadcn mapping** - Our standard `conversion.md` flow
2. **Google's react-components** - Full modular React architecture with AST validation

Both are presented in Step 4 when `design.tool_used` = `stitch`.

---

## Reference: Stitch MCP Tools

| Tool | Purpose |
|------|---------|
| `list_projects` | Get all Stitch projects |
| `get_project` | Get project details |
| `create_project` | Create new project |
| `list_screens` | Get all screens in project |
| `get_screen` | Get screen details + download URLs |
| `generate_screen_from_text` | Generate new screen from prompt |
