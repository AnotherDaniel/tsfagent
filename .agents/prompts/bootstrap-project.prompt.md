---
description: "Bootstrap a new project with tsftemplate quality infrastructure and tsfagent skills. Use when: creating a new project, setting up OFT/TSF scaffolding, initializing a trustable project from scratch."
agent: "project-manager"
tools: [read, edit, search, execute]
argument-hint: "Project name and brief description of what it will do"
---

Bootstrap a new trustable software project using tsftemplate scaffolding.

The user will provide a **project name** and optionally a brief description of the project's purpose.

## Steps

1. **Create the project directory and initialize git** (if not already in one):

   ```bash
   mkdir <project-name> && cd <project-name>
   git init
   ```

2. **Add tsftemplate as a submodule**:

   ```bash
   git submodule add https://github.com/AnotherDaniel/tsftemplate
   ```

3. **Copy scaffolding from tsftemplate**:

   ```bash
   # CI workflow
   cp -r tsftemplate/.github .github

   # OFT configuration
   cp tsftemplate/.env.oft .env.oft

   # TSF graph root
   cp tsftemplate/.dotstop.dot .dotstop.dot

   # Documentation skeleton
   cp -r tsftemplate/docs docs
   cp tsftemplate/mkdocs.yaml mkdocs.yaml

   # Project-level TSF statements directory
   mkdir -p trustable/<project-name>

   # Upstream TSF statements
   cp -r tsftemplate/trustable/upstream trustable/upstream
   ```

4. **Add tsfagent as a submodule**:

   ```bash
   git submodule add https://github.com/AnotherDaniel/tsfagent
   ```

5. **Update `.env.oft`** to include project-specific paths:

   ```bash
   OFT_FILE_PATTERNS="*.md docs .github trustable/<project-name>"
   ```

6. **Update `.dotstop.dot`** — replace `TSFTEMPLATE` references with the project's prefix (UPPERCASE version of project name).

7. **Rename/update TSF template statements** in `trustable/<project-name>/`:
   - Copy tsftemplate's example project statements as a starting point if desired
   - Update the PREFIX in all statement IDs to match the project

8. **Update the CI workflow** (`.github/workflows/release.yaml`):
   - Replace `TSFTEMPLATE` TSF IDs with the project's prefix
   - Adjust file references and paths to match the project structure

9. **Create a project specification document** at `docs/specification.md` with the BCP14 preamble:

   ```markdown
   # <Project Name> Specification

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174]
   when, and only when, they appear in *italics*.

   ## Requirements

   <!-- Requirements will be added here by specification-writer -->
   ```

10. **Create initial commit**:

    ```bash
    git add -A
    git commit -m "Initial project setup with tsftemplate scaffolding"
    ```

11. **Report** what was created and confirm the project is ready for the user to describe their first goal.
