# Generative UI spectrum

Generative UI is not one pattern. It is a spectrum of how much control the
developer, the agent, and the frontend renderer each hold. An ADK application
can mix multiple points on the spectrum in the same user journey.

Use [AG-UI](/integrations/ag-ui/) when those surfaces need the full
agent-frontend transport: streaming messages, run activity, reasoning, tool
calls, state, frontend tools, human decisions, and UI specifications. Use
[A2UI](/integrations/a2ui/) when the declarative part of that loop should be a
catalog-backed visual language that renders with native frontend components.

```text
ADK agent logic
  -> AG-UI transport layer or custom ADK APIs
       messages, state, tools, activity, human decisions
       optional payloads:
         - A2UI visual-language descriptors
         - MCP Apps resources
         - open UI surfaces
  -> Native frontend renderer
```

## Choose a band

| Band | Developer controls | Agent controls | Best fit |
|---|---|---|---|
| [Controlled generative UI](controlled-generative-ui.md) | The component, props schema, validation, styling, and interaction model. | When to render the component and what data to pass. | Product-critical UI that must stay fully owned by the application. |
| [Declarative generative UI with A2UI](declarative-generative-ui-a2ui.md) | The catalog of approved components and their renderers. | How to assemble those components for the user's current intent. | Rich UI variety without arbitrary generated code. |
| [Open generative UI](open-generative-ui.md) | The sandbox, renderer boundary, permissions, and lifecycle. | The surface shape or tool-owned UI payload to show. | Exploratory visualizations and rich surfaces that do not fit a fixed component or catalog. |
| [MCP Apps](mcp-apps.md) | The MCP server, app host, permissions, and placement in the product. | Which app-capable tool to invoke and what context to pass. | Tool-owned and third-party app experiences inside the ADK frontend. |

## How AG-UI and A2UI compose

A2UI defines the declarative visual language. AG-UI carries that UI
specification and the surrounding interaction events across agent frameworks
and frontend surfaces. Use A2UI for the UI shape; use AG-UI for the live
agent-user transport.

That composition gives ADK apps two important options:

1. AG-UI can carry A2UI payloads alongside interaction events such as messages,
   tool calls, state, frontend tools, and human decisions.
2. A2UI can also travel over A2A, REST, MCP, or custom streams when the app
   already has another client contract that preserves payload metadata,
   ordering, versioning, and client capabilities.

Next, compare the feature-level examples in
[Frontend patterns](../patterns.md).
