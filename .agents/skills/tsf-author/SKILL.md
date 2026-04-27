---
name: tsf-author
description: "Author Trustable Software Framework (TSF) quality statements and evidence strategies. Use when: defining TSF quality goals, creating project-level TSF statements, breaking down goals into falsifiable sub-statements, planning evidence for TSF statements, writing normative/context statement pairs, scoring TSF nodes. Keywords: TSF, trudag, trustable, quality goal, evidence, normative, statement, .dotstop, tsftemplate, tsffer, tsflink, falsifiable."
---

# TSF Author

Create and maintain Trustable Software Framework (TSF) quality statements — from high-level quality goals through falsifiable sub-statements, with evidence strategies that prove each claim.

## When to Use

- Defining new TSF quality goals for a project
- Breaking down quality goals into falsifiable, evidence-backed statements
- Creating project-level TSF statement files in `trustable/<project>/`
- Planning what evidence is needed to prove a TSF statement
- Writing normative/context statement pairs
- Reviewing or improving existing TSF statements

## Handoff

### Inputs

- `req~` identifiers from the specification — align TSF statements with these to ensure quality goals map to implementable requirements
- Existing TSF graph structure (`.dotstop.dot`, upstream statements)

### Outputs

- TSF statement IDs (`PREFIX-DESCRIPTOR`) — consumed by **developer** to wire `tsffer` CI evidence steps
- `_CONTEXT` evidence plans — consumed by **developer** to know what artifacts to produce and which `req~` IDs to reference via `openfasttrace` evidence type
- Graph links (parent → child) in `.dotstop.dot`

### Related Skills

This skill is the second step in the tsftemplate triad:

1. **specification-writer** decomposes user goals into `req~` items
2. **tsf-author** (this skill) creates TSF quality statements and evidence strategies
3. **developer** implements requirements, adds OFT markers, and wires CI evidence

## TSF Statement Concepts

### Statement Hierarchy

TSF organizes quality claims in a directed acyclic graph (DAG):

```
TRUSTABLE-SOFTWARE (root)
  └─ TT-EXPECTATIONS (TSF Tenet — abstract quality pillar)
       └─ TA-BEHAVIOURS (Trustable Assertion — mid-level claim)
            └─ ECLIPSE-PROJECT_SCOPE (Upstream — process/standard demand)
                 └─ MYPROJECT-FEATURE_SPEC (Project — concrete, provable claim)
```

Each layer increases specificity. **Project-level statements** are the leaf nodes where evidence is attached.

### TSF Tenet Branches

The root `TRUSTABLE-SOFTWARE` node has six tenet branches (TT-*). These divide into two categories that require very different treatment:

#### Feature-aligned tenets (evolve with project functionality)

These tenets are closely tied to what the software *does*. Project statements under them grow as features are added and must be updated when behaviour changes.

| Tenet | Statement | Child Assertions | What It Covers |
|-------|-----------|-----------------|----------------|
| **TT-EXPECTATIONS** | "Documentation specifying what XYZ is expected to do, and what it must not do, and how this is verified." | TA-BEHAVIOURS, TA-CONSTRAINTS, TA-INDICATORS, TA-MISBEHAVIOURS | Specified behaviours, constraints on deployment, misbehaviour identification (STPA analysis fits here), monitoring indicators |
| **TT-RESULTS** | "Evidence that XYZ does what it is supposed to do, and does not do what it must not do." | TA-ANALYSIS, TA-DATA, TA-VALIDATION | Test data collection, analysis of results, validation through systematic testing |
| **TT-CHANGES** | "XYZ is actively maintained, with regular updates and regression prevention." | TA-FIXES, TA-UPDATES | Bug triage and fixes, dependency and configuration updates under change control |

**Guidance**: When adding a new feature or requirement, consider whether it needs project statements under TT-EXPECTATIONS (specifying the behaviour) and TT-RESULTS (proving the behaviour). STPA-derived constraints from the stpa-analyst skill typically map to TA-MISBEHAVIOURS (under TT-EXPECTATIONS) and TA-VALIDATION (under TT-RESULTS).

#### Process-aligned tenets (set up once, stable across features)

These tenets describe *how* the software is built, managed, and assured. Project statements under them are typically set up during project bootstrap and change only when the development process, tooling, or supply chain changes — not when features are added.

| Tenet | Statement | Child Assertions | What It Covers |
|-------|-----------|-----------------|----------------|
| **TT-CONFIDENCE** | "Confidence is achieved by measuring and analysing behaviour and evidence over time." | TA-CONFIDENCE, TA-METHODOLOGIES | SME confidence scoring, development processes (code review, commit records, security policy, contribution process) |
| **TT-CONSTRUCTION** | "Tools are provided to build XYZ from trusted sources with full reproducibility." | TA-ITERATIONS, TA-RELEASES, TA-TESTS | Build reproducibility, release artifacts, test environments, version control |
| **TT-PROVENANCE** | "All inputs and attestations are provided with known provenance." | TA-INPUTS, TA-SUPPLY_CHAIN | IP compliance, SBOM generation, dependency mirroring, supply chain controls |

**Guidance**: These branches align primarily with the project's CI/CD setup, build tooling, and organisational processes. The Eclipse upstream statements (ECLIPSE-*) typically sit beneath the TA-assertions in these branches — covering code review, commit records, security policy, build instructions, formal releases, etc. Project statements here prove that the project follows these processes.

### Choosing the Right Parent Node

When creating a new project statement, link it to the most specific existing node that it supports:

| Your statement is about... | Link to parent |
|---------------------------|---------------|
| A specified feature behaviour or capability | TA-BEHAVIOURS → via an ECLIPSE-* node if one exists, or directly |
| A constraint on how the software may be used/deployed | TA-CONSTRAINTS |
| Monitoring or early warning of misbehaviour | TA-INDICATORS |
| Identifying and mitigating prohibited behaviours (STPA) | TA-MISBEHAVIOURS |
| Test results, coverage reports | TA-DATA or TA-VALIDATION |
| Analysis of test or monitoring data | TA-ANALYSIS |
| Bug fixes or vulnerability patches | TA-FIXES |
| Dependency or configuration updates | TA-UPDATES |
| Build process or reproducibility | TA-RELEASES or TA-ITERATIONS |
| Development methodology or process compliance | TA-METHODOLOGIES |
| IP, licensing, SBOM | TA-INPUTS |
| Supply chain or source mirroring | TA-SUPPLY_CHAIN |
| SME confidence assessment | TA-CONFIDENCE |

### Statement Types

| Type | `normative` | Purpose | File naming |
|------|-------------|---------|-------------|
| **Normative statement** | `true` | Falsifiable claim that must be proven | `PREFIX-DESCRIPTOR.md` |
| **Context companion** | `false` | Rationale, guidance, evidence suggestions | `PREFIX-DESCRIPTOR_CONTEXT.md` |

Every normative statement MUST have a context companion. The context companion explains the *why* behind the statement and specifies what kinds of evidence would satisfy it — this is essential for guiding evidence collection in CI workflows.

## TSF Statement File Format

### Normative Statement

```markdown
---
normative: true

publish:
    group: "<Report Section>"
#EVIDENCE_REF#
score: 
    <ScorerName>: <0.0-1.0>
---

<Falsifiable claim text — a concise, verifiable assertion about the project.>
```

### Context Companion (Required)

The `_CONTEXT` file is mandatory for every normative statement. It provides rationale and — critically — specifies what evidence would prove the statement is met.

```markdown
---
normative: false
---

**Guidance**

<Why this statement matters. What it means in practice. Scope and assumptions.>

**Evidence**

Evidence for compliance might include (but is not limited to):

* <Specific artifact type 1 and what it demonstrates>
* <Specific artifact type 2 and what it demonstrates>
* <Specific artifact type 3 and what it demonstrates>

**Confidence scoring**

<Rationale for the SME score assigned to the normative statement.>
```

The **Evidence** section is the bridge between the abstract claim and the concrete CI workflow that will collect proof. Each evidence suggestion should describe both *what* the artifact is and *what it demonstrates* about the statement.

### Frontmatter Fields

| Field | Required | Values | Purpose |
|-------|----------|--------|---------|
| `normative` | Yes | `true` / `false` | Whether this is a binding assertion or informational context |
| `publish.group` | For normative | String (e.g. `"Eclipse Process"`, `"Security"`, `"Testing"`) | Groups statement in the TSF report |
| `#EVIDENCE_REF#` | For normative | Placeholder literal | `tsflink` replaces this with actual evidence links during CI |
| `score.<Name>` | For normative | `0.0` – `1.0` | SME confidence that proving this statement contributes to the parent quality goal |

### Score Semantics

- `1.0`: Fully proves the parent claim when evidence is provided
- `0.7–0.9`: Strongly supports, but other factors also matter
- `0.3–0.6`: Partial contribution; other statements cover remaining aspects
- `< 0.3`: Minor or indirect contribution

## Statement Naming & File Placement

### ID Convention

- Format: `PREFIX-DESCRIPTOR`
- `PREFIX`: project name in UPPERCASE (e.g., `TSFTEMPLATE`)
- `DESCRIPTOR`: UPPERCASE with underscores, concise label (e.g., `PROJECT_README`, `SECURITY_POLICY`, `BUILD_REPRODUCIBILITY`)
- File name: `PREFIX-DESCRIPTOR.md` in `trustable/<project>/`

### trudag Commands

#### Create a statement

```bash
trudag manage create-item <PREFIX> <DESCRIPTOR> <directory>
# Example:
trudag manage create-item TSFTEMPLATE BUILD_REPRODUCIBILITY trustable/tsftemplate
# Creates: trustable/tsftemplate/TSFTEMPLATE-BUILD_REPRODUCIBILITY.md
```

After creation, populate the frontmatter and write the statement text.

#### Add an existing statement file to the graph

```bash
trudag manage add-item /path/to/ITEM.md
```

#### Link a parent statement to a child (record reasoning)

```bash
trudag manage create-link <PARENT-ID> <CHILD-ID>
# Example: link an upstream Eclipse statement to a project statement
trudag manage create-link UPSTREAM.ECLIPSE.ECLIPSE-BUILD_INSTRUCTIONS TSFTEMPLATE-BUILD_REPRODUCIBILITY
```

This updates `.dotstop.dot` automatically. The parent-child direction means "the parent's claim is supported by the child."

#### Review items (mark as reviewed after inspection)

```bash
trudag manage set-item <ITEM-ID>
# Review item and all its links:
trudag manage set-item <ITEM-ID> --links
```

#### Clear suspect links (after re-evaluating a changed item)

```bash
trudag manage set-link <PARENT-ID> <CHILD-ID>
```

#### Inspect and lint

```bash
trudag manage show-item <ITEM-ID>    # Inspect a specific item
trudag manage show-link <PARENT> <CHILD>  # Inspect a specific link
trudag manage lint                    # Show suspect links and unreviewed items
trudag plot                           # Visualize the entire graph (SVG)
```

#### Score and publish

```bash
trudag score      # Calculate scores recursively from evidence
trudag publish    # Generate markdown report for mkdocs
```

## Writing Falsifiable Statements

A good TSF statement is a **concise, verifiable assertion** about what the project does or provides. Apply these criteria:

### Statement Quality Rules

1. **Falsifiable**: It must be possible to determine with evidence whether the claim is true or false
2. **Singular**: One provable claim per statement; split compound claims into separate nodes
3. **Concrete**: Avoid vague qualifiers ("properly", "adequately", "as needed")
4. **Scoped**: State what is included and what is not
5. **Evidence-aware**: Write the statement with a clear idea of what proof looks like

### Good vs Bad Examples

| Quality | Statement | Problem |
|---------|-----------|---------|
| Good | "The project performs automated dependency vulnerability scanning on every CI run." | Clear, verifiable, evidence is obvious |
| Good | "All test environments are constructed from version-controlled configuration and are reproducible." | Scoped, falsifiable, can be proven with artifacts |
| Bad | "The project handles security well." | Vague — what does "well" mean? Not falsifiable |
| Bad | "Code quality is ensured through best practices." | No specifics, no measurable outcome |

### Breaking Down Goals

When a quality goal is too broad for a single statement, decompose it:

1. **Identify the abstract goal** (e.g., "The project is secure")
2. **List concrete, observable aspects** that contribute to that goal:
   - Vulnerability scanning is automated
   - Security policy exists and is followed
   - Dependencies are pinned and reviewed
   - Secrets are not committed to the repository
3. **Write one TSF statement per aspect**, each with its own evidence strategy
4. **Link statements** to their parent in the TSF graph via trudag

## Evidence Planning

When creating a TSF statement, simultaneously plan how it will be proven. This is the critical connection between claims and trust. Evidence should be described in the context companion and later implemented in the CI workflow.

### Evidence Types

TSF supports these evidence reference types (via Python renderers provided in `.dotstop_extensions/` in submodule `tsftemplate`):

| Type | Use Case | Example |
|------|----------|---------|
| `github` | File or file section in the repository | SECURITY.md, CI config, specific code lines |
| `download_url` | Release artifact accessible via URL | Uploaded README, test report, SBOM |
| `webpage` | External reference or policy page | Eclipse Foundation Security Policy, standards docs |
| `openfasttrace` | OFT requirement tracing result | `req~security-policy` tracing status |

It is possible/desirable to extend this list of evidence renderers according to project needs.

### Choosing Evidence for a Statement

Match statement type to appropriate evidence:

| Statement About | Suitable Evidence |
|----------------|-------------------|
| Documentation exists | `github` reference to the file + `download_url` of release artifact |
| Process is followed | `webpage` reference to process definition + `github` reference to configuration enforcing it |
| Code behavior | `openfasttrace` requirement trace (links spec → impl → test) |
| External compliance | `webpage` reference to standard + `github` reference to project's compliance artifact |
| CI/CD practice | `github` reference to workflow file + `download_url` of generated report |
| Test coverage | `openfasttrace` trace + `download_url` of test/coverage report |

### Evidence Strength

Not all evidence is equally convincing. Prefer stronger forms:

1. **Strongest**: Automated, machine-verifiable (CI test results, OFT traces, SBOM scans)
2. **Strong**: Auditable artifacts (signed release assets, commit history, review records)
3. **Moderate**: Documented references (policy pages, configuration files)
4. **Weak**: Self-attestation without supporting artifacts

A single statement may have multiple evidence items. Combining evidence types strengthens the claim — for example, pairing an OFT requirement trace (proves implementation exists) with a test report download (proves it passes).

### Recording Evidence in CI

Evidence is recorded in CI workflows using `tsffer` GitHub action steps. The **developer** skill contains the full set of tsffer CI templates (github, openfasttrace, download_url, webpage, multi-reference). When authoring evidence plans in `_CONTEXT` files, describe *what* evidence is needed and *why*, then let the developer skill handle the CI implementation details.

## Procedure

### Creating a New Quality Goal with Statements

1. **Read the existing TSF graph.** Review `trustable/<project>/` for current statements and `.dotstop.dot` for the graph structure. Understand what parent nodes exist upstream.
2. **Define the quality goal.** Write a one-sentence description of what you want to prove about the project (e.g., "The project manages dependency security proactively").
3. **Decompose into falsifiable statements.** Break the goal into 2–5 concrete, singular claims that together cover the goal. Each must be independently provable.
4. **For each statement:**
   a. Create the statement file via `trudag manage create-item PREFIX DESCRIPTOR trustable/<project>/`
   b. Write the normative frontmatter (`normative: true`, `publish.group`, `#EVIDENCE_REF#`, `score`)
   c. Write the falsifiable claim text
   d. Create a `_CONTEXT` companion with guidance, rationale, and concrete evidence suggestions — each evidence item should describe the artifact type and what it proves
5. **Plan evidence.** For each statement, identify at least one evidence item from the evidence types table and note what CI workflow steps would produce or reference it.
6. **Link in the graph.** Use `trudag manage create-link <PARENT-ID> <CHILD-ID>` to connect new statement nodes to the appropriate parent in the TSF DAG.
7. **Review.** Run `trudag manage lint` to check for suspect links or unreviewed items.

### Improving Existing Statements

1. Read the current statement and its context companion (if any).
2. Check against the statement quality rules — is it falsifiable, singular, concrete, scoped?
3. If the statement is compound, propose splitting into multiple statements.
4. If evidence strategy is missing or unclear, create or update the `_CONTEXT` companion.
5. Verify the score reflects actual contribution to the parent goal.

## Quality Checklist

Before finalizing a TSF statement, verify:

- [ ] `normative: true` is set in frontmatter
- [ ] `publish.group` is specified and matches the reporting category
- [ ] `#EVIDENCE_REF#` placeholder is present (for tsflink injection)
- [ ] `score` has at least one SME assessment between 0.0 and 1.0
- [ ] Statement text is a single, falsifiable assertion
- [ ] Statement avoids vague qualifiers ("properly", "adequate", "good")
- [ ] A `_CONTEXT` companion file exists with guidance and concrete evidence suggestions
- [ ] Each evidence suggestion in `_CONTEXT` describes the artifact AND what it proves
- [ ] At least one concrete evidence type has been identified
- [ ] File name matches `PREFIX-DESCRIPTOR.md` convention
- [ ] Statement is linked to an appropriate parent node via `trudag manage create-link`
- [ ] `trudag manage lint` reports no suspect links for this statement

## Example: Defining a "Dependency Security" Quality Goal

### Goal Decomposition

Quality goal: "The project manages dependency security proactively."

Decomposed statements:

1. **MYPROJECT-DEPENDENCY_SCANNING** — Automated CVE scanning runs on every CI build
2. **MYPROJECT-DEPENDENCY_PINNING** — All dependencies are pinned to specific versions
3. **MYPROJECT-DEPENDENCY_UPDATES** — Dependencies are reviewed and updated within 30 days of a known vulnerability

### Statement File: MYPROJECT-DEPENDENCY_SCANNING.md

```markdown
---
normative: true

publish:
    group: "Security"
#EVIDENCE_REF#
score: 
    JaneDoe: 0.8
---

The project performs automated dependency vulnerability scanning as part of every CI pipeline run, and build results are published as release artifacts.
```

### Context File: MYPROJECT-DEPENDENCY_SCANNING_CONTEXT.md

```markdown
---
normative: false
---

**Guidance**

Automated dependency scanning ensures that known vulnerabilities in third-party libraries are detected early, before they reach production. This is a foundational practice for supply chain security.

The scanning tool should cover both direct and transitive dependencies, and results should be available for audit as part of each release.

**Evidence**

Evidence for this statement could include:

- `github` reference to the CI workflow file configuring the scanning step
- `download_url` reference to the scan report published as a release artifact
- `openfasttrace` trace linking `req~dependency-scanning` to its implementation and tests

**Confidence scoring**

Score of 0.8 reflects that automated scanning is a strong but not complete measure — it depends on scanner coverage and database freshness. Manual review of flagged issues is captured in a separate statement.
```