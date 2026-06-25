# Custom frontends with ADK APIs

Build a custom frontend with ADK APIs when you want to own the client contract
yourself. In this path, your application reads ADK Runtime events from the API
server, maps them into UI state, and sends user input or tool responses back to
the runtime boundary.

This is the most flexible path and the most responsibility. It is a good fit
when you are building an internal frontend SDK, adapting ADK to an existing
application runtime, or integrating ADK events into a client protocol you
already operate.

## Runtime path

```text
ADK agent
  -> Runner and session services
  -> ADK Event stream
  -> API server /run or /run_sse
  -> Your frontend adapter or SDK
  -> Application UI
```

Start with the [API Server](/runtime/api-server/) and inspect the `/run_sse`
stream for your agent. Then build an adapter that converts ADK events into the
state shape your UI expects.

## Adapter responsibilities

| Responsibility | What to handle |
|---|---|
| Messages | Stream assistant text and preserve message ordering across events. |
| Sessions | Track app name, user ID, session ID, resume behavior, and state updates. |
| Tool calls | Render pending tool calls, tool results, errors, and progress states. |
| User input | Send new messages and any required tool or human responses back to ADK. |
| State | Decide which ADK state deltas become visible application state. |
| Errors and lifecycle | Surface run start, completion, cancellation, and failure states clearly. |
| Auth and permissions | Keep user identity, authorization, and client-only capabilities outside the agent prompt unless intentionally shared. |

## When to use this path

Use custom frontends with ADK APIs when:

- You already have a frontend runtime or client SDK and need ADK to plug into
  it.
- You want the lowest-level access to ADK events.
- You need a frontend contract that is specific to your product or platform.
- You are experimenting with a new client surface before adopting a standard
  protocol.

Use [AG-UI](/integrations/ag-ui/) instead when you want an existing
application-facing event protocol for streaming messages, state, tool calls,
generative UI, and human-in-the-loop interactions.

Add [A2UI](/integrations/a2ui/) when the agent should return structured UI
payloads that a renderer can display.

## Next steps

- [Use the API Server](/runtime/api-server/)
- [Understand the Event Loop](/runtime/event-loop/)
- [Compare frontend patterns](/runtime/frontend-interfaces/patterns/)
