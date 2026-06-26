# Open generative UI

Open generative UI gives the agent more room to produce or select a rich
surface. The host application still owns the sandbox, permissions, placement,
and lifecycle.

<p class="frontend-pattern-example-caption">CopilotKit AG-UI frontend backed by Google ADK.</p>

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="Host an open UI surface"
import { useRenderToolCall } from "@copilotkit/react-core/v2";

useRenderToolCall({
  name: "showOpenSurface",
  render: ({ result }) => (
    <SandboxedSurface
      src={result.url}
      title={result.title}
      permissions={result.permissions}
    />
  ),
});
```

The ADK agent can request a richer surface through the AG-UI stream. The
frontend decides whether the surface is trusted, where it appears, and what it
can do.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/open-gen-ui/preview" title="Open generative UI showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/open-gen-ui/preview){ target="_blank" }

</div>

</div>

Use this band for exploratory visualizations, tool-produced views, and surfaces
that need more freedom than a fixed component or A2UI catalog.

Next, compare [MCP Apps](mcp-apps.md) or return to the
[generative UI spectrum](generative-ui-spectrum.md).
