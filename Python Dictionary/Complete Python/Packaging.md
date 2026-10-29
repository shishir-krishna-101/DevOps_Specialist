# Packaging

Packaging is the part of Python development that turns reusable source code into something that can be built, installed, tested, distributed, and maintained consistently.

For DevOps engineers, packaging matters when automation grows beyond a single script: shared internal tools, command-line utilities, deployment helpers, and libraries all benefit from a predictable package structure.

> **Verified fact:** The Python Packaging User Guide documents modern Python packaging workflows, including project metadata, build systems, source distributions, wheels, dependency declarations, and publishing workflows.

---

## 1. Package vs Distribution

These terms are related but should not be treated as identical.

- A **Python package** is source-code structure used for organizing importable Python code.
- A **distribution package** is the installable/distributable artifact described by Python packaging standards and tooling.

A useful mental model is:

```text
Python source code
      ↓
project metadata + build configuration
      ↓
build
      ↓
distribution artifacts
      ├── source distribution
      └── wheel
      ↓
install
      ↓
usable Python project
```

**Repository design decision:** This guide uses the term *project* when discussing the whole distributable unit and *package* when discussing importable source structure.

---

## 2. Why Packaging Matters in DevOps

A one-file script can be enough for a small task. Packaging becomes valuable when you need:

- reusable automation modules
- a stable command-line tool
- dependency declarations
- repeatable installation
- testable project structure
- build artifacts
- versioned releases
- internal distribution

For example:

```text
infra-tool/
├── pyproject.toml
├── src/
│   └── infra_tool/
│       ├── __init__.py
│       └── cli.py
└── tests/
    └── test_cli.py
```

This is an example repository layout, not a requirement that every Python project must use this exact structure.

---

## 3. `pyproject.toml`

Modern Python packaging uses `pyproject.toml` as the central configuration file for project metadata and build-related configuration.

A minimal example:

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"

[project]
name = "example-tool"
version = "0.1.0"
description = "Example DevOps automation tool"
requires-python = ">=3.11"
dependencies = []
```

> **Important:** The exact build backend and dependency choices are project decisions. Do not copy this example blindly into a production project without checking the current packaging documentation for the selected backend.

The Packaging User Guide defines `pyproject.toml` configuration through the applicable standards and documents how build systems consume it.

---

## 4. Build System vs Project Metadata

A useful distinction is:

```text
[build-system]
    ↓
How the project is built

[project]
    ↓
What the project is
```

The `[build-system]` table declares the tools needed to build the project and identifies the build backend.

The `[project]` table contains standardized project metadata such as the project name, version, description, Python requirement, and dependencies.

This separation allows build tooling and project metadata to be reasoned about independently.

---

## 5. Dependencies

Project dependencies should be declared as project metadata rather than relying on an undocumented developer environment.

For example:

```toml
[project]
name = "example-tool"
version = "0.1.0"
dependencies = [
    "some-library>=1.0",
]
```

Dependency specifications should reflect the compatibility policy of the project.

### DevOps lesson

A production automation tool should make its runtime requirements explicit.

Avoid a workflow such as:

```text
works on my laptop
        ↓
copy script to server
        ↓
missing package
        ↓
manual installation
```

Prefer:

```text
project metadata
        ↓
build/install tooling
        ↓
known runtime dependencies
```

**Common mistake:** Treating the packages installed in your personal virtual environment as the project's dependency specification.

---

## 6. Optional Dependencies

Some projects need additional dependencies only for particular features or development workflows.

Python packaging metadata supports optional dependency groups through the `project.optional-dependencies` table.

Conceptually:

```text
core project
   │
   ├── runtime dependencies
   │
   ├── test extras
   │
   └── documentation extras
```

Use optional dependencies when an additional feature set should be installable without making every user install every dependency.

---

## 7. Source Distribution and Wheel

Python packaging commonly produces two important artifact types:

### Source distribution

A source distribution contains the project's source and supporting files needed to build/install it.

### Wheel

A wheel is a built distribution format designed for installation.

A project may publish both forms so that consumers and tooling have appropriate distribution artifacts available.

```text
project
   ↓
build
   ├── source distribution
   └── wheel
```

**Verified fact:** The Python Packaging User Guide documents both source distributions and wheels as standard distribution formats.

---

## 8. Building a Project

The Packaging User Guide recommends using standard Python packaging tools and build workflows rather than manually constructing distribution archives.

A common workflow is:

```text
write source
   ↓
write metadata/configuration
   ↓
build artifacts
   ↓
inspect artifacts
   ↓
test installation
   ↓
publish when ready
```

A typical modern build command is:

```bash
python -m build
```

This command comes from the `build` package and is intentionally kept separate from the project build backend.

**Important:** The build frontend and the build backend are different roles. The frontend requests a build; the backend performs the project-specific build according to its configuration.

---

## 9. Installing From an Artifact

A useful packaging test is to install the artifact rather than only running code from the repository checkout.

Conceptually:

```text
source checkout
      ↓
      build
      ↓
artifact
      ↓
clean environment
      ↓
install
      ↓
run tests / CLI
```

This catches packaging problems that a direct source checkout can hide, such as missing package data or incorrect package discovery.

---

## 10. Console Scripts and CLI Tools

A DevOps automation project may expose a command-line entry point through project metadata.

The standardized project metadata supports command-line entry points through `[project.scripts]`.

Example:

```toml
[project.scripts]
infra-check = "infra_tool.cli:main"
```

This means the installed project can expose a command named `infra-check` that invokes the specified Python function.

**Repository design decision:** For this repository, CLI examples should keep the command implementation small and place reusable logic in importable modules rather than putting the whole application inside a command entry point.

---

## 11. Versioning

A distributable project needs a versioning strategy.

The project metadata includes a `version` field, and Python packaging standards define how version identifiers are interpreted.

For learning purposes:

```text
0.1.0
│ │ │
│ │ └── patch
│ └──── minor
└────── major
```

Do not assume that this diagram alone defines the project's release policy. Versioning policy should be explicitly chosen and documented by the project maintainers.

**Common mistake:** Changing package behavior without considering whether downstream consumers can safely adopt the new release.

---

## 12. Package Discovery

When a project contains source packages, the build backend needs to know which packages belong in the distribution.

A source layout can help keep importable application code separate from repository-level files:

```text
project/
├── pyproject.toml
├── src/
│   └── example_package/
│       ├── __init__.py
│       └── module.py
└── tests/
```

The exact package-discovery configuration depends on the selected build backend.

> **No-hallucination rule:** Do not copy backend-specific discovery settings between tools. Verify the configuration against the documentation for the chosen build backend.

---

## 13. Package Data and Non-Python Files

Some applications need files other than `.py` modules:

- templates
- configuration defaults
- schemas
- static resources
- documentation assets

Those files need to be intentionally included in the built distribution when the application requires them.

A project that works directly from a repository checkout can still fail after installation if required package data was never included in the artifact.

**Practical lesson:** Test the built artifact in a clean environment.

---

## 14. Build Isolation

Modern Python build workflows use an isolated build environment so that build dependencies can be separated from the user's runtime environment.

This matters because:

```text
runtime environment
       ≠
build environment
```

A project should not depend accidentally on unrelated packages installed globally on the developer's machine.

The exact isolation behavior is controlled by the packaging/build workflow and its standards; follow the Packaging User Guide and selected backend documentation.

---

## 15. Virtual Environments and Packaging

Packaging and virtual environments solve different problems.

- A **package/distribution** describes software that can be installed and distributed.
- A **virtual environment** provides an isolated Python environment in which packages can be installed.

A practical development flow is:

```text
project
  ↓
create virtual environment
  ↓
install project/dependencies
  ↓
develop + test
  ↓
build artifact
  ↓
test artifact in clean environment
```

See the dedicated virtual-environment topic in the Python for DevOps section for environment management fundamentals.

---

## 16. Publishing

Publishing means making built distribution artifacts available through a package repository or another distribution mechanism.

The Packaging User Guide documents workflows for publishing packages to package indexes.

A safe release pipeline should conceptually look like:

```text
commit
  ↓
test
  ↓
build
  ↓
inspect artifacts
  ↓
test installation
  ↓
publish
  ↓
verify installation
```

**Operational lesson:** Do not make publishing the first time you discover that the artifact cannot be installed correctly.

---

## 17. Internal DevOps Tools

Packaging is especially useful for internal automation.

Example:

```text
company automation project
├── inventory
├── deployment checks
├── environment validation
├── health checks
└── CLI
```

Instead of distributing copied scripts manually, a team can maintain a versioned project and distribute its built artifacts through an approved internal mechanism.

The exact distribution mechanism is an organizational decision and is outside the scope of Python's packaging standards.

---

## 18. Reproducibility

Packaging improves reproducibility, but packaging alone does not guarantee identical environments.

You still need to consider:

- Python version
- dependency constraints
- operating system
- native dependencies
- environment variables
- external services
- credentials and configuration

A reliable automation project makes these requirements visible rather than relying on undocumented machine state.

---

## 19. Common Mistakes

### Mistake 1: Treating a source checkout as an installed package

Always test the actual distribution artifact.

### Mistake 2: Omitting dependencies

If the application imports a runtime dependency, its installation requirements must be represented appropriately in project metadata.

### Mistake 3: Confusing build and runtime dependencies

The tools required to build a project are not automatically the same as the dependencies required when users run it.

### Mistake 4: Depending on global packages

A project should not silently rely on whatever happens to be installed on the developer's machine.

### Mistake 5: Publishing without testing the artifact

Build first, install in a clean environment, and verify behavior before release.

### Mistake 6: Copying backend configuration blindly

Build-backend configuration is tool-specific. Verify it against authoritative documentation.

### Mistake 7: Treating version numbers as decoration

A version is part of the project's release contract. Change it deliberately.

---

## 20. Practical Exercises

### Exercise 1 — Package a CLI

Create a small DevOps utility with:

- a `pyproject.toml`
- an importable package
- a CLI entry point
- one unit-test module

Build it and install it in a clean virtual environment.

### Exercise 2 — Inspect the artifact

Build source and wheel distributions. Inspect their contents and identify:

- Python modules
- metadata
- package data
- test files that are intentionally excluded or included

### Exercise 3 — Dependency declaration

Take a script that currently relies on manually installed libraries. Convert those runtime requirements into project metadata and test installation in a fresh environment.

### Exercise 4 — Broken package

Intentionally omit a required package data file. Build the project, install the artifact, and identify why the installed program differs from the source checkout.

### Exercise 5 — Release pipeline design

Design a CI pipeline with these stages:

```text
test → build → artifact validation → release
```

For each stage, explain what failure should prevent the next stage.

---

## 21. Knowledge Check

1. What is the difference between a package and a distribution package?
2. What role does `pyproject.toml` play?
3. What is the purpose of `[build-system]`?
4. What kind of information belongs in `[project]`?
5. What is a wheel?
6. What is a source distribution?
7. Why should an artifact be tested separately from a source checkout?
8. What is the difference between build and runtime dependencies?
9. Why are virtual environments not a replacement for packaging metadata?
10. What problem does `[project.scripts]` solve?
11. Why should backend-specific configuration be verified against backend documentation?
12. What should happen before publishing a distribution?

---

## 22. Completion Checklist

You should be able to:

- [ ] Explain package vs distribution.
- [ ] Explain the role of `pyproject.toml`.
- [ ] Distinguish build metadata from project metadata.
- [ ] Declare runtime dependencies.
- [ ] Explain source distributions and wheels.
- [ ] Build a Python project using a standard build workflow.
- [ ] Install and test a built artifact in a clean environment.
- [ ] Expose a CLI through project metadata.
- [ ] Explain package-data considerations.
- [ ] Distinguish packaging from virtual environments.
- [ ] Design a basic build-and-release workflow.
- [ ] Verify backend-specific configuration from authoritative documentation.

---

## 23. Verified Facts vs Repository Design

### Verified facts

- Python packaging uses standardized project metadata and build configuration.
- `pyproject.toml` is used for project/build configuration.
- Python packaging supports source distributions and wheels.
- Project metadata can declare dependencies and command-line entry points.
- Build systems and build frontends have distinct roles.

### Repository design decisions

- Examples use a `src/` layout because it clearly separates source code from repository-level files.
- Exercises emphasize DevOps automation and clean-environment installation.
- This guide intentionally avoids prescribing one build backend for every project.

### Demonstrations

The `pyproject.toml` snippets in this file are learning examples. They are not intended to define a universal production configuration.

---

## Authoritative References

- Python Packaging User Guide: https://packaging.python.org/en/latest/
- Packaging Python Projects: https://packaging.python.org/en/latest/tutorials/packaging-projects/
- Writing your `pyproject.toml`: https://packaging.python.org/en/latest/guides/writing-pyproject-toml/
- Python `venv`: https://docs.python.org/3/library/venv.html

If a packaging behavior depends on a particular build backend, consult that backend's current official documentation rather than assuming that configuration is portable across tools.
