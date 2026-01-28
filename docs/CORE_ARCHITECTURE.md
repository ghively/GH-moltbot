# Core Architecture

This document details the core architecture of Moltbot, focusing on the Gateway System, Agent Runtime, Channel Integrations, and Tool System.

## Gateway System

The Gateway is the central control plane of the application. It manages network connections, authentication, session state, and event routing.

### Key Components

*   **Server Implementation** (`src/gateway/server.impl.ts`): The main entry point that initializes the HTTP and WebSocket servers. It coordinates the startup of various subsystems like discovery, maintenance timers, and sidecars.
*   **Authentication** (`src/gateway/auth.ts`): Handles security. Supports multiple modes:
    *   **Token**: Simple bearer token.
    *   **Password**: Password-based auth.
    *   **Tailscale**: Integration with Tailscale for identity verification.
*   **Session Management**:
    *   **Runtime State** (`src/gateway/server-runtime-state.ts`): Holds in-memory state of active connections, chat runs, and buffers.
    *   **Session Keys** (`src/gateway/server-session-key.ts`): Resolves unique keys for sessions based on channel, agent, and user context.
*   **Event Routing**:
    *   **WebSocket Handlers** (`src/gateway/server-ws-runtime.ts`): Processes incoming WebSocket messages and routes them to appropriate handlers.
    *   **Methods** (`src/gateway/server-methods.ts`): Defines the API methods available over the gateway protocol.
*   **Discovery**:
    *   **mDNS/Bonjour**: Broadcasts presence to local network for auto-discovery.

### Data Flow

1.  **Connection**: Client connects via WebSocket.
2.  **Auth**: Gateway verifies credentials (token/password/Tailscale).
3.  **Session**: A session context is established.
4.  **Message**: Incoming messages are routed to the `Agent Runtime` or handled directly (e.g., control commands).
5.  **Response**: Responses are sent back via WebSocket or HTTP.

## Agent Runtime

The Agent Runtime is responsible for the intelligence of the bot. It executes the AI loop, manages context, and invokes tools.

### Key Components

*   **Runner** (`src/agents/pi-embedded-runner/run.ts`): The core loop (`runEmbeddedPiAgent`). It manages the interaction with LLMs.
    *   **Context Window Guard**: Monitors token usage and prevents overflows.
    *   **Auth Profiles**: Manages API keys and provider rotation/failover.
    *   **Lanes**: Manages concurrency and task queueing for agents.
*   **Model Abstraction**:
    *   Supports multiple providers (OpenAI, Anthropic, etc.).
    *   Handles "thinking" models (reasoning steps).
*   **Lifecycle**:
    *   **Initialization**: Sets up the agent environment and workspace.
    *   **Execution**: Sends prompts to the LLM, processes responses, and executes tools.
    *   **Failover**: Handles errors and retries with backup providers/models.

### Architecture

The runtime operates as an "embedded" agent, meaning it runs within the same process as the Gateway (or a managed child process). It uses a "snapshot" of the configuration and skills to ensure consistency during a run.

## Channel Integrations

Channels are the interfaces through which the bot communicates with the outside world (e.g., WhatsApp, Telegram).

### Architecture

*   **Registry** (`src/channels/registry.ts`): Maintains a list of available channels and their metadata (labels, icons, documentation paths).
*   **Plugin System**: Channels are implemented as plugins. They register themselves with the Gateway.
*   **Normalization**: The system normalizes channel IDs and aliases (e.g., "gchat" -> "googlechat").

### Standard Channels

*   **WhatsApp**: Uses `@whiskeysockets/baileys`.
*   **Telegram**: Uses `grammy`.
*   **Discord**: Uses `@discordjs/rest` (or similar).
*   **Slack**: Uses `@slack/bolt`.
*   **Signal**: Uses `signal-cli`.

## Tool System

Tools extend the agent's capabilities, allowing it to interact with the system and external APIs.

### Architecture

*   **Tool Definitions**: Tools are defined with schemas (Input/Output) and implementation logic.
*   **Execution**: The Agent Runtime invokes tools based on LLM "tool calls".
*   **Sandbox** (`src/agents/sandbox.ts`): Tools execute within a controlled environment (often with Docker or restricted filesystem access) to ensure security.
*   **Built-in Tools**:
    *   **Browser**: Headless browser control.
    *   **Filesystem**: Read/write access to the workspace.
    *   **Gateway**: Control gateway functions.

### Discovery

Tools are discovered from:
1.  **Built-in set**: Core tools provided by the runtime.
2.  **Skills**: External skills (plugins) that provide additional tools.
