# Custom frontends with ADK APIs

Build a custom frontend with ADK APIs when you want to own the client contract
yourself. This is the native ADK API path: your application reads ADK Runtime
events from the API server, maps them into UI state, and sends user input or
tool responses back to the runtime boundary.

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

At a high level, the adapter does this:

```text
/run_sse Event
  -> normalize message, tool, state, error, or lifecycle event
  -> update application UI state
  -> send user input, tool results, or human decisions back to ADK
```

## Minimal browser adapter

Browser clients usually call `/run_sse` with `fetch()` instead of `EventSource`
because the ADK API server uses `POST` for agent runs. The skeleton below creates
a session, streams events, and normalizes the event shapes that most frontends
need to handle.

```ts title="adk-client.ts"
type AdkPart =
  | { text?: string }
  | { functionCall?: { id: string; name: string; args: unknown } }
  | { functionResponse?: { id: string; name: string; response: unknown } };

type AdkEvent = {
  content?: { role?: string; parts?: AdkPart[] };
  actions?: {
    stateDelta?: Record<string, unknown>;
    requestedAuthConfigs?: Record<string, unknown>;
  };
  error?: string;
  id?: string;
  timestamp?: number;
};

type NormalizedEvent =
  | { type: "message"; role?: string; text: string }
  | { type: "tool_call"; id: string; name: string; args: unknown }
  | { type: "tool_result"; id: string; name: string; result: unknown }
  | { type: "state"; delta: Record<string, unknown> }
  | { type: "human_or_auth_request"; configs: Record<string, unknown> }
  | { type: "error"; message: string };

const apiBase = "http://localhost:8000";

export async function ensureSession(
  appName: string,
  userId: string,
  sessionId: string,
  initialState: Record<string, unknown> = {},
) {
  const response = await fetch(
    `${apiBase}/apps/${appName}/users/${userId}/sessions/${sessionId}`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(initialState),
    },
  );

  if (!response.ok && response.status !== 409) {
    throw new Error(`Could not create ADK session: ${response.status}`);
  }
}

export async function runAdkTurn({
  appName,
  userId,
  sessionId,
  text,
  onEvent,
}: {
  appName: string;
  userId: string;
  sessionId: string;
  text: string;
  onEvent: (event: NormalizedEvent) => void;
}) {
  const response = await fetch(`${apiBase}/run_sse`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      appName,
      userId,
      sessionId,
      streaming: true,
      newMessage: {
        role: "user",
        parts: [{ text }],
      },
    }),
  });

  if (!response.ok || !response.body) {
    throw new Error(`ADK run failed: ${response.status}`);
  }

  for await (const event of readSseEvents(response.body)) {
    for (const normalized of normalizeAdkEvent(event)) {
      onEvent(normalized);
    }
  }
}

async function* readSseEvents(stream: ReadableStream<Uint8Array>) {
  const reader = stream.pipeThrough(new TextDecoderStream()).getReader();
  let buffer = "";

  while (true) {
    const { value, done } = await reader.read();
    if (done) break;
    buffer += value;

    let boundary = buffer.indexOf("\n\n");
    while (boundary !== -1) {
      const frame = buffer.slice(0, boundary);
      buffer = buffer.slice(boundary + 2);
      boundary = buffer.indexOf("\n\n");

      const data = frame
        .split("\n")
        .filter((line) => line.startsWith("data:"))
        .map((line) => line.slice("data:".length).trim())
        .join("\n");

      if (data) yield JSON.parse(data) as AdkEvent;
    }
  }
}

function normalizeAdkEvent(event: AdkEvent): NormalizedEvent[] {
  if (event.error) return [{ type: "error", message: event.error }];

  const normalized: NormalizedEvent[] = [];
  for (const part of event.content?.parts ?? []) {
    if ("text" in part && part.text) {
      normalized.push({
        type: "message",
        role: event.content?.role,
        text: part.text,
      });
    }
    if ("functionCall" in part && part.functionCall) {
      normalized.push({
        type: "tool_call",
        id: part.functionCall.id,
        name: part.functionCall.name,
        args: part.functionCall.args,
      });
    }
    if ("functionResponse" in part && part.functionResponse) {
      normalized.push({
        type: "tool_result",
        id: part.functionResponse.id,
        name: part.functionResponse.name,
        result: part.functionResponse.response,
      });
    }
  }

  const stateDelta = event.actions?.stateDelta;
  if (stateDelta && Object.keys(stateDelta).length > 0) {
    normalized.push({ type: "state", delta: stateDelta });
  }

  const authConfigs = event.actions?.requestedAuthConfigs;
  if (authConfigs && Object.keys(authConfigs).length > 0) {
    normalized.push({
      type: "human_or_auth_request",
      configs: authConfigs,
    });
  }

  return normalized;
}
```

Use the normalized events to update your application's message list, tool-call
registry, shared state store, and approval UI. When the user submits another
message, a tool result, or a human decision, send another `/run_sse` request with
the `newMessage.parts` shape your adapter defines for that response.

For production browser apps, put the ADK API server behind an authenticated
backend or edge proxy. The frontend adapter should not expose service
credentials, privileged tools, or unrestricted session state directly to the
browser.

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

When you choose this path, you also own the client protocol over time:
reconnect behavior, resume semantics, streaming text assembly, state
reconciliation, tool lifecycle mapping, human-response mapping, authorization
boundaries, and versioning.

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
