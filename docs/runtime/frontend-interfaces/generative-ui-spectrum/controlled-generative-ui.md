# Controlled generative UI

Controlled UI is the safest first step. The application owns the component,
props schema, validation, styling, and interaction model. The agent decides when
to show it and what typed data to pass through AG-UI.

<p class="frontend-pattern-example-caption">CopilotKit AG-UI frontend backed by Google ADK.</p>

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="Controlled UI example"
import { useComponent } from "@copilotkit/react-core/v2";

useComponent({
  agentId: "adk_agent",
  name: "showRenewalRiskCard",
  parameters: renewalRiskCardSchema,
  render: RenewalRiskCard,
});
```

The ADK agent emits a typed tool call through the frontend protocol. The
frontend renders a known component and sends the result back through the stream.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/gen-ui-tool-based/preview" title="Controlled generative UI showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/gen-ui-tool-based/preview){ target="_blank" }

</div>

</div>

Use this band when the application must keep exact ownership of UI behavior,
validation, state updates, and styling.

Next, compare
[declarative generative UI with A2UI](declarative-generative-ui-a2ui.md) or
return to the [spectrum overview](index.md).
