# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the **Agent Skills** open format specification and reference implementation. Agent Skills are folders of instructions, scripts, and resources that AI agents can discover and use to perform specialized tasks. The repo has two main components:

1. **`docs/`** - Mintlify documentation site
2. **`skills-ref/`** - Python reference implementation for parsing and validating skills

## Documentation Site (`docs/`)

The documentation is built with [Mintlify](https://mintlify.com).

### Commands

```bash
# Install Mintlify CLI (required for local development)
npm i -g mint

# Run local development server (must run from /docs directory)
cd docs && mint dev

# Update Mintlify CLI if issues occur
mint update
```

Local preview available at `http://localhost:3000`

### Architecture

- **Navigation**: Defined in `docs/docs.json` under `navigation.pages`
- **Content pages**: MDX files (Markdown + JSX) in `/docs`
- **Components**: React components in `docs/snippets/` (e.g., LogoCarousel)
- **Deployment**: Automatic on push to `main` branch

### Adding/Editing Documentation

1. Create or edit `.mdx` files in `/docs`
2. Add filename (without extension) to `docs/docs.json` navigation array
3. Run `mint dev` from `/docs` directory to preview

## Reference Implementation (`skills-ref/`)

Python library for parsing, validating, and generating prompts from Agent Skills. Requires Python 3.11+.

### Setup

```bash
# macOS/Linux
cd skills-ref
uv sync
source .venv/bin/activate

# Windows (PowerShell)
cd skills-ref
uv sync
.venv\Scripts\Activate.ps1
```

### Commands

```bash
# Code quality (format and lint)
uv run ruff format .
uv run ruff check --fix .

# Run tests
uv run pytest

# CLI usage
skills-ref validate path/to/skill
skills-ref read-properties path/to/skill
skills-ref to-prompt path/to/skill-a path/to/skill-b
```

### Architecture

The `skills_ref` package is organized into focused modules:

- **`parser.py`** - Locates SKILL.md files and extracts YAML frontmatter
- **`validator.py`** - Validates skill properties against the specification (name format, description length, directory matching, etc.)
- **`prompt.py`** - Generates `<available_skills>` XML for agent prompts (Anthropic-recommended format)
- **`models.py`** - `SkillProperties` dataclass for type-safe skill metadata
- **`errors.py`** - Exception hierarchy (`SkillError`, `ParseError`, `ValidationError`)
- **`cli.py`** - Click-based CLI with validate/read-properties/to-prompt commands

### Adding Features

When modifying the library:
1. Add code to appropriate module in `src/skills_ref/`
2. Add corresponding tests in `tests/`
3. Run `uv run ruff format .` before committing
4. Run `uv run pytest` to verify tests pass

## Agent Skills Format

A skill is a folder containing a `SKILL.md` file with YAML frontmatter:

```yaml
---
name: skill-name
description: What this skill does and when to use it
license: MIT
compatibility: Requires Python 3.11+
---

# Instructions for the agent...
```

**Required structure**: `skill-name/SKILL.md` (directory name must match `name` field)

**Optional directories**: `scripts/`, `references/`, `assets/`

**Progressive disclosure pattern**: Agents load name+description first (~100 tokens), then full SKILL.md when activated (<5000 tokens), then scripts/references on demand.

See `docs/specification.mdx` for complete format details.
