# Frontend interfaces

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
- **AG-UI** provides a stable application-facing event protocol for the full
  back-and-forth between agents and application frontends: messages, activity,
  state, tools, frontend actions, generative UI, and human-in-the-loop flows.
- **A2UI** defines declarative generative UI payloads. The agent assembles
  trusted, catalog-defined building blocks, and a client renderer displays them
  with native application components.
- **Frontend frameworks and channels** render the final experience for users.

## Choose by layer

These are complementary layers. Choose the interaction contract your frontend
needs, then add any UI payload specs or rendering surfaces your experience
requires.

| Layer | Choose | Use it for |
|---|---|---|
| Development and debugging | [ADK Web](/runtime/web-interface/) | Build, inspect, and debug an agent locally. Do not use it as a production product UI. |
| Runtime transport | [API Server](/runtime/api-server/) | Expose ADK agent runs through `/run` for batch-style calls or `/run_sse` for event streams. |
| Client event contract | [Custom ADK APIs](/runtime/frontend-interfaces/custom-frontends/) or [AG-UI](/integrations/ag-ui/) | Use custom ADK APIs when you want to own the adapter and client contract yourself. Use AG-UI for a production-oriented application event protocol across web, mobile, chat, and other surfaces. |
| Structured UI payload | [A2UI](/integrations/a2ui/) | Add portable declarative UI built from approved component catalogs to the stream you already chose. |
| Rendering surface | Frontend frameworks and channels | Render the final experience in React, Vue, React Native, Flutter, Slack, Microsoft Teams, or another client. |

## Recommended decision path

1. Start with [ADK Web](/runtime/web-interface/) while you build and debug the
   agent.
2. Use the [API Server](/runtime/api-server/) to inspect the `/run_sse` event
   stream your agent produces.
3. Choose the client-facing event contract:
   [custom ADK APIs](/runtime/frontend-interfaces/custom-frontends/) when you
   want to own the full mapping from ADK events to your product's client
   protocol; [AG-UI](/integrations/ag-ui/) when application clients need
   streaming messages, lifecycle events, state, tool calls, frontend actions,
   generative UI, or human-in-the-loop flows.
4. Add [A2UI](/integrations/a2ui/) when the agent should return structured UI
   payloads assembled from trusted component catalogs. A2UI is composable with
   AG-UI, A2A, REST, MCP, and custom streams.
5. Render the chosen contract in the framework or channel your users actually
   use.

## Frontend contracts

<div class="grid cards" markdown>

-   :material-api:{ .lg .middle } **Custom ADK APIs**

    ---

    Use the native ADK API path when you want to build a frontend adapter or SDK
    from the ADK Runtime event stream yourself.

    [:octicons-arrow-right-24: Build a custom frontend](/runtime/frontend-interfaces/custom-frontends/)

-   :material-transit-connection-variant:{ .lg .middle } **AG-UI**

    ---

    Use AG-UI when you need a production-oriented application event protocol:
    streaming messages, shared state, tool rendering, frontend actions,
    generative UI, and human-in-the-loop interactions.

    [:octicons-arrow-right-24: Build with AG-UI](/integrations/ag-ui/)

</div>

## Structured UI payloads

<div class="grid cards" markdown>

-   :material-card-multiple-outline:{ .lg .middle } **A2UI**

    ---

    Use A2UI when the agent should assemble declarative UI from a catalog of
    trusted application components. Carry it through AG-UI, A2A, REST, MCP, or
    a custom stream.

    [:octicons-arrow-right-24: Generate UI with A2UI](/integrations/a2ui/)

</div>

## Frontend patterns

Use the generative UI spectrum to decide how much control belongs to the
developer, the agent, and the client renderer. Then pick the specific frontend
patterns your application needs: controlled generative UI, declarative A2UI
payloads, open-ended UI surfaces, MCP Apps, tool rendering, in-app generative
UI, shared state, and human-in-the-loop. The patterns page shows each feature
with code and a live embedded showcase.

[:octicons-arrow-right-24: Understand the generative UI spectrum](/runtime/frontend-interfaces/patterns/generative-ui-spectrum/)

[:octicons-arrow-right-24: Explore frontend patterns](/runtime/frontend-interfaces/patterns/)
