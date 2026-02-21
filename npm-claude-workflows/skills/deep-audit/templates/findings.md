# Deep Audit Findings — Consolidated & Triaged

## Metadata
- **Date**: {{DATE}}
- **Mode**: {{MODE}}
- **Scope**: {{SCOPE_DESCRIPTION}}
- **Agents**: {{AGENT_COUNT}} ({{AGENT_NAMES}})
- **Commit**: {{COMMIT_HASH}}

## Accepted Findings

<!-- Findings that passed validation and relevance checks. Grouped by severity. -->

### P1 — Critical

<!-- P1 findings are always ACCEPTED unless INVALID. -->

=== FINDING ===
id: F-001
agent: <source-agent>
severity: P1
confidence: <80-100>
file: <path>
line: <number or range>
dimension: <dimension name>
title: <one-line>
description: |
  <2-4 sentences>
suggestion: |
  <specific fix>
=== END FINDING ===

### P2 — Important

<!-- ... accepted P2 findings ... -->

### P3 — Minor

<!-- ... accepted P3 findings ... -->

## Skipped Findings

<!-- Valid findings that were triaged as not worth acting on. -->

| ID | Title | Severity | Category | Reason |
|----|-------|----------|----------|--------|
<!-- Categories: over-optimization, overkill, low-value-polish, linter-territory -->

## Invalid Findings

<!-- Findings that failed validation — file/line mismatch, issue misread, code already handles it, or suggestion would break things. -->

| ID | Title | Reason |
|----|-------|--------------------|

## Triage Summary
- **Total**: {{TOTAL_FINDINGS}}
- **Accepted**: {{ACCEPTED_COUNT}} ({{P1_COUNT}} P1, {{P2_COUNT}} P2, {{P3_COUNT}} P3)
- **Skipped**: {{SKIPPED_COUNT}}
- **Invalid**: {{INVALID_COUNT}}
- **Duplicates merged**: {{DUPES_MERGED}}

## Dimension Summaries

<!-- One per audited dimension, consolidated from agent outputs -->

=== DIMENSION SUMMARY ===
dimension: <name>
score: <1-10>
p1_count: <N>
p2_count: <N>
p3_count: <N>
assessment: |
  <2-3 sentences>
=== END DIMENSION SUMMARY ===
