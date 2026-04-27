---
name: quality-reviewer
description: "Review overall project quality: TSF graph coverage and content, OFT tracing gaps, evidence relevance, code best practices, and CI structure. Use when: auditing quality artifacts, reviewing traceability completeness, checking TSF graph health, assessing evidence quality, verifying project structure and CI setup. Keywords: review, audit, quality, coverage, gap, TSF graph, OFT trace, evidence, best practices, CI, lint."
---

# Quality Reviewer

Assess the overall quality posture of a tsfagent-managed project — reviewing the alignment between specification, TSF graph, implementation, evidence, and CI infrastructure.

This skill is a diagnostic tool, not a builder. It reads and analyses existing artifacts, identifies gaps and weaknesses, and produces a structured report with prioritised recommendations. It does not create or modify files.

## When to Use

- After an implementation cycle, to verify completeness before release
- When onboarding to an existing project, to understand its quality posture
- Periodically, to catch drift between specification and implementation
- When the user explicitly asks for a quality audit or review

## Handoff

### Inputs

- The complete project directory — specification, TSF statements, source code, tests, CI workflows, and configuration files

### Outputs

- A structured **quality review report** (printed to chat, not written to a file) covering all review dimensions
- Prioritised **recommendations** that can be addressed by the other skills:
  - Missing requirements → **specification-writer**
  - TSF graph gaps or weak statements → **tsf-author**
  - Missing implementations, tests, or evidence wiring → **developer**
  - STPA concerns → **stpa-analyst**

### Related Skills

This skill reviews the output of all other skills in the tsfagent framework:

1. **specification-writer** — requirements completeness and quality
2. **tsf-author** — TSF graph structure and statement quality
3. **developer** — implementation coverage, test quality, evidence wiring, git hygiene
4. **stpa-analyst** — whether hazard analysis has been performed when warranted

## Review Dimensions

The review covers six dimensions. Each produces findings rated as **gap** (missing, must fix), **weakness** (present but inadequate), or **ok** (satisfactory).

### 1. OFT Tracing Completeness

Check that every specification requirement has the coverage its `Needs:` directive demands.

**Procedure**:

1. Read `docs/specification.md` (or equivalent) and catalogue all `req~` items with their `Needs:` directives
2. Run OFT tracing (or manually scan source and test files) to find coverage markers (`[impl->...]`, `[utest->...]`, `[itest->...]`)
3. Cross-reference: for each `req~`, verify every needed artifact type has a matching marker

**Check for**:

- `req~` items with `Needs: impl` but no `[impl->...]` marker anywhere
- `req~` items with `Needs: utest` but no `[utest->...]` marker in test files
- `req~` items with `Needs: itest` but no `[itest->...]` marker
- Orphaned markers that reference non-existent `req~` IDs
- Version mismatches between markers and current `req~` versions
- Files containing markers that are not listed in `.env.oft` `OFT_FILE_PATTERNS`

### 2. TSF Graph Coverage and Structure

Check that the TSF directed acyclic graph is well-formed and that project statements adequately cover the quality hierarchy.

**Procedure**:

1. Read `.dotstop.dot` to understand the graph structure — nodes and edges
2. Run `trudag manage lint` to identify suspect links and unreviewed items
3. Run `trudag score` to see evidence propagation scores
4. Catalogue all project-level statements in `trustable/<project>/`
5. Check that each normative statement has a `_CONTEXT` companion

**Check for**:

- Suspect links flagged by `trudag manage lint` (child changed after link review)
- Unreviewed items (new statements never inspected)
- Project statements with a score of 0.0 (no evidence linked)
- Normative statements without a `_CONTEXT` companion
- Feature-aligned tenets (TT-EXPECTATIONS, TT-RESULTS, TT-CHANGES) with no project-level descendants — these should grow with functionality
- Process-aligned tenets (TT-CONFIDENCE, TT-CONSTRUCTION, TT-PROVENANCE) with no project-level descendants — these should be set up during bootstrap
- Orphaned project statements not linked to any parent in the graph

### 3. TSF Statement Quality

Assess whether TSF statements are well-written and falsifiable.

**Procedure**:

1. Read each project-level normative statement in `trustable/<project>/`
2. Evaluate against the falsifiability criteria from the tsf-author skill
3. Read each `_CONTEXT` companion and assess whether it provides actionable evidence guidance

**Check for**:

- Vague or unfalsifiable normative statements (e.g. "The system is secure" — not testable)
- Statements that are too broad (should be decomposed into sub-statements)
- `_CONTEXT` files that lack concrete evidence suggestions or don't reference `req~` IDs
- Statements that don't align with any specification requirement (disconnected quality claims)
- Statements linked to the wrong parent node (e.g. a behavioural claim under TT-CONSTRUCTION)

### 4. Evidence Relevance and Completeness

Assess whether CI evidence actually proves what the TSF statements claim.

**Procedure**:

1. Read `.github/workflows/release.yaml` (or equivalent CI workflow)
2. Catalogue all `tsffer` steps — their reference types, referenced files/URLs, and `asset_tsf_ids`
3. For each TSF project statement, check whether it has at least one `tsffer` step targeting it
4. Evaluate whether the evidence type and content actually support the claim

**Check for**:

- TSF statements with no `tsffer` evidence step at all
- Evidence that is irrelevant to the claim (e.g. a file reference to an unrelated source file)
- `openfasttrace` references using versioned requirement IDs (should be bare, e.g. `req~name` not `req~name~1`)
- Missing `tsflink` step (should run after all `tsffer` steps)
- Evidence plans in `_CONTEXT` files that are not reflected in actual CI steps

### 5. Code and Test Quality

Assess whether the implementation follows best practices for the project's programming environment.

**Procedure**:

1. Identify the project's primary language(s) and framework(s) from source files
2. Review source code for idiomatic structure, naming, and error handling
3. Review test files for meaningful assertions (not just smoke tests)
4. Check git history for commit hygiene if accessible

**Check for**:

- Tests that don't actually assert the behaviour specified by the requirement (rubber-stamp tests)
- OFT markers placed far from the code they supposedly cover
- Dead code or commented-out code with OFT markers (false coverage)
- Missing error handling at system boundaries
- Non-idiomatic patterns for the language/framework in use
- Large, unfocused commits or commits without requirement references

### 6. Project Structure and CI

Assess whether the overall project layout and CI pipeline follow tsfagent conventions.

**Procedure**:

1. Check for the expected file layout (see copilot-instructions.md)
2. Verify CI workflow structure — OFT trace step, tsffer steps, tsflink step, in correct order
3. Check that `.env.oft` `OFT_FILE_PATTERNS` covers all directories containing OFT markers

**Check for**:

- Missing expected files (`.env.oft`, `.dotstop.dot`, `docs/specification.md`, CI workflow)
- `.env.oft` patterns that don't cover directories containing markers
- CI workflow that runs `tsflink` before all `tsffer` steps
- CI workflow missing the OFT tracing step
- TSF statements directory not matching the project prefix convention
- Submodule directories (`tsftemplate/`, `tsfagent/`) that have been modified (should be read-only)

## Procedure

### Running a Full Quality Review

1. **Gather context.** Read the following files and directories to build a complete picture:
   - `docs/specification.md` — all `req~` items
   - `trustable/<project>/` — all TSF statement files
   - `.dotstop.dot` — graph structure
   - `.env.oft` — OFT configuration
   - `.github/workflows/` — CI pipeline
   - Source and test directories — implementation files with OFT markers

2. **Run tool checks** (if trudag and OFT are available):
   ```bash
   trudag manage lint
   trudag score
   ```

3. **Evaluate each dimension.** Work through the six review dimensions above, collecting findings.

4. **Produce the report.** Present findings in the structured format below.

### Report Format

```markdown
# Quality Review: <Project Name>

## Summary

| Dimension | Status | Findings |
|-----------|--------|----------|
| OFT Tracing | 🟢/🟡/🔴 | <one-line summary> |
| TSF Graph Coverage | 🟢/🟡/🔴 | <one-line summary> |
| TSF Statement Quality | 🟢/🟡/🔴 | <one-line summary> |
| Evidence Relevance | 🟢/🟡/🔴 | <one-line summary> |
| Code & Test Quality | 🟢/🟡/🔴 | <one-line summary> |
| Project Structure & CI | 🟢/🟡/🔴 | <one-line summary> |

## Detailed Findings

### OFT Tracing
<gaps, weaknesses, and ok items>

### TSF Graph Coverage
<gaps, weaknesses, and ok items>

### TSF Statement Quality
<gaps, weaknesses, and ok items>

### Evidence Relevance
<gaps, weaknesses, and ok items>

### Code & Test Quality
<gaps, weaknesses, and ok items>

### Project Structure & CI
<gaps, weaknesses, and ok items>

## Recommendations

Prioritised list of actions, each tagged with the skill that should address it:

1. **[gap]** <description> → **<skill-name>**
2. **[weakness]** <description> → **<skill-name>**
...
```

Use status indicators:
- 🟢 **Green**: All checks pass, no issues found
- 🟡 **Yellow**: Minor weaknesses or non-critical gaps
- 🔴 **Red**: Critical gaps that break traceability or leave quality claims unproven

### Running a Focused Review

When asked to review a specific dimension (e.g. "check my OFT coverage"), run only that section of the procedure and report findings for that dimension alone. Use the same finding format (gap/weakness/ok) but skip the full summary table.

## Quality Guidance

- **Be specific.** Don't say "some requirements lack coverage" — list the exact `req~` IDs.
- **Be actionable.** Every finding should have a clear next step and the skill that handles it.
- **Prioritise.** Gaps (missing, broken) before weaknesses (present but inadequate) before suggestions.
- **Don't fix, report.** This skill identifies problems; the other skills fix them. The user decides what to act on.
- **Respect scope.** Only review artifacts within the project directory. Do not evaluate upstream/submodule quality — those are maintained externally.
