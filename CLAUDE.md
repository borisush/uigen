# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in natural language, Claude AI generates the code using tool calls, and components render instantly in a sandboxed iframe.

## Commands

```bash
# Setup (install deps, generate Prisma client, run migrations)
npm run setup

# Development
npm run dev              # Start with Turbopack
npm run dev:daemon       # Start in background, logs to logs.txt

# Build & Production
npm run build
npm start

# Testing
npm test                 # Run all tests with Vitest
npm test -- --watch      # Watch mode
npm test -- src/lib/__tests__/file-system.test.ts  # Single test file

# Code Quality
npm run lint

# Database
npm run db:reset         # Reset and re-run migrations
npx prisma studio        # GUI for database
```

## Architecture

### Three-Panel Layout
- **Left (35%)**: Chat interface for AI conversation
- **Right Top**: Live preview in sandboxed iframe
- **Right Bottom**: File tree + Monaco code editor

### Core Data Flow
```
User Message → POST /api/chat → Claude AI (streaming)
                                    ↓
                              Tool Calls (str_replace_editor, file_manager)
                                    ↓
                              FileSystemContext.handleToolCall()
                                    ↓
                              VirtualFileSystem updates
                                    ↓
                              UI refresh (Preview, Editor, FileTree)
```

### Virtual File System (`src/lib/file-system.ts`)
- All code lives in-memory, never written to disk
- `serialize()`/`deserialize()` for database persistence
- Entry point is always `/App.jsx`

### AI Provider System (`src/lib/provider.ts`)
- **With `ANTHROPIC_API_KEY`**: Uses Claude (claude-haiku-4-5)
- **Without key**: MockLanguageModel returns static demo components

### AI Tools (`src/lib/tools/`)
- `str_replace_editor`: Create/view/edit files (create, str_replace, insert, view, undo_edit)
- `file_manager`: Rename/delete files

### JSX Transform & Preview (`src/lib/transform/jsx-transformer.ts`)
- Babel standalone compiles JSX in browser
- Creates blob URLs for each file
- Import map resolves local files and external deps (via esm.sh)

### Authentication (`src/lib/auth.ts`)
- JWT stored in HttpOnly cookie (7 days)
- Anonymous users can work; signing up converts their work to a project
- Anonymous work persisted in sessionStorage (`src/lib/anon-work-tracker.ts`)

### Database (Prisma + SQLite)
- `User`: email, password (bcrypt)
- `Project`: name, messages (JSON), data (serialized VirtualFS)

## Key Contexts

- **FileSystemProvider** (`src/lib/contexts/file-system-context.tsx`): Manages virtual file system state, handles AI tool calls
- **ChatProvider** (`src/lib/contexts/chat-context.tsx`): Wraps Vercel AI SDK's `useChat`, manages conversation

## Server Actions (`src/actions/`)

- `signUp`, `signIn`, `signOut`, `getUser` - Auth operations
- `createProject`, `getProject`, `getProjects` - Project CRUD

## Code Style

- Use comments sparingly. Only comment complex code.

## Testing

Tests use Vitest + React Testing Library. Test files are colocated in `__tests__/` directories next to their source files.
