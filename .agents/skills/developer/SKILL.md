---
name: developer
description: "Implement code and tests for OFT specification requirements, with full traceability. Use when: implementing a requirement from the specification, writing test cases for requirements, adding impl~/utest~/itest~ OFT coverage markers, creating tsffer CI steps for TSF evidence, connecting code to TSF quality goals. Keywords: implement, code, test, impl~, utest~, itest~, coverage, OFT tag, tsffer, evidence, CI, traceability."
---

# Developer

Implement specification requirements as production code and tests, with OpenFastTrace coverage markers and TSF evidence linkage in CI workflows.

This skill is the implementation layer in the tsftemplate triad:
- **specification-writer** defines *what* must be built (`req~`)
- **tsf-author** defines *why* it matters and what proof looks like (TSF statements)
- **developer** builds it, tests it, and wires up evidence (`impl~`, `utest~`, `tsffer`)

## When to Use

- Implementing a requirement from the specification (`req~` → code)
- Writing test cases that cover specification requirements
- Adding OFT coverage markers (`impl~`, `utest~`, `itest~`) to code and tests
- Creating `tsffer` CI workflow steps to record evidence for TSF statements
- Reviewing whether existing implementations have proper traceability

## Handoff

### Inputs

- `req~<name>~<version>` identifiers with `Needs:` directives — from the specification (created by **specification-writer**)
- TSF statement IDs (`PREFIX-DESCRIPTOR`) — from **tsf-author**, identifying which TSF nodes need evidence
- `_CONTEXT` evidence plans — from **tsf-author**, describing what artifacts to produce and which evidence types to use

### Outputs

- Production code and tests with OFT coverage markers (`[impl->...]`, `[utest->...]`, etc.)
- `tsffer` CI workflow steps recording evidence for TSF statements
- Updated `.env.oft` if new file patterns are needed

## OFT Coverage Markers

Coverage markers tell OpenFastTrace that a requirement is implemented or tested. The marker format depends on the file type.

### Marker Format by File Type

The **tag format** is the universal OFT coverage marker that works across all supported file types. Prefer it for consistency.

#### Tag format (preferred — all programming and config files)

The tag format is embedded in comments appropriate to each language:

```python
# [impl->req~dependency-scanning~1]
def scan_dependencies():
    ...
```

```java
// [impl->req~authentication-timeout~1]
public void configureSessionTimeout(Duration timeout) {
    ...
}
```

```yaml
# [impl->req~ci-scanning~1]
name: Dependency scan
```

```bash
# [impl->req~build-reproducibility~1]
docker build --no-cache -t myproject:latest .
```

For tests, replace `impl` with the appropriate test type:

```python
# [utest->req~dependency-scanning~1]
def test_scan_dependencies_finds_known_cve():
    ...
```

```java
// [utest->req~authentication-timeout~1]
@Test
void testSessionExpiresAfterTimeout() {
    ...
}
```

The tag format `[<type>-><spec-item-id>]` is recognized in: Python, Java, C/C++, C#, Go, Rust, TypeScript, JavaScript, Shell, YAML, TOML, JSON, SQL, Lua, Swift, and many more (see OFT user guide for the full list).

#### Markdown files (`.md`) — alternative format

Markdown files support both the tag format in HTML comments and OFT's native backtick-ID format:

**Tag format in HTML comment (preferred for consistency):**

```markdown
<!-- [impl->req~project-readme~1] -->
```

**Native markdown format (also valid):**

```markdown
<!--
`impl~project-readme~1`
Covers:
- req~project-readme~1
-->
```

The native format is more verbose but allows covering multiple requirements from a single marker. Use it when a single document section implements several requirements.

### Tag Format Reference

The OFT tag format for programming languages and markup files is:

```
[<artifact-type>-><specification-item-id>]
```

Where:
- `artifact-type`: `impl`, `utest`, `itest`, `stest`, or any custom type
- `specification-item-id`: the full `req~name~version` identifier

Optional extended forms:

```
[<artifact-type>~<name>~<revision>-><specification-item-id>]
```

This allows naming and versioning the covering item explicitly.

### Standard Artifact Types

| Type | Meaning | Use for |
|------|---------|---------|
| `impl` | Implementation | Production code, configuration, documentation files |
| `utest` | Unit test | Isolated unit tests |
| `itest` | Integration test | Tests involving multiple components or external systems |
| `stest` | System test | End-to-end system-level tests |

Choose the type that matches what the requirement's `Needs:` directive expects.

## Procedure

### Implementing a Requirement

1. **Read the requirement.** Find the `req~` item in the specification document. Understand what it demands (check BCP14 keywords), what tracing it needs (`Needs: impl`, `Needs: impl, utest`, etc.), and any dependencies.

2. **Identify or create the target file.** Determine where the implementation belongs — source code, configuration, documentation, or infrastructure file.

3. **Write the implementation.** Implement the requirement to the highest quality standards. Follow existing project conventions for style, structure, and error handling.

4. **Add the OFT coverage marker.** Place the marker as close as possible to the implementation:
   - In code files: use the tag format (`[impl->req~name~version]`) in a comment directly above the implementing function, class, or block
   - In markdown files: use the backtick-ID format in an HTML comment at the top of the relevant section
   - In config/YAML files: use the tag format in a comment above the relevant configuration block

5. **Verify the marker is discoverable.** Ensure the file type and location are included in the project's `.env.oft` `OFT_FILE_PATTERNS` configuration. If not, update `.env.oft`.

### Writing Tests for a Requirement

1. **Check the `Needs:` directive.** Only write tests if the requirement specifies `utest`, `itest`, or `stest` coverage.

2. **Write the test.** Each test should verify a specific, falsifiable aspect of the requirement. Name tests descriptively to reflect the requirement being verified.

3. **Add the OFT coverage marker.** Place the tag directly above the test function or class:
   ```python
   # [utest->req~session-timeout~1]
   def test_session_expires_after_configured_timeout():
       ...
   ```

4. **One marker per test function.** If a single test covers multiple requirements, add multiple markers:
   ```python
   # [utest->req~session-timeout~1]
   # [utest->req~session-cleanup~1]
   def test_expired_session_is_cleaned_up():
       ...
   ```

### Connecting to TSF Evidence via CI

After implementing code and tests, wire up evidence collection in the CI workflow so that TSF statements can reference proof of implementation.

1. **Identify the TSF statement(s)** that this implementation supports. Check `trustable/<project>/` for the relevant `PREFIX-DESCRIPTOR.md` files and their `_CONTEXT` companions for evidence guidance.

2. **Choose appropriate tsffer reference types** based on what was implemented:

   | What you produced | tsffer reference type | Example |
   |------------------|-----------------------|---------|
   | A source file | `github` | Reference to the file with line range |
   | A release artifact | `download_url` | URL of uploaded build output |
   | A requirement implementation | `openfasttrace` | OFT trace for `req~name` |
   | A policy/process doc | `webpage` | URL to external reference |

3. **Add tsffer action steps** to the release workflow (`.github/workflows/release.yaml`):

#### GitHub file reference

```yaml
- name: tsffer <descriptive name> evidence
  uses: AnotherDaniel/tsffer@v0.5.5
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    mode: reference
    reference_properties: |
      {
        "reference_type": "github",
        "repository": "${{ github.repository }}",
        "path": "<file-path>#L<start>-L<end>",
        "ref": "${{ github.ref }}"
      }
    asset_description: "<What this file proves>"
    asset_name: "<Short label>"
    asset_tsf_ids: "<TSF-STATEMENT-ID>"
```

#### OFT requirement trace

```yaml
- name: tsffer OFT <requirement-name> evidence
  uses: AnotherDaniel/tsffer@v0.5.5
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    mode: reference
    reference_properties: |
      {
        "reference_type": "openfasttrace",
        "requirement_id": "req~<name>"
      }
    asset_description: "Tracing info for OFT requirement"
    asset_name: "OFT Requirement <name>"
    asset_tsf_ids: "<TSF-STATEMENT-ID>"
```

#### Download URL reference (for release artifacts)

```yaml
- name: Upload <artifact> to release
  uses: svenstaro/upload-release-action@v2
  id: upload_artifact
  with:
    repo_token: ${{ secrets.GITHUB_TOKEN }}
    file: <path-to-artifact>
    tag: ${{ github.ref }}
- name: tsffer <artifact> evidence
  uses: AnotherDaniel/tsffer@v0.5.5
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    mode: reference
    reference_properties: |
      {
        "reference_type": "download_url",
        "url": "${{ steps.upload_artifact.outputs.browser_download_url }}",
        "description": "<Artifact description>"
      }
    asset_description: "<What this artifact proves>"
    asset_name: "<Short label>"
    asset_tsf_ids: "<TSF-STATEMENT-ID>"
```

#### Multiple references in a single step

Multiple evidence items for the same TSF statement can be combined in a JSON array:

```yaml
reference_properties: |
  [
    {
      "reference_type": "github",
      "repository": "${{ github.repository }}",
      "path": "src/scanner.py",
      "ref": "${{ github.ref }}"
    },
    {
      "reference_type": "webpage",
      "url": "https://example.org/standard",
      "description": "Compliance standard reference"
    }
  ]
asset_tsf_ids: "MYPROJECT-SCANNING"
```

#### Targeting multiple TSF statements

A single evidence item can support multiple TSF statements via comma-separated IDs:

```yaml
asset_tsf_ids: "MYPROJECT-PROJECT_README,MYPROJECT-PROJECT_SCOPE"
```

## End-to-End Traceability Example

Given requirement `req~dependency-scanning~1` with `Needs: impl, utest` and TSF statement `MYPROJECT-DEPENDENCY_SCANNING`:

### 1. Implementation (`src/scanner.py`)

```python
# [impl->req~dependency-scanning~1]
def scan_dependencies(lockfile_path: str) -> list[Vulnerability]:
    """Scan project dependencies for known CVEs using the OSV database."""
    lockfile = parse_lockfile(lockfile_path)
    return query_osv(lockfile.packages)
```

### 2. Unit test (`tests/test_scanner.py`)

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

### 3. CI evidence (`.github/workflows/release.yaml`)

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

### 4. Update `.env.oft` if needed

```bash
OFT_FILE_PATTERNS="*.md *.py docs src tests .github trustable/myproject"
```

## Quality Checklist

Before finalizing an implementation, verify:

- [ ] Every `Needs: impl` requirement has an `impl~` or `[impl->...]` marker in code
- [ ] Every `Needs: utest` requirement has a `[utest->...]` marker in a test file
- [ ] Every `Needs: itest` requirement has an `[itest->...]` marker in an integration test
- [ ] Coverage markers reference the correct `req~name~version`
- [ ] Markers are placed close to the implementing code (not in an unrelated location)
- [ ] Files containing markers are covered by `.env.oft` `OFT_FILE_PATTERNS`
- [ ] Tests are meaningful (verify the requirement, not just exercise code)
- [ ] For each requirement linked to a TSF statement, a `tsffer` CI step records evidence
- [ ] `tsffer` steps use `openfasttrace` reference type with bare requirement ID (no version suffix)
- [ ] Implementation follows existing project conventions for code style and structure
- [ ] Each commit is small, focused, and references the relevant `req~` or TSF statement ID
- [ ] No generated files, build outputs, secrets, or submodule changes committed

## Verification

After implementation is complete, run these checks to confirm the full traceability chain is intact.

### OFT Tracing

Run OpenFastTrace to verify all requirements are covered:

```bash
# Using the run-oft action locally, or the OFT CLI directly:
oft trace $(cat .env.oft | grep OFT_FILE_PATTERNS | cut -d'"' -f2)
```

- **Exit code 0**: All `Needs:` directives are satisfied
- **Exit code 1**: Coverage gaps exist — the report shows which `req~` items are missing `impl`, `utest`, or `itest` markers

Common failures and fixes:

| OFT Error | Cause | Fix |
|-----------|-------|-----|
| `req~name~1` needs `impl` but has no coverage | Missing `[impl->req~name~1]` marker | Add marker to implementing code |
| `req~name~1` needs `utest` but has no coverage | Missing `[utest->req~name~1]` marker | Add marker to test file |
| Marker found but not matched | File not in `OFT_FILE_PATTERNS` | Update `.env.oft` |
| Version mismatch | Marker references wrong version | Update marker to match current `req~` version |

### TSF Graph Integrity

```bash
# Check for suspect links and unreviewed items
trudag manage lint

# Visualize the graph to inspect structure
trudag plot

# Calculate scores to verify evidence propagation
trudag score
```

- **Suspect links**: A child statement changed after the link was reviewed. Run `trudag manage set-link <PARENT> <CHILD>` after re-evaluating.
- **Unreviewed items**: New statements not yet reviewed. Run `trudag manage set-item <ID>` after inspection.
- **Score of 0.0**: Statement has no evidence linked. Check that `tsffer` CI steps reference the correct `asset_tsf_ids`.

### CI Evidence Completeness

Review `.github/workflows/release.yaml` and verify:

1. Every TSF statement in `trustable/<project>/` has at least one `tsffer` step with its ID in `asset_tsf_ids`
2. Every `openfasttrace` reference uses the bare requirement ID (no `~version` suffix)
3. The `tsflink` step runs after all `tsffer` steps
4. The OFT trace step's `file-patterns` match `.env.oft` `OFT_FILE_PATTERNS`

## Git Commit Practices

Commits are part of the traceability chain — they connect code changes to the requirements that motivated them.

**Commit scope**: Make small, focused commits. Each commit should address a single requirement or a single logical change. Do not bundle unrelated changes.

- One commit per `req~` implementation (code + OFT marker)
- One commit per test addition (test code + OFT marker)
- One commit per CI evidence wiring (`tsffer` step addition)
- Separate commits for configuration changes (`.env.oft`, `.dotstop.dot`)

**Commit message format**:

```
<short summary of what changed> (req~<name>~<version>)

<optional body explaining why, referencing the requirement or TSF statement>
```

Examples:

```
Add dependency scanner implementation (req~dependency-scanning~1)
```

```
Add unit tests for session timeout (req~session-timeout~1)

Covers edge cases: zero timeout, negative value, exceeding max.
```

```
Wire tsffer evidence for dependency scanning (MYPROJECT-DEPENDENCY_SCANNING)
```

```
Update .env.oft to include test fixtures directory
```

**Rules**:

- First line: imperative mood, max ~72 characters, reference the `req~` ID or TSF statement ID in parentheses when applicable
- Body (optional): explain *why*, not *what* — the diff shows what changed
- Never commit generated files, build outputs, or secrets
- Never commit changes to submodule directories (`tsftemplate/`, `tsfagent/`)
- Run tests before committing — do not commit code that breaks the build
