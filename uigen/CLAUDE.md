# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run setup        # First-time setup: install deps, generate Prisma client, run migrations
npm run dev          # Start dev server with Turbopack (uses node-compat.cjs)
npm run build        # Production build
npm run lint         # Run ESLint
npm run test         # Run all Vitest tests
npm run db:reset     # Reset SQLite database (destructive)
```

Run a single test file:
```bash
npx vitest run src/lib/__tests__/file-system.test.ts
```

The app runs without an `ANTHROPIC_API_KEY` — a mock provider returns static components instead.

## Architecture

UIGen is a Next.js 15 (App Router) application where users describe React components in chat and get live previews. The core flow: user message → `/api/chat` → Claude streams text + tool calls → client applies tool calls to virtual file system → iframe renders live preview.

### Three-panel layout (`src/app/main-content.tsx`)
- Left: `ChatInterface` (messages + input)
- Right top: Live preview iframe or code editor tabs
- Right bottom: `FileTree` + Monaco `CodeEditor`

### State management via two React contexts
- **`FileSystemProvider`** (`src/lib/contexts/file-system-context.tsx`): owns the `VirtualFileSystem` instance, processes AI tool calls (`onToolCall` callback), and exposes file CRUD to the UI
- **`ChatProvider`** (`src/lib/contexts/chat-context.tsx`): wraps Vercel AI SDK's `useChat`, serializes the file system and sends it to `/api/chat`, pipes tool call results back to `FileSystemProvider`

### Virtual file system (`src/lib/file-system.ts`)
In-memory tree using Maps. Never writes to disk. Authenticated users get their file system serialized to JSON and saved in the `Project.data` Prisma field. The AI operates on it via two tools exposed from `/api/chat`:
- **`str_replace_editor`**: view, create, replace, and insert into files
- **`file_manager`**: rename and delete files

Every AI-generated project must have `/App.jsx` as the entry point. The AI uses `@/` import alias for local files.

### Live preview (`src/components/preview/PreviewFrame.tsx`)
On every file system change, the preview:
1. Auto-detects `App.jsx`/`App.tsx` as entry
2. Transpiles JSX in-browser via Babel Standalone (`src/lib/transform/jsx-transformer.ts`)
3. Builds an ES module import map for third-party packages
4. Renders into a sandboxed iframe using blob URLs

### AI provider (`src/lib/provider.ts`)
Returns a real Anthropic client (claude-haiku-4-5) when `ANTHROPIC_API_KEY` is set, otherwise returns a mock provider that simulates a 4-step tool call sequence generating a placeholder component.

### Authentication (`src/lib/auth.ts`)
JWT stored in httpOnly cookies (7-day expiry, `jose` library). Server actions in `src/actions/` handle sign-up, sign-in, sign-out, and project CRUD. `src/middleware.ts` protects `/api/projects/*` and `/api/filesystem/*`.

### Database
Prisma with SQLite (`prisma/schema.prisma`). Generated client output: `src/generated/prisma`. Two models: `User` and `Project` (stores messages and file system as JSON strings).

### Anonymous user flow
Unauthenticated users can use the app; work is stored in `sessionStorage` via `src/lib/anon-work-tracker.ts`. On sign-in/sign-up, the anonymous session data (messages + file system) is migrated to the new project.

### System prompt (`src/lib/prompts/generation.tsx`)
Instructs Claude to style with Tailwind (not inline styles), use `/App.jsx` as the entry, and always import local files with the `@/` alias.

## Key conventions
- Path alias `@/*` maps to `src/*`
- Tests use Vitest + React Testing Library with jsdom environment (`vitest.config.mts`)
- UI components are shadcn/ui (new-york style) in `src/components/ui/`
- `"use client"` for interactive components, `"use server"` for server actions
