---
catalog_title: AG-UI
catalog_description: Build interactive chat UIs with streaming, state sync, and agentic actions
catalog_icon: /integrations/assets/ag-ui.png
---

# AG-UI protocol for ADK frontends

Connect your ADK agents to full-featured applications with rich, responsive UIs.
[AG-UI](https://docs.ag-ui.com/) is an open protocol that handles streaming
events, client state, and bidirectional communication between agents and
application frontends.

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
tool-call rendering, shared state, frontend tools, human approvals, or
generative UI.

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
breakdown, see [Frontend patterns](/runtime/frontend-interfaces/patterns/).

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

## Example client: CopilotKit

CopilotKit is one AG-UI client implementation. Use it when you want a packaged
React client, runtime wiring, and components on top of an AG-UI-compatible ADK
adapter.

To create a sample application with an ADK agent and a CopilotKit web client:

1. Create the app:

    ```bash
    npx copilotkit@latest create -f adk
    ```

2. Set your Google API key:

    ```bash
    export GOOGLE_API_KEY="your-api-key"
    ```

3. Install dependencies and run:

    ```bash
    npm install && npm run dev
    ```

This starts two servers:

- **http://localhost:3000** - The web UI (open this in your browser)
- **http://localhost:8000** - The ADK agent API (backend only)

Open [http://localhost:3000](http://localhost:3000) in your browser to chat with
your agent.

## Capability map

The live examples for these capabilities live on the
[Frontend patterns](/runtime/frontend-interfaces/patterns/) page, where each
pattern is shown with code beside an embedded ADK-backed showcase.

| Capability | Use it when | Live pattern |
|---|---|---|
| Chat and streaming messages | Users need conversational interaction with an agent. | [Frontend patterns](/runtime/frontend-interfaces/patterns/) |
| Controlled generative UI | The app owns the React/native component and the agent selects when to render it. | [Controlled generative UI](/runtime/frontend-interfaces/patterns/#controlled-generative-ui) |
| Declarative UI payloads | The agent should return portable structured UI data. | [A2UI declarative UI](/runtime/frontend-interfaces/patterns/#a2ui-declarative-ui) |
| MCP Apps and open UI surfaces | The agent returns an app-like surface through MCP or another sandboxed UI path. | [Open generative UI and MCP Apps](/runtime/frontend-interfaces/patterns/#open-generative-ui-and-mcp-apps) |
| Tool rendering | The frontend should show tool calls, progress, results, and failures as first-class UI. | [Tool rendering](/runtime/frontend-interfaces/patterns/#tool-rendering) |
| Frontend tools and context | The agent needs approved application context or client-side actions. | [Frontend tools and context](/runtime/frontend-interfaces/patterns/#frontend-tools-and-context) |
| Shared state | The UI and agent need an explicit synchronized state boundary. | [Shared state](/runtime/frontend-interfaces/patterns/#shared-state) |
| Human-in-the-loop | The run needs user review, approval, revision, or selection before continuing. | [Human-in-the-loop](/runtime/frontend-interfaces/patterns/#human-in-the-loop) |

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
