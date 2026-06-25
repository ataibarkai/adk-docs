# Frontend interfaces

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-typescript">TypeScript</span><span class="lst-go">Go</span><span class="lst-java">Java</span>
</div>

ADK gives you several ways to run an agent during development and production.
When your product owns the user experience, the frontend usually needs a
client-facing contract on top of the ADK Runtime event stream. That contract
decides how the UI receives streaming messages, tool calls, state changes,
errors, human approvals, and optional structured UI payloads.

Use this section when you are moving beyond `adk web` or raw API testing and
want to connect an ADK agent to a browser, mobile app, desktop app, chat
surface, or another user-facing client.

## How the pieces fit

ADK runs the agent. The frontend interface translates runtime events into a
client experience:

```text
ADK agent
  -> Runner and session services
  -> ADK Event stream
  -> API server /run or /run_sse
  -> Frontend interface
  -> Application UI
```

The layers solve different problems:

- **ADK Runtime events** are the source of truth for what the agent did.
- **The API server** exposes those events over HTTP through `/run` and
  `/run_sse`.
- **AG-UI** provides a stable application-facing event protocol for streaming
  agent runs, state, tools, generative UI, and human-in-the-loop flows.
- **A2UI** defines structured UI payloads that an agent can produce and a
  client renderer can display.
- **Frontend frameworks and channels** render the final experience for users.

## Choose an interface

| Surface | Use it for | Runtime relationship |
|---|---|---|
| ADK Web | Local development, debugging, inspecting events, and editing session state. | Runs against ADK agents and shows runtime behavior in a development UI. |
| API Server | Programmatic testing, backend integrations, and custom clients that want the raw ADK event stream. | Exposes ADK agent runs through `/run` and `/run_sse`. |
| Custom frontends with ADK APIs | Frontend adapters, SDKs, or clients where you own the full event mapping. | Consumes ADK events directly and adapts them for the target UI. |
| AG-UI | Production application UIs that need streaming messages, lifecycle events, state sync, tool calls, generative UI, and human-in-the-loop interactions. | Maps ADK events and sessions into a stable client-facing event protocol. |
| A2UI | Agent responses that should include portable structured UI payloads such as cards, forms, charts, and tables. | Produces renderable payloads that can travel through AG-UI, A2A, MCP, REST, or another stream. |
| Frontend frameworks and channels | Rendering chat, controls, tools, and app-specific views in a browser, mobile app, or collaboration surface. | Consume the chosen frontend contract and send user input or tool results back to the runtime boundary. |

## Recommended implementation path

1. Start with [ADK Web](/runtime/web-interface/) while you build and debug the
   agent.
2. Use the [API Server](/runtime/api-server/) to inspect the `/run_sse` event stream
   your agent produces.
3. Choose whether your client should consume raw ADK events, AG-UI events, or
   structured A2UI payloads.
4. Build a [custom frontend with ADK APIs](/runtime/frontend-interfaces/custom-frontends/)
   when you want to own the full client contract yourself.
5. Add [AG-UI](/integrations/ag-ui/) when application clients need a
   stable streaming event contract.
6. Add [A2UI](/integrations/a2ui/) when the agent should return structured
   UI payloads that a client renderer can display.

## Current frontend paths

<div class="grid cards" markdown>

-   :material-transit-connection-variant:{ .lg .middle } **AG-UI**

    ---

    Use AG-UI when you need the most complete frontend integration path:
    streaming messages, shared state, tool rendering, generative UI, and
    human-in-the-loop interactions.

    [:octicons-arrow-right-24: Build with AG-UI](/integrations/ag-ui/)

-   :material-card-multiple-outline:{ .lg .middle } **A2UI**

    ---

    Use A2UI when the agent should describe structured UI that can be rendered
    across different clients and component catalogs.

    [:octicons-arrow-right-24: Generate UI with A2UI](/integrations/a2ui/)

-   :material-api:{ .lg .middle } **Custom frontends with ADK APIs**

    ---

    Use the API server when you want to build a frontend adapter or SDK from the
    ADK Runtime event stream yourself.

    [:octicons-arrow-right-24: Build a custom frontend](/runtime/frontend-interfaces/custom-frontends/)

</div>

## Frontend patterns

After you choose the interface, pick the UI patterns your application needs:
controlled generative UI, declarative A2UI payloads, open-ended UI surfaces,
tool rendering, in-app generative UI, and human-in-the-loop.

[:octicons-arrow-right-24: Explore frontend patterns](/runtime/frontend-interfaces/patterns/)
