# Contributing to Skills MCP Server

Thanks for helping improve Skills MCP Server. This guide covers how to set up the project, add a workflow profile, and get a pull request merged.

Good places to start are issues labeled [`good first issue`](https://github.com/rajpreetcodes/Skills_MCP_Server/issues?q=is%3Aissue+state%3Aopen+label%3A%22good+first+issue%22). Comment on an issue before starting so two people don't build the same thing.

## Table of contents

- [Project layout](#project-layout)
- [Development setup](#development-setup)
- [Running tests](#running-tests)
- [Adding a workflow profile](#adding-a-workflow-profile)
- [Adding a skill](#adding-a-skill)
- [Code style](#code-style)
- [Commit conventions](#commit-conventions)
- [Pull request checklist](#pull-request-checklist)

## Project layout

```
src/         Server code (entry point: python -m src.server)
profiles/    Workflow profiles (YAML), one file per profile
tests/       pytest suite (registry, search, profiles, server)
config.json  Default configuration
```

## Development setup

Requires Python 3.10 or later.

```bash
git clone https://github.com/<your-username>/Skills_MCP_Server.git
cd Skills_MCP_Server

python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -e .
```

Fork the repository first and clone your fork. Then create a branch for your change:

```bash
git checkout -b feat/add-docker-profile
```

To run the server against a skills folder, point `--skill-root` at any directory that contains skill folders with a `SKILL.md` file:

```bash
python -m src.server --transport stdio --skill-root /path/to/skills
```

You can also set `SKILL_ROOT` as an environment variable or in `config.json`. See the README for HTTP mode and client setup.

## Running tests

```bash
# Full suite
pytest tests/ -v

# A single file
pytest tests/test_profiles.py -v
```

Tests must pass before you open a PR. If you add behavior, add a test for it.

## Adding a workflow profile

A profile is a YAML file in `profiles/` that bundles skills for a common workflow, such as `product-builder` or `senior-engineer`. Most profile issues follow this pattern.

### 1. Create the file

Name the file after the profile, in lowercase with hyphens: `profiles/docker-devops.yaml`. The `name` field must match the file name without the extension.

### 2. Follow the schema

```yaml
name: docker-devops
description: Containerization and deployment workflow for Docker-based services
skills:
  - skill-one
  - skill-two
profiles:
  - other-profile        # optional: nest an existing profile
stages:
  plan:
    - skill-one
  build:
    - skill-two
  validate:
    - skill-one
```

| Field         | Required | Notes                                                                 |
| ------------- | -------- | --------------------------------------------------------------------- |
| `name`        | Yes      | Unique, lowercase, hyphenated. Matches the file name.                 |
| `description` | Yes      | One line describing the workflow and who it is for.                   |
| `skills`      | Yes\*    | Flat list of skill names to include.                                  |
| `profiles`    | No       | Names of other profiles to include. Do not create circular references. |
| `stages`      | No       | Ordered phases, each with a list of skills. Used by `resolve_profile`. |

\*Provide `skills`, `stages`, or both. Look at an existing file in `profiles/` and mirror its structure so your profile loads the same way.

### 3. Check your profile

- Every skill name should correspond to a skill that exists in a skills library. Do not invent names.
- Keep stages in a sensible order (for example research, plan, implement, validate).
- Keep the profile focused. A handful of well-chosen skills beats a long list.
- Make sure the YAML parses and that `tests/test_profiles.py` passes.

You can also confirm the profile loads by running the server and calling `list_profiles` and `resolve_profile` from an MCP client or the MCP Inspector:

```bash
npx @modelcontextprotocol/inspector python -m src.server --transport stdio
```

## Adding a skill

Skills live in your skills directory, not in this repository. Each skill is a folder with a `SKILL.md` file:

```
my-new-skill/
  SKILL.md
  references/        # optional supporting files
```

Frontmatter fields: `name`, `description`, `category`, `version`, `tags`, `triggers`, `allowed-tools`, `license`. Call `refresh_skills` (or restart the server) after adding one.

## Code style

- Follow [PEP 8](https://peps.python.org/pep-0008/) and add type hints to new functions.
- Keep functions small and give them clear names. Add a short docstring for public functions.
- **Never `print()` to stdout.** In STDIO mode stdout carries MCP protocol frames, so stray output breaks the connection. Use the `logging` module, which is routed to stderr.
- Do not add telemetry or network calls. The server is local-first and read-only with respect to skill files.
- Do not commit secrets, tokens, or personal paths. Use `.env.example` for new settings.
- Keep unrelated changes out of your PR.

## Commit conventions

Use [Conventional Commits](https://www.conventionalcommits.org/): `type(scope): short summary`.

| Type        | Use for                                   |
| ----------- | ----------------------------------------- |
| `feat:`     | A new feature or profile                  |
| `fix:`      | A bug fix                                 |
| `docs:`     | Documentation only                        |
| `test:`     | Adding or fixing tests                    |
| `refactor:` | Code change that is not a fix or feature  |
| `chore:`    | Tooling, dependencies, housekeeping       |

Examples:

```
feat(profiles): add Docker containerization workflow profile
fix(search): handle empty query without raising
docs: add CONTRIBUTING.md
```

Write the summary in the imperative mood, keep it under about 72 characters, and add a body if the reason for the change isn't obvious. Reference the issue in the PR description with `Closes #<number>`.

## Pull request checklist

Before you request review, confirm that:

- [ ] The PR addresses one issue and links it (`Closes #<number>`).
- [ ] `pytest tests/ -v` passes locally.
- [ ] `git diff --check` reports no whitespace errors.
- [ ] New or changed behavior has tests.
- [ ] Documentation is updated (README, this guide, or docstrings) where needed.
- [ ] New profiles follow the schema above and reference real skills.
- [ ] Commit messages follow the conventions above.
- [ ] No secrets, local paths, or unrelated changes are included.

## Questions

Open an issue if something here is unclear or out of date. Improvements to this guide are welcome too.

By contributing, you agree that your contributions are licensed under the project's [MIT License](LICENSE).
