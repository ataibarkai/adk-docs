---
hide:
  - toc
---

# Frontend patterns

ADK agents can power many kinds of user interfaces. The frontend decision is not
only which framework renders the app, but which runtime contract carries
messages, tool calls, state, approvals, and structured UI between ADK and the
client.

This page treats [AG-UI](/integrations/ag-ui/) as the most complete frontend
path for production application UIs. It can carry every pattern below. A2UI gets
its own standalone lane as a declarative UI payload format, and a custom ADK API
client remains available when you want to build your own frontend SDK directly
on `/run_sse`.

!!! note "Current runnable examples"

    The embedded examples use the CopilotKit showcase because it already
    demonstrates these frontend patterns against agent backends. The same
    pattern applies to ADK when ADK Runtime events are adapted into AG-UI events
    or when A2UI payloads are carried through your chosen client stream.

## Choose the layer first

| Layer | Use it when | What it owns |
|---|---|---|
| ADK Runtime | You are building, running, and deploying agents. | Agent execution, sessions, tools, and runtime events. |
| Custom ADK frontend client | You need a product-specific SDK or want to map `/run_sse` yourself. | Raw event consumption, state mapping, retries, and client contract design. |
| AG-UI | Your application needs streaming messages, tool rendering, frontend tools, shared state, generative UI, or human approval flows. | A stable client-facing event protocol across web, mobile, and other frontend surfaces. |
| A2UI | The agent should return a portable structured UI payload. | A declarative component payload that can travel through AG-UI, A2A, MCP, REST, or another stream. |
| Frontend frameworks and channels | You are rendering the final user experience. | React, Vue, React Native, Flutter, Slack, Microsoft Teams, or another client surface. |

## Feature examples

### Controlled generative UI

Use controlled generative UI when the agent can choose *when* to show a UI
surface, but the application keeps ownership of the component, styling,
validation, and interaction model. This is usually the first rich UI pattern to
add to an AG-UI frontend.

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="React client"
import { useComponent } from "@copilotkit/react-core/v2";
import { z } from "zod";

const renewalRiskCard = z.object({
  accountName: z.string(),
  riskLevel: z.enum(["low", "medium", "high"]),
  annualRecurringRevenue: z.string(),
  summary: z.string(),
  signals: z.array(z.string()),
  nextSteps: z.array(z.string()),
});

export function RenewalRiskUI() {
  useComponent({
    agentId: "adk_agent",
    name: "showRenewalRiskCard",
    description: "Render a renewal risk card for an account.",
    parameters: renewalRiskCard,
    render: RenewalRiskCard,
  });

  return null;
}
```

The agent emits a typed tool call. The frontend renders a known application
component and sends the result back through the AG-UI stream.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/langgraph-python/gen-ui-tool-based/preview" title="Controlled generative UI showcase" loading="lazy"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/langgraph-python/gen-ui-tool-based/preview){ target="_blank" }

</div>

</div>

### A2UI declarative UI

Use A2UI when the agent should return a portable UI payload instead of selecting
one hard-coded component. A2UI can stand alone as a payload format, or it can
travel through AG-UI when the same frontend also needs streaming messages,
state, tools, and approvals.

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```python title="ADK-side tool shape"
from copilotkit import a2ui

CATALOG_ID = "copilotkit://renewal-catalog"
SURFACE_ID = "renewal-plan"

def display_renewal_plan(account_name: str, signals: list[str]) -> str:
    return a2ui.render(
        operations=[
            a2ui.create_surface(SURFACE_ID, catalog_id=CATALOG_ID),
            a2ui.update_components(SURFACE_ID, RENEWAL_PLAN_SCHEMA),
            a2ui.update_data_model(
                SURFACE_ID,
                {
                    "accountName": account_name,
                    "riskLevel": "high",
                    "signals": signals,
                    "nextSteps": ["Schedule sponsor review"],
                },
            ),
        ],
    )
```

The agent returns structured A2UI operations. The client renderer maps the
payload to an approved component catalog.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/langgraph-python/declarative-gen-ui/preview" title="A2UI declarative UI showcase" loading="lazy"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/langgraph-python/declarative-gen-ui/preview){ target="_blank" }

</div>

</div>

### Open generative UI and MCP Apps

Use open generative UI when the agent needs to display a richer tool-owned
surface, such as an MCP App, rather than a component that lives entirely in the
host product. AG-UI can carry the run events while the frontend hosts the app
surface with the right sandbox, permissions, and lifecycle.

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```ts title="Runtime configuration"
import { CopilotRuntime } from "@copilotkit/runtime/v2";
import { HttpAgent } from "@ag-ui/client";

export const runtime = new CopilotRuntime({
  agents: {
    adk_agent: new HttpAgent({
      url: process.env.ADK_AG_UI_URL ?? "http://localhost:8000/",
    }),
  },
  mcpApps: {
    servers: [
      {
        type: "http",
        url: process.env.MCP_SERVER_URL ?? "https://mcp.excalidraw.com",
        serverId: "excalidraw",
        agentId: "adk_agent",
      },
    ],
  },
});
```

The runtime makes an app-capable MCP server available. The agent can call it,
and the frontend renders the returned app surface instead of hand-building a
custom component for every tool.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/langgraph-python/mcp-apps/preview" title="MCP Apps showcase" loading="lazy"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/langgraph-python/mcp-apps/preview){ target="_blank" }

</div>

</div>

### Tool rendering

Use tool rendering when the most important UI is the status, arguments, result,
or error for a backend tool call. The frontend can provide a custom renderer for
important tools and a default renderer for the long tail.

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="React client"
import { useRenderToolCall } from "@copilotkit/react-core";

export function ToolCallRenderers() {
  useRenderToolCall({
    name: "lookup_renewal_risk",
    render: ({ args, result, status }) => (
      <ToolCard
        title={`Renewal risk for ${args.accountName}`}
        status={status}
        result={result}
      />
    ),
  });

  return null;
}
```

The ADK agent still owns the tool call. The client owns how progress, results,
empty states, retries, and errors appear to the user.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/langgraph-python/tool-rendering-default-catchall/preview" title="Tool rendering showcase" loading="lazy"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/langgraph-python/tool-rendering-default-catchall/preview){ target="_blank" }

</div>

</div>

### In-app generative UI

Use in-app generative UI when the agent should read application context or ask
the frontend to take a domain action, such as selecting a record, updating a
filter, opening a panel, or triggering a browser/native capability.

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="React client"
import { useAgentContext, useFrontendTool } from "@copilotkit/react-core/v2";
import { z } from "zod";

export function DashboardBridge({ selectedAccount, onSelectAccount }) {
  useAgentContext({
    description: "Current dashboard account and visible worklist.",
    value: { selectedAccount },
  });

  useFrontendTool({
    agentId: "adk_agent",
    name: "selectAccount",
    description: "Select an account in the dashboard.",
    parameters: z.object({ accountName: z.string() }),
    handler: async ({ accountName }) => onSelectAccount(accountName),
    followUp: false,
  });

  return null;
}
```

The app exposes safe context and safe actions. The agent can respond in the UI
without receiving unlimited access to the browser or application internals.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/langgraph-python/frontend-tools/preview" title="In-app generative UI showcase" loading="lazy"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/langgraph-python/frontend-tools/preview){ target="_blank" }

</div>

</div>

### Shared state

Use shared state when the frontend and agent need a durable, synchronized view
of the same working object. The UI can write state as the user navigates, and
the agent can write state back when it discovers or produces useful context.

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="React client"
import { useAgent, UseAgentUpdate } from "@copilotkit/react-core/v2";

export function AccountBrief({ account }) {
  const { agent } = useAgent({
    agentId: "adk_agent",
    updates: [UseAgentUpdate.OnStateChanged],
  });

  function updateFocus(operatorFocus: string) {
    agent.setState({
      ...agent.state,
      accountBrief: {
        accountId: account.id,
        company: account.company,
        operatorFocus,
      },
    });
  }

  return <FocusSelect onChange={updateFocus} />;
}
```

Shared state turns the frontend from a passive renderer into part of the
agent's working context while preserving an explicit state boundary.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/langgraph-python/shared-state-read-write/preview" title="Shared state showcase" loading="lazy"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/langgraph-python/shared-state-read-write/preview){ target="_blank" }

</div>

</div>

### Human-in-the-loop

Use human-in-the-loop when an agent run must pause for review before continuing:
approving an action, revising generated content, filling missing fields, or
canceling a risky step. The frontend owns the review experience and sends the
decision back through the run stream.

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="React client"
import { useHumanInTheLoop } from "@copilotkit/react-core/v2";
import { z } from "zod";

export function RenewalApproval() {
  useHumanInTheLoop({
    agentId: "adk_agent",
    name: "reviewRenewalOutreach",
    description: "Ask the user to approve or revise outreach.",
    parameters: z.object({
      accountName: z.string(),
      draftMessage: z.string(),
      riskReason: z.string(),
    }),
    render: ({ args, status, respond }) => (
      <ApprovalCard
        draft={args.draftMessage}
        disabled={status !== "executing"}
        onApprove={() => respond({ decision: "approved" })}
        onRevise={(note) => respond({ decision: "revise", note })}
      />
    ),
  });

  return null;
}
```

The agent asks for a decision. The application presents the right approval UI,
records the human response, and lets the agent continue with that result.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/langgraph-python/hitl-in-chat/preview" title="Human-in-the-loop showcase" loading="lazy"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/langgraph-python/hitl-in-chat/preview){ target="_blank" }

</div>

</div>

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

You can also build a
[custom frontend adapter](/runtime/frontend-interfaces/custom-frontends/)
directly on the ADK API server. This is the lowest-level option: your client or
backend-for-frontend reads `/run_sse`, maps ADK events into UI state, and sends
user input or tool responses back through the runtime boundary.

Choose this path when you need a custom client contract, already have a
frontend runtime abstraction, or want to build an ADK-specific SDK for your own
product.

## Keep the boundaries explicit

The frontend interface should adapt ADK events for clients; it should not
replace the agent, the session model, or the deployment model. A good frontend
integration keeps these responsibilities separate:

- ADK owns agent execution, tools, sessions, and runtime events.
- The frontend interface owns the client-facing event or payload contract.
- The application owns rendering, permissions, auth context, and user
  experience.
