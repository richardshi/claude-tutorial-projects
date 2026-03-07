# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

E-commerce data utilities project providing TypeScript query functions over a SQLite database, with Claude Agent SDK integration for agentic workflows.

## Development Commands

```bash
# Install dependencies and configure Claude hooks
npm run setup

# Run the entry point (initializes DB schema)
npx tsx src/main.ts

# Run the agent SDK example
npm run sdk

# Type-check without emitting
npx tsc --noEmit
```

## Architecture

- **`src/main.ts`** — Opens `ecommerce.db` (SQLite) and runs `createSchema()` to initialize all tables. Executed as a daily cron job.
- **`src/schema.ts`** — Defines all 12 tables via `CREATE TABLE IF NOT EXISTS`. Reference this for column names and constraints when writing queries.
- **`src/queries/`** — All query modules. Each exports async functions accepting a `Database` instance.
- **`sdk.ts`** — Example of using `@anthropic-ai/claude-agent-sdk` (`query()` API). Use as a reference for agent-powered integrations.

## Query Pattern

All query functions follow this pattern (use `db.get()` for single rows, `db.all()` for multiple):

```typescript
export function getCustomerByEmail(db: Database, email: string): Promise<any> {
  return new Promise((resolve, reject) => {
    db.get(`SELECT * FROM customers WHERE email = ?`, [email], (err, row) => {
      if (err) reject(err);
      else resolve(row);
    });
  });
}
```

## Hooks (configured via `.claude/settings.example.json`)

Two hooks run automatically on file writes/edits to `src/queries/`:

- **`hooks/query_hook.js`** (PreToolUse) — Uses Claude agent SDK to detect duplicate query logic. Blocks the write with feedback if a new function duplicates existing functionality. Currently bypassed (`process.exit(0)` at top).
- **`hooks/tsc.js`** (PostToolUse) — Runs TypeScript type-checking after every file write; exits with code 2 on errors, blocking the operation.
- **Prettier** also runs automatically (PostToolUse) on all written/edited files.

To activate hooks, copy `.claude/settings.example.json` to `~/.claude/settings.json` or the project-level `.claude/settings.json`.

## Available Skill

- **`/audit`** — Audits and fixes vulnerable npm dependencies, cross-referencing `.claude/skills/audit/known-issues.md` to skip previously investigated unfixable issues.

## Critical Guidance

- All database queries must be written in `./src/queries/`
- The database file is `ecommerce.db` in the project root (created at runtime)
- TypeScript strict mode is enabled — all new code must pass `tsc --noEmit`
