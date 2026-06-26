# MCP Apps

MCP Apps sit near the open end of the spectrum. The app surface comes from the
tool ecosystem, while the host frontend controls placement, permissions, and
lifecycle.

<p class="frontend-pattern-example-caption">CopilotKit AG-UI frontend backed by Google ADK.</p>

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```ts title="MCP Apps over an AG-UI runtime"
const runtime = new CopilotRuntime({
  agents: { adk_agent },
  mcpApps: {
    servers: [{
      type: "http",
      url: "https://mcp.excalidraw.com",
      serverId: "excalidraw",
      agentId: "adk_agent",
    }],
  },
});
```

The runtime makes an app-capable MCP server available. The agent can call it,
and the frontend renders the returned app surface without hand-building a
custom component for every tool.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/mcp-apps/preview" title="MCP Apps showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/mcp-apps/preview){ target="_blank" }

</div>

</div>

Use this band for app-capable tools and third-party app experiences inside an
ADK frontend.

Return to the [spectrum overview](index.md) or compare the broader
[frontend patterns](../patterns.md).
