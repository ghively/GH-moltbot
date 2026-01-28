# Repository Structure Analysis

This document provides a comprehensive map of the Moltbot repository structure.

## Root Directory

The root directory serves as the entry point for the monorepo, containing configuration files and the main source directories.

### Configuration Files

-   **`package.json`**: Defines the project metadata, scripts, and dependencies. It acts as the manifest for the root workspace.
-   **`pnpm-workspace.yaml`**: Defines the pnpm workspace configuration, including member packages (`.`, `ui`, `packages/*`, `extensions/*`).
-   **`tsconfig.json`**: Base TypeScript configuration for the project, targeting ES2022 and NodeNext module resolution.
-   **`vitest.config.ts`** (and variants): Configuration for Vitest testing framework, with specific configs for e2e, unit, and gateway tests.
-   **`moltbot.mjs`**: The CLI entry point for the application.

### Key Directories

#### `/src` - Core Application Code

This directory contains the core logic of the Moltbot application.

-   **`gateway/`**: Implements the Gateway System, including WebSocket control plane, session management, and event handling.
-   **`agents/`**: Contains the Agent Runtime, responsible for agent initialization, tool execution, and session state.
-   **`channels/`**: Defines the channel integration architecture and shared logic for communication channels.
-   **`tools/`** (inside `agents/`): Contains tool definitions and execution logic.
-   **`config/`**: Handles configuration loading, validation, and schema definitions.
-   **`plugins/`**: Plugin system implementation.
-   **`cli/`**: CLI command implementations.

#### `/packages` - Monorepo Packages

Contains shared packages used across the repository.

-   **`clawdbot/`**: A package likely containing shared logic or a legacy component of the system (formerly named Clawdbot).

#### `/apps` - Application Entry Points

Contains platform-specific application wrappers.

-   **`android/`**: Android application project.
-   **`ios/`**: iOS application project.
-   **`macos/`**: macOS application project.
-   **`shared/`**: Shared code for the apps.

#### `/extensions` - Extension System

Contains extensions that integrate external services or add functionality.

-   **Examples**: `discord`, `slack`, `whatsapp`, `google-gemini-cli-auth`, `memory-lancedb`.
-   These extensions likely follow a plugin architecture to extend the core bot's capabilities.

#### `/skills` - Skills/Plugins Architecture

Contains "skills" which are specialized capabilities or tools the agent can use.

-   **Examples**: `github`, `spotify-player`, `weather`, `notion`, `openai-image-gen`.
-   Skills seem to be modular functional units that can be enabled/disabled.

#### `/ui` - Frontend Components

Contains the frontend web application code.

-   **Tech Stack**: Vite, TypeScript, React (implied by typical usage, check `package.json` in `ui` to confirm).
-   **Purpose**: Provides a user interface for interacting with or controlling the bot.

#### `/docs` - Existing Documentation

Contains project documentation in Markdown format.

-   Includes architecture notes, debugging guides, setup instructions, and platform specific docs.

## Dependencies

-   **Internal**: The project uses a workspace structure where packages can depend on each other.
-   **External**: Key dependencies include `@whiskeysockets/baileys` (WhatsApp), `@slack/bolt`, `@grammyjs/runner` (Telegram), `express`, `ws` (WebSockets), `playwright-core` (Browser automation).
