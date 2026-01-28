# Development Guide

This guide covers the setup, build process, and testing workflows for Moltbot developers.

## Prerequisites

*   **Node.js**: Version 22.12.0 or higher.
*   **pnpm**: Version 10.23.0 or higher (managed via `packageManager` in `package.json`).
*   **Git**: For version control.
*   **Docker** (Optional): For running containerized tests or sandboxes.

## Setup

1.  **Clone the repository**:
    ```bash
    git clone <repository-url>
    cd moltbot
    ```

2.  **Install dependencies**:
    ```bash
    pnpm install
    ```
    This will install dependencies for the root workspace and all packages.

## Build System

Moltbot uses a monorepo structure. The build process is managed by `pnpm`.

*   **Build everything**:
    ```bash
    pnpm build
    ```
    This executes the following pipeline:
    1.  `pnpm canvas:a2ui:bundle`: Bundles the A2UI (Agent to UI) components using `rolldown` (if sources are present).
    2.  `tsc -p tsconfig.json`: Compiles the core TypeScript source code to `dist/`.
    3.  `scripts/canvas-a2ui-copy.ts`: Copies the A2UI bundle to the distribution folder.
    4.  `scripts/copy-hook-metadata.ts`: Copies `HOOK.md` documentation to the distribution folder.
    5.  `scripts/write-build-info.ts`: Generates build metadata.

*   **Build UI**:
    ```bash
    pnpm ui:build
    ```
    Builds the frontend application in `ui/`.

*   **Prepack**:
    Runs `pnpm build` and `pnpm ui:build` before packaging.

## Development Workflows

### Running the Gateway

To start the Gateway in development mode (with hot reload):

```bash
pnpm dev
# OR specifically for gateway
pnpm gateway:dev
```

This starts the node process defined in `scripts/run-node.mjs`.

### Running the UI

To start the frontend development server:

```bash
pnpm ui:dev
```

### Type Checking & Linting

*   **Lint**:
    ```bash
    pnpm lint
    ```
    Uses `oxlint` for fast linting.

*   **Format**:
    ```bash
    pnpm format
    ```
    Uses `oxfmt`.

## Testing Strategies

Moltbot uses **Vitest** for testing. There are multiple test configurations for different scopes.

### Unit Tests
Run standard unit tests:
```bash
pnpm test
```
This runs tests matching `src/**/*.test.ts`, excluding e2e tests.

### E2E Tests
Run end-to-end tests:
```bash
pnpm test:e2e
```
These tests verify full flows, often involving the gateway and agent runtime.

### Live Tests
Run live integration tests (requiring real API keys):
```bash
pnpm test:live
```
**Note**: You need to configure environment variables (e.g., `ANTHROPIC_API_KEY`) for these to pass.

### Docker Tests
Run tests involving Docker containers:
```bash
pnpm test:docker:all
```

## Debugging

*   **Logs**: The application uses a structured logging system. Logs are output to stdout/stderr.
*   **Environment Variables**:
    *   `CLAWDBOT_DEBUG=1`: Enable debug logging.
    *   `CLAWDBOT_GATEWAY_PORT`: Override the gateway port.

## Code Style

*   **TypeScript**: Strict mode is enabled. Use types explicitly.
*   **Formatting**: Prettier/oxfmt is used. Run `pnpm format:fix` to auto-format.
*   **Naming**:
    *   Files: `kebab-case.ts`
    *   Classes: `PascalCase`
    *   Functions/Variables: `camelCase`

## Dependency Management

*   **Adding a dependency**:
    ```bash
    pnpm add <package-name>
    ```
    Use `-w` to add to the root workspace if it's a dev tool, or navigate to the specific package/app.

*   **Syncing Plugins**:
    ```bash
    pnpm plugins:sync
    ```
    Ensures plugin versions are consistent.

## Dependencies

*   **Internal Packages**: Packages in `packages/` can be dependencies of the root or other packages.
*   **External Critical Dependencies**:
    *   `@whiskeysockets/baileys`: WhatsApp integration.
    *   `@slack/bolt`: Slack integration.
    *   `ws`: WebSocket communication.
    *   `express`: HTTP server (likely for some endpoints or sidecars).
    *   `playwright-core`: Browser automation for tools.
