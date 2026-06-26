# Declarative generative UI with A2UI

Declarative UI gives the agent more freedom without handing it arbitrary code.
Developers provide a catalog of approved UI components, and the agent assembles
those components into a UI for the current task.

<p class="frontend-pattern-example-caption">CopilotKit AG-UI frontend backed by Google ADK.</p>

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```python title="A2UI prompt setup"
schema_manager = A2uiSchemaManager(
    version=VERSION_0_9,
    catalogs=[BasicCatalog.get_config(version=VERSION_0_9)],
)

instruction = schema_manager.generate_system_prompt(
    role_description="You are a renewal assistant.",
    workflow_description="Return structured UI when useful.",
    ui_description="Use cards, tables, buttons, and forms.",
    include_schema=True,
    include_examples=True,
)
```

A2UI defines what the agent can assemble. AG-UI can carry those A2UI messages
alongside state, tools, activity, human review, and user responses.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/declarative-gen-ui/preview" title="A2UI declarative UI showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/declarative-gen-ui/preview){ target="_blank" }

</div>

</div>

Use this band when the UI should vary by user intent while staying inside a
trusted component catalog and renderer.

Next, compare [open generative UI](open-generative-ui.md) or
[MCP Apps](mcp-apps.md), or return to the [spectrum overview](index.md).
