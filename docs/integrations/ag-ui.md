---
catalog_title: AG-UI
catalog_description: Agent-User Interaction transport for application frontends
catalog_icon: /integrations/assets/ag-ui.png
---

# AG-UI protocol for ADK frontends

Connect your ADK agents to full-featured applications with rich, responsive UIs.
[AG-UI](https://docs.ag-ui.com/) is the transport layer for the live
agent-frontend loop. It is an event-based, transport-agnostic Agent-User
Interaction protocol that connects agent backends with application frontends.

AG-UI clients exist across React, Kotlin, Java, Go, and command-line surfaces:

- [CopilotKit](https://copilotkit.ai) provides tooling and components to tightly
  integrate your agent with web applications
- Clients for
  [Kotlin](https://github.com/ag-ui-protocol/ag-ui/tree/main/sdks/community/kotlin),
  [Java](https://github.com/ag-ui-protocol/ag-ui/tree/main/sdks/community/java),
  [Go](https://github.com/ag-ui-protocol/ag-ui/tree/main/sdks/community/go/example/client),
  and [CLI
  implementations](https://github.com/ag-ui-protocol/ag-ui/tree/main/apps/client-cli-example/src)
  in TypeScript

## Where AG-UI fits

ADK Runtime events are the source of truth for an agent run. AG-UI adapts those
events into the client-facing interaction pipe for application frontends. Use
AG-UI when your UI needs more than a single text response: streaming messages,
agent activity, reasoning, tool-call rendering, shared state, frontend tools,
human approvals, or UI specifications such as A2UI.

The layers are:

```text
ADK agent and Runtime events
  -> ADK-to-AG-UI adapter
  -> AG-UI transport layer
  -> Client implementation, such as CopilotKit
  -> Application UI
```

For a broader map of ADK frontend options, see
[Frontend interfaces](/runtime/frontend-interfaces/). For the pattern-level
breakdown, see
[Generative UI spectrum](/runtime/frontend-interfaces/patterns/generative-ui-spectrum/)
and [Frontend patterns](/runtime/frontend-interfaces/patterns/).

## Adapter shape

An ADK-to-AG-UI adapter keeps the runtime boundary explicit:

| ADK Runtime event | AG-UI contract | UI responsibility |
|---|---|---|
| Model text and partial text | Message and delta events | Append streaming assistant output without losing ordering. |
| `functionCall` parts | Tool-call events | Render pending tool work, arguments, progress, and cancellation affordances. |
| `functionResponse` parts | Tool-result events | Resolve the matching tool call and render the result or error state. |
| `actions.stateDelta` | State events | Project approved session state into application state. |
| Run start, completion, and errors | Lifecycle events | Show loading, completion, retry, and failure states. |
| Human-input requests | Human-in-the-loop events | Present an approval or input UI and send the decision back to ADK. |

## Example AG-UI clients

AG-UI is a protocol, not a single frontend SDK. Choose a client implementation
for the surface you are building:

| Client | Use it for |
|---|---|
| CopilotKit | Packaged React client, runtime wiring, and components on top of an AG-UI-compatible ADK adapter. To scaffold an ADK sample, run `npx copilotkit@latest create -f adk`. |
| Community clients | Kotlin, Java, Go, command-line, or custom clients that consume AG-UI events directly. |

## Capability map

The live examples for these capabilities live under
[Frontend patterns](/runtime/frontend-interfaces/patterns/), with each pattern
shown on its own page beside an embedded ADK-backed showcase.

| Capability | Use it when | Live pattern |
|---|---|---|
| Chat and streaming messages | Users need conversational interaction with an agent. | [Frontend patterns](/runtime/frontend-interfaces/patterns/) |
| Controlled generative UI | The app owns the React/native component and the agent selects when to render it. | [Controlled generative UI](/runtime/frontend-interfaces/patterns/controlled-generative-ui/) |
| Declarative UI payloads | The agent should assemble portable structured UI from approved component catalogs. | [Declarative generative UI with A2UI](/runtime/frontend-interfaces/patterns/declarative-generative-ui-a2ui/) |
| Open UI surfaces | The agent returns a richer sandboxed UI surface. | [Open generative UI](/runtime/frontend-interfaces/patterns/open-generative-ui/) |
| MCP Apps | The agent invokes app-capable tools from MCP servers. | [MCP Apps](/runtime/frontend-interfaces/patterns/mcp-apps/) |
| Tool rendering | The frontend should show tool calls, progress, results, and failures as first-class UI. | [Tool rendering](/runtime/frontend-interfaces/patterns/tool-rendering/) |
| Frontend tools and context | The agent needs approved application context or client-side actions. | [Frontend tools and context](/runtime/frontend-interfaces/patterns/frontend-tools-and-context/) |
| Shared state | The UI and agent need an explicit synchronized state boundary. | [Shared state](/runtime/frontend-interfaces/patterns/shared-state/) |
| Human-in-the-loop | The run needs user review, approval, revision, or selection before continuing. | [Human-in-the-loop](/runtime/frontend-interfaces/patterns/human-in-the-loop/) |

## A2UI with AG-UI

A2UI and AG-UI operate at different layers. A2UI is the visual language: the
agent assembles catalog-backed components, and the frontend renderer turns them
into native application UI. AG-UI is the transport layer that carries those UI
specifications inside the broader interaction stream, alongside messages, agent
activity, reasoning, tool calls, state, frontend tools, human-in-the-loop
events, lifecycle events, and user responses.

Use them together when an ADK app needs declarative generative UI and a full
bidirectional frontend protocol:

```text
ADK agent logic
  -> AG-UI interaction transport
       carries messages, state, tools, activity, and A2UI specifications
  -> Web, mobile, chat, or custom frontend renderer
```

Use A2UI for the UI shape. Use AG-UI for the live agent-user transport.

## Resources

To learn the protocol and see running examples:

- [AG-UI docs](https://docs.ag-ui.com/)
- [AG-UI protocol repository](https://github.com/ag-ui-protocol/ag-ui)
- [AG-UI Dojo](https://dojo.ag-ui.com)
- [Frontend patterns](/runtime/frontend-interfaces/patterns/)

For CopilotKit implementation examples:

- [Agentic Generative UI](https://docs.copilotkit.ai/adk/generative-ui/agentic)
- [A2UI with CopilotKit](https://docs.copilotkit.ai/adk/generative-ui/a2ui)
- [Human in the Loop](https://docs.copilotkit.ai/adk/human-in-the-loop)
- [Frontend Actions](https://docs.copilotkit.ai/adk/frontend-actions)
