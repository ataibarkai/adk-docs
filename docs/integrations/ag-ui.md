---
catalog_title: AG-UI
catalog_description: Build interactive chat UIs with streaming, state sync, and agentic actions
catalog_icon: /integrations/assets/ag-ui.png
---

# AG-UI protocol for ADK frontends

Connect your ADK agents to full-featured applications with rich, responsive UIs.
[AG-UI](https://docs.ag-ui.com/) is an open protocol for the live
agent-frontend loop: streaming events, client state, tool calls, frontend
actions, generative UI, human review, and bidirectional communication between
agents and application frontends.

[AG-UI](https://github.com/ag-ui-protocol/ag-ui) provides a consistent interface
to empower rich clients across technology stacks, from mobile to the web and
even the command line. There are a number of different clients that support
AG-UI:

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
events into a stable client-facing protocol for application frontends. Use
AG-UI when your UI needs more than a single text response: streaming messages,
agent activity, reasoning, tool-call rendering, shared state, frontend tools,
human approvals, or generative UI.

The layers are:

```text
ADK agent and Runtime events
  -> ADK-to-AG-UI adapter
  -> AG-UI client-facing event protocol
  -> Client implementation, such as CopilotKit
  -> Application UI
```

For a broader map of ADK frontend options, see
[Frontend interfaces](/runtime/frontend-interfaces/). For the pattern-level
breakdown, see
[Generative UI spectrum](/runtime/frontend-interfaces/generative-ui-spectrum/)
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

The live examples for these capabilities live on the
[Frontend patterns](/runtime/frontend-interfaces/patterns/) page, where each
pattern is shown with code beside an embedded ADK-backed showcase.

| Capability | Use it when | Live pattern |
|---|---|---|
| Chat and streaming messages | Users need conversational interaction with an agent. | [Frontend patterns](/runtime/frontend-interfaces/patterns/) |
| Controlled generative UI | The app owns the React/native component and the agent selects when to render it. | [Controlled generative UI](/runtime/frontend-interfaces/patterns/#controlled-generative-ui) |
| Declarative UI payloads | The agent should assemble portable structured UI from approved component catalogs. | [A2UI declarative UI](/runtime/frontend-interfaces/patterns/#a2ui-declarative-ui) |
| MCP Apps and open UI surfaces | The agent returns an app-like surface through MCP or another sandboxed UI path. | [Open generative UI and MCP Apps](/runtime/frontend-interfaces/patterns/#open-generative-ui-and-mcp-apps) |
| Tool rendering | The frontend should show tool calls, progress, results, and failures as first-class UI. | [Tool rendering](/runtime/frontend-interfaces/patterns/#tool-rendering) |
| Frontend tools and context | The agent needs approved application context or client-side actions. | [Frontend tools and context](/runtime/frontend-interfaces/patterns/#frontend-tools-and-context) |
| Shared state | The UI and agent need an explicit synchronized state boundary. | [Shared state](/runtime/frontend-interfaces/patterns/#shared-state) |
| Human-in-the-loop | The run needs user review, approval, revision, or selection before continuing. | [Human-in-the-loop](/runtime/frontend-interfaces/patterns/#human-in-the-loop) |

## A2UI with AG-UI

A2UI and AG-UI operate at different layers. A2UI defines a declarative UI
contract: the agent assembles catalog-backed components, and the frontend
renderer turns them into native application UI. AG-UI carries that contract
inside the broader interaction stream, alongside messages, agent activity,
reasoning, tool calls, state, frontend tools, human-in-the-loop events,
lifecycle events, and user responses.

Use them together when an ADK app needs declarative generative UI and a full
bidirectional frontend protocol:

```text
ADK agent logic
  -> A2UI declarative UI contract
  -> AG-UI interaction stream
  -> Web, mobile, chat, or custom frontend renderer
```

Use A2UI for the UI shape. Use AG-UI for the live agent-user channel.

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
