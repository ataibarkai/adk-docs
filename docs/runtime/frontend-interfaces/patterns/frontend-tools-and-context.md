# Frontend tools and context

Use frontend tools and context for in-app generative UI: cases where the agent
should read application context or ask the frontend to take a domain action,
such as selecting a record, updating a filter, opening a panel, or triggering a
browser/native capability.

<p class="frontend-pattern-example-caption">CopilotKit AG-UI frontend backed by Google ADK.</p>

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="CopilotKit AG-UI client example"
import {
  useAgentContext,
  useFrontendTool,
} from "@copilotkit/react-core/v2";
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
without receiving unlimited access to the browser or application internals. In
ADK, the adapter decides which frontend tool calls are exposed to the agent and
how their results return to the runtime.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/frontend-tools/preview" title="In-app generative UI showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/frontend-tools/preview){ target="_blank" }

</div>

</div>

Return to [Frontend patterns](../patterns.md).
