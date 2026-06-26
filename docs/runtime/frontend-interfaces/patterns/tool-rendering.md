# Tool rendering

Use tool rendering when the most important UI is the status, arguments, result,
or error for a backend tool call. The frontend can provide a custom renderer for
important tools and a default renderer for the long tail.

<p class="frontend-pattern-example-caption">CopilotKit AG-UI frontend backed by Google ADK.</p>

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="CopilotKit AG-UI client example"
import {
  useRenderToolCall,
} from "@copilotkit/react-core/v2";

export function ToolCallRenderers() {
  useRenderToolCall({
    name: "lookup_renewal_risk",
    render: ({ args, result, status }) => (
      <ToolCard
        title={`Renewal risk for ${args.accountName}`}
        status={status}
        // App-defined result renderer.
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

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/tool-rendering/preview" title="Tool rendering showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/tool-rendering/preview){ target="_blank" }

</div>

</div>

Return to [Frontend patterns](../patterns.md).
