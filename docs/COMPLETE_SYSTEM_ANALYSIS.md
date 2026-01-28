# Moltbot Complete System Analysis & Documentation Database

> **Comprehensive architectural analysis of the Moltbot codebase**
> Generated: 2025-01-28
> Version: 2026.1.27-beta.1

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture Layers](#architecture-layers)
3. [Core Components](#core-components)
4. [Data Flow](#data-flow)
5. [Entry Points](#entry-points)
6. [Gateway System](#gateway-system)
7. [Agent Runtime](#agent-runtime)
8. [Tools System](#tools-system)
9. [Channel System](#channel-system)
10. [Plugin System](#plugin-system)
11. [Configuration System](#configuration-system)
12. [Authentication & Security](#authentication--security)
13. [Concurrency Model](#concurrency-model)
14. [State Management](#state-management)
15. [Frontend Interfaces](#frontend-interfaces)
16. [Extensions Directory](#extensions-directory)
17. [Skills Directory](#skills-directory)
18. [Testing Strategy](#testing-strategy)
19. [Development Workflow](#development-workflow)
20. [Customization Guide](#customization-guide)
21. [Technology Stack](#technology-stack)
22. [File Reference](#file-reference)

---

## System Overview

### Purpose

**Moltbot** is a personal AI assistant platform that:
- Runs on your own hardware (local-first)
- Answers across multiple communication channels
- Supports single-user scenarios (not multi-tenant)
- Emphasizes privacy and data ownership
- Provides extensible architecture through plugins

### Key Characteristics

| Attribute | Description |
|-----------|-------------|
| **Type** | Personal AI Assistant |
| **Architecture** | Event-driven gateway + embedded agent runtime |
| **Channels** | 7 built-in + 30+ extension channels |
| **Platforms** | macOS, Linux, Windows/WSL2, iOS, Android |
| **Runtime** | Node.js 22+ with TypeScript |
| **Protocol** | WebSocket + HTTP (REST) |
| **Agent Framework** | Pi Agent (@mariozechner/pi-agent-core) |
| **License** | MIT |

### Supported Channels

**Built-in (Core):**
1. Telegram (Bot API)
2. WhatsApp (QR link via Baileys)
3. Discord (Bot API)
4. Google Chat (Chat API webhook)
5. Slack (Socket Mode)
6. Signal (signal-cli)
7. iMessage (WIP)

**Extensions:**
Matrix, Mattermost, Nextcloud Talk, Twitch, LINE, Zalo, Zalo Personal, BlueBubbles, Nostr, Tlon, MS Teams, and 20+ more.

### Project Statistics

| Metric | Count |
|--------|-------|
| TypeScript Source Files | 1,000+ |
| Extensions | 30+ |
| Skills | 50+ |
| Built-in Tools | 60+ |
| RPC Methods | 30+ |
| Test Files | 200+ |
| Lines of Code | ~150,000+ |

---

## Architecture Layers

The system is organized into 8 distinct layers, from low-level entry points to user interfaces.

```
┌─────────────────────────────────────────────────────────┐
│ Layer 8: Frontend Interfaces                           │
│ (Web UI, TUI, Mobile Apps)                             │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│ Layer 7: Configuration System                          │
│ (YAML config, environment variables, validation)       │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│ Layer 6: Plugin System                                 │
│ (Channel plugins, skill plugins, tool plugins)         │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│ Layer 5: Tools System                                  │
│ (60+ built-in tools, execution approval)               │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│ Layer 4: Channel System                                │
│ (7 built-in + 30+ extensions, message normalization)   │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│ Layer 3: Agent Runtime                                 │
│ (Pi agent, embedded execution, tool calling)           │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│ Layer 2: Gateway System (Control Plane)                │
│ (WebSocket, HTTP, RPC, session management)            │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│ Layer 1: Entry Points                                  │
│ (CLI, commands, routing)                               │
└─────────────────────────────────────────────────────────┘
```

---

## Core Components

### Component Interaction Diagram

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Channels   │────▶│   Gateway    │────▶│   Agent      │
│ (Plugins)    │     │  (Server)    │     │  (Runtime)   │
└──────────────┘     └──────────────┘     └──────────────┘
       │                     │                     │
       │                     │                     │
       ▼                     ▼                     ▼
  ┌─────────┐         ┌─────────┐         ┌─────────┐
  │ External │         │ Clients │         │   LLM   │
  │ Services │         │ (WS/HTTP)│       │ APIs    │
  └─────────┘         └─────────┘         └─────────┘
```

### Core Component Responsibilities

| Component | Responsibilities | Key Files |
|-----------|------------------|-----------|
| **CLI** | Command parsing, routing, execution | `src/cli/` |
| **Gateway** | WebSocket server, HTTP server, session management | `src/gateway/` |
| **Agent Runtime** | LLM interaction, tool execution, context management | `src/agents/` |
| **Channels** | External platform integration, message normalization | `src/channels/`, `extensions/` |
| **Tools** | Agent capabilities, command execution | `src/agents/tools/` |
| **Plugins** | Dynamic loading of extensions and skills | `src/plugins/` |
| **Config** | Configuration loading, validation, migration | `src/config/` |
| **Frontend** | User interfaces (web, terminal, mobile) | `ui/`, `src/tui/`, `apps/` |

---

## Data Flow

### Complete Message Lifecycle

```
┌──────────────────────────────────────────────────────────────────┐
│ STEP 1: Message Ingestion                                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  User sends message via WhatsApp/Telegram/etc                   │
│                    ↓                                             │
│  Channel plugin receives raw message                            │
│                    ↓                                             │
│  Message normalized to standard format                          │
│                    ↓                                             │
│  Channel forwards to Gateway WebSocket                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 2: Gateway Processing                                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Gateway receives WebSocket message                             │
│                    ↓                                             │
│  Authentication check (token/password/Tailscale)                │
│                    ↓                                             │
│  Session resolution (find or create session)                    │
│                    ↓                                             │
│  Message queued in agent lane                                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 3: Agent Execution                                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Session lane acquired (prevents concurrent processing)          │
│                    ↓                                             │
│  Global lane acquired (system-wide serialization)               │
│                    ↓                                             │
│  Context window guard check (token budget)                      │
│                    ↓                                             │
│  Model resolution (provider/model selection)                    │
│                    ↓                                             │
│  Auth profile selection (API key with rotation)                 │
│                    ↓                                             │
│  Prompt construction:                                           │
│    - System instructions                                        │
│    - Conversation history                                       │
│    - Available tools                                            │
│    - Context compaction (if needed)                             │
│                    ↓                                             │
│  LLM inference (API call to model provider)                     │
│                    ↓                                             │
│  Response parsing:                                              │
│    - If tool calls: execute tools → feed back to LLM            │
│    - If text response: continue to delivery                     │
│                    ↓                                             │
│  Final response generation                                      │
│                    ↓                                             │
│  Lane release                                                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 4: Response Delivery                                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Gateway receives response from agent                           │
│                    ↓                                             │
│  Format conversion (markdown/plain for channel)                 │
│                    ↓                                             │
│  Channel lookup (find originating channel)                      │
│                    ↓                                             │
│  Delivery via channel plugin                                    │
│                    ↓                                             │
│  Session transcript updated                                     │
│                    ↓                                             │
│  Event broadcast to connected clients                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 5: User Receives Response                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Message delivered to user's device                             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Error Handling & Failover Flow

```
Agent execution attempt
        ↓
   Error occurs?
        ├─ No → Continue normally
        └─ Yes → Classify error type
                ↓
        ┌──────┴──────┐
        │             │
   Rate Limit?   API Error?
        │             │
        ↓             ↓
 Switch auth   Try fallback
   profile     model
        │             │
        └──────┬──────┘
               ↓
         Retry attempt
               ↓
         Success?
          ├─ Yes → Continue
          └─ No → Report error
```

---

## Entry Points

### CLI Entry Point Flow

```
moltbot.mjs (shebang script)
    ↓
src/entry.ts
    - Process title setup
    - Warning filter installation
    - Experimental warning suppression (with respawn)
    - Windows argv normalization
    ↓
src/cli/run-main.ts
    - dotenv loading
    - Environment normalization
    - Runtime check (Node version)
    - CLI routing (tryRouteCli)
    - Commander.js program building
    - Plugin registration
    - Argument parsing
```

### File: `src/entry.ts`

**Purpose:** Main entry point for all CLI execution

**Key Responsibilities:**
- Enable Node.js compile cache
- Install process warning filters
- Ensure experimental warnings are suppressed (respawns if needed)
- Normalize Windows argv (removes node.exe duplicates)
- Parse CLI profile arguments
- Route to CLI runner

**Code Flow:**
```typescript
1. Enable compile cache (if available)
2. Install warning filter
3. Check if experimental warning suppression needed
4. If needed: respawn process with --disable-warning=ExperimentalWarning
5. Normalize Windows argv (remove node.exe references)
6. Parse CLI profile arguments (--profile)
7. Import and run CLI
```

### File: `src/cli/run-main.ts`

**Purpose:** Main CLI runner with routing

**Key Responsibilities:**
- Load environment configuration
- Attempt command routing (bypasses full Commander.js)
- Fallback to Commander.js if routing fails
- Register plugin CLI commands
- Handle errors gracefully

**Routing System:**
- `tryRouteCli()` attempts direct command execution
- Finds routed commands via registry
- Prepares command context (config, plugins)
- Returns true if routed, false if fallback needed
- Faster than full Commander.js initialization

### File: `src/cli/program.ts`

**Purpose:** Commander.js program building

**Exports:**
- `buildProgram()` - Constructs the CLI program
- Delegates to `src/cli/program/build-program.js`

### Available CLI Commands

| Command | Description | Source |
|---------|-------------|--------|
| `moltbot` | Start gateway (default) | `src/cli/gateway-cli.ts` |
| `moltbot gateway` | Explicit gateway mode | `src/cli/gateway-cli.ts` |
| `moltbot tui` | Terminal UI | `src/cli/tui-cli.ts` |
| `moltbot agent` | Run agent in CLI mode | `src/cli/node-cli.ts` |
| `moltbot message` | Send messages | `src/cli/` |
| `moltbot channels` | Channel management | `src/cli/channels-cli.ts` |
| `moltbot config` | Configuration | `src/cli/config-cli.ts` |
| `moltbot models` | Model management | `src/cli/models-cli.ts` |
| `moltbot skills` | Skill management | `src/cli/skills-cli.ts` |
| `moltbot doctor` | Diagnostics | `src/commands/` |
| `moltbot onboard` | Onboarding wizard | `src/wizard/` |

---

## Gateway System

### Overview

The Gateway is the **central control plane** for Moltbot. It manages:
- WebSocket connections for real-time communication
- HTTP server for REST APIs
- Authentication and authorization
- Session management
- Event routing and broadcasting
- Channel lifecycle
- Plugin management
- Discovery (mDNS/Bonjour)

### File: `src/gateway/server.impl.ts`

**Function:** `startGatewayServer(port, opts)`

**Startup Sequence:**

```typescript
1. Set port in environment (CLAWDBOT_GATEWAY_PORT)
2. Read config snapshot
3. Migrate legacy config (if needed)
4. Resolve gateway runtime configuration
5. Initialize sidecars:
   - Browser control server
   - Gmail watcher
   - Internal hooks
   - Channels
   - Plugin services
6. Create runtime state
7. Load plugins
8. Create channel manager
9. Start HTTP server
10. Start WebSocket server
11. Start discovery service (mDNS)
12. Start maintenance timers
13. Return GatewayServer handle
```

**GatewayServer Type:**
```typescript
type GatewayServer = {
  close: (opts?: {
    reason?: string;
    restartExpectedMs?: number | null;
  }) => Promise<void>;
};
```

### Gateway Components

| Component | File | Purpose |
|-----------|------|---------|
| **Server Implementation** | `server.impl.ts` | Main server orchestration |
| **WebSocket Runtime** | `server-ws-runtime.ts` | WebSocket connection handling |
| **HTTP Server** | `server-http.ts` | REST API endpoints |
| **Authentication** | `auth.ts` | Token/password/Tailscale auth |
| **Session Management** | `server-runtime-state.ts` | In-memory state |
| **Channel Management** | `server-channels.ts` | Channel lifecycle |
| **Discovery** | `server-discovery-runtime.ts` | mDNS/Bonjour announcement |
| **Methods** | `server-methods/*.ts` | RPC implementations |
| **Cron Service** | `server-cron.ts` | Scheduled tasks |
| **Node Registry** | `node-registry.ts` | Connected nodes |
| **Exec Approval** | `exec-approval-manager.ts` | Command approval system |
| **Hooks** | `hooks.ts` | Event hooks |
| **Tailscale** | `server-tailscale.ts` | Tailscale integration |

### Gateway Options

```typescript
type GatewayServerOptions = {
  // Bind address policy
  bind?: "loopback" | "lan" | "tailnet" | "auto";

  // Advanced host override
  host?: string;

  // Feature toggles
  controlUiEnabled?: boolean;
  openAiChatCompletionsEnabled?: boolean;
  openResponsesEnabled?: boolean;

  // Configuration overrides
  auth?: GatewayAuthConfig;
  tailscale?: GatewayTailscaleConfig;

  // Test options
  allowCanvasHostInTests?: boolean;
  wizardRunner?: (opts, runtime, prompter) => Promise<void>;
};
```

### WebSocket Protocol

**Connection Flow:**
```
Client connects → wss://gateway:port
        ↓
Authentication challenge
        ↓
Client sends credentials
        ↓
Gateway validates
        ↓
Connection established
        ↓
Client can send:
  - RPC method calls
  - Event subscriptions
  - Chat messages
        ↓
Gateway broadcasts:
  - Agent events
  - Chat messages
  - Presence updates
  - Health status
```

### RPC Methods

**Location:** `src/gateway/server-methods/`

| Category | Methods | Description |
|----------|---------|-------------|
| **System** | `health`, `status`, `shutdown` | System operations |
| **Config** | `config.get`, `config.set`, `config.apply`, `config.schema` | Config management |
| **Sessions** | `sessions.list`, `sessions.preview`, `sessions.reset`, `sessions.delete` | Session CRUD |
| **Channels** | `channels.status`, `channels.logout` | Channel operations |
| **Agents** | `agents.list`, `agent.wait` | Agent management |
| **Models** | `models.list` | Model listing |
| **Nodes** | `node.pair.request`, `node.list`, `node.invoke` | Node operations |
| **Tools** | `tools.list`, `tools.exec` | Tool operations |
| **Skills** | `skills.list`, `skills.enable`, `skills.disable` | Skill management |
| **Chat** | `chat.send`, `chat.history` | Chat operations |
| **Logs** | `logs.tail` | Log streaming |
| **Cron** | `cron.list` | Scheduled tasks |
| **Wizard** | `wizard.start` | Onboarding |
| **Browser** | `browser.*` | Browser control |
| **Exec Approval** | `exec-approvals.*` | Command approval |
| **Usage** | `usage.stats` | Usage tracking |

**Method Signature Pattern:**
```typescript
async function methodName(
  params: ParamsType,
  context: RequestContext
): Promise<ResultType>;
```

### Gateway Events

**Location:** `src/gateway/` (GATEWAY_EVENTS)

Clients can subscribe to:

| Event | Payload | Description |
|-------|---------|-------------|
| `connect.challenge` | Auth challenge | Sent on connection |
| `agent` | AgentEvent | Agent activity |
| `chat` | ChatMessage | New chat messages |
| `presence` | PresenceUpdate | User/node presence |
| `health` | HealthStatus | Health changes |
| `heartbeat` | Heartbeat | Periodic heartbeat |
| `shutdown` | ShutdownInfo | System shutting down |

### Gateway Startup Sidecars

**File:** `src/gateway/server-startup.ts`

Sidecars started during gateway initialization:

| Sidecar | Purpose | Trigger |
|---------|---------|---------|
| **Browser Control** | Headless browser automation | `gateway.browser.enabled` |
| **Gmail Watcher** | Email-triggered hooks | `hooks.gmail.account` set |
| **Internal Hooks** | Event handlers | `hooks.internal.enabled` |
| **Channels** | Communication platforms | Configured in config |
| **Plugin Services** | Extension services | Plugins loaded |

---

## Agent Runtime

### Overview

The Agent Runtime is the **intelligence layer** of Moltbot, built on the Pi Agent Framework.

### File: `src/agents/pi-embedded-runner/run.ts`

**Function:** `runEmbeddedPiAgent(params)`

**Purpose:** Execute an embedded Pi agent run

**Core Flow:**

```typescript
1. Resolve lanes (session + global)
2. Acquire session lane lock
3. Acquire global lane lock
4. Resolve workspace directory
5. Resolve model (provider/model)
6. Evaluate context window guard
7. Resolve auth profile (API key)
8. Build payloads (system prompt, history, tools)
9. Run attempt:
    a. Send to LLM
    b. Parse response
    c. Execute tools (if any)
    d. Generate final response
10. Handle failover (if needed)
11. Release locks
12. Return result
```

### Agent Parameters

```typescript
type RunEmbeddedPiAgentParams = {
  // Session identification
  sessionId: string;
  sessionKey?: string;

  // Input
  prompt: string;
  image?: Buffer;

  // Model selection
  provider?: string;
  model?: string;

  // Configuration
  config?: MoltbotConfig;
  agentDir?: string;
  workspaceDir?: string;

  // Runtime
  lane?: string;
  enqueue?: (task, opts) => Promise<void>;

  // Message channel
  messageChannel?: string;
  messageProvider?: string;

  // Output format
  toolResultFormat?: "markdown" | "plain";

  // Skills
  skillsSnapshot?: SkillsSnapshot;
};
```

### Agent Result

```typescript
type EmbeddedPiRunResult = {
  // Responses from the agent
  payloads: EmbeddedPiPayload[];

  // Execution metadata
  meta: EmbeddedPiAgentMeta;

  // Final assistant message
  assistantMessage?: string;

  // Usage statistics
  usage?: UsageLike;
};
```

### Pi Agent Components

| Component | File | Purpose |
|-----------|------|---------|
| **Runner** | `run.ts` | Main execution loop |
| **Lanes** | `lanes.ts` | Concurrency management |
| **Model** | `model.ts` | Model abstraction |
| **History** | `history.ts` | Conversation history |
| **Compact** | `compact.ts` | Context compaction |
| **System Prompt** | `system-prompt.ts` | Prompt construction |
| **Payloads** | `run/payloads.ts` | Request/response building |
| **Attempt** | `run/attempt.ts` | Single execution attempt |
| **Images** | `run/images.ts` | Image handling |
| **Abort** | `abort.ts` | Run abortion |
| **Runs** | `runs.ts` | Run management |
| **Session Manager** | `session-manager-*.ts` | Session lifecycle |
| **Cache TTL** | `cache-ttl.ts` | Cache management |
| **Extensions** | `extensions.ts` | Extension handling |

### Context Window Management

**File:** `src/agents/context-window-guard.ts`

**Purpose:** Prevent context overflow and manage compaction

**Key Constants:**
```typescript
const CONTEXT_WINDOW_HARD_MIN_TOKENS = 2000;  // Absolute minimum
const CONTEXT_WINDOW_WARN_BELOW_TOKENS = 8000; // Warning threshold
const DEFAULT_CONTEXT_TOKENS = 200000;         // Claude default
```

**Evaluation:**
```typescript
type ContextWindowInfo = {
  tokens: number;
  source: "config" | "model" | "default";
};

type ContextWindowGuard = {
  info: ContextWindowInfo;
  shouldWarn: boolean;  // Below 8000 tokens
  shouldBlock: boolean; // Below 2000 tokens
};
```

**Compaction Strategy:**
When context exceeds limit:
1. Remove oldest messages
2. Summarize if needed
3. Keep system prompt
4. Preserve recent context

### Auth Profile System

**File:** `src/agents/auth-profiles.ts`

**Purpose:** Manage API keys with rotation and failover

**Profile Structure:**
```typescript
type AuthProfile = {
  id: string;
  provider: string;
  apiKey: string;
  baseUrl?: string;
  lastUsed?: number;
  lastGood?: number;     // Last successful use
  failures?: number;     // Consecutive failure count
  cooldownUntil?: number; // Rate limit cooldown
};
```

**Rotation Strategy:**
1. **LastUsed优先** - Prefer recently used profiles
2. **Round-robin** - If no lastUsed, rotate evenly
3. **Cooldown respect** - Skip profiles in cooldown
4. **Failure tracking** - Track consecutive failures
5. **Fallback** - Use backup models if configured

**Profile Storage:**
```
~/.moltbot/config/auth-profiles.json
```

### Model Resolution

**File:** `src/agents/model-selection.ts`

**Hierarchy:**
```
1. Explicit provider/model in params
2. Configured agent model
3. Configured default model
4. Hardcoded default (anthropic/claude-sonnet-4.5)
```

**Model Catalog:**
```typescript
type ModelCatalogEntry = {
  provider: string;
  model: string;
  contextWindow?: number;
  supportsImages?: boolean;
  supportsThinking?: boolean;
  maxThinking?: number;
};
```

### Failover Handling

**File:** `src/agents/failover-error.ts`

**FailoverError:**
```typescript
class FailoverError extends Error {
  reason: FailoverReason;
  fallbackModel?: string;
}

type FailoverReason =
  | "rate_limit"
  | "api_error"
  | "auth_error"
  | "context_overflow"
  | "timeout"
  | "unknown";
```

**Failover Flow:**
```
Error during execution
        ↓
Classify error type
        ↓
Is failover enabled?
        ├─ No → Throw error
        └─ Yes → Check fallbacks
                ↓
        Fallback model configured?
                ├─ No → Throw error
                └─ Yes → Switch to fallback
                        ↓
                Retry with fallback
                        ↓
                Success?
                        ├─ Yes → Return result
                        └─ No → Throw error
```

---

## Tools System

### Overview

Tools are the **capabilities** the agent can invoke. 60+ built-in tools are available, with more provided by skills.

### File: `src/agents/tools/`

**Tool Definition Pattern:**
```typescript
export const toolName: AgentTool = {
  name: "tool_name",
  label: "Tool Label",
  description: "What this tool does",
  parameters: TypeBox.Schema,
  execute: async (toolCallId, args, context) => {
    // Tool logic
    return {
      content: [
        { type: "text", text: "result" }
      ]
    };
  }
};
```

### Tool Categories

#### Session & Agent Tools

| Tool | File | Purpose |
|------|------|---------|
| `sessions-list-tool` | `sessions-list-tool.ts` | List active sessions |
| `sessions-send-tool` | `sessions-send-tool.ts` | Send message to session |
| `sessions-spawn-tool` | `sessions-spawn-tool.ts` | Spawn subagent |
| `sessions-history-tool` | `sessions-history-tool.ts` | View session history |
| `session-status-tool` | `session-status-tool.ts` | Get session status |
| `agents-list-tool` | `agents-list-tool.ts` | List available agents |
| `agent-step` | `agent-step.ts` | Step through agent execution |

#### Communication Tools

| Tool | File | Purpose |
|------|------|---------|
| `message-tool` | `message-tool.ts` | Generic messaging |
| `whatsapp-actions` | `whatsapp-actions.ts` | WhatsApp-specific |
| `telegram-actions` | `telegram-actions.ts` | Telegram-specific |
| `discord-actions` | `discord-actions.ts` | Discord actions |
| `discord-actions-guild` | `discord-actions-guild.ts` | Discord guild management |
| `discord-actions-moderation` | `discord-actions-moderation.ts` | Discord moderation |
| `discord-actions-messaging` | `discord-actions-messaging.ts` | Discord messaging |
| `slack-actions` | `slack-actions.ts` | Slack actions |

#### Web & Browser Tools

| Tool | File | Purpose |
|------|------|---------|
| `browser-tool` | `browser-tool.ts` | Browser automation |
| `web-fetch` | `web-fetch.ts` | HTTP fetching (SSRF protected) |
| `web-search` | `web-search.ts` | Web search |
| `web-tools` | `web-tools.ts` | Web utilities |

#### Utility Tools

| Tool | File | Purpose |
|------|------|---------|
| `image-tool` | `image-tool.ts` | Image processing |
| `memory-tool` | `memory-tool.ts` | Memory/knowledge storage |
| `cron-tool` | `cron-tool.ts` | Scheduled task management |
| `tts-tool` | `tts-tool.ts` | Text-to-speech |
| `canvas-tool` | `canvas-tool.ts` | Canvas/A2UI interaction |
| `nodes-tool` | `nodes-tool.ts` | Node management |

#### System Tools

| Tool | File | Purpose |
|------|------|---------|
| `gateway-tool` | `gateway-tool.ts` | Gateway control |
| `gateway` | `gateway.ts` | Gateway operations |

### Tool Execution

**Execution Flow:**
```
Agent requests tool call
        ↓
Tool validation (schema check)
        ↓
Approval required?
        ├─ Yes → Create approval request
        │         ↓
        │     Wait for user decision
        │         ↓
        │     Approved?
        │         ├─ No → Return error
        │         └─ Yes → Continue
        └─ No → Continue
        ↓
Execute tool
        ↓
Return result
        ↓
Feed back to agent
```

### Execution Approval

**File:** `src/gateway/exec-approval-manager.ts`

**Approval Request:**
```typescript
type ExecApprovalRequestPayload = {
  command: string;
  cwd?: string;
  host?: string;
  security?: string;
  ask?: string;
  agentId?: string;
  resolvedPath?: string;
  sessionKey?: string;
};
```

**Approval Record:**
```typescript
type ExecApprovalRecord = {
  id: string;  // UUID
  request: ExecApprovalRequestPayload;
  createdAtMs: number;
  expiresAtMs: number;
  resolvedAtMs?: number;
  decision?: "approve" | "deny";
  resolvedBy?: string;
};
```

**Timeouts:**
- Default: 5 minutes
- Configurable per tool type

### Tool Discovery

Tools are discovered from:
1. **Built-in tools** - `src/agents/tools/`
2. **Skill tools** - Enabled skills
3. **Plugin tools** - Loaded plugins
4. **Channel tools** - Active channels

**Discovery Order:**
```
Built-in → Skills → Plugins → Channels
```

---

## Channel System

### Overview

Channels are **integrations with communication platforms**. They normalize different platforms into a standard message format.

### File: `src/channels/registry.ts`

**Channel Metadata:**
```typescript
type ChannelMeta = {
  id: string;
  label: string;
  selectionLabel: string;
  detailLabel: string;
  docsPath: string;
  docsLabel: string;
  blurb: string;
  systemImage: string;  // SF Symbol name
  selectionDocsPrefix?: string;
  selectionDocsOmitLabel?: boolean;
  selectionExtras?: string[];
};
```

### Built-in Channels

| Channel | ID | Description |
|---------|-------|-------------|
| **Telegram** | `telegram` | Bot API via @BotFather |
| **WhatsApp** | `whatsapp` | QR link via Baileys |
| **Discord** | `discord` | Bot API |
| **Google Chat** | `googlechat` | Chat API webhook |
| **Slack** | `slack` | Socket Mode |
| **Signal** | `signal` | signal-cli |
| **iMessage** | `imessage` | Apple Messages (WIP) |

**Default Channel:** WhatsApp

**Channel Order:** Telegram → WhatsApp → Discord → Google Chat → Slack → Signal → iMessage

### Channel Aliases

```typescript
const CHANNEL_ALIASES: Record<string, ChannelId> = {
  "imsg": "imessage",
  "google-chat": "googlechat",
  "gchat": "googlechat",
};
```

### Channel Plugin Interface

```typescript
type ChannelPlugin = {
  id: string;
  meta: ChannelMeta;
  connect: (ctx: ChannelContext) => Promise<ConnectionHandle>;
  disconnect: (ctx: ChannelContext) => Promise<void>;
};
```

**ChannelContext:**
```typescript
type ChannelContext = {
  config: ChannelConfig;
  gateway: GatewayHandle;
  log: Logger;
  workspaceDir: string;
};
```

### Message Normalization

All channels normalize messages to:

```typescript
type NormalizedMessage = {
  // Source
  channelId: string;
  conversationId: string;

  // Sender
  sender: {
    id: string;
    name?: string;
    avatar?: string;
  };

  // Content
  body: string;
  attachments?: Attachment[];

  // Metadata
  timestamp: number;
  replyTo?: string;
};
```

### Extension Channels

**Location:** `extensions/`

| Channel | Description |
|---------|-------------|
| `matrix` | Matrix protocol |
| `mattermost` | Mattermost |
| `nextcloud-talk` | Nextcloud Talk |
| `twitch` | Twitch |
| `line` | LINE |
| `zalo` | Zalo |
| `zalouser` | Zalo Personal |
| `bluebubbles` | BlueBubbles |
| `nostr` | Nostr protocol |
| `tlon` | Tlon |
| `msteams` | Microsoft Teams |

---

## Plugin System

### Overview

The plugin system enables **dynamic loading of extensions** (channels, skills, tools).

### File: `src/plugins/loader.ts`

**Plugin Types:**
1. **Channel plugins** - Communication platforms
2. **Skill plugins** - Agent capabilities
3. **Tool plugins** - Individual tools
4. **Hook plugins** - Event handlers

### Plugin Discovery

**Discovery Order:**
```
1. Bundled plugins (src/plugins/*)
2. Extension plugins (extensions/*)
3. User plugins (workspace/plugins/*)
```

**Plugin Manifest:**
```typescript
type PluginManifest = {
  id: string;
  name: string;
  version: string;
  description?: string;
  type: "channel" | "skill" | "tool" | "hook";
  main: string;
  configSchema?: object;
  dependencies?: string[];
};
```

### Plugin Registry

**File:** `src/plugins/registry.ts`

**Registry Structure:**
```typescript
type PluginRegistry = {
  channels: PluginEntry[];
  skills: PluginEntry[];
  tools: PluginEntry[];
  hooks: PluginEntry[];
};
```

**PluginEntry:**
```typescript
type PluginEntry = {
  plugin: Plugin;
  manifest: PluginManifest;
  state: "loaded" | "enabled" | "disabled" | "error";
  error?: Error;
};
```

### Plugin Services

**File:** `src/plugins/services.ts`

**Purpose:** Host services for plugins

**Services Provided:**
- Configuration access
- Logging
- Workspace access
- Gateway communication
- Tool registration

### Plugin Lifecycle

```
Discovery
    ↓
Validation (schema check)
    ↓
Loading (dynamic import)
    ↓
Registration (add to registry)
    ↓
Enable (if configured)
    ↓
Initialization (plugin.init)
    ↓
Active (plugin operational)
    ↓
Disable (on shutdown)
    ↓
Unload (remove from registry)
```

---

## Configuration System

### Overview

Configuration is **YAML-based** with environment variable substitution and validation.

### File: `src/config/config.ts`

**Main Loader:**
```typescript
function loadConfig(): MoltbotConfig {
  // Load from file
  // Substitute env vars
  // Validate against schema
  // Apply runtime overrides
  // Migrate legacy config
}
```

### Config Locations

| Platform | Config Path |
|----------|-------------|
| **macOS** | `~/.moltbot/config.yaml` |
| **Linux** | `~/.moltbot/config.yaml` |
| **Windows** | `%USERPROFILE%\.moltbot\config.yaml` |

### Environment Variable Substitution

**Syntax:**
```yaml
apiKey: ${ANTHROPIC_API_KEY}
port: ${PORT:-18789}  # Default value
```

**Supported Formats:**
- `${VAR_NAME}` - Required
- `${VAR_NAME:-default}` - With default
- `${VAR_NAME:?error}` - Fail if missing

### Config Schema

**File:** `src/config/types.ts`

**Root Schema:**
```typescript
type MoltbotConfig = {
  // Gateway settings
  gateway: GatewayConfig;

  // Agent settings
  agents: AgentsConfig;

  // Model catalog
  models: ModelsConfig;

  // Channel configs
  channels: ChannelsConfig;

  // Skills
  skills: SkillsConfig;

  // Tools
  tools: ToolsConfig;

  // Hooks
  hooks: HooksConfig;

  // UI settings
  ui: UiConfig;

  // Logging
  logging: LoggingConfig;

  // Cron jobs
  cron?: CronConfig;
};
```

### Gateway Config

**File:** `src/config/types.gateway.ts`

```typescript
type GatewayConfig = {
  // Server
  port?: number;
  bind?: "loopback" | "lan" | "tailnet" | "auto";
  host?: string;

  // Authentication
  auth: GatewayAuthConfig;

  // Control UI
  controlUi?: {
    enabled?: boolean;
  };

  // HTTP endpoints
  http?: {
    endpoints?: {
      chatCompletions?: { enabled?: boolean };
      responses?: { enabled?: boolean };
    };
  };

  // Tailscale
  tailscale?: GatewayTailscaleConfig;

  // Remote transport
  remote?: GatewayRemoteConfig;
};
```

### Agents Config

**File:** `src/config/types.agents.ts`

```typescript
type AgentsConfig = {
  defaults: {
    model?: string;
    provider?: string;
    context?: number;
    thinking?: "none" | "low" | "medium" | "high";
    temperature?: number;
    maxTokens?: number;

    // Fallbacks
    fallbacks?: FallbackConfig[];

    // Concurrency
    concurrency?: number;

    // Limits
    limits?: AgentLimitsConfig;
  };

  // Multiple agents
  agents?: Record<string, AgentConfig>;

  // Profiles
  profiles?: AuthProfileConfig[];
};
```

### Models Config

**File:** `src/config/types.models.ts`

```typescript
type ModelsConfig = {
  catalog: Record<string, ModelCatalogEntry>;
  aliases?: Record<string, string>;
};
```

**Catalog Entry:**
```typescript
type ModelCatalogEntry = {
  provider: string;
  model: string;

  // Capabilities
  contextWindow?: number;
  supportsImages?: boolean;
  supportsThinking?: boolean;
  maxThinking?: number;

  // API
  baseUrl?: string;
  headers?: Record<string, string>;
};
```

### Legacy Migration

**Files:**
- `src/config/legacy-migrate.ts`
- `src/config/legacy.migrations.part-1.ts`
- `src/config/legacy.migrations.part-2.ts`
- `src/config/legacy.migrations.part-3.ts`

**Migration Rules:**
```typescript
type MigrationRule = {
  from: string[];
  to: string;
  transform?: (value: any) => any;
  delete?: boolean;
};
```

---

## Authentication & Security

### Gateway Authentication

**File:** `src/gateway/auth.ts`

**Auth Modes:**

| Mode | Description |
|------|-------------|
| **Token** | Bearer token in header |
| **Password** | Password in payload |
| **Tailscale** | Tailscale identity verification |
| **Device Token** | Device-specific token |

**Authentication Flow:**
```
Client connects
      ↓
Send auth challenge
      ↓
Client responds with credentials
      ↓
Validate credentials:
  - Token: timing-safe comparison
  - Password: hash comparison
  - Tailscale: whois lookup
      ↓
Authentication successful?
      ├─ Yes → Connection established
      └─ No → Connection rejected
```

**Security Features:**
- **Timing-safe comparisons** (prevents timing attacks)
- **Loopback detection** (recognizes local requests)
- **Tailscale whois** (identity verification)
- **Trusted proxies** (X-Forwarded-For handling)
- **Constant-time token compare** (`crypto.timingSafeEqual`)

### Execution Approval

**File:** `src/gateway/exec-approval-manager.ts`

**Purpose:** Require user approval for dangerous commands

**Approval Request:**
```typescript
type ExecApprovalRequestPayload = {
  command: string;        // Command to execute
  cwd?: string;          // Working directory
  host?: string;         // Target host (remote)
  security?: string;     // Security context
  ask?: string;          // Prompt text for user
  agentId?: string;      // Requesting agent
  resolvedPath?: string; // Resolved executable path
  sessionKey?: string;   // Session identifier
};
```

**Approval Flow:**
```
Agent requests dangerous command
      ↓
Create approval request (UUID)
      ↓
Start timeout timer (default: 5 min)
      ↓
Wait for user decision
      ↓
[UI shows approval prompt]
      ↓
User approves or denies
      ↓
Decision recorded
      ↓
Result returned to agent
```

**Approval States:**
- `pending` - Waiting for user
- `approved` - User approved
- `denied` - User denied
- `expired` - Timeout elapsed

### Security Module

**Location:** `src/security/`

| File | Purpose |
|------|---------|
| `audit.ts` | Security auditing |
| `audit-fs.ts` | Filesystem auditing |
| `external-content.ts` | External content sanitization |
| `fix.ts` | Security fixes |
| `windows-acl.ts` | Windows ACL management |

### Content Security

**External Content Handling:**
- SSRF protection in web fetch
- File access restrictions
- Command execution sandboxing
- Path traversal prevention

---

## Concurrency Model

### Lane System

**File:** `src/agents/pi-embedded-runner/lanes.ts`

**Purpose:** Serialize operations to prevent race conditions

**Lane Types:**

| Lane Type | Scope | Purpose |
|-----------|-------|---------|
| **Session Lane** | Per session | Serialize messages within a conversation |
| **Global Lane** | System-wide | Serialize cross-session operations |

**Lock Ordering:**
```
Session Lane → Global Lane
```

This prevents deadlocks by always acquiring locks in the same order.

**Example:**
```
Session A (WhatsApp):
  [Lane A lock] → [Global lock] → Execute → [Global unlock] → [Lane A unlock]

Session B (Telegram):
  [Lane B lock] → [Global lock] → Execute → [Global unlock] → [Lane B unlock]

Result: Sessions A and B can run concurrently, but each session's
        messages are processed in order.
```

### Command Queue

**File:** `src/process/command-queue.ts`

**Purpose:** Queue commands for execution in lanes

**Queue Type:**
```typescript
type CommandQueue = {
  enqueue: (lane: string, task: Task, opts?: QueueOptions) => Promise<void>;
};
```

**Execution:**
```typescript
await enqueueCommandInLane("session-123", async () => {
  // This runs exclusively in the session lane
  await doSomething();
});
```

### Concurrency Limits

**Configuration:**
```yaml
agents:
  defaults:
    concurrency: 3  # Max parallel agent runs
```

**Concurrency Control:**
- Session lanes: Unlimited (one per session)
- Global lane: Serializes access to shared resources
- Agent runs: Limited by config

---

## State Management

### In-Memory State

**File:** `src/gateway/server-runtime-state.ts`

**State Components:**

| State | Description | Lifetime |
|-------|-------------|----------|
| **Sessions** | Active sessions | Until disconnect |
| **Chat buffers** | In-flight messages | Until delivered |
| **Connections** | WebSocket clients | Until disconnect |
| **Lanes** | Active locks | Until completion |
| **Channels** | Active channels | Until shutdown |
| **Nodes** | Connected nodes | Until disconnect |

**RuntimeState:**
```typescript
type RuntimeState = {
  // Sessions
  sessions: Map<string, SessionState>;

  // Connections
  clients: Set<GatewayWsClient>;

  // Channels
  channels: Map<string, ChannelState>;

  // Nodes
  nodes: NodeRegistry;

  // Health
  health: HealthState;

  // Presence
  presence: PresenceState;
};
```

### Filesystem State

**Locations:**

| Data | Location | Format |
|------|----------|--------|
| **Config** | `~/.moltbot/config.yaml` | YAML |
| **Auth profiles** | `~/.moltbot/config/auth-profiles.json` | JSON |
| **Sessions** | `~/.moltbot/sessions/` | JSON |
| **Transcripts** | `~/.moltbot/sessions/*/transcript.json` | JSON |
| **Logs** | `~/.moltbot/logs/` | Text |
| **Agent data** | `~/.moltbot/agents/` | Mixed |

### Session Storage

**File:** `src/config/sessions/`

**Session Components:**

| File | Purpose |
|------|---------|
| `store.ts` | Session storage backend |
| `transcript.ts` | Transcript persistence |
| `metadata.ts` | Session metadata |
| `paths.ts` | Path resolution |
| `session-key.ts` | Session key generation |
| `main-session.ts` | Main session logic |
| `group.ts` | Group session logic |

**Session Structure:**
```
~/.moltbot/sessions/
├── {session-key}/
│   ├── transcript.json
│   ├── metadata.json
│   └── context.json
```

### Databases (Optional)

**Extensions:**
- `memory-lancedb` - Vector database for long-term memory
- `memory-core` - Core memory functions

**Usage:**
- Optional per-session memory
- Vector search for relevant context
- Persistent across sessions

---

## Frontend Interfaces

### Web UI

**Location:** `ui/`

**Tech Stack:**
- React
- Vite
- TypeScript

**Features:**
- Real-time WebSocket connection
- Chat interface
- Configuration management
- Session management
- Canvas/A2UI support
- Live updates

**Build:**
```bash
pnpm ui:build
```

**Dev Mode:**
```bash
pnpm ui:dev
```

### Terminal UI (TUI)

**Location:** `src/tui/`

**Features:**
- Interactive menus
- Wizard flows
- Status displays
- Terminal-based chat

**Command:**
```bash
moltbot tui
```

### Mobile Apps

**Location:** `apps/`

#### iOS App

**Directory:** `apps/ios/`

**Tech Stack:**
- Swift
- SwiftUI
- Xcode (xcodegen)

**Build:**
```bash
pnpm ios:gen      # Generate Xcode project
pnpm ios:build    # Build app
pnpm ios:run      # Run on simulator
```

#### Android App

**Directory:** `apps/android/`

**Tech Stack:**
- Kotlin
- Gradle

**Build:**
```bash
pnpm android:assemble  # Build APK
pnpm android:install   # Install on device
pnpm android:run       # Run app
```

#### macOS App

**Directory:** `apps/macos/`

**Tech Stack:**
- Swift
- SwiftUI

**Build:**
```bash
pnpm mac:package   # Build .app bundle
pnpm mac:open      # Open app
```

#### Shared Code

**Directory:** `apps/shared/`

**Purpose:** Shared code between iOS, Android, and macOS apps

---

## Extensions Directory

### Overview

**Location:** `extensions/`

Extensions are **external packages** that integrate additional services or add functionality.

### Channel Extensions

| Extension | Channel ID | Description |
|-----------|------------|-------------|
| `discord` | `discord` | Discord bot API |
| `slack` | `slack` | Slack Socket Mode |
| `telegram` | `telegram` | Telegram Bot API |
| `whatsapp` | `whatsapp` | WhatsApp Web (Baileys) |
| `signal` | `signal` | Signal (signal-cli) |
| `imessage` | `imessage` | Apple Messages |
| `googlechat` | `googlechat` | Google Chat |
| `matrix` | `matrix` | Matrix protocol |
| `line` | `line` | LINE |
| `zalo` | `zalo` | Zalo |
| `zalouser` | `zalouser` | Zalo Personal |
| `bluebubbles` | `bluebubbles` | BlueBubbles |
| `mattermost` | `mattermost` | Mattermost |
| `nextcloud-talk` | `nextcloud-talk` | Nextcloud Talk |
| `twitch` | `twitch` | Twitch |
| `nostr` | `nostr` | Nostr protocol |
| `tlon` | `tlon` | Tlon |
| `msteams` | `msteams` | Microsoft Teams |

### Feature Extensions

| Extension | Purpose |
|-----------|---------|
| `memory-lancedb` | Vector database memory |
| `memory-core` | Core memory functions |
| `google-gemini-cli-auth` | Gemini CLI authentication |
| `qwen-portal-auth` | Qwen portal authentication |
| `copilot-proxy` | GitHub Copilot proxy |
| `diagnostics-otel` | OpenTelemetry diagnostics |
| `llm-task` | LLM task processing |
| `voice-call` | Voice calling |
| `lobster` | ??? |

### Extension Structure

```
extensions/{extension-name}/
├── package.json
├── index.ts
├── README.md
└── src/
    └── ...
```

### Adding Extensions

Extensions are discovered automatically from the `extensions/` directory and can be enabled via configuration.

---

## Skills Directory

### Overview

**Location:** `skills/`

Skills are **modular capabilities** that provide tools to the agent.

### Skill Categories

#### Productivity

| Skill | Description |
|-------|-------------|
| `github` | GitHub integration |
| `notion` | Notion integration |
| `trello` | Trello integration |
| `things-mac` | Things task manager (macOS) |
| `tmux` | Terminal multiplexer control |

#### Communication

| Skill | Description |
|-------|-------------|
| `slack` | Slack actions |
| `discord` | Discord actions |
| `wacli` | WhatsApp CLI |
| `blucli` | ??? |

#### Information

| Skill | Description |
|-------|-------------|
| `weather` | Weather information |
| `summarize` | Text summarization |
| `gifgrep` | GIF search |

#### Media

| Skill | Description |
|-------|-------------|
| `spotify-player` | Spotify control |
| `sonoscli` | Sonos control |
| `openai-image-gen` | Image generation |
| `video-frames` | Video frame extraction |

#### Notes & Knowledge

| Skill | Description |
|-------|-------------|
| `obsidian` | Obsidian integration |
| `bear-notes` | Bear Notes (macOS) |
| `apple-notes` | Apple Notes |
| `apple-reminders` | Apple Reminders |

#### Development

| Skill | Description |
|-------|-------------|
| `coding-agent` | Coding assistance |
| `gemini` | Google Gemini |
| `model-usage` | Model usage tracking |

#### Voice & Audio

| Skill | Description |
|-------|-------------|
| `openai-whisper` | Speech recognition (local) |
| `openai-whisper-api` | Speech recognition (API) |
| `sherpa-onnx-tts` | Text-to-speech |

#### Home Automation

| Skill | Description |
|-------|-------------|
| `openhue` | Philips Hue control |
| `voice-call` | Voice calling |

#### Utilities

| Skill | Description |
|-------|-------------|
| `1password` | 1Password integration |
| `canvas` | Canvas/A2UI support |
| `session-logs` | Session logging |
| `food-order` | Food ordering |
| `goplaces` | Location services |
| `local-places` | Local places |
| `ordercli` | Ordering system |

### Skill Structure

```
skills/{skill-name}/
├── skill.json        # Skill manifest
├── README.md
└── src/
    └── ...
```

### Skill Manifest

```json
{
  "id": "skill-name",
  "name": "Skill Name",
  "version": "1.0.0",
  "description": "What this skill does",
  "tools": [
    {
      "name": "tool_name",
      "description": "Tool description"
    }
  ]
}
```

---

## Testing Strategy

### Test Types

| Test Type | Command | Description |
|-----------|---------|-------------|
| **Unit Tests** | `pnpm test` | Unit tests (Vitest) |
| **E2E Tests** | `pnpm test:e2e` | End-to-end tests |
| **Live Tests** | `pnpm test:live` | Tests with real API keys |
| **Docker Tests** | `pnpm test:docker:all` | Full integration in Docker |
| **Coverage** | `pnpm test:coverage` | Coverage report |

### Test Configuration

**Files:**
- `vitest.config.ts` - Main Vitest config
- `vitest.e2e.config.ts` - E2E test config
- `vitest.live.config.ts` - Live test config

**Coverage Thresholds:**
```typescript
thresholds: {
  lines: 70,
  functions: 70,
  branches: 70,
  statements: 70
}
```

### Test Files

**Pattern:** `**/*.test.ts`

**Count:** 200+ test files

**Examples:**
- `src/gateway/gateway.e2e.test.ts`
- `src/agents/pi-embedded-runner/run/attempt.test.ts`
- `src/config/config.test.ts`

### Docker Tests

**Scripts:** `scripts/e2e/*.sh`

| Test | Script |
|------|--------|
| Onboard | `scripts/e2e/onboard-docker.sh` |
| Gateway Network | `scripts/e2e/gateway-network-docker.sh` |
| Live Models | `scripts/test-live-models-docker.sh` |
| Live Gateway | `scripts/test-live-gateway-models-docker.sh` |
| QR Import | `scripts/e2e/qr-import-docker.sh` |
| Doctor Switch | `scripts/e2e/doctor-install-switch-docker.sh` |
| Plugins | `scripts/e2e/plugins-docker.sh` |
| Cleanup | `scripts/test-cleanup-docker.sh` |

---

## Development Workflow

### Setup

```bash
# Clone repository
git clone https://github.com/moltbot/moltbot.git
cd moltbot

# Install dependencies
pnpm install

# Build project
pnpm build

# Build UI
pnpm ui:build
```

### Development Mode

**Gateway (with auto-reload):**
```bash
pnpm gateway:watch
```

**Gateway (manual restart):**
```bash
pnpm gateway:dev
```

**UI Dev Server:**
```bash
pnpm ui:dev
```

### Building

```bash
# Full build
pnpm build

# Core build
tsc -p tsconfig.json

# UI build
pnpm ui:build
```

### Linting & Formatting

```bash
# Lint
pnpm lint

# Lint and fix
pnpm lint:fix

# Format check
pnpm format

# Format fix
pnpm format:fix
```

### Testing

```bash
# Unit tests
pnpm test

# E2E tests
pnpm test:e2e

# Live tests (requires API keys)
pnpm test:live

# All tests
pnpm test:all
```

### Release

```bash
# Run prepack
pnpm prepack

# Check release
pnpm release:check

# Publish (npm)
npm publish
```

---

## Customization Guide

### Easy Customizations (Low Risk)

#### 1. Add New Skills

**Location:** `skills/`

**Steps:**
1. Create directory: `skills/my-skill/`
2. Add manifest: `skill.json`
3. Implement tools
4. Enable in config

**Template:**
```json
{
  "id": "my-skill",
  "name": "My Skill",
  "version": "1.0.0",
  "description": "Does something useful",
  "tools": []
}
```

#### 2. Add New Tools

**Location:** `src/agents/tools/`

**Template:**
```typescript
import { Type } from "@sinclair/typebox";

export const myTool = {
  name: "my_tool",
  label: "My Tool",
  description: "Does something",
  parameters: Type.Object({
    input: Type.String()
  }),
  execute: async (_toolCallId, args) => {
    return {
      content: [{ type: "text", text: "Result" }]
    };
  }
};
```

#### 3. UI Theming

**Location:** `ui/`

**Files to modify:**
- CSS variables for colors
- Component styling
- Layout

#### 4. Config Defaults

**Location:** `src/config/defaults.ts`

**Modify:** Default values for configuration options

#### 5. Add Hooks

**Location:** `src/hooks/bundled/`

**Steps:**
1. Create directory: `my-hook/`
2. Add handler: `handler.ts`
3. Implement hook logic

### Medium Customizations (Moderate Risk)

#### 1. Add New Channels

**Location:** `extensions/`

**Steps:**
1. Create extension: `extensions/my-channel/`
2. Implement plugin interface
3. Register in `src/channels/registry.ts`
4. Add to `CHAT_CHANNEL_ORDER`
5. Configure in `config.yaml`

**Template:**
```typescript
import { createChannelPlugin } from "moltbot/plugin-sdk";

const myChannelPlugin = createChannelPlugin({
  id: "my-channel",
  meta: {
    label: "My Channel",
    systemImage: "bubble.left",
    blurb: "My custom channel"
  },
  connect: async (ctx) => {
    // Connect to service
    return { client };
  },
  disconnect: async (ctx) => {
    // Cleanup
  }
});

export default {
  id: "my-channel",
  register(api) {
    api.registerChannel({ plugin: myChannelPlugin });
  }
};
```

#### 2. Modify Agent Behavior

**Location:** `src/agents/pi-embedded-runner/`

**Files:**
- `run.ts` - Main execution loop
- `system-prompt.ts` - System prompt
- `compact.ts` - Compaction strategy

#### 3. Add RPC Methods

**Location:** `src/gateway/server-methods/`

**Steps:**
1. Create file: `my-method.ts`
2. Implement handler
3. Register in `listGatewayMethods()`
4. Add to protocol schema

#### 4. Custom Model Providers

**Files:**
- `src/config/types.models.ts` - Add provider type
- `src/agents/model-selection.ts` - Add resolution logic
- `src/agents/auth-profiles.ts` - Add auth handling

#### 5. Change Authentication

**Location:** `src/gateway/auth.ts`

**Warning:** Security implications

### Advanced Customizations (High Risk)

#### 1. Core Gateway Changes

**Location:** `src/gateway/`

**Files:**
- `server.impl.ts` - Main server
- `server-ws-runtime.ts` - WebSocket handling
- `server-runtime-state.ts` - State management

**Warning:** Affects all functionality

#### 2. Concurrency Model

**Location:** `src/agents/pi-embedded-runner/lanes.ts`

**Warning:** Can cause race conditions

#### 3. Session Management

**Location:** `src/config/sessions/`

**Files:**
- `store.ts` - Storage backend
- `transcript.ts` - Transcript handling

**Warning:** Data loss potential

#### 4. Protocol Changes

**Location:** `src/gateway/protocol/`

**Warning:** Breaking changes for clients

#### 5. Plugin System

**Location:** `src/plugins/`

**Warning:** Affects all extensibility

### Recommended Approach

1. **Start Small** - Add skills/tools first
2. **Understand Flow** - Trace message path
3. **Read Tests** - E2E tests show usage
4. **Use Extensions** - Plugins over core changes
5. **Maintain Compatibility** - Consider upstream updates
6. **Test Thoroughly** - Run full test suite

### Key Files to Understand First

| File | Purpose |
|------|---------|
| `ARCHITECTURE.md` | High-level overview |
| `API_REFERENCE.md` | RPC methods |
| `ADDON_DEVELOPMENT.md` | Extension guide |
| `src/gateway/server.impl.ts` | Server startup |
| `src/agents/pi-embedded-runner/run.ts` | Agent loop |
| `src/channels/registry.ts` | Channel registry |

---

## Technology Stack

### Core

| Technology | Version | Purpose |
|------------|---------|---------|
| **TypeScript** | 5.9+ | Type-safe JavaScript |
| **Node.js** | 22.12.0+ | Runtime |
| **pnpm** | 10.23.0+ | Package manager |

### Key Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `@mariozechner/pi-agent-core` | 0.49.3 | Agent framework |
| `@mariozechner/pi-ai` | 0.49.3 | AI integration |
| `@mariozechner/pi-coding-agent` | 0.49.3 | Coding agent |
| `@mariozechner/pi-tui` | 0.49.3 | Terminal UI |
| `@whiskeysockets/baileys` | 7.0.0-rc.9 | WhatsApp |
| `@slack/bolt` | 4.6.0 | Slack |
| `@grammyjs/runner` | 2.0.3 | Telegram runner |
| `@grammyjs/transformer-throttler` | 1.2.1 | Telegram throttling |
| `@discordjs/rest` | Latest | Discord |
| `ws` | 8.19.0 | WebSocket |
| `express` | 5.2.1 | HTTP server |
| `hono` | 4.11.4 | HTTP framework |
| `playwright-core` | 1.58.0 | Browser automation |
| `vitest` | 4.0.18 | Testing |

### Frontend

| Technology | Purpose |
|------------|---------|
| **React** | UI framework |
| **Vite** | Build tool |
| **Swift/SwiftUI** | iOS/macOS apps |
| **Kotlin** | Android app |

### Development Tools

| Tool | Purpose |
|------|---------|
| **vitest** | Testing framework |
| **oxlint** | Linter |
| **oxfmt** | Formatter |
| **tsc** | TypeScript compiler |
| **rolldown** | Bundler (A2UI) |

---

## File Reference

### Directory Structure

```
moltbot/
├── src/                          # Core source code
│   ├── gateway/                  # Gateway system (100+ files)
│   │   ├── server.impl.ts        # Main server
│   │   ├── auth.ts               # Authentication
│   │   ├── server-ws-runtime.ts  # WebSocket
│   │   ├── server-runtime-state.ts  # State
│   │   ├── server-methods/       # RPC methods (30+ files)
│   │   └── protocol/             # Protocol schemas
│   ├── agents/                   # Agent runtime (200+ files)
│   │   ├── pi-embedded-runner/   # Pi agent execution
│   │   ├── tools/                # Built-in tools (60+ files)
│   │   ├── auth-profiles/        # API key management
│   │   └── model-*.ts            # Model handling
│   ├── channels/                 # Channel abstractions
│   │   └── registry.ts           # Channel registry
│   ├── config/                   # Configuration (100+ files)
│   │   ├── config.ts             # Main loader
│   │   ├── types*.ts             # Schema definitions
│   │   └── sessions/             # Session management
│   ├── plugins/                  # Plugin system
│   │   ├── loader.ts             # Plugin loading
│   │   └── registry.ts           # Plugin registry
│   ├── cli/                      # CLI (100+ files)
│   │   ├── program.ts            # Command definitions
│   │   └── gateway-cli.ts        # Gateway command
│   ├── hooks/                    # Hooks system
│   ├── tui/                      # Terminal UI
│   └── ...
├── extensions/                   # Channel extensions (30+)
│   ├── discord/
│   ├── slack/
│   ├── telegram/
│   ├── whatsapp/
│   └── ...
├── skills/                       # Agent skills (50+)
│   ├── github/
│   ├── notion/
│   ├── weather/
│   └── ...
├── ui/                           # Web UI
│   ├── src/
│   └── package.json
├── apps/                         # Mobile apps
│   ├── ios/
│   ├── android/
│   ├── macos/
│   └── shared/
├── docs/                         # Documentation
├── scripts/                      # Build/utility scripts
├── moltbot.mjs                   # CLI entry point
├── package.json                  # Root package
├── pnpm-workspace.yaml           # Workspace config
├── tsconfig.json                 # TypeScript config
└── vitest.config.ts              # Test config
```

### File Count by Directory

| Directory | Files |
|-----------|-------|
| `src/gateway/` | 100+ |
| `src/agents/` | 200+ |
| `src/config/` | 100+ |
| `src/cli/` | 100+ |
| `src/plugins/` | 30+ |
| `src/hooks/` | 30+ |
| `extensions/` | 30+ |
| `skills/` | 50+ |
| **Total** | **1,000+** |

### Critical Files

| File | Lines | Purpose |
|------|-------|---------|
| `src/gateway/server.impl.ts` | ~500 | Main server |
| `src/agents/pi-embedded-runner/run.ts` | ~500 | Agent loop |
| `src/gateway/auth.ts` | ~300 | Authentication |
| `src/config/config.ts` | ~400 | Config loader |
| `src/channels/registry.ts` | ~200 | Channel registry |
| `src/plugins/loader.ts` | ~400 | Plugin loading |

---

## Appendix A: Configuration Reference

### Complete Config Schema

```yaml
# Gateway Settings
gateway:
  port: 18789
  bind: loopback | lan | tailnet | auto
  host: "optional override"
  auth:
    token: "bearer-token"
    password: "password"
    tailscale:
      enabled: true
  controlUi:
    enabled: true
  tailscale:
    enabled: false
    mode: proxy | serve

# Agent Settings
agents:
  defaults:
    model: "anthropic/claude-sonnet-4.5"
    provider: "anthropic"
    context: 200000
    thinking: none | low | medium | high
    temperature: 0.7
    maxTokens: 4096
    concurrency: 3
    fallbacks:
      - model: "backup-model"
        provider: "backup-provider"

  # Multiple agents
  agents:
    my-agent:
      model: "anthropic/claude-opus-4.5"
      system: "Custom system prompt"

  # Auth profiles
  profiles:
    - id: "anthropic-pro"
      provider: "anthropic"
      apiKey: "${ANTHROPIC_API_KEY}"

# Models
models:
  catalog:
    anthropic/claude-sonnet-4.5:
      provider: anthropic
      model: claude-sonnet-4-20250514
      contextWindow: 200000
      supportsImages: true
      supportsThinking: false

# Channels
channels:
  whatsapp:
    enabled: true
  telegram:
    enabled: true
    botToken: "${TELEGRAM_BOT_TOKEN}"

# Skills
skills:
  weather:
    enabled: true
    apiKey: "${WEATHER_API_KEY}"

# Tools
tools:
  allowlist:
    - "browser:*"
    - "web:*"
  denylist:
    - "exec:*"

# Hooks
hooks:
  internal:
    enabled: true
  gmail:
    enabled: false
    account: "example@gmail.com"
    model: "anthropic/claude-sonnet-4.5"

# UI
ui:
  accentColor: "#ff6b6b"
  assistantName: "Clawd"
  assistantAvatar: "🦞"

# Logging
logging:
  level: info
  file: ~/.moltbot/logs/moltbot.log
```

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `CLAWDBOT_GATEWAY_TOKEN` | Auth token override |
| `CLAWDBOT_GATEWAY_PORT` | Port override |
| `CLAWDBOT_DEBUG` | Enable debug mode |
| `CLAWDBOT_SKIP_CHANNELS` | Skip channel startup |
| `CLAWDBOT_DISABLE_ROUTE_FIRST` | Disable CLI routing |
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `OPENAI_API_KEY` | OpenAI API key |

---

## Appendix B: API Reference

### WebSocket Protocol

**Connection:**
```javascript
const ws = new WebSocket('ws://localhost:18789');

// Authentication
ws.send(JSON.stringify({
  jsonrpc: '2.0',
  method: 'connect.challenge',
  params: {
    token: 'your-token'
  },
  id: 1
}));
```

**RPC Call:**
```javascript
ws.send(JSON.stringify({
  jsonrpc: '2.0',
  method: 'sessions.list',
  params: {},
  id: 2
}));
```

**Event Subscription:**
```javascript
ws.send(JSON.stringify({
  jsonrpc: '2.0',
  method: 'events.subscribe',
  params: {
    events: ['chat', 'agent', 'health']
  },
  id: 3
}));
```

### HTTP Endpoints

**Chat Completions (OpenAI-compatible):**
```http
POST /v1/chat/completions
Content-Type: application/json
Authorization: Bearer your-token

{
  "model": "anthropic/claude-sonnet-4.5",
  "messages": [
    { "role": "user", "content": "Hello" }
  ]
}
```

**OpenResponses API:**
```http
POST /v1/responses
Content-Type: application/json
Authorization: Bearer your-token

{
  "query": "Hello",
  "session_key": "session-id"
}
```

---

## Appendix C: Troubleshooting

### Common Issues

**Port in use:**
```bash
# Find process using port 18789
lsof -i :18789

# Kill process
kill -9 <PID>
```

**Config errors:**
```bash
# Validate config
moltbot doctor

# Show config
moltbot config get
```

**Channel connection issues:**
```bash
# Check channel status
moltbot channels status

# Restart channel
moltbot channels logout <channel>
```

**Agent not responding:**
```bash
# Check agent status
moltbot agents list

# View session
moltbot sessions list
```

### Debug Mode

```bash
# Enable debug logging
CLAWDBOT_DEBUG=1 moltbot gateway

# Verbose mode
moltbot gateway --verbose

# Trace mode
moltbot gateway --trace
```

---

## Conclusion

Moltbot is a **production-grade, well-architected** personal AI assistant platform with:

- **Modular design** - Everything is pluggable
- **Multi-channel support** - 7 built-in + 30+ extensions
- **Robust agent runtime** - Based on Pi Agent framework
- **Comprehensive tooling** - 60+ built-in tools
- **Strong security** - Multiple auth modes, approval system
- **Extensive documentation** - This file + existing docs

**For customization:**
1. Start with skills and tools
2. Understand the data flow
3. Use extension points
4. Test thoroughly
5. Consider upstream compatibility

**Key files to understand:**
- `src/gateway/server.impl.ts` - Everything starts here
- `src/agents/pi-embedded-runner/run.ts` - Agent execution
- `src/channels/registry.ts` - Channel system
- `src/config/config.ts` - Configuration

**Resources:**
- GitHub: https://github.com/moltbot/moltbot
- Website: https://molt.bot
- Discord: https://discord.gg/clawd
- Documentation: https://docs.molt.bot

---

*This document is a comprehensive reference for understanding and customizing Moltbot. For the most up-to-date information, always refer to the source code and official documentation.*
