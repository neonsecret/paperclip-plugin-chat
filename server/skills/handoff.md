## Skill: Agent Handoff

When the user @-mentions an agent with a work request, you should **hand off** the work by creating a Paperclip task assigned to that agent.

### When to trigger

- User says `@AgentName do something` — create a task assigned to that agent
- User says `@AgentName` with a question (e.g., "what's your status?") — query the agent's tasks instead, don't create a new task
- User uses `/handoff` — explicitly wants to assign work to an agent

### How to hand off

1. **Extract the intent** from the user's message — what work needs to be done?
2. **Create an issue** via `POST /api/companies/{companyId}/issues` with:
   - `title`: A concise summary of the work (you write this, don't just copy the user's raw message)
   - `description`: Full context including what the user asked for, any relevant details from the conversation, and acceptance criteria
   - `assigneeAgentId`: The agent's ID (provided in the `[Mentioned agents]` block)
   - `priority`: Infer from context — default to `medium` unless urgency is clear
   - `status`: `todo`
3. **Confirm the handoff** to the user with:
   - The task title and ID
   - Who it's assigned to
   - A link: `[View task](/issues/{issueId})`

### Example

User: `@CEO create a marketing plan for Q3`

You should:
```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$PAPERCLIP_COMPANY_ID/issues" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -H "X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID" \
  -d '{
    "title": "Create Q3 marketing plan",
    "description": "Create a comprehensive marketing plan for Q3. Include target audience, channels, budget allocation, and key milestones.",
    "assigneeAgentId": "ceo-agent-id",
    "priority": "medium",
    "status": "todo"
  }'
```

Then respond:
> Handed off to **CEO**: "Create Q3 marketing plan" — [View task](/issues/PAP-123)

### Multiple agents

If the user mentions multiple agents, create separate tasks for each unless the work is clearly collaborative — in that case, create one task and mention the others in the description.

### Context from conversation

When creating the task description, include relevant context from the current conversation. The agent picking up the task won't have access to this chat, so the description should be self-contained.
