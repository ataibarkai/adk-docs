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
client experience. AG-UI and A2UI are complementary layers: AG-UI is the
transport layer for agent-user interaction, while A2UI is the visual language
for declarative UI.

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
- **AG-UI** is the transport layer: an event-based, transport-agnostic pipe
  between agent backends and application frontends for messages, activity,
  state, tools, frontend actions, human-in-the-loop flows, and UI
  specifications.
- **A2UI** is the visual language: an Agent-to-User Interface specification for
  declarative UI widgets. The agent assembles trusted, catalog-defined building
  blocks, and a client renderer displays them with native application
  components.
- **Frontend frameworks and channels** render the final experience for users.

## Choose by layer

These are complementary layers. Choose the interaction contract your frontend
needs, then add any UI payload specs or rendering surfaces your experience
requires.

| Layer | Choose | Use it for |
|---|---|---|
| Development and debugging | [ADK Web](/runtime/web-interface/) | Build, inspect, and debug an agent locally. Do not use it as a production product UI. |
| Runtime transport | [API Server](/runtime/api-server/) | Expose ADK agent runs through `/run` for batch-style calls or `/run_sse` for event streams. |
| Client event transport | [AG-UI](/integrations/ag-ui/) or [custom ADK APIs](/runtime/frontend-interfaces/custom-frontends/) | Use AG-UI for the recommended interoperable interaction pipe across web, mobile, chat, and other surfaces. Use custom ADK APIs when you need to own the lower-level adapter and client contract yourself. |
| Visual UI language | [A2UI](/integrations/a2ui/) | Add portable declarative UI widgets, generated from approved component catalogs, to a stream that preserves payload metadata, ordering, versioning, and client capabilities. |
| Rendering surface | Frontend frameworks and channels | Render the final experience in React, Vue, React Native, Flutter, Slack, Microsoft Teams, or another client. |

## Recommended decision path

1. Start with [ADK Web](/runtime/web-interface/) while you build and debug the
   agent.
2. Use the [API Server](/runtime/api-server/) to inspect the `/run_sse` event
   stream your agent produces.
3. Choose the client-facing event transport:
   [AG-UI](/integrations/ag-ui/) when application clients need streaming
   messages, lifecycle events, state, tool calls, frontend actions, UI
   specifications, or human-in-the-loop flows; use
   [custom ADK APIs](/runtime/frontend-interfaces/custom-frontends/) when you
   need to own the lower-level mapping from ADK events to your product's client
   protocol.
4. Add [A2UI](/integrations/a2ui/) when the agent should return structured UI
   payloads assembled from trusted component catalogs. A2UI is the visual
   language; AG-UI or another transport carries it to the client renderer.
5. Render the chosen contract in the framework or channel your users actually
   use.

## Frontend contracts

<div class="grid cards" markdown>

-   :material-transit-connection-variant:{ .lg .middle } **AG-UI**

    ---

    Use AG-UI when you need the transport layer for agent-user interaction:
    streaming messages, shared state, tool rendering, frontend actions, UI
    specifications, and human-in-the-loop interactions.

    [:octicons-arrow-right-24: Build with AG-UI](/integrations/ag-ui/)

-   :material-api:{ .lg .middle } **Custom ADK APIs**

    ---

    Use the native ADK API path when you need a lower-level adapter or SDK built
    directly from the ADK Runtime event stream.

    [:octicons-arrow-right-24: Build a custom frontend](/runtime/frontend-interfaces/custom-frontends/)

</div>

## Visual language

<div class="grid cards" markdown>

-   :material-card-multiple-outline:{ .lg .middle } **A2UI**

    ---

    Use A2UI when the agent should speak UI by assembling declarative widgets
    from a catalog of trusted application components. Carry it through AG-UI,
    A2A, REST, MCP, or a custom stream.

    [:octicons-arrow-right-24: Generate UI with A2UI](/integrations/a2ui/)

</div>

## Frontend patterns

Each pattern page pairs implementation code with a live ADK-backed showcase.
Start with the concrete patterns your application needs, then use the spectrum
page to compare how much control belongs to the developer, the agent, and the
client renderer.

[:octicons-arrow-right-24: Explore frontend patterns](/runtime/frontend-interfaces/patterns/)

[:octicons-arrow-right-24: Compare the generative UI spectrum](/runtime/frontend-interfaces/patterns/generative-ui-spectrum/)
