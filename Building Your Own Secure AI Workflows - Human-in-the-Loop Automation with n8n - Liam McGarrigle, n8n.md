# Building your own secure AI workflows: Human-in-the-loop automation with n8n

| Field | Details |
| --- | --- |
| **Speaker** | Liam McGarrigle |
| **Affiliation** | n8n |
| **Type** | Workshop |
| **Date** | 2026-05-02 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=tDArkCqjA-c) |
| **Tags** | `rag-context-and-search` `agent-engineering` `apis-mcp-and-protocols` `reasoning-and-models` `safety-and-governance` `vision-and-video` |

## Summary

Liam McGarrigle, a developer advocate at n8n, builds a visual AI agent that manages
Gmail and Google Calendar — and then puts a human in the loop before it can send
email or create events. n8n is a drag-and-drop workflow canvas, but its fields also
accept small JavaScript expressions, so you can stay visual and drop into code only
where needed. The stated goal is less about building agents (now easy) and more about
seeing what an agent can do, inspecting what it did, and fixing the part that broke.

The build is incremental: add a **Chat Trigger**, attach an **AI Agent**, give it a
model (via OpenRouter, e.g. Sonnet 4.6), add **Simple Memory** so it recalls the
conversation, then expose Gmail and Calendar as **tools** beneath the agent. A key
idea is that n8n exposes each API field individually: only the fields you explicitly
delegate with `$fromAI` are filled by the model. That turns tool access into an
argument-by-argument decision rather than all-or-nothing API permission.

The security heart of the workshop is the human review step, which sits on the
connection between the agent and a tool. It intercepts the tool call before it runs,
so the model doesn't have to choose a separate "ask permission" tool correctly.
McGarrigle guards sending, replying, and event creation, and leaves archiving
unguarded as a personal risk choice. The back half covers improving the approval
preview, connecting n8n to external agents through instance-level MCP, and separating
action approval from workflow-change approval.

## Key Takeaways

- **A Chat Trigger → AI Agent → model + memory + tools** is the basic shape; only the
  model is required at first.
- **Memory is not automatic.** Attach Simple Memory (or Postgres/Redis) or the agent
  won't remember earlier messages.
- **Expressions (`{{ }}`)** mix static text, JavaScript helpers like `$now`, and values
  dragged from other nodes.
- **You choose exactly which arguments the model controls.** Only fields delegated
  with `$fromAI` are model-filled; the rest stay fixed.
- **Human review sits on the agent→tool connection**, intercepting the call before
  execution — no need for the model to pick the right review tool.
- **Name nodes and write descriptions carefully** — the node name becomes the tool
  name and the description becomes the tool description the model sees.
- **Instance-level MCP** lets external agents (e.g. Claude) call your workflows,
  enabled per workflow.
- **Automatic triggers (schedules, webhooks) can coexist with human authorization** of
  sensitive actions.

## Detailed Notes

### What we're building, and the chat entry point · 01:00
- n8n is a visual canvas that also accepts one-off JavaScript in any field — e.g.
  `.toUpperCase()` works directly without a separate Code node.
- The project is a simple Gmail and Calendar agent ending in a human-in-the-loop
  review step. A companion Notion guide has follow-along steps and the completed
  workflow JSON you can paste onto the canvas if you fall behind.
- n8n began in 2019 as a workflow-integration tool. Its primitives still explain the
  agent: a **trigger** starts an execution, **actions** call applications, and
  **branches** choose what happens next (e.g. a lead goes to Google Contacts if
  personal, Salesforce if business).
- Start with a **Chat Trigger**. A Slack event is another entry point, but the
  built-in chat keeps the first test local to the editor. Enable **Make available in
  Chat Hub** and publish to get a chat interface inside n8n; a test message appears in
  the workflow's **Executions** tab, verifying the entry point before any AI is wired in.
- McGarrigle's positioning: the winners in agents will be those who can *see* what
  their agent does and tweak it — control and visibility are the differentiator.

### Attach a model, then give it memory · 14:24
- Add an **AI Agent** node; its lower connectors accept a chat model, memory, and
  tools. Only the model is required at this stage.
- You can connect many providers — if one isn't listed, the OpenAI node often works by
  changing its base URL (also the route for an enterprise proxy). The demo uses
  OpenRouter with **Sonnet 4.6**, and the credential must match the provider node; an
  OpenRouter token won't work in another provider's node.
- With only a model, the agent can greet you but can't recall the first message — the
  app hasn't supplied history. Attach **Simple Memory**, which stores history in n8n
  and uses the Chat Trigger's **session ID** to tie messages to the right conversation.
- The memory's context-window setting controls how much prior conversation is sent;
  McGarrigle raises the default `5` toward `50` for longer chats, warning that past
  tokens cost money. Other choices: Postgres (queryable from another app) or Redis
  (integrate with an existing system).

### Map inputs, connect tools, and constrain arguments · 20:16
- A field can hold a fixed value or an **expression**. Inside `{{ }}` the content is
  evaluated; surrounding text stays literal. Drag a value from the trigger into another
  field to create a reference, e.g. `Request: {{ $json.chatInput }}`.
- Attach Gmail and Calendar actions as **tools beneath the agent**, not as a fixed
  sequence after it, so the agent chooses when to call them.
- Credential gotcha: opening a credential edits it; choosing **Add New Credential**
  creates another one. Gmail and Calendar each need their own Google sign-in here.
- Each operation exposes individual fields, and **only explicitly delegated fields are
  filled by the model.** Example: fix the subject and message, let the model set only
  `To`. The restriction comes from the configured interface, not a prompt.
- Setup cost: every model-supplied field must be configured. The auto-fill button
  inserts a helper expression; writing `$fromAI('to')` directly does the same and
  allows richer templates — useful when generated text should occupy only part of a
  field. An `undefined` preview just means no value has been supplied yet.

### Put instructions at the level where they apply · 29:03
- Rename an auto-named action to something direct like **Send an Email** and write the
  description manually. The **node name is the tool name**; the description tells the
  model what the tool is for.
- McGarrigle keeps email-specific instructions (including writing-style rules) inside
  the tool description, so copying the tool into another workflow carries them along.
- The **system message** holds the agent's identity, purpose, and general behavior.
  Start small and add instructions in response to observed failures. Two practical
  rules: ask for missing information rather than invent it, and never send unresolved
  placeholders (e.g. a prospective client's missing name).
- Add `The current date and time is {{ $now }}.` to the system message so the model
  doesn't search around an irrelevant date.
- Field descriptions resolve ambiguity: a Gmail thread ID differs from a message ID.
  And Calendar's **`summary`** field is really the event title — describe it as the
  title and even rename the model-facing key from `summary` to `title`.

### Inspect the execution, then add human approval · 38:03
- A first test shows a credential-access error: a credential can look selected while
  the project lacks access. Fix it from Home → Credentials → Share with the project.
  Selection and permission are separate.
- After a run, inspect the node output and the detailed execution log: it shows the
  actual operation (e.g. Gmail's *Get Many Messages* with its date-bounded search) even
  when the result is summarized in chat.
- Seeing it touch a real account makes the risk concrete — an accidental email would
  reach another person. So McGarrigle guards **sending, replying, and event creation**
  and leaves **archiving** unguarded as a personal risk decision. Coverage follows the
  consequence of each operation, not one blanket rule.
- To add a gate:
  1. Select the connection between the agent and its tool.
  2. Click the connection's plus button to add human review.
  3. Choose the review channel (the demo uses native n8n chat).
  4. Rename the approval button (e.g. **Send**) and add a decline option.
  5. Leave **Block User Input** off if the reviewer may respond with changes.
- A typed response denies the pending call, letting you correct the proposal instead of
  approving it.
- First guarded test hits a useful error: the Chat Trigger's **Response Mode** must
  change from **Streaming** to **Using Response Nodes**, giving workflow nodes control
  over what the user receives, including the review request.
- That also means the agent's normal final answer needs an explicit delivery node. Map
  the correct field — the agent returns `output`, not `message`: `{{ $json.output }}`.

### Show the proposal before approval · 46:35
- The default review message only says the agent wants to call the email tool — not
  enough to approve a send. You need the recipient and content.
- Inside the review message, `$tool.name` names the proposed tool and
  `$tool.parameters` exposes its arguments. Rendering the object raw gives
  `[object Object]`; use `{{ $tool.parameters.toJSONString() }}` to inspect it.
- A labeled message is clearer than a raw object: "The agent wants to send an email,"
  then `To:`, `Subject:`, and `Message:` pulled from `$tool.parameters`. These describe
  the pending call; they don't execute it. One review node can guard multiple tools,
  but its message must fit their different arguments.
- Mental model: **the agent calls the original tool; n8n intercepts the call.** The
  model doesn't have to choose a review tool correctly. Since it may still ask
  conversational permission, the tool description can explain that the call will be
  reviewed before running.
- Repair the Calendar preview with real data: open **Executions**, find the Chat Hub run
  (test runs are marked with a flask), and use **Copy to editor**. Then select the real
  timestamp field and apply `toDateTime()` followed by `format()` (Luxon-based; `DDDD`
  and `TT` give readable output), applying it only in the review message.

### External agents, MCP, and workflow governance · 58:46
- **Instance-level MCP:** enable it under `Settings → Instance level MCP` and grant
  access per workflow. Connection Details gives a server URL; connect via OAuth through
  the Claude connector or an access token. McGarrigle says the native integration works
  inside n8n and can create, read, and execute workflows — including sending a chat
  message into the email-and-calendar bot.
- A company-wide approval question: where does work wait, and which reviews take
  longest? An approval request puts an execution in **Waiting**, later moving to
  **Finished** after resuming. Waiting start/end times plus Slack details could be
  parsed in another workflow; he says human-review audit reporting needs polish.
- To move to Slack, replace the chat entry, final response, and review delivery with
  Slack equivalents. A schedule can trigger runs (e.g. "clear the inbox") while guarded
  actions still send approval requests to Slack — the same pattern he uses for GitHub
  issues and pull requests. Unanswered requests need a policy: he suggests **Limit Wait
  Time**, e.g. auto-deny after ten minutes.
- Two different approvals:
  - **Action approval:** a Slack trigger provides the requester's ID, but a vacation
    request should route to a manager or decision channel, not back to the requester.
  - **Workflow-change approval:** copy the workflow, edit the copy, and have its
    maintainer reintegrate reviewed work — or use Enterprise dev/staging/production
    environments with designated change managers. Autosave is not a review process.
- No approval button (e.g. a phone agent)? Telling the model to accept a proposal does
  not authorize the tool; if another model decides whether a spoken reply counts as
  approval, that interpretation becomes part of authorization. A custom sub-workflow
  (ask, get `true`/`false`, branch with an If node) is workflow-defined logic, not the
  built-in gate.
- To let an org keep its own interface and use n8n as an engine, a **Webhook Trigger**
  provides an HTTP entry point. And as the assistant grows, don't pile every domain onto
  one agent: use a **subagent tool** so the main agent calls specialists
  (email-and-calendar, GitHub issues, Jira), each with a model suited to its task.

## Resources

- [Google Calendar Events resource](https://developers.google.com/workspace/calendar/api/v3/reference/events)
  — reference for event titles (`summary`), start/end, and other fields.
- Understanding n8n Cloud concurrency — which executions count toward concurrency limits.
