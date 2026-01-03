---
description: Perform a non-destructive cross-artifact consistency and quality analysis across spec.md, plan.md, and tasks.md after task generation, including quantitative metrics validation for success criteria.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Identify inconsistencies, duplications, ambiguities, and underspecified items across the three core artifacts (`spec.md`, `plan.md`, `tasks.md`) before implementation. This command MUST run only after `/speckit.tasks` has successfully produced a complete `tasks.md`.

The analysis includes validation of success criteria to ensure they include proper quantitative metrics (time, performance, volume) and qualitative measures (user satisfaction, task completion), while remaining technology-agnostic and user-focused.

## Operating Constraints

**STRICTLY READ-ONLY**: Do **not** modify any files. Output a structured analysis report. Offer an optional remediation plan (user must explicitly approve before any follow-up editing commands would be invoked manually).

**Constitution Authority**: The project constitution (`/memory/constitution.md`) is **non-negotiable** within this analysis scope. Constitution conflicts are automatically CRITICAL and require adjustment of the spec, plan, or tasks—not dilution, reinterpretation, or silent ignoring of the principle. If a principle itself needs to change, that must occur in a separate, explicit constitution update outside `/speckit.analyze`.

## Execution Steps

### 1. Initialize Analysis Context

Run `{SCRIPT}` once from repo root and parse JSON for FEATURE_DIR and AVAILABLE_DOCS. Derive absolute paths:

- SPEC = FEATURE_DIR/spec.md
- PLAN = FEATURE_DIR/plan.md
- TASKS = FEATURE_DIR/tasks.md

Abort with an error message if any required file is missing (instruct the user to run missing prerequisite command).
For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

### 2. Load Artifacts (Progressive Disclosure)

Load only the minimal necessary context from each artifact:

**From spec.md:**

- Overview/Context
- Functional Requirements
- Non-Functional Requirements
- User Stories
- Success Criteria
- Edge Cases (if present)

**From plan.md:**

- Architecture/stack choices
- Data Model references
- Phases
- Technical constraints

**From tasks.md:**

- Task IDs
- Descriptions
- Phase grouping
- Parallel markers [P]
- Referenced file paths

**From constitution:**

- Load `/memory/constitution.md` for principle validation

### 3. Build Semantic Models

Create internal representations (do not include raw artifacts in output):

- **Requirements inventory**: Each functional + non-functional requirement with a stable key (derive slug based on imperative phrase; e.g., "User can upload file" → `user-can-upload-file`)
- **User story/action inventory**: Discrete user actions with acceptance criteria
- **Success criteria inventory**: Extract all success criteria with their measurable outcomes
- **Task coverage mapping**: Map each task to one or more requirements or stories (inference by keyword / explicit reference patterns like IDs or key phrases)
- **Constitution rule set**: Extract principle names and MUST/SHOULD normative statements

### 4. Detection Passes (Token-Efficient Analysis)

Focus on high-signal findings. Limit to 50 findings total; aggregate remainder in overflow summary.

#### A. Duplication Detection

- Identify near-duplicate requirements
- Mark lower-quality phrasing for consolidation

#### B. Ambiguity Detection

- Flag vague adjectives (fast, scalable, secure, intuitive, robust) lacking measurable criteria
- Flag unresolved placeholders (TODO, TKTK, ???, `<placeholder>`, etc.)

#### C. Underspecification

- Requirements with verbs but missing object or measurable outcome
- User stories missing acceptance criteria alignment
- Tasks referencing files or components not defined in spec/plan

#### D. Constitution Alignment

- Any requirement or plan element conflicting with a MUST principle
- Missing mandated sections or quality gates from constitution

#### E. Coverage Gaps

- Requirements with zero associated tasks
- Tasks with no mapped requirement/story
- Non-functional requirements not reflected in tasks (e.g., performance, security)

#### F. Inconsistency

- Terminology drift (same concept named differently across files)
- Data entities referenced in plan but absent in spec (or vice versa)
- Task ordering contradictions (e.g., integration tasks before foundational setup tasks without dependency note)
- Conflicting requirements (e.g., one requires Next.js while other specifies Vue)

#### G. Quantitative Metrics Validation

Validate success criteria for proper quantitative analysis:

- **Missing Measurements**: Success criteria lacking specific quantitative metrics (numbers, percentages, time values, counts)
  - Flag criteria with vague terms: "fast", "quickly", "efficiently", "many", "few", "some"
  - Flag criteria missing numeric thresholds: "better performance", "improved user experience"
  
- **Technology Leakage**: Success criteria mentioning implementation details
  - Flag technology-specific metrics: API response times, database TPS, framework-specific measures
  - Flag infrastructure references: server names, service endpoints, cache hit rates
  - Recommend user-facing metric alternatives
  
- **Unverifiable Criteria**: Success criteria that cannot be objectively tested
  - Flag subjective statements without measurement methods: "users will be happy", "system is intuitive"
  - Flag criteria missing verification approach: no mention of how to measure the outcome
  
- **Missing Quantitative Balance**: Analyze the distribution of quantitative vs. qualitative criteria
  - Warn if all criteria are qualitative (lacking measurable numbers)
  - Warn if all criteria are purely technical (lacking user-focused outcomes)
  - Recommend including both quantitative metrics and qualitative measures

**Good quantitative criteria patterns to recognize:**

- Time-based: "Users can complete X in under Y minutes/seconds"
- Volume-based: "System handles N concurrent users/requests"
- Percentage-based: "X% of users successfully complete Y on first attempt"
- Rate-based: "Z operations per second/minute/hour"
- Improvement-based: "Reduce X by Y%" (with baseline context)

### 5. Severity Assignment

Use this heuristic to prioritize findings:

- **CRITICAL**: Violates constitution MUST, missing core spec artifact, or requirement with zero coverage that blocks baseline functionality
- **HIGH**: Duplicate or conflicting requirement, ambiguous security/performance attribute, untestable acceptance criterion, success criteria with technology leakage or missing all quantitative metrics
- **MEDIUM**: Terminology drift, missing non-functional task coverage, underspecified edge case, success criteria with vague measurements or unverifiable outcomes
- **LOW**: Style/wording improvements, minor redundancy not affecting execution order, minor imbalance in quantitative vs. qualitative criteria

### 6. Produce Compact Analysis Report

Output a Markdown report (no file writes) with the following structure:

## Specification Analysis Report

| ID | Category    | Severity | Location(s)          | Summary                                  | Recommendation                         |
|----|-------------|----------|----------------------|------------------------------------------|----------------------------------------|
| A1 | Duplication | HIGH     | spec.md:L120-134     | Two similar requirements ...             | Merge phrasing; keep clearer version   |

(Add one row per finding; generate stable IDs prefixed by category initial.)

**Coverage Summary Table:**

| Requirement Key | Has Task? | Task IDs | Notes |
|-----------------|-----------|----------|-------|

**Success Criteria Quality Analysis:**

| Criterion ID | Has Quantitative Metric? | Technology-Agnostic? | User-Focused? | Issues |
|--------------|--------------------------|----------------------|---------------|--------|

**Constitution Alignment Issues:** (if any)

**Unmapped Tasks:** (if any)

**Metrics:**

- Total Requirements
- Total Tasks
- Total Success Criteria
- Coverage % (requirements with >=1 task)
- Success Criteria Quality:
  - Count with quantitative metrics
  - Count with qualitative measures only
  - Count with technology leakage
  - Count with missing measurements
- Ambiguity Count
- Duplication Count
- Critical Issues Count

### 7. Provide Next Actions

At end of report, output a concise Next Actions block:

- If CRITICAL issues exist: Recommend resolving before `/speckit.implement`
- If only LOW/MEDIUM: User may proceed, but provide improvement suggestions
- Provide explicit command suggestions: e.g., "Run /speckit.specify with refinement", "Run /speckit.plan to adjust architecture", "Manually edit tasks.md to add coverage for 'performance-metrics'"

### 8. Offer Remediation

Ask the user: "Would you like me to suggest concrete remediation edits for the top N issues?" (Do NOT apply them automatically.)

## Operating Principles

### Context Efficiency

- **Minimal high-signal tokens**: Focus on actionable findings, not exhaustive documentation
- **Progressive disclosure**: Load artifacts incrementally; don't dump all content into analysis
- **Token-efficient output**: Limit findings table to 50 rows; summarize overflow
- **Deterministic results**: Rerunning without changes should produce consistent IDs and counts

### Analysis Guidelines

- **NEVER modify files** (this is read-only analysis)
- **NEVER hallucinate missing sections** (if absent, report them accurately)
- **Prioritize constitution violations** (these are always CRITICAL)
- **Use examples over exhaustive rules** (cite specific instances, not generic patterns)
- **Report zero issues gracefully** (emit success report with coverage statistics)

## Context

{ARGS}
