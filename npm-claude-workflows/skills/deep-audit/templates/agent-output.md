# <Agent Name> — Audit Output

## Findings

=== FINDING ===
agent: <agent-name>
severity: P1|P2|P3
confidence: <80-100>
file: <path>
line: <number or range>
dimension: <dimension name>
title: <one-line title>
description: |
  <2-4 sentences explaining the issue>
suggestion: |
  <specific fix with code snippet if applicable>
=== END FINDING ===

<!-- Repeat for each finding (no particular order required) -->

## Dimension Summaries

=== DIMENSION SUMMARY ===
dimension: <name>
score: <1-10>
p1_count: <N>
p2_count: <N>
p3_count: <N>
assessment: |
  <2-3 sentence assessment of this dimension>
=== END DIMENSION SUMMARY ===

<!-- One per dimension covered by this agent -->
