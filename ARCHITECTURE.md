# Moltbot Architecture

This document provides a high-level overview of the Moltbot system architecture, designed to help developers understand how the components interact.

## System Overview

Moltbot is a modular, event-driven AI agent platform built on Node.js and TypeScript. It uses a monorepo structure managed by pnpm.

### Key Subsystems

1.  **Gateway System**: The central nervous system. It handles WebSocket connections, authentication, session management, and routes messages between channels and the agent runtime.
2.  **Agent Runtime**: The intelligence layer. It executes the "Pi" agent loop, manages context windows, invokes tools, and interacts with LLM providers (OpenAI, Anthropic, etc.).
3.  **Channel Integrations**: Plugins that connect the bot to external communication platforms like WhatsApp, Telegram, Discord, and Slack.
4.  **Tool System**: A framework for extending the agent's capabilities with tools (coding, browser control, system access) and skills.
5.  **Frontend (UI)**: A React/Vite application for managing and interacting with the bot.

## Repository Structure

*   **`src/`**: Core application code (Gateway, Agent, Channels, Tools).
*   **`packages/`**: Shared internal packages.
*   **`apps/`**: Platform-specific application wrappers (iOS, Android, macOS).
*   **`extensions/`**: Third-party or modular extensions (integrations).
*   **`skills/`**: specialized agent capabilities.
*   **`ui/`**: Web-based control interface.

## Core Components

### Gateway (`src/gateway`)

The Gateway is the entry point. It runs an HTTP/WebSocket server that:
*   **Authenticates** clients (Token, Password, Tailscale).
*   **Manages Sessions**: Tracks active conversations and user contexts.
*   **Routes Events**: Dispatches incoming messages to the Agent Runtime or handles control commands.
*   **Discovery**: Uses mDNS/Bonjour to announce presence on the local network.

### Agent Runtime (`src/agents`)

The Agent Runtime executes the AI logic:
*   **Embedded Runner**: Runs the agent loop within the main process (`runEmbeddedPiAgent`).
*   **Context Management**: Monitors token usage to prevent context overflow.
*   **Auth Profiles**: Rotates API keys and handles provider failover.
*   **Lanes**: Manages concurrency to ensure orderly processing of messages.

### Channels (`src/channels`)

Channels are plugins that normalize external platforms into a standard message format:
*   **Registry**: Maintains a list of active channels.
*   **Plugins**: specific implementations (e.g., WhatsApp via Baileys) are loaded as plugins.

### Tools (`src/agents/tools`)

Tools allow the agent to perform actions:
*   **Sandboxing**: Tools can run in a restricted environment (`src/agents/sandbox.ts`).
*   **Discovery**: Tools are aggregated from core capabilities, plugins, and enabled skills.
*   **Policy**: Granular permissions control which tools can be used in which context.

## Data Flow

### Message Ingestion
1.  **Channel Plugin** receives a message (e.g., from WhatsApp).
2.  **Normalization**: The message is converted to a standard Moltbot format.
3.  **Gateway**: The channel forwards the message to the Gateway.
4.  **Session Resolution**: The Gateway determines the active session or creates a new one.
5.  **Agent Dispatch**: The message is queued in the appropriate Agent Lane.

### Agent Execution
1.  **Prompt Construction**: The runtime builds a prompt including system instructions, conversation history, and available tools.
2.  **LLM Inference**: The prompt is sent to the selected Model Provider (e.g., Anthropic).
3.  **Reasoning**: If using a "thinking" model, reasoning steps are processed.
4.  **Tool Execution**: If the model requests a tool call, the runtime executes it (potentially in a sandbox) and feeds the result back.
5.  **Response**: The final text response is generated.

### Response Delivery
1.  **Gateway**: The response payload is received from the Agent Runtime.
2.  **Channel Plugin**: The Gateway forwards the response to the originating Channel.
3.  **Delivery**: The Channel Plugin sends the message to the user.

## State Management

*   **In-Memory**: Active sessions, chat buffers, and connection state are held in memory.
*   **Filesystem**: Session transcripts, configuration, and logs are persisted to disk.
*   **Databases**: Some extensions (e.g., `memory-lancedb`) use vector databases for long-term recall.

## Concurrency

*   **Lanes**: The system uses "Lanes" to serialize operations within a specific scope (e.g., a single session) while allowing parallel processing across different sessions.
