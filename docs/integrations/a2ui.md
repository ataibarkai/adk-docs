---
catalog_title: A2UI
catalog_description: Generate rich, structured UIs from your agents using the Agent-to-UI protocol
catalog_icon: /integrations/assets/a2ui.svg
---

# A2UI - declarative generative UI for ADK

A2UI is a declarative generative UI spec. Instead of returning only text, an ADK
agent can assemble a UI from a catalog of approved components: cards, forms,
charts, tables, buttons, and domain-specific views. The agent outputs
declarative JSON, and a renderer on the client turns it into approved
application UI.

It's transport-agnostic: A2UI payloads work over A2A, MCP, REST, WebSockets,
AG-UI, or any other protocol. A2UI describes *what* to render; the client and
its renderer decide *how* to render it.

!!! tip "A2UI and frontend protocols"
    A2UI is a transport-agnostic declarative UI contract. It answers what UI the
    agent wants to show, not how the full conversation stream is delivered. Pair
    A2UI with [AG-UI](/integrations/ag-ui/) when your ADK app also needs
    bidirectional streaming messages, agent activity, tool calls, state sync,
    lifecycle events, frontend actions, or human-in-the-loop flows. See
    [Frontend interfaces](/runtime/frontend-interfaces/) for the full ADK
    frontend map.

## What A2UI owns

| Layer | Responsibility |
|---|---|
| Component catalog | The approved components the agent may use, including descriptions, prop schemas, examples, and renderer mappings. |
| Agent prompt | The schema and examples that teach the model how to assemble valid A2UI messages. |
| Validation | The boundary that treats generated UI as untrusted until it matches the selected catalog and protocol version. |
| Renderer | The client-side implementation that maps A2UI descriptors to native UI in your design system. |

AG-UI, A2A, REST, MCP, or a custom stream can carry the validated payload. A2UI
does not replace the ADK Runtime, your session model, or the frontend event
protocol.

## ADK quickstart

This journey mirrors the official A2UI ADK flow: start with a normal ADK agent,
test it in `adk web`, then upgrade the response from text to declarative UI.

### Install ADK and the A2UI SDK

```bash
pip install -U google-adk a2ui-agent-sdk
```

!!! warning "Version note"
    The current `a2ui-agent-sdk` package requires Python 3.14 or newer. The
    snippets below use the current SDK import paths and show v0.9 constants
    because those are the stable paths used by the A2UI agent development guide.

### 1. Start with a plain ADK agent

Create an ADK agent and tool the normal way. Verify the text-only experience
first so you know the agent logic works before adding UI generation.

```python
from google.adk.agents import Agent

def get_resources() -> list[dict]:
    """Return project resources and their status."""
    return [
        {"name": "auth-service", "status": "healthy", "region": "us-west1"},
        {"name": "events-db", "status": "warning", "issue": "Storage at 92%"},
    ]

root_agent = Agent(
    model="gemini-flash-latest",
    name="cloud_dashboard",
    description="Reports on cloud resources.",
    instruction=(
        "When users ask about project resources, call get_resources and "
        "summarize the result in plain text."
    ),
    tools=[get_resources],
)
```

Run the agent locally:

```bash
adk web
```

Ask a prompt such as `What's running in my project?`. At this point the agent
should answer with text.

### 2. Set up the Schema Manager

The `A2uiSchemaManager` loads component catalogs and generates system prompts
that teach the LLM how to produce valid A2UI JSON.

```python
from a2ui.schema.constants import VERSION_0_9
from a2ui.schema.manager import A2uiSchemaManager
from a2ui.basic_catalog.provider import BasicCatalog

schema_manager = A2uiSchemaManager(
    version=VERSION_0_9,
    catalogs=[
        BasicCatalog.get_config(
            version=VERSION_0_9,
            examples_path="examples",
        ),
    ],
)
```

!!! note
    `A2uiSchemaManager` instances are version-specific. If your agent supports
    multiple A2UI protocol versions, preconfigure one schema manager per version
    and use `try_activate_a2ui_extension` at request time to select the active
    manager.

!!! tip
    If you omit the `catalogs` parameter, the schema manager uses the
    [Basic Catalog](https://a2ui.org/concepts/catalogs/) maintained by the
    A2UI team, which includes common components like Text, Card, Button,
    Image, and more. You can also create [custom catalogs](#custom-catalog-configuration)
    with domain-specific components, or mix the basic catalog with your own
    — see [Backend implementation notes](#backend-implementation-notes) below.

### 3. Generate the A2UI system prompt

The `generate_system_prompt` method combines your agent's role description with
the A2UI JSON schema and few-shot examples, so the LLM knows exactly how to
format its output.

```python
instruction = schema_manager.generate_system_prompt(
    role_description=(
        "You are a cloud infrastructure assistant. When users ask about "
        "resources, call get_resources before answering."
    ),
    workflow_description=(
        "Analyze the user's request and return structured UI when appropriate."
    ),
    ui_description=(
        "Use cards for resource summaries, tables for comparisons, buttons "
        "for drill-down actions, and forms when you need user input. Respond "
        "only with valid A2UI JSON."
    ),
    include_schema=True,
    include_examples=True,
)
```

### 4. Upgrade the ADK agent

Use the generated instruction as the ADK agent's system prompt:

```python
from google.adk.agents import Agent

root_agent = Agent(
    model="gemini-flash-latest",
    name="cloud_dashboard",
    description="An agent that generates rich UI responses.",
    instruction=instruction,
    tools=[get_resources],
)
```

Run `adk web` again and ask the same prompt. The agent should now produce A2UI
JSON instead of a wall of text. The exact rendering path depends on the client:
ADK Web can be used for local experiments, while production apps should use a
client renderer or carry A2UI through an application protocol such as AG-UI.

If your local client shows raw JSON, that is still useful for the quickstart:
copy the model text into `llm_output_text` in the validation step below. In a
production adapter, `llm_output_text` is the text content you receive from the
ADK event stream before you parse, validate, and forward the A2UI payload to a
renderer.

### 5. Validate A2UI output

Always validate the LLM's JSON output before sending it to the client. The SDK
provides parsing, fixing, and validation utilities:

```python
from a2ui.parser.parser import parse_response
from a2ui.a2a.parts import parse_response_to_parts

# Get the active catalog's validator
selected_catalog = schema_manager.get_selected_catalog()

# Option A: Manual parse + validate
response_parts = parse_response(llm_output_text)
for part in response_parts:
    if part.a2ui_json:
        selected_catalog.validator.validate(part.a2ui_json)

# Option B: One-liner that returns A2A Parts
parts = parse_response_to_parts(
    llm_output_text,
    validator=selected_catalog.validator,
    fallback_text="Here's what I found.",
)
```

A2UI JSON is untrusted model output until it validates against the selected
catalog. Validate before rendering, and use deterministic fallback text or retry
logic when validation fails.

When you carry A2UI through A2A, the payload is wrapped in a `DataPart` so
renderers can identify it. The v1.0 A2A extension uses the MIME type
`application/a2ui+json`:

```json
{
  "kind": "data",
  "data": [
    {
      "version": "v1.0",
      "createSurface": {
        "surfaceId": "hello",
        "catalogId": "https://example.com/catalog.json"
      }
    }
  ],
  "metadata": {
    "mimeType": "application/a2ui+json"
  }
}
```

After validation, carry the payload through the stream your frontend already
uses: A2A, AG-UI, REST, Server-Sent Events, or a custom ADK API adapter.

## Backend implementation notes

The live side-by-side A2UI example is in
[Declarative generative UI with A2UI](/runtime/frontend-interfaces/patterns/declarative-generative-ui-a2ui/).
The sections below are backend reference snippets for catalog selection,
catalog configuration, and capability advertisement.

### Dynamic catalog selection

For agents that need different UI components depending on context (e.g., charts
for data queries, forms for configuration), resolve the catalog at runtime and
store it in session state:

```python
from a2ui.schema.constants import A2UI_CLIENT_CAPABILITIES_KEY

async def _prepare_session(self, context, run_request, runner):
    session = await super()._prepare_session(context, run_request, runner)

    # Determine client capabilities from request metadata
    capabilities = context.message.metadata.get(A2UI_CLIENT_CAPABILITIES_KEY)

    # Select the right catalog
    a2ui_catalog = self.schema_manager.get_selected_catalog(
        client_ui_capabilities=capabilities
    )
    examples = self.schema_manager.load_examples(a2ui_catalog, validate=True)

    # Store lightweight catalog metadata in session state for tool access
    await runner.session_service.append_event(
        session,
        Event(
            actions=EventActions(
                state_delta={
                    "system:a2ui_enabled": True,
                    "system:a2ui_catalog_id": a2ui_catalog.catalog_id,
                    "system:a2ui_component_ids": [
                        component.id for component in a2ui_catalog.components
                    ],
                    "system:a2ui_example_count": len(examples),
                }
            ),
        ),
    )
    return session
```

### Custom catalog configuration

You can define your own component catalogs for domain-specific UI:

```python
from a2ui.schema.manager import CatalogConfig

schema_manager = A2uiSchemaManager(
    version=VERSION_0_9,
    catalogs=[
        BasicCatalog.get_config(version=VERSION_0_9),
        CatalogConfig.from_path(
            name="my_dashboard_catalog",
            catalog_path="catalogs/dashboard.json",
            examples_path="catalogs/dashboard_examples",
        ),
    ],
)
```

### Agent card capabilities

Orchestrator agents can aggregate A2UI capabilities from sub-agents and
advertise them in the agent card:

```python
from a2ui.a2a.extension import get_a2ui_agent_extension

# Collect catalog IDs from sub-agents
supported_catalog_ids = set()
for subagent in subagents:
    for extension in subagent_card.capabilities.extensions:
        if extension.uri == "https://a2ui.org/a2a-extension/a2ui/v0.9":
            supported_catalog_ids.update(
                extension.params.get("supportedCatalogIds") or []
            )

# Advertise in the orchestrator's AgentCard
agent_card = AgentCard(
    capabilities=AgentCapabilities(
        extensions=[
            get_a2ui_agent_extension(
                version=VERSION_0_9,
                supported_catalog_ids=list(supported_catalog_ids),
            )
        ]
    )
)
```

## Samples

The A2UI repository includes ADK sample agents you can run immediately:

| Sample | Description |
|---|---|
| [restaurant_finder](https://github.com/a2ui-project/a2ui/tree/main/samples/agent/adk/restaurant_finder) | Static schema agent for searching and displaying restaurant information |
| [rizzcharts](https://github.com/a2ui-project/a2ui/tree/main/samples/community/agent/adk/rizzcharts) | Dynamic catalog agent that selects chart components based on context |
| [orchestrator](https://github.com/a2ui-project/a2ui/tree/main/samples/community/agent/adk/orchestrator) | Multi-agent setup that delegates to sub-agents and aggregates UI capabilities |

## Resources

- [A2UI specification](https://a2ui.org/)
- [A2UI GitHub repository](https://github.com/a2ui-project/a2ui)
- [A2UI Python SDK (`a2ui-agent-sdk`)](https://pypi.org/project/a2ui-agent-sdk/)
- [Agent development guide](https://github.com/a2ui-project/a2ui/blob/main/agent_sdks/python/a2ui_agent/agent_development.md)
- [Component gallery](https://a2ui.org/reference/components/)
- [A2A protocol](https://a2a-protocol.org)
