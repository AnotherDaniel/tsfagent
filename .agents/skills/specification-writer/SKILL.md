---
name: specification-writer
description: "Write and maintain OpenFastTrace (OFT) requirements specifications in markdown. Use when: creating new requirements, adding requirements to specification documents, reviewing requirement format, generating req~ identifiers, setting Needs/Depends tracing directives. Keywords: requirement, specification, OFT, OpenFastTrace, req~, BCP14, RFC 2119, tracing."
---

# Specification Writer

Write formal requirements in OpenFastTrace (OFT) markdown format, following IETF BCP14 keyword conventions and proper traceability structure.

## When to Use

- Adding requirements to an existing specification document
- Reviewing or correcting OFT requirement syntax
- Breaking down high-level goals into traceable requirements

This skill assumes a specification document already exists. Read the existing document first to understand its section structure, naming conventions, and BCP14 preamble before adding requirements.

## Handoff

### Inputs

- User goals or feature descriptions to decompose into formal requirements
- Existing specification documents to extend

### Outputs

- `req~<name>~<version>` identifiers with `Needs:` directives — consumed by **developer** to implement and test
- Requirement IDs — consumed by **tsf-author** to align TSF quality statements with specification items

### Related Skills

This skill is the first step in the tsftemplate triad:

1. **specification-writer** (this skill) decomposes user goals into `req~` items
2. **tsf-author** creates TSF quality statements aligned with these requirements
3. **developer** implements requirements, adds OFT markers, and wires CI evidence

## Requirement Format

Each requirement follows this exact pattern:

```markdown
#### <Descriptive title>

`req~<short-name>~<version>`:
<Requirement text using BCP14 keywords in *italic*.>

Needs: <tracing-directives>
```

### Requirement ID Rules

- Format: `` `req~<short-name>~<version>` ``
- `short-name`: lowercase, hyphen-separated, concise identifier (e.g. `project-readme`, `security-policy`)
- `version`: integer starting at `1`; increment when:
  - The obligation level changes (e.g., *SHOULD* becomes *MUST*)
  - The scope of what is required changes (e.g., adding or removing conditions)
  - The verifiable outcome changes
  - Do NOT increment for: typo fixes, rewording that preserves meaning, formatting changes
- The ID line MUST end with a colon (`:`)
- The requirement text follows immediately on the next line

### BCP14 Keyword Usage

Use IETF BCP14 keywords in *italic* to express obligation levels:

| Keyword | Meaning |
|---------|---------|
| *MUST* / *MUST NOT* | Absolute requirement or prohibition |
| *REQUIRED* / *SHALL* | Synonyms for *MUST* |
| *SHOULD* / *SHOULD NOT* | Recommended unless valid reason to deviate |
| *RECOMMENDED* | Synonym for *SHOULD* |
| *MAY* / *OPTIONAL* | Truly optional behavior |

Only use these keywords in their BCP14 sense. Do NOT italicize them when used in their ordinary English sense.

### Tracing Directives

The `Needs:` line declares what artifact types must reference this requirement for it to be considered covered:

| Directive | Meaning |
|-----------|---------|
| `impl` | An implementation must reference this requirement |
| `utest` | A unit test must reference this requirement |
| `itest` | An integration test must reference this requirement |
| `stest` | A system-level test must reference this requirement |

Combine directives as needed (e.g. `Needs: impl, utest, itest`).

Choose directives based on the requirement's nature:
- Documentation/process requirements: `Needs: impl`
- Behavioral/functional requirements: `Needs: impl, utest`
- Requirements involving external systems or integration points: `Needs: impl, utest, itest`
- Design constraints: `Needs: impl`

### Dependencies

When a requirement depends on another, add a `Depends:` block:

```markdown
Depends:
- req~other-requirement~1
```

Dependencies are listed as bullet items, each referencing a full requirement ID.

## Procedure

### Decomposing User Goals into Requirements

When the user describes a high-level goal or feature (e.g., "I want vulnerability scanning"), break it down into formal requirements:

1. **Identify distinct concerns.** List the concrete, separable aspects of the goal. For "vulnerability scanning" this might be: scanning runs automatically, known CVEs are flagged, results are available as artifacts, and scanning covers transitive dependencies.
2. **Apply the singularity test.** Each concern should be independently testable. If a candidate requirement contains "and" joining two verifiable outcomes, split it.
3. **Choose the right granularity.**
   - Too coarse: requirement cannot be meaningfully tested or traced to specific code → split further
   - Too fine: requirement describes an implementation detail rather than a verifiable outcome → merge up
   - Right level: one `req~` item maps naturally to one function/module and one or more test cases
4. **Assign tracing directives per requirement type:**
   - The requirement describes *behavior or logic*: `Needs: impl, utest`
   - The requirement describes *system integration or external interaction*: `Needs: impl, utest, itest`
   - The requirement describes *a document or process artifact*: `Needs: impl`
5. **Check for dependencies between the resulting requirements** and add `Depends:` where one presupposes another.

### Writing New Requirements

1. **Identify the target specification document.** Look for an existing `specification.md` or equivalent in the `docs/` directory.
2. **Determine the appropriate section.** Group related requirements under a descriptive heading hierarchy.
3. **Choose a requirement ID.** Use a concise, hyphen-separated name that reflects the requirement's subject. Check existing IDs to avoid collisions and maintain consistent naming style.
4. **Write the requirement text.** Each requirement should be:
   - **Singular**: one testable/verifiable concern per requirement
   - **Falsifiable**: it must be possible to determine whether the requirement is met
   - **Precise**: use BCP14 keywords to express obligation level unambiguously
   - **Self-contained**: understandable without reading other requirements (dependencies clarify relationships, not meaning)
5. **Set tracing directives.** Add `Needs:` with appropriate coverage types.
6. **Declare dependencies.** Add `Depends:` if this requirement presupposes another.

### Adding to an Existing Specification

1. Read the existing specification to understand the section structure and naming conventions.
2. Find the correct section or create a new subsection if the topic is not yet covered.
3. Follow the naming pattern of existing requirement IDs (e.g., if existing IDs use `project-` prefix, continue that convention).
4. Add the new requirement following the format above.
5. Verify the target document's directory is included in `.env.oft` `OFT_FILE_PATTERNS`. If not, update `.env.oft`.

## Quality Checklist

Before finalizing a requirement, verify:

- [ ] ID follows `req~<name>~<version>` format with colon suffix
- [ ] BCP14 keywords are italicized and used in their normative sense
- [ ] Requirement is singular (one concern) and falsifiable
- [ ] `Needs:` directive is present and appropriate
- [ ] `Depends:` is declared if prerequisite requirements exist
- [ ] Requirement text does not duplicate an existing requirement
- [ ] Section placement is logical within the document hierarchy

## Example

```markdown
### Security requirements

#### Security policy

`req~security-policy~1`:
The project *MUST* provide a publicly accessible security policy that describes the process for reporting and handling security vulnerabilities.

Needs: impl

#### Dependency vulnerability scanning

`req~dependency-scanning~1`:
The project *SHOULD* perform automated vulnerability scanning of all direct and transitive dependencies as part of the CI pipeline.

Needs: impl, test

Depends:
- req~security-policy~1
```
