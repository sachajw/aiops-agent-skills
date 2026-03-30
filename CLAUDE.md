# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the **Agent Skills** open format specification and reference implementation. Agent Skills are folders of instructions, scripts, and resources that AI agents can discover and use to perform specialized tasks. The repo has two main components:

1. **`docs/`** - Mintlify documentation site (CC-BY-4.0 license)
2. **`skills-ref/`** - Python reference implementation for parsing and validating skills (Apache 2.0 license)

Example skills live in a separate repo: https://github.com/anthropics/skills

Sub-CLAUDE.md files exist in `docs/CLAUDE.md` and `skills-ref/CLAUDE.md` with component-specific guidance.

## Documentation Site (`docs/`)

Built with [Mintlify](https://mintlify.com). Auto-deploys on push to `main`.

```bash
npm run dev           # Start local dev server (localhost:3000) — runs `cd docs && npx mint dev`
npm i -g mint         # Install Mintlify CLI globally
mint update           # Update Mintlify CLI if issues occur
```

- **Navigation**: `docs/docs.json` → `navigation.pages` array
- **Content**: MDX files in `/docs`; React components in `docs/snippets/`
- **Adding pages**: Create `.mdx` file, add filename (no extension) to the navigation array
- **Logo carousel**: `docs/snippets/LogoCarousel.jsx`, images in `docs/images/logos/`

## Reference Implementation (`skills-ref/`)

Python library (3.11+) for parsing, validating, and generating prompts from Agent Skills. Uses `hatchling` build system, `click` for CLI, `strictyaml` for frontmatter parsing.

**Note**: Per CONTRIBUTING.md, code contributions to `skills-ref/` are not being accepted at this time (bug reports and feedback welcome).

```bash
cd skills-ref
uv sync                                    # Install dependencies

# Code quality
uv run ruff format .                       # Format
uv run ruff check --fix .                  # Lint

# Testing
uv run pytest                              # All tests
uv run pytest tests/test_parser.py -v      # Single test file
uv run pytest tests/test_parser.py::test_parse_valid_frontmatter -v  # Single test

# CLI
skills-ref validate path/to/skill
skills-ref read-properties path/to/skill
skills-ref to-prompt path/to/skill-a path/to/skill-b
```

### Architecture

`src/skills_ref/` — six focused modules:

- **`models.py`** — `SkillProperties` dataclass (name, description, license, compatibility, allowed_tools, metadata). Serializes to dict with hyphenated keys (`allowed-tools`).
- **`parser.py`** — `find_skill_md()` locates SKILL.md (prefers uppercase, accepts lowercase). `read_properties()` parses YAML frontmatter via `strictyaml` and returns `SkillProperties`.
- **`validator.py`** — Validates against spec: NFKC-normalized names (max 64 chars, lowercase, alphanumeric+hyphens, must match directory), descriptions (max 1024), and allowed fields only. Returns error list.
- **`prompt.py`** — Generates `<available_skills>` XML for agent system prompts.
- **`errors.py`** — Exception hierarchy: `SkillError` → `ParseError`, `ValidationError` (carries `errors` list).
- **`cli.py`** — Click CLI. All commands accept skill directory path or direct SKILL.md path (auto-resolves to parent dir).

### Public API (`__init__.py` exports)

`SkillError`, `ParseError`, `ValidationError`, `SkillProperties`, `find_skill_md`, `validate`, `read_properties`, `to_prompt`

## Agent Skills Format

A skill is `skill-name/SKILL.md` where the directory name matches the `name` frontmatter field. Optional subdirectories: `scripts/`, `references/`, `assets/`. Progressive disclosure: name+description (~100 tokens) → full SKILL.md (<5000 tokens) → scripts/references on demand. See `docs/specification.mdx` for complete spec.
