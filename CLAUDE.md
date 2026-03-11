# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Trellis is a multi-platform AI coding framework that provides structured workflows for AI-assisted development. It supports Claude Code, Cursor, OpenCode, iFlow, Codex, Kilo, Kiro, Gemini CLI, Antigravity, and Qoder.

## Common Commands

```bash
# Install dependencies
pnpm install

# Build the project
pnpm build

# Run tests
pnpm test                    # Run all tests
pnpm test:watch             # Watch mode
pnpm test -- <file>         # Run specific test file

# Lint and typecheck
pnpm lint                   # ESLint
pnpm lint:fix               # Fix lint issues
pnpm typecheck              # TypeScript type checking
pnpm lint:py                # Pyright type checking
pnpm lint:all               # Run all linters

# Format
pnpm format                 # Prettier format
pnpm format:check           # Check formatting

# Development
pnpm dev                   # Watch mode for TypeScript
pnpm start                  # Run CLI directly

# Release (maintainers only)
pnpm release                # Patch release
pnpm release:minor          # Minor release
pnpm release:major          # Major release
```

## Architecture

The codebase is organized as follows:

- **`src/cli/`** - Entry point and CLI infrastructure
- **`src/commands/`** - CLI command implementations (init, update, etc.)
- **`src/configurators/`** - Platform-specific setup logic for each supported AI tool
- **`src/templates/`** - Template files used during initialization
- **`src/migrations/`** - Migration scripts for upgrading existing setups
- **`src/types/`** - TypeScript type definitions
- **`src/utils/`** - Utility functions
- **`src/constants/`** - Constants and configuration

The CLI is built with Commander.js. Each platform (cursor, opencode, claude, etc.) has its own configurator in `src/configurators/`.

## Testing

Tests use Vitest and are located in `test/` mirroring the `src/` structure. Test files follow the pattern `*.test.ts`.
