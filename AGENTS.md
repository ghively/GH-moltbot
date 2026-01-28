# Instructions for AI Agents

Welcome, Agent. This file is your primary entry point for working with the Moltbot repository.

## 🧭 Orientation & Documentation

Before making changes, familiarize yourself with the system using the following documentation:

*   **Architecture Overview**: Read [ARCHITECTURE.md](./ARCHITECTURE.md) to understand the system design, components (Gateway, Agent Runtime, Channels), and data flow.
*   **Repository Structure**: Refer to [docs/REPO_STRUCTURE.md](./docs/REPO_STRUCTURE.md) for a map of the directory layout.
*   **Core Architecture Details**: See [docs/CORE_ARCHITECTURE.md](./docs/CORE_ARCHITECTURE.md) for deep dives into specific subsystems.
*   **Developer Guide**: [DEVELOPMENT_GUIDE.md](./DEVELOPMENT_GUIDE.md) covers setup, build scripts, and testing workflows.
*   **API Reference**: [API_REFERENCE.md](./API_REFERENCE.md) documents the Gateway protocol, RPC methods, and configuration.
*   **Extension Development**: [ADDON_DEVELOPMENT.md](./ADDON_DEVELOPMENT.md) explains how to create plugins, channels, and tools.

## 🚀 Quick Start Commands

The project uses `pnpm`.

*   **Install Dependencies**: `pnpm install`
*   **Build Project**: `pnpm build`
*   **Run Linter**: `pnpm lint`
*   **Run Unit Tests**: `pnpm test` (Excludes E2E)
*   **Run E2E Tests**: `pnpm test:e2e`

## 📂 Codebase Navigation

*   **`src/`**: Core logic.
    *   `gateway/`: Server & WebSocket handling.
    *   `agents/`: Agent runtime & tools.
    *   `channels/`: Channel abstractions.
    *   `config/`: Configuration schemas.
*   **`extensions/`**: Channel plugins (WhatsApp, Discord, etc.).
*   **`skills/`**: Capability plugins.

*Tip: Check the `INDEX.md` file in each major directory for a quick content summary.*

## ✅ Verification Requirements

For **every** change you make, you MUST perform the following checks:

1.  **Build**: Ensure `pnpm build` passes without errors.
2.  **Lint**: Run `pnpm lint` and fix any issues.
3.  **Test**:
    *   Run `pnpm test` to verify unit tests.
    *   If you modify critical paths, run `pnpm test:e2e`.
    *   **Always** add or update tests for new functionality.

## 💡 Coding Conventions

*   **TypeScript**: Use strict typing. Avoid `any`.
*   **Style**: Follow existing patterns. Use `pnpm format:fix` to apply formatting.
*   **Documentation**: Add JSDoc comments to all exported functions and classes. Update Markdown documentation if you change architecture or APIs.
