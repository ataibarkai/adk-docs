# Frontend patterns

ADK agents can power many kinds of user interfaces. The important decision is
not just which UI framework you use, but how much control stays in the
application and how much structure the agent can return at runtime.

Most production frontends combine multiple patterns. For example, an app might
use AG-UI for streaming and state, render known tool calls with application
components, and allow selected agents to return A2UI payloads for portable
cards or forms.

## Pattern map

| Pattern | What the agent emits | What the frontend owns | Current ADK path |
|---|---|---|---|
| Controlled generative UI | Tool calls, tool results, and state changes. | Predefined components, styling, layout, validation, and user interactions. | [AG-UI](/integrations/ag-ui/) with frontend tools or tool-call rendering. |
| A2UI declarative UI | Structured A2UI JSON payloads such as cards, forms, tables, and charts. | Renderer, component catalog, styling, validation, and user interaction handling. | [A2UI](/integrations/a2ui/), optionally carried over AG-UI, A2A, MCP, REST, or another stream. |
| Open generative UI | A richer UI resource or app surface, often backed by an external tool or MCP server. | The host surface, sandboxing, permissions, and lifecycle of the rendered UI. | AG-UI plus an MCP Apps-style renderer when your client supports it. |
| Tool rendering | Function calls, function responses, progress events, and errors. | Progress cards, result views, confirmations, and retry affordances. | [AG-UI](/integrations/ag-ui/) or a custom ADK event client. |
| In-app generative UI | State deltas, frontend-tool calls, and app-context requests. | Application state, permissions, browser or native APIs, and domain-specific widgets. | [AG-UI](/integrations/ag-ui/) for bidirectional runtime interaction. |
| Human-in-the-loop | A tool call or interrupt that needs user review before continuing. | Confirmation UI, form input, audit messaging, and the response sent back to the agent. | [AG-UI](/integrations/ag-ui/) with human approval or response components. |

## AG-UI as the complete frontend path

AG-UI is the most complete frontend interface when an ADK agent needs to behave
like part of an application instead of a backend that only returns text. Use it
when the client needs a stable runtime contract for:

- streaming assistant messages and lifecycle events
- tool-call rendering and frontend tools
- shared state between agent and application
- generative UI attached to tool calls or message streams
- human-in-the-loop approvals and responses
- A2UI payloads traveling through the same client-facing stream

## A2UI with AG-UI

A2UI and AG-UI solve different layers of the frontend problem. A2UI describes a
piece of UI the agent wants the client to render. AG-UI carries runtime events
between the agent backend and the application UI.

Use them together when:

1. The ADK agent should generate a structured UI payload.
2. The application still needs a bidirectional stream for messages, tools,
   state, lifecycle events, and user responses.
3. The frontend can render the selected A2UI catalog safely inside the
   product's design system.

Use A2UI without AG-UI when you only need portable UI payloads and already have
another transport or runtime contract.

## Custom frontends with ADK APIs

You can also build a [custom frontend adapter](/runtime/frontend-interfaces/custom-frontends/)
directly on the ADK API server. This is the lowest-level option: your client or
backend-for-frontend reads `/run_sse`, maps ADK events into UI state, and sends
user input or tool responses back through the runtime boundary.

Choose this path when you need a custom client contract, already have a frontend
runtime abstraction, or want to build an ADK-specific SDK for your own product.

## Keep the boundaries explicit

The frontend interface should adapt ADK events for clients; it should not
replace the agent, the session model, or the deployment model. A good frontend
integration keeps these responsibilities separate:

- ADK owns agent execution, tools, sessions, and runtime events.
- The frontend interface owns the client-facing event or payload contract.
- The application owns rendering, permissions, auth context, and user
  experience.
