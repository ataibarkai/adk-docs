# Human-in-the-loop

Use human-in-the-loop when an agent run must pause for review before continuing:
approving an action, revising generated content, filling missing fields, or
canceling a risky step. The frontend owns the review experience and sends the
decision back through the run stream.

<p class="frontend-pattern-example-caption">CopilotKit AG-UI frontend backed by Google ADK.</p>

<div class="frontend-pattern-example" markdown>

<div class="frontend-pattern-copy" markdown>

```tsx title="CopilotKit AG-UI client example"
import {
  useHumanInTheLoop,
} from "@copilotkit/react-core/v2";
import { z } from "zod";

export function RenewalApproval() {
  useHumanInTheLoop({
    agentId: "adk_agent",
    name: "reviewRenewalOutreach",
    description:
      "Ask the user to approve or revise outreach.",
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
records the human response, and lets the agent continue with that result. The
backend still needs a pause/resume or human-input mechanism, such as ADK
[human input for workflows](/graphs/human-input/), that the frontend contract
can surface.

</div>

<div class="frontend-pattern-demo" markdown>

<iframe src="https://showcase.copilotkit.ai/integrations/google-adk/hitl-in-chat/preview" title="Human-in-the-loop showcase"></iframe>

[:octicons-link-external-16: Open showcase](https://showcase.copilotkit.ai/integrations/google-adk/hitl-in-chat/preview){ target="_blank" }

</div>

</div>

Return to [Frontend patterns](../patterns.md).
