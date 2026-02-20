# Deep Audit — {{DATE}}

## Scope

| Field | Value |
|-------|-------|
| **Mode** | {{MODE}} |
| **Scope** | {{SCOPE_DESCRIPTION}} |
| **Agents Run** | {{AGENT_COUNT}} |
| **Stack** | {{DETECTED_STACK}} |
| **Duration** | {{DURATION}} |
| **Commit** | `{{COMMIT_HASH}}` |

## Scorecard

| Dimension | Score | P1 | P2 | P3 | Assessment |
|-----------|------:|---:|---:|---:|------------|
{{SCORECARD_ROWS}}

**Overall Health: {{OVERALL_SCORE}}/10** — {{OVERALL_LABEL}}

## Findings

{{#IF_P1_COUNT}}
### P1 — Critical

{{P1_FINDINGS}}
{{/IF_P1_COUNT}}

{{#IF_P2_COUNT}}
### P2 — Important

{{P2_FINDINGS}}
{{/IF_P2_COUNT}}

{{#IF_P3_COUNT}}
### P3 — Minor

{{P3_FINDINGS}}
{{/IF_P3_COUNT}}

{{#IF_NO_FINDINGS}}
No findings above the confidence threshold. The codebase looks healthy across all audited dimensions.
{{/IF_NO_FINDINGS}}

### Finding Detail Template

<!-- Each finding renders as: -->
<!--
#### F-NNN: {{TITLE}} ({{SEVERITY}})

| | |
|---|---|
| **File** | `{{FILE}}:{{LINE}}` |
| **Dimension** | {{DIMENSION}} |
| **Confidence** | {{CONFIDENCE}}% |
| **Agent** | {{AGENT}} |

{{DESCRIPTION}}

**Suggestion:** {{SUGGESTION}}

---
-->

## Action Plan

Top {{ACTION_PLAN_COUNT}} prioritized fixes:

{{ACTION_PLAN_ITEMS}}

{{#IF_REFACTOR_PLAN}}
## Refactoring Roadmap

> **{{THEME_COUNT}} themes** | **{{QUICK_WIN_COUNT}} quick wins** | **Total effort: {{TOTAL_EFFORT}}**

{{EXECUTION_SUMMARY}}

### Quick Wins

{{QUICK_WIN_ITEMS}}

### Phase 1 — Safe Refactors

{{PHASE_1_THEMES}}

### Phase 2 — Enablers

{{PHASE_2_THEMES}}

### Phase 3 — High Impact

{{PHASE_3_THEMES}}

### Phase 4 — Polish

{{PHASE_4_THEMES}}

### Theme Detail Template

<!-- Each theme renders as: -->
<!--
#### T-NNN: {{THEME_NAME}}

| | |
|---|---|
| **Effort** | {{EFFORT}} |
| **Risk** | {{RISK}} |
| **Phase** | {{PHASE}} |
| **Findings** | {{FINDING_IDS}} |
| **Dependencies** | {{DEPENDENCIES}} |
| **Coverage Gate** | {{COVERAGE_GATE}} |
| **Blast Radius** | {{BLAST_RADIUS}} |

{{SUMMARY}}

**Refactoring Steps:**

{{STEPS}}

**Files Involved:**

{{FILES}}

**Testing:**
- *Before:* {{TESTS_BEFORE}}
- *After:* {{TESTS_AFTER}}

{{#IF_WARNINGS}}
**Warnings:** {{WARNINGS}}
{{/IF_WARNINGS}}

---
-->
{{/IF_REFACTOR_PLAN}}

## Statistics

| Metric | Value |
|--------|-------|
| Total Findings | {{TOTAL_FINDINGS}} |
| P1 (Critical) | {{P1_COUNT}} |
| P2 (Important) | {{P2_COUNT}} |
| P3 (Minor) | {{P3_COUNT}} |
| Agents Run | {{AGENT_COUNT}} |
| Dimensions Covered | {{DIMENSION_COUNT}} |

### Per-Agent Breakdown

| Agent | Findings | P1 | P2 | P3 | Dimensions |
|-------|--------:|---:|---:|---:|------------|
{{AGENT_BREAKDOWN_ROWS}}
