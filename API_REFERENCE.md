# API Reference

This document outlines the available APIs, RPC methods, Events, and Configuration options for Moltbot.

## Gateway Protocol

The Gateway communicates via WebSocket. Messages follow a JSON-RPC 2.0-like structure or a custom event format.

### RPC Methods

The following methods are available to authenticated clients.

#### Core
*   `health`: Get system health status.
*   `status`: Get general gateway status.
*   `logs.tail`: Stream logs.
*   `shutdown`: Initiate shutdown.

#### Configuration
*   `config.get`: Retrieve current configuration.
*   `config.set`: Update configuration.
*   `config.apply`: Apply configuration changes.
*   `config.schema`: Get configuration schema.

#### Channels
*   `channels.status`: Get status of all channels.
*   `channels.logout`: Disconnect a channel.

#### Sessions
*   `sessions.list`: List active sessions.
*   `sessions.preview`: Preview a session transcript.
*   `sessions.reset`: Clear session history.
*   `sessions.delete`: Delete a session.

#### Agents & Models
*   `agents.list`: List available agents.
*   `models.list`: List available models.
*   `agent.wait`: Wait for agent readiness.

#### Pairing & Nodes
*   `node.pair.request`: Request pairing with a node.
*   `node.list`: List connected nodes.
*   `node.invoke`: Invoke a command on a node.

#### Misc
*   `wizard.start`: Start the onboarding wizard.
*   `cron.list`: List scheduled tasks.

### Events (`GATEWAY_EVENTS`)

Clients can subscribe to these events:

*   `connect.challenge`: Auth challenge on connection.
*   `agent`: Agent activity events.
*   `chat`: New chat messages.
*   `presence`: User/Node presence updates.
*   `health`: Health status changes.
*   `heartbeat`: Periodic heartbeat.
*   `shutdown`: System shutting down.

## Configuration

Configuration is managed via `config.yaml` (or similar) and environment variables.

### Key Config Sections (`MoltbotConfig`)

*   **`auth`**: Authentication settings.
*   **`gateway`**: Server settings (port, TLS, bind address).
*   **`agents`**: Agent definitions and bindings.
*   **`channels`**: Configuration for specific channels (WhatsApp, Telegram, etc.).
*   **`models`**: Model provider settings (API keys, aliases).
*   **`tools`**: Global tool policies and settings.
*   **`plugins`**: Plugin specific configuration.
*   **`skills`**: Skill settings.
*   **`ui`**: UI customization (accent color, assistant name/avatar).
*   **`logging`**: Logging verbosity and output.

### Environment Variables

*   `CLAWDBOT_GATEWAY_TOKEN`: Auth token override.
*   `CLAWDBOT_GATEWAY_PORT`: Server port override.
*   `CLAWDBOT_DEBUG`: Enable debug mode.

## CLI Reference

The `moltbot` CLI (via `moltbot.mjs`) supports:

*   `moltbot`: Start the gateway.
*   `moltbot tui`: Start the Terminal User Interface.
*   `moltbot doctor`: Run system diagnostics.
*   `moltbot gateway`: Explicitly run gateway mode.

Run `moltbot --help` for full usage.
