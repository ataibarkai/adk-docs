# Shared state

Use shared state when the frontend and agent need a durable, synchronized view
of the same working object. The UI can write state as the user navigates, and
the agent can write state back when it discovers or produces useful context.

<p class="frontend-pattern-example-caption">CopilotKit AG-UI frontend backed by Google ADK.</p>

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="CopilotKit AG-UI client example"
import {
  useAgent,
  UseAgentUpdate,
} from "@copilotkit/react-core/v2";

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

  return <FocusSelect onChange={updateFocus} />; // App-defined control.
}
```

Shared state turns the frontend from a passive renderer into part of the
agent's working context while preserving an explicit state boundary. In ADK,
the adapter should map application-facing state events to the ADK session or
state model deliberately; do not treat shared state as unrestricted mutation of
agent internals.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/shared-state-read-write/preview" title="Shared state showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/shared-state-read-write/preview){ target="_blank" }

</div>

</div>

Return to [Frontend patterns](../patterns.md).
