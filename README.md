# tsfagent

An AI agent framework for building trustable software projects from the ground up — from a user's high-level goal through formal specification, quality assurance, and traceable implementation.

## The Idea

Writing high-quality software means more than working code. It means traceable requirements, provable quality claims, tested implementations, and auditable evidence that ties everything together. Doing this manually is tedious and error-prone; the overhead discourages adoption of the very practices that build trust.

**tsfagent** automates this workflow using a set of specialized AI agents. You describe what you want to build, and the agents handle the engineering rigor:

1. **Decompose** your goal into formal, traceable specification requirements
2. **Define** quality goals as falsifiable claims in the Trustable Software Framework (TSF)
3. **Implement** code and tests with full traceability markers
4. **Wire up** CI evidence so every quality claim is backed by proof

The result is a project where every line of code traces back to a requirement, every requirement connects to a quality goal, and every quality goal has machine-verifiable evidence.

## Components

### Agent

| Agent | Purpose |
|-------|---------|
| **project-manager** | Orchestrates the full workflow — takes user goals and drives them through specification, TSF authoring, and implementation |

### Skills

| Skill | Purpose |
|-------|---------|
| **specification-writer** | Decomposes user goals into formal OFT requirements (`req~` items) with BCP14 keywords and tracing directives |
| **tsf-author** | Creates TSF quality statements aligned with requirements — falsifiable claims with evidence strategies |
| **developer** | Implements requirements as code and tests, adds OFT coverage markers, wires `tsffer` CI evidence steps |
| **stpa-analyst** | Performs System-Theoretic Process Analysis on project code/architecture, deriving safety constraints that feed into the other three skills |
| **quality-reviewer** | Audits overall project quality — TSF graph coverage, OFT tracing gaps, evidence relevance, code practices, CI structure |
### Prompts

| Prompt | Purpose |
|--------|---------|
| **bootstrap-project** | Sets up a new project with tsftemplate scaffolding — directories, CI workflow, OFT config, TSF graph |

### Template

The [tsftemplate](https://github.com/AnotherDaniel/tsftemplate) submodule provides the quality infrastructure:

- TSF statement hierarchy and upstream quality framework (Eclipse, Trustable tenets)
- OpenFastTrace (OFT) configuration and requirement tracing setup
- CI workflow templates with `tsffer` evidence recording and `tsflink` report generation
- mkdocs-based documentation and TSF report publishing

## How It Works

```
User Goal
    │
    ▼
┌──────────────────┐
│  project-manager │  Orchestrates the pipeline
└────────┬─────────┘
         │
         ├──────────────────────────┐
         │                          ▼
         │                   ┌─────────────┐
         │                   │  stpa-      │  Hazard analysis
         │                   │  analyst    │  (optional)
         │                   └──────┬──────┘
         │                          │ constraints
    ┌────┴────┐                     │
    ▼         ▼                     ▼
┌────────┐ ┌──────────┐
│  spec- │ │   tsf-   │    Requirements + quality goals
│ writer │ │  author  │     (informed by STPA if used)
└───┬────┘ └────┬─────┘
    │           │
    └─────┬─────┘
          ▼
    ┌───────────┐
    │ developer │  Implements, tests, wires evidence
    └───────────┘
          │
          ▼
    ┌─────────────────┐
    │ quality-reviewer │  Audits quality posture (optional)
    └─────────────────┘
          │
          ▼
    Traceable, quality-assured project
```

### Workflow in Detail

1. **You describe a goal** — e.g., "Build a CLI tool that scans project dependencies for known CVEs and reports them"

2. **project-manager** analyzes the goal and plans the work:
   - What specification requirements are needed
   - What quality claims should be made
   - What implementation and testing is required
   - Whether an STPA hazard analysis would add value (recommended for safety/reliability-critical systems)

3. **(Optional) stpa-analyst** performs a System-Theoretic Process Analysis:
   - Models the system's control structure (controllers, actions, feedback)
   - Identifies losses, hazards, and unsafe control actions
   - Derives safety constraints that become candidate requirements and quality goals
   - Produces `docs/stpa-analysis.md` as a structured analysis document

4. **specification-writer** decomposes the goal into formal `req~` items:
   - Each requirement is singular, falsifiable, and uses BCP14 keywords (*MUST*, *SHOULD*, etc.)
   - Tracing directives (`Needs: impl, utest`) define what coverage is expected
   - Requirements are placed in the project's specification document

5. **tsf-author** creates TSF quality statements:
   - Maps requirements to quality goals in the TSF hierarchy
   - Writes falsifiable normative statements with `_CONTEXT` companions
   - Plans what evidence will prove each claim (file references, OFT traces, test reports)
   - Links statements into the TSF directed acyclic graph via `trudag`

6. **developer** implements everything:
   - Writes production code with `[impl->req~name~version]` OFT markers
   - Writes tests with `[utest->req~name~version]` markers
   - Adds `tsffer` CI workflow steps to record evidence for TSF statements
   - Ensures `.env.oft` covers all relevant file patterns

7. **The CI pipeline** (from tsftemplate) runs OFT tracing, records evidence via `tsffer`, links it to TSF statements via `tsflink`, and publishes a trust report.

8. **(Optional) quality-reviewer** audits the project's quality posture:
   - Checks OFT tracing completeness — every `req~` item has the coverage its `Needs:` directive demands
   - Reviews TSF graph structure — lint, scores, orphaned or unlinked statements
   - Assesses TSF statement quality — falsifiability, evidence plans, parent-node alignment
   - Verifies evidence relevance — CI evidence actually proves what the statements claim
   - Reviews code and test quality for language/framework best practices
   - Produces a prioritised report with recommendations mapped to the responsible skill

## Getting Started

### Prerequisites

- [VS Code](https://code.visualstudio.com/) with [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) (agent mode)
- [OpenFastTrace](https://github.com/itsallcode/openfasttrace) — requirements tracing tool
- [trudag](https://github.com/AnotherDaniel/trudag) — TSF graph management CLI
- Git

### Setting Up a New Project

You can bootstrap a new project either manually or using the included prompt.

#### Quick Setup (using the bootstrap prompt)

Open VS Code in your new project directory and run the `/bootstrap-project` prompt:

> `/bootstrap-project my-scanner — A CLI tool that scans dependencies for CVEs`

The prompt drives the project-manager agent through all scaffolding steps automatically.

#### Manual Setup

1. **Create your project repository**

   ```bash
   mkdir my-project && cd my-project
   git init
   ```

2. **Add tsftemplate as a submodule**

   tsftemplate provides the quality infrastructure — TSF upstream statements, CI workflow templates, OFT configuration, and documentation scaffolding.

   ```bash
   git submodule add https://github.com/AnotherDaniel/tsftemplate
   ```

3. **Copy the scaffolding into your project**

   From tsftemplate, set up the initial project structure:

   ```bash
   # CI workflow with OFT + tsffer + tsflink
   cp -r tsftemplate/.github .github

   # OFT configuration
   cp tsftemplate/.env.oft .env.oft

   # TSF graph root
   cp tsftemplate/.dotstop.dot .dotstop.dot

   # Documentation skeleton
   cp -r tsftemplate/docs docs
   cp tsftemplate/mkdocs.yaml mkdocs.yaml

   # Project-level TSF statements directory
   mkdir -p trustable/my-project

   # Upstream TSF statements (reference hierarchy)
   cp -r tsftemplate/trustable/upstream trustable/upstream

   # tsftemplate trudag evidence renderers
   cp -r tsftemplate/.dotstop_extensions .dotstop_extensions
   ```

4. **Add tsfagent as a submodule** (for the agent skills)

   ```bash
   git submodule add https://github.com/AnotherDaniel/tsfagent
   ```

   The `.agents/` directory in tsfagent provides the skills, agents, and prompts. VS Code's Copilot agent mode picks these up automatically.

5. **Start the workflow**

   Open your project in VS Code and invoke the project-manager agent:

   > *"I want to build a CLI tool that scans project dependencies for known CVEs and generates a report with severity ratings."*

   The agent will drive the specification → TSF → implementation pipeline from there.

### Project Structure After Setup

```
my-project/
├── .env.oft                    # OFT file pattern configuration
├── .dotstop.dot                # TSF graph definition
├── .github/
│   └── workflows/
│       └── release.yaml        # CI with OFT + tsffer + tsflink
├── docs/
│   └── specification.md        # OFT requirements
├── src/                        # Implementation (created by developer)
├── tests/                      # Tests (created by developer)
├── trustable/
│   ├── my-project/             # Project-level TSF statements
│   └── upstream/               # Upstream TSF hierarchy
├── tsftemplate/                # Template submodule (reference)
└── tsfagent/                   # Agent skills submodule
```

## Key Concepts

### OpenFastTrace (OFT)

A requirements tracing tool that verifies every specification item is implemented and tested. Requirements are written as `req~name~version` in markdown; implementations and tests reference them with coverage markers like `[impl->req~name~1]`. OFT checks that all `Needs:` directives are satisfied.

### Trustable Software Framework (TSF)

A quality framework that organizes provable claims about a project in a directed acyclic graph. Each claim is a markdown file with YAML frontmatter, linked to parent quality goals. Evidence from CI workflows is attached to prove each claim. The graph is scored to produce a trust report.

### The Traceability Chain

```
User Goal
  → req~ (specification requirement)
    → impl~ / utest~ (OFT coverage markers in code and tests)
      → tsffer (CI evidence recording)
        → TSF statement (quality claim with proof)
          → Trust report (auditable quality assessment)
```

## Hazard Analysis with STPA

The **stpa-analyst** skill provides an optional but powerful analytical step that can run before or alongside the specification/TSF/implementation pipeline. It performs a System-Theoretic Process Analysis (STPA) on your project's code or architecture.

### What STPA Does

SPTA is a top-down hazard analysis method (based on the STAMP model by Nancy Leveson at MIT). Unlike failure-focused methods like FMEA, STPA identifies hazards arising from unsafe *interactions* between components — including cases where every component works correctly but the system as a whole enters a hazardous state.

The analysis proceeds through four steps:

1. **Define purpose** — Identify losses (undesirable outcomes), hazards (system states leading to losses), and system-level constraints
2. **Model control structure** — Map controllers, control actions, feedback, and controlled processes in the system
3. **Identify unsafe control actions** — For each control action, systematically check: what if it's not provided? Provided when it shouldn't be? Too early/late? Wrong duration?
4. **Identify loss scenarios** — Why each unsafe control action might occur (stale state, dropped messages, race conditions, API violations, etc.)

### When to Use It

- Systems where safety, reliability, or data integrity are critical
- Projects with complex component interactions or concurrency
- When you want a systematic way to derive requirements rather than ad-hoc brainstorming
- As a periodic review when the system architecture changes

### How It Connects to the Pipeline

The STPA analysis produces three types of derived outputs:

| STPA Output | Feeds Into | Becomes |
|-------------|------------|---------|
| Controller constraints | **specification-writer** | `req~` items with `Needs: impl, utest, itest` |
| Hazard/constraint pairs | **tsf-author** | TSF statements under "Safety" or "Reliability" groups |
| Loss scenarios | **developer** | Fault injection and validation test cases |

The analysis document (`docs/stpa-analysis.md`) is a living artifact — it should be revisited when the architecture changes, new components are added, or test results reveal unexpected behaviours (consistent with the RAFIA iterative cycle).

### Example Usage

Invoke the stpa-analyst on your project:

> *"Perform an STPA analysis on the task scheduler module in src/scheduler/"*

The analyst will:
1. Read the code to understand the system's components and interactions
2. Present identified losses and hazards for your review
3. Model the control structure and identify unsafe control actions
4. Produce `docs/stpa-analysis.md` with the full analysis and derived outputs
5. Hand off candidate requirements, TSF statements, and test scenarios to the other skills

## End-to-End Example

This example shows the complete chain from user goal through to CI evidence, using a dependency scanning feature.

### 1. User Goal

> "I want my project to scan dependencies for known vulnerabilities on every CI run."

### 2. Specification (specification-writer)

Added to `docs/specification.md`:

```markdown
#### Dependency vulnerability scanning

`req~dependency-scanning~1`:
The project *MUST* perform automated vulnerability scanning of all direct and transitive
dependencies on every CI pipeline run and *MUST* make the results available as a release artifact.

Needs: impl, utest
```

### 3. TSF Statement (tsf-author)

Created `trustable/my-project/MYPROJECT-DEPENDENCY_SCANNING.md`:

```markdown
---
normative: true

publish:
    group: "Security"
#EVIDENCE_REF#
score:
    JaneDoe: 0.8
---

The project performs automated dependency vulnerability scanning as part of every CI pipeline
run, and build results are published as release artifacts.
```

And `trustable/my-project/MYPROJECT-DEPENDENCY_SCANNING_CONTEXT.md`:

```markdown
---
normative: false
---

**Guidance**

Automated dependency scanning ensures that known vulnerabilities in third-party libraries are
detected before they reach production.

**Evidence**

- `openfasttrace` trace of `req~dependency-scanning` proving implementation and test coverage exist
- `github` reference to the CI workflow file configuring the scanning step
- `download_url` reference to the scan report published as a release artifact

**Confidence scoring**

Score of 0.8 reflects automated scanning is strong but depends on scanner coverage and database freshness.
```

Linked in the graph:

```bash
trudag manage create-link UPSTREAM.ECLIPSE.ECLIPSE-BUILD_INSTRUCTIONS MYPROJECT-DEPENDENCY_SCANNING
```

### 4. Implementation (developer)

`src/scanner.py`:

```python
# [impl->req~dependency-scanning~1]
def scan_dependencies(lockfile_path: str) -> list[Vulnerability]:
    """Scan project dependencies for known CVEs using the OSV database."""
    lockfile = parse_lockfile(lockfile_path)
    return query_osv(lockfile.packages)
```

`tests/test_scanner.py`:

```python
# [utest->req~dependency-scanning~1]
def test_scan_finds_known_vulnerability():
    vulns = scan_dependencies("tests/fixtures/lockfile_with_cve.lock")
    assert any(v.cve_id == "CVE-2024-1234" for v in vulns)

# [utest->req~dependency-scanning~1]
def test_scan_returns_empty_for_clean_dependencies():
    vulns = scan_dependencies("tests/fixtures/clean_lockfile.lock")
    assert vulns == []
```

### 5. CI Evidence (developer)

Added to `.github/workflows/release.yaml`:

```yaml
- name: tsffer dependency scanning evidence
  uses: AnotherDaniel/tsffer@v0.5.5
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    mode: reference
    reference_properties: |
      {
        "reference_type": "openfasttrace",
        "requirement_id": "req~dependency-scanning"
      }
    asset_description: "OFT tracing for dependency scanning requirement"
    asset_name: "OFT dependency-scanning"
    asset_tsf_ids: "MYPROJECT-DEPENDENCY_SCANNING"
```

### 6. Result

When CI runs on a tagged release:

- OFT verifies `req~dependency-scanning~1` is covered by `impl` and `utest` markers
- tsffer records the OFT trace result as evidence for `MYPROJECT-DEPENDENCY_SCANNING`
- tsflink links all evidence to TSF statements and generates the trust report
- The trust report shows `MYPROJECT-DEPENDENCY_SCANNING` with a score of 0.8 and linked evidence

## Verification

After the agents complete their work, verify the full pipeline:

### OFT Tracing

```bash
# Run OpenFastTrace to check all requirements are covered
oft trace $(cat .env.oft | grep OFT_FILE_PATTERNS | cut -d'"' -f2)
```

Exit code 0 means all `Needs:` directives are satisfied. Exit code 1 means coverage gaps exist — check the report for which `req~` items are missing markers.

### TSF Graph

```bash
# Check for suspect links and unreviewed items
trudag manage lint

# Visualize the graph
trudag plot

# Calculate scores
trudag score
```

### Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| OFT reports missing `impl` coverage | No `[impl->req~name~1]` marker in code | Add marker to implementing function/file |
| OFT reports missing `utest` coverage | No `[utest->req~name~1]` marker in tests | Add marker to test function |
| OFT finds marker but doesn't match | File not in `.env.oft` `OFT_FILE_PATTERNS` | Update `.env.oft` to include the file's directory |
| trudag reports suspect links | A child statement changed after link was reviewed | Re-evaluate and run `trudag manage set-link` |
| TSF statement scores 0.0 | No evidence linked | Check `tsffer` CI steps reference the correct `asset_tsf_ids` |

## Project Layout Reference

| Path | Purpose | Created By |
|------|---------|------------|
| `docs/stpa-analysis.md` | STPA hazard analysis (if performed) | stpa-analyst |
| `docs/specification.md` | OFT requirements specification | specification-writer |
| `trustable/<project>/` | Project-level TSF statements | tsf-author |
| `trustable/upstream/` | Upstream TSF hierarchy | tsftemplate (scaffolding) |
| `.dotstop.dot` | TSF graph definition | tsf-author (via trudag) |
| `.env.oft` | OFT file pattern configuration | developer / bootstrap |
| `.github/workflows/release.yaml` | CI pipeline with OFT + tsffer + tsflink | developer |
| `src/`, `tests/` | Implementation and tests with OFT markers | developer |

## Tools

| Tool | Purpose | Repository |
|------|---------|------------|
| **OpenFastTrace** | Requirements tracing | [itsallcode/openfasttrace](https://github.com/itsallcode/openfasttrace) |
| **trudag** | TSF graph management | [AnotherDaniel/trudag](https://github.com/AnotherDaniel/trudag) |
| **tsffer** | CI evidence recording (GitHub Action) | [AnotherDaniel/tsffer](https://github.com/AnotherDaniel/tsffer) |
| **tsflink** | Evidence linking + report generation (GitHub Action) | [AnotherDaniel/tsflink](https://github.com/AnotherDaniel/tsflink) |

## License

[Eclipse Public License 2.0](LICENSE)
