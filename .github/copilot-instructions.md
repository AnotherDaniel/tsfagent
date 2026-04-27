# tsfagent Project Instructions

## Vocabulary

| Term | Meaning |
|------|---------|
| **OFT** | OpenFastTrace — requirements tracing tool |
| **TSF** | Trustable Software Framework — quality claim hierarchy |
| **trudag** | CLI for managing the TSF directed acyclic graph |
| **tsffer** | GitHub Action for recording evidence in CI releases |
| **tsflink** | GitHub Action for linking evidence to TSF statements and generating reports |
| **BCP14** | IETF keywords (*MUST*, *SHOULD*, *MAY*, etc.) used in specification requirements |

## Naming Conventions

### OFT Requirements

- ID format: `req~<lowercase-hyphen-name>~<version>` (e.g., `req~dependency-scanning~1`)
- Coverage markers: `[impl->req~name~version]`, `[utest->req~name~version]`, `[itest->req~name~version]`
- Tracing directives: `Needs: impl`, `Needs: impl, utest`, `Needs: impl, utest, itest`
- Standard artifact types: `impl`, `utest`, `itest`, `stest`

### TSF Statements

- ID format: `PREFIX-DESCRIPTOR` in UPPERCASE with underscores (e.g., `MYPROJECT-DEPENDENCY_SCANNING`)
- File naming: `PREFIX-DESCRIPTOR.md` (normative) + `PREFIX-DESCRIPTOR_CONTEXT.md` (companion)
- Placement: `trustable/<project>/`

### Cross-References

- A `req~` ID maps to a TSF statement by convention: `req~dependency-scanning~1` ↔ `MYPROJECT-DEPENDENCY_SCANNING`
- tsffer `openfasttrace` references use bare requirement IDs without version: `req~dependency-scanning`

## Tool Versions

| Tool | Version | Reference |
|------|---------|-----------|
| `tsffer` | `AnotherDaniel/tsffer@v0.5.5` | GitHub Action |
| `tsflink` | `AnotherDaniel/tsflink@v0.3.0` | GitHub Action |
| `upload-release-action` | `svenstaro/upload-release-action@v2` | For release artifacts |

## File Layout

| Path | Purpose |
|------|---------|
| `docs/specification.md` | OFT requirements specification |
| `trustable/<project>/` | Project-level TSF statements |
| `trustable/upstream/` | Upstream TSF hierarchy (from tsftemplate) |
| `.dotstop.dot` | TSF graph definition |
| `.env.oft` | OFT file pattern configuration |
| `.github/workflows/release.yaml` | CI pipeline with OFT + tsffer + tsflink |

## Workflow Model

This project uses three specialized skills orchestrated by the **project-manager** agent, with an optional analytical skill:

1. **stpa-analyst** (optional) — performs hazard analysis, deriving safety constraints
2. **specification-writer** — decomposes goals into `req~` items
3. **tsf-author** — creates TSF quality statements and evidence plans
4. **developer** — implements code, tests, OFT markers, and CI evidence

The pipeline flows: (STPA analysis →) specification → TSF statements → implementation → CI evidence → trust report.

## Quality Rules

- Every `req~` with `Needs: impl` must have a corresponding `[impl->...]` marker
- Every `req~` with `Needs: utest` must have a corresponding `[utest->...]` marker
- Every normative TSF statement must have a `_CONTEXT` companion
- Every TSF statement should have at least one tsffer evidence source in CI
- OFT file patterns in `.env.oft` must cover all files containing markers
- Run `trudag manage lint` to verify graph integrity after changes
