---
hide:
  - toc
---

# Frontend patterns

ADK agents can power many kinds of user interfaces. Start by choosing the
runtime contract, then choose the specific UI pattern your product needs.

[AG-UI](/integrations/ag-ui/) is the production-oriented event protocol path for
application UIs. It can carry agent activity, reasoning, tool calls, frontend
actions, shared state, human review, and the full generative UI spectrum.

!!! note "Current runnable examples"

    The examples use a CopilotKit AG-UI frontend backed by Google ADK agents.
    Use them to study frontend behavior, then decide whether your ADK app should
    expose raw ADK APIs, AG-UI, A2UI payloads, or a custom stream.

## Choose the layer first

| Layer | Use it when | What it owns |
|---|---|---|
| ADK Runtime | You are building, running, and deploying agents. | Agent execution, sessions, tools, and runtime events. |
| Custom ADK frontend client | You need a product-specific SDK or want to map `/run_sse` yourself. | Raw event consumption, state mapping, retries, and client contract design. |
| AG-UI | Your application needs streaming messages, tool rendering, frontend tools, shared state, generative UI, or human approval flows. | A stable client-facing event protocol across web, mobile, chat, and other frontend surfaces. |
| A2UI | The agent should return a portable structured UI payload assembled from trusted components. | A declarative component payload that can travel through AG-UI, A2A, MCP, REST, or another stream. |
| Frontend frameworks and channels | You are rendering the final user experience. | React, Vue, React Native, Flutter, Slack, Microsoft Teams, or another client surface. |

## Generative UI spectrum

| Pattern | Use it when |
|---|---|
| [Controlled generative UI](generative-ui-spectrum/controlled-generative-ui.md) | The app owns the component and the agent decides when to show it. |
| [Declarative generative UI with A2UI](generative-ui-spectrum/declarative-generative-ui-a2ui.md) | The agent assembles trusted catalog components into a UI payload. |
| [Open generative UI](generative-ui-spectrum/open-generative-ui.md) | The agent or tool needs a richer surface than fixed components or a catalog. |
| [MCP Apps](generative-ui-spectrum/mcp-apps.md) | The frontend hosts app-capable tool surfaces from MCP servers. |

## Interaction patterns

| Pattern | Use it when |
|---|---|
| [Tool rendering](patterns/tool-rendering.md) | The user needs to see tool progress, arguments, results, or errors. |
| [Frontend tools and context](patterns/frontend-tools-and-context.md) | The agent should read app context or ask the frontend to take safe domain actions. |
| [Shared state](patterns/shared-state.md) | The agent and frontend need to collaborate over the same working object. |
| [Human-in-the-loop](patterns/human-in-the-loop.md) | The run must pause for user review, approval, revision, or cancellation. |

## A2UI with AG-UI

A2UI defines the declarative UI contract: what catalog-backed surface the agent
wants the client to render. AG-UI carries the live interaction stream between
the agent backend and the application UI.

Use them together when the ADK agent should generate structured UI and the
application still needs messages, agent activity, reasoning, tools, state,
frontend actions, and user responses.

Use A2UI without AG-UI when you only need portable UI payloads and already have
another transport or runtime contract.

## Custom frontends with ADK APIs

You can also build a
[custom frontend adapter](/runtime/frontend-interfaces/custom-frontends/)
directly on the ADK API server. This is the lowest-level option: your client or
backend-for-frontend reads `/run_sse`, maps ADK events into UI state, and sends
user input or tool responses back through the runtime boundary.

## Keep the boundaries explicit

- ADK owns agent execution, tools, sessions, and runtime events.
- The frontend interface owns the client-facing event or payload contract.
- The application owns rendering, permissions, auth context, and user
  experience.
