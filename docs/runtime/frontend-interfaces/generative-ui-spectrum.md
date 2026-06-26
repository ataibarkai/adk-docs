# Generative UI spectrum

Generative UI is not one pattern. It is a spectrum of how much control the
developer, the agent, and the frontend renderer each hold. An ADK application
can mix multiple points on the spectrum in the same user journey.

Use [AG-UI](/integrations/ag-ui/) when those surfaces need the full
agent-frontend loop: streaming messages, run activity, reasoning, tool calls,
state, frontend tools, and human decisions. Use [A2UI](/integrations/a2ui/) when
the declarative part of that loop should be a catalog-backed UI spec that can
render across frontends.

```text
ADK agent logic
  -> Runtime/client contract
       AG-UI or custom ADK APIs: live interaction events
       A2UI: declarative UI messages
       MCP Apps or open UI: app-like surfaces
  -> Frontend renderer
```

## Choose a band

| Band | Developer controls | Agent controls | Best fit |
|---|---|---|---|
| Controlled generative UI | The component, props schema, validation, styling, and interaction model. | When to render the component and what data to pass. | Product-critical UI that must stay fully owned by the application. |
| Declarative generative UI with A2UI | The catalog of approved components and their renderers. | How to assemble those components for the user's current intent. | Rich UI variety without arbitrary generated code. |
| Open generative UI and MCP Apps | The sandbox, app host, permissions, and lifecycle boundary. | The surface shape, app URL, or external app/tool surface to invoke. | One-off visualizations, exploratory applets, tool-owned surfaces, and third-party app experiences. |

## Controlled generative UI

Controlled UI is the safest first step. The app owns the component; the agent
selects when to show it.

<p class="frontend-pattern-example-caption">CopilotKit AG-UI frontend backed by Google ADK.</p>

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="Controlled UI example"
useComponent({
  agentId: "adk_agent",
  name: "showRenewalRiskCard",
  parameters: renewalRiskCardSchema,
  render: RenewalRiskCard,
});
```

The ADK agent emits a typed tool call through the frontend protocol. The
frontend renders a known component and sends the result back through the stream.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/gen-ui-tool-based/preview" title="Controlled generative UI showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/gen-ui-tool-based/preview){ target="_blank" }

</div>

</div>

## Declarative generative UI with A2UI

Declarative UI gives the agent more freedom without handing it arbitrary code.
Developers provide a catalog of approved UI components, and the agent assembles
those components into a UI for the current task.

<p class="frontend-pattern-example-caption">CopilotKit AG-UI frontend backed by Google ADK.</p>

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```python title="A2UI prompt setup"
schema_manager = A2uiSchemaManager(
    version=VERSION_0_9,
    catalogs=[BasicCatalog.get_config(version=VERSION_0_9)],
)

instruction = schema_manager.generate_system_prompt(
    role_description="You are a renewal assistant.",
    workflow_description="Return structured UI when useful.",
    ui_description="Use cards, tables, buttons, and forms.",
    include_schema=True,
    include_examples=True,
)
```

A2UI defines what the agent can assemble. AG-UI can carry those A2UI messages
alongside state, tools, activity, human review, and user responses.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/declarative-gen-ui/preview" title="A2UI declarative UI showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/declarative-gen-ui/preview){ target="_blank" }

</div>

</div>

## Open generative UI and MCP Apps

Open UI lets the agent or a tool-owned app control more of the surface. The host
application should keep a clear sandbox, permission model, and lifecycle.

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

MCP Apps sit near the open end of the spectrum: the app surface comes from the
tool ecosystem, while the host frontend controls where and how it appears.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/mcp-apps/preview" title="MCP Apps showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/mcp-apps/preview){ target="_blank" }

</div>

</div>

## How AG-UI and A2UI compose

A2UI defines the declarative UI contract. AG-UI carries that contract and the
surrounding interaction events across agent frameworks and frontend surfaces.
Use A2UI for the UI shape; use AG-UI for the live agent-user channel.

That composition gives ADK apps two important advantages:

1. AG-UI can carry A2UI alongside interaction events such as messages, tool
   calls, state, frontend tools, and human decisions.
2. A2UI can also travel over A2A, REST, MCP, or custom streams when the app
   already has another client contract.

Next, compare the feature-level examples in
[Frontend patterns](patterns.md).
