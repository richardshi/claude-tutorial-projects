# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository structure

This is a monorepo containing multiple independent tutorial/learning projects:

| Project | Description |
|---------|-------------|
| `uigen/` | Next.js 15 app where users describe React components in chat and get live previews, powered by Claude API |

Each project has its own `CLAUDE.md` with project-specific commands, architecture details, and conventions.

## Git conventions

- `.gitignore` lives at the repo root and prefixes all project-specific paths with their subdirectory (e.g., `uigen/node_modules`).
- When adding a new project, copy only the project files — never copy a nested `.git` directory. If accidentally copied, remove it with `rm -rf <project>/.git`, then run `git rm --cached -f <project> && git add <project>` to re-stage as normal tracked files.
