# Addon Development

Moltbot is designed to be extensible through **Extensions** (Addons) and **Skills**. This guide explains how to create them.

## Extensions

Extensions are packages located in `extensions/` that integrate external services or add core functionality. They typically implement the **Channel Plugin** interface.

### Creating a New Channel Extension

1.  **Create Directory**: Create a new directory in `extensions/<your-channel>`.
2.  **`package.json`**: Define the package.
3.  **Implement Plugin**: Create an `index.ts` that exports a plugin definition.

#### Example Structure
```
extensions/my-channel/
├── package.json
├── index.ts
└── src/
    └── bot.ts
```

#### Plugin Interface Example
A channel plugin typically provides connection logic and message translation.

```typescript
import { createChannelPlugin } from "clawdbot/plugin-sdk";
import type { MoltbotPluginApi } from "clawdbot/plugin-sdk";

// Define the plugin implementation
const myChannelPlugin = createChannelPlugin({
  id: "my-channel",
  meta: {
    label: "My Channel",
    systemImage: "bubble.left", // SF Symbol name or similar
    blurb: "Connects to My Channel service",
  },
  connect: async (ctx) => {
    ctx.log.info("Connecting to My Channel...");

    // Simulate connection to external service
    const client = new MyExternalClient(ctx.config?.token);

    client.on("message", (msg) => {
        // Forward message to Gateway
        ctx.gateway.send({
            channelId: "my-channel",
            body: msg.text,
            sender: { id: msg.userId, name: msg.userName },
            conversationId: msg.chatId
        });
    });

    await client.connect();

    return {
        // Return a handle to the connection
        client,
    };
  },
  disconnect: async (ctx) => {
    // Cleanup
    if (ctx.state?.client) {
        await ctx.state.client.disconnect();
    }
  }
});

// Export the plugin definition
export default {
  id: "my-channel",
  name: "My Channel Integration",
  description: "Integration for My Channel",
  configSchema: {}, // specific config schema if needed
  register(api: MoltbotPluginApi) {
    // Register the channel with the runtime
    api.registerChannel({ plugin: myChannelPlugin });
  },
};
```

## Skills

Skills are modular capabilities located in `skills/`. They provide specialized tools to the agent.

### Creating a Skill

1.  **Create Directory**: Create `skills/<skill-name>`.
2.  **`skill.json`** (or similar manifest): Define the skill metadata.
3.  **Implementation**: Scripts or binaries that the agent can execute.

Skills can be:
*   **Javascript/TypeScript**: Node.js scripts.
*   **Binaries**: Compiled executables.
*   **APIs**: Wrappers around HTTP APIs.

### Skill Lifecycle
*   **Installation**: Skills are "installed" into the agent's workspace.
*   **Discovery**: The agent discovers tools provided by the skill.
*   **Execution**: The agent invokes the skill's tools via the `exec` tool or specialized bridges.

## Custom Tools

You can create custom tools directly in the codebase or via plugins.

### Tool Definition Example
Here is an example of a tool that fetches weather data.

```typescript
import { Type } from "@sinclair/typebox";
import type { AnyAgentTool } from "./pi-tools.types";

// Define schema using TypeBox
const WeatherSchema = Type.Object({
  location: Type.String({ description: "City and state, e.g. San Francisco, CA" }),
  unit: Type.Optional(Type.String({ enum: ["celsius", "fahrenheit"], default: "celsius" })),
});

export function createWeatherTool(apiKey: string): AnyAgentTool {
  return {
    name: "get_weather",
    label: "Get Weather",
    description: "Get the current weather for a location.",
    parameters: WeatherSchema,
    execute: async (_toolCallId, args) => {
      // Args are typed based on schema
      const { location, unit } = args as { location: string; unit?: string };

      // Perform the logic (e.g., fetch from API)
      const response = await fetch(`https://api.weather.com/v1?q=${location}&key=${apiKey}`);
      const data = await response.json();

      return {
          content: [
              {
                  type: "text",
                  text: `The weather in ${location} is ${data.temp} degrees ${unit}.`
              }
          ]
      };
    },
  };
}
```

## Plugin Architecture Patterns

*   **Hook System**: Extensions can register hooks (e.g., `onMessage`, `onStartup`) to intercept or augment behavior.
*   **Service Injection**: Plugins can expose services to other plugins via the Gateway registry.
*   **Middleware**: Custom middleware can be added to the HTTP server for handling webhooks.

## Best Practices

1.  **Isolation**: Keep extensions self-contained. Avoid modifying core code.
2.  **Configuration**: Use the centralized config system. Define your config schema.
3.  **Error Handling**: Gracefully handle connection failures and API errors.
4.  **Logging**: Use the provided logger instance, not `console.log`.
