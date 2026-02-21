# Shared Agent Instructions

Every audit agent MUST read this file before starting work. It defines the output format, confidence rules, false positive filters, and tool guidelines that all agents share.

## Output Format

Every agent MUST produce output in exactly this format.

### Finding Block

```
=== FINDING ===
agent: <agent-file-name without .md>
severity: P1|P2|P3
confidence: <80-100>
file: <relative file path>
line: <line number or range, e.g., 42 or 42-58>
dimension: <dimension name>
title: <concise one-line title>
description: |
  <2-4 sentences explaining the issue, why it matters, and concrete evidence>
suggestion: |
  <specific fix or approach — code snippet if helpful, but keep it brief>
=== END FINDING ===
```

### Dimension Summary Block

One per dimension the agent covers:

```
=== DIMENSION SUMMARY ===
dimension: <dimension name>
score: <1-10>
p1_count: <number>
p2_count: <number>
p3_count: <number>
assessment: |
  <2-3 sentences summarizing the dimension's health and key patterns observed>
=== END DIMENSION SUMMARY ===
```

### Output Order

1. All `=== FINDING ===` blocks (sorted by severity: P1 first, then P2, then P3)
2. All `=== DIMENSION SUMMARY ===` blocks

### Output Destination

Write your complete output to `_bmad-output/deep-audit/agents/<your-agent-name>.md` following the agent output template. After writing, print: `[OUTPUT WRITTEN] _bmad-output/deep-audit/agents/<your-agent-name>.md`

### Important Rules

- Do NOT include findings below 80% confidence
- Do NOT report findings outside your assigned dimensions
- Do NOT suggest fixes that introduce new problems
- Do NOT report the same issue multiple times across different files — report the pattern once and list affected files
- If no findings for a dimension, still include the DIMENSION SUMMARY with score and assessment
- Keep descriptions factual and evidence-based; avoid vague language like "could potentially" or "might cause issues"

## Confidence Threshold

Agents MUST only report findings with **confidence >= 80%** (on a 0-100 scale).

- **90-100**: Very high confidence — clear violation with concrete evidence
- **80-89**: High confidence — strong signal with reasonable certainty
- **Below 80**: Do NOT report — risk of false positive outweighs value

When assessing confidence, consider:
- Is this a definitive violation or a judgment call?
- Could there be a valid reason for this pattern you can't see?
- Would a senior engineer agree this is an issue?

## False Positive Prevention

Agents MUST NOT report:
- Issues a linter or formatter would catch (eslint, prettier, stylelint)
- Subjective style preferences that a senior engineer might reasonably disagree with
- Pre-existing patterns the codebase uses consistently (these are intentional conventions, not bugs)
- Potential issues that depend on runtime state, specific inputs, or environment config you cannot verify
- Micro-optimizations with negligible real-world impact

## Severity Definitions

| Level | Label | Meaning | Action |
|-------|-------|---------|--------|
| **P1** | Critical | Security vulnerability, data loss risk, or production blocker | Fix before next deploy |
| **P2** | Important | Significant quality issue that degrades maintainability or reliability | Fix within current sprint |
| **P3** | Minor | Code quality improvement; low risk but worth addressing | Fix when touching the file |

## Tool Usage Guidelines

### When Serena MCP is Available

If `find_symbol`, `find_referencing_symbols`, or other Serena MCP tools are available in your tool list, prefer them over Read/Grep for targeted code exploration:

| Task | Without Serena | With Serena |
|------|---------------|-------------|
| Find all usages of a function | Grep for function name | `find_referencing_symbols` |
| Understand module dependencies | Read import statements across files | `find_symbol` + references |
| Check type definitions | Grep for `interface`/`type` keywords | `find_symbol` with type filter |
| Trace call chains | Read multiple files following imports | `find_referencing_symbols` recursively |
| Find implementations | Grep for class/function names | `find_symbol` with implementation filter |

**Fallback**: If Serena tools are not available or return errors, fall back to Read/Grep. Do not fail the audit because an MCP tool is unavailable.

### General Tool Guidelines

- **Prefer targeted reads**: Read specific functions/sections rather than entire files when possible
- **Use Glob first**: Find relevant files before reading them
- **Batch searches**: Make parallel Grep calls when checking for multiple patterns
