# Getting Started with n8n: From Simple AI Agents to Multi-Agent Workflows and MCP

A practical, beginner-friendly collection of **seven importable n8n workflows**. Follow the exercises in order to understand AI agents, sequential and parallel orchestration, supervisor agents, MCP tools, and real-world calendar/email automation.

> **Important:** These are instructional/demo workflows, not guaranteed production-ready integrations. Import them into **your own n8n workspace**, reconnect credentials, and test with dummy data before using real accounts, appointments, or email recipients.

## Learning path

| Step | File | What you learn | Level |
| --- | --- | --- | --- |
| 1 | `Simple_Agent.json` | Chat trigger, AI Agent, model, memory, calculator tool | Beginner |
| 2 | `Sequential_Multiagent.json` | Pass one agent's output to the next | Beginner–intermediate |
| 3 | `Multiagent_Parallel.json` | Branch a request to independent agents | Intermediate |
| 4 | `Multiagent Supervisor.json` | A supervisor delegates work to specialist agent tools | Intermediate |
| 5 | `MCP Server.json` | Expose Google Calendar, Gmail and Sheets operations as MCP tools | Advanced |
| 6 | `auto scheduling and emailing agent.json` | Connect a single AI agent to an MCP server | Advanced |
| 7 | `Multi-agent scheduling assistant.json` | Coordinator + calendar, contacts and email specialists using MCP | Advanced |

## 1. What is n8n?

[n8n](https://n8n.io/) is a visual workflow automation platform. A **workflow** connects **nodes** that receive input, process data, call services or AI models, and pass results onward.

Key concepts:

- **Trigger:** starts a workflow (for these examples, usually an incoming chat message).
- **Node:** a step such as an AI Agent, OpenAI Chat Model, or Google Calendar operation.
- **Connection:** determines where data flows; AI connections attach models, memory and tools to an agent.
- **AI Agent:** uses instructions, a language model, and optionally tools/memory to decide what to do.
- **Credentials:** authorize access to external services. They must be configured in your own workspace.
- **Execution:** a single run; inspect execution logs and individual node inputs/outputs to debug.
- **MCP (Model Context Protocol):** a standard interface through which an agent can discover and call supported tools exposed by a server.

## 2. Start using n8n

### Option A — n8n Cloud (easiest for a workshop)

1. Open https://n8n.io/ and choose the n8n Cloud option.
2. Create or sign in to your account; open your n8n editor workspace.
3. Create a new workflow and import **Step 1** using the instructions below.
4. Add working AI model credentials and run a chat test.


### Import a JSON workflow

1. Download or place the supplied `.json` file on your computer.
2. In n8n, create a **New Workflow**.
3. In the workflow editor menu (typically **three dots**), select **Import from File** and choose the JSON. Menu wording can vary by n8n version.
4. Check that the nodes and their connections appear on the canvas.
5. Open each **OpenAI Chat Model** node, select/create credentials, and confirm the chosen model is available to your account.
6. For later exercises, also reconnect Google and MCP integrations.
7. **Save** the workflow. For chat-triggered exercises, use **Open chat** / the built-in test chat and send a message. For MCP server access, publish/activate the server workflow as required by your n8n version.
8. Examine each node's input/output and the **Executions** view.

**OpenAI setup:** Create an OpenAI API key using your own OpenAI API project, add it as an OpenAI credential in n8n, and select it in the model nodes. A ChatGPT subscription does not by itself include API usage. Models in the files include `gpt-4o-mini` and, in the sequential example, `gpt-4o`; you can choose another supported model if necessary.

> Imported credential references or AI-gateway settings might not work in a different n8n workspace. Re-select the credentials in every relevant node. Never put API keys in workflow JSON, screenshots, or a public repository.

## 3. Exercise 1 — Simple AI Agent

**File:** `Simple_Agent.json`  
**Purpose:** Understand the four essential ingredients of a useful agent: **instructions, LLM, memory, and tools**.

```text
Chat Trigger ──▶ AI Agent ──▶ Chat response
                     ├── OpenAI Chat Model (gpt-4o-mini)
                     ├── Simple Memory
                     └── Calculator
```

The system instructions ask the agent to behave like a student assistant, remember conversational context and use the calculator for calculations.

**Try in the built-in chat:**

```text
Hello! My name is Ahmed. I am studying computer science.
What is 125 * 48 + 75?
What did I tell you I am studying?
```

**Observe:** How the model, memory and calculator attach to the agent, and when the calculator tool is invoked. Memory is scoped to the configured chat/session behavior; do not assume permanent cross-user storage.

## 4. Exercise 2 — Sequential Multi-Agent Workflow

**File:** `Sequential_Multiagent.json`  
**Purpose:** Learn an ordered pipeline where each agent relies on the previous agent's output.

```text
Chat Trigger
     │
     ▼
Flight Recommendation Agent
     │  output
     ▼
Hotel Recommendation Agent
     │  output
     ▼
Travel Summary Agent
     │
     ▼
Final travel response
```

- **Flight agent:** provides a short flight suggestion.
- **Hotel agent:** receives the previous output via `{{ $json.output }}`, chooses a hotel and is instructed to preserve the flight details.
- **Summary agent:** receives the hotel agent's output and produces the final recommendation.

**Test prompt:**

```text
Plan a budget-friendly 3-day trip from Karachi to Islamabad next month.
Suggest a flight, hotel, and short travel summary.
```

**Learning point:** Each stage runs after the previous stage and can lose information if an earlier agent does not pass it along. The example contains **no live flight or hotel booking/search tools**: suggestions must not be treated as verified availability or prices.

## 5. Exercise 3 — Parallel Multi-Agent Workflow

**File:** `Multiagent_Parallel.json`  
**Purpose:** Send one request to multiple specialist agents on separate branches.

```text
                         ┌─▶ Flight Recommendation Agent ─┐
Chat Trigger ─────────────┼─▶ Hotel Recommendation Agent ──┼─▶ Travel Summary Agent
                         └─▶ Tourist Place Agent ──────────┘
```

- **Flight agent:** flight recommendation.
- **Hotel agent:** accommodation recommendation.
- **Tourist-place agent:** attractions to visit.
- **Travel summary agent:** intended to consolidate those suggestions.

**Test prompt:**

```text
I want a 4-day budget trip to Lahore.
Recommend a flight, hotel, and attractions to visit.
```

**Important implementation note:** This uploaded JSON connects three branches directly to the Travel Summary Agent and uses `{{ $json.output }}` as its prompt. That connection **does not explicitly aggregate** all three branch outputs into a single item; the summary agent may be invoked with branch-specific data rather than a combined result. For a reliable final combined summary, add an explicit **Merge/aggregation** step, collect each branch's result into clearly named fields, and then invoke the summary agent with all three fields. Check the execution order and output on your installed n8n version. The workflow is useful to demonstrate branching, but its fan-in needs verification/correction.

**Learning point:** Parallel-style branching is appropriate when tasks are independent; aggregation is a separate design responsibility.

## 6. Exercise 4 — Supervisor Multi-Agent Workflow

**File:** `Multiagent Supervisor.json`  
**Purpose:** Understand **delegation**: the supervisor AI Agent decides which specialist tool-agents to call.

```text
Chat Trigger ──▶ Supervisor AI Agent ──▶ Final admission response
                     ├── Marks Verification Agent (tool)
                     ├── Document Verification Agent (tool)
                     └── Admission Summary Agent (tool)
```

Specialists in the uploaded file:

1. **Marks Verification Agent:** checks reported marks and percentage eligibility.
2. **Document Verification Agent:** is instructed to check Aadhaar card and mark-sheet presence.
3. **Admission Summary Agent:** consolidates findings into a brief admission summary.

**Sample exercise:**

```text
Demo admission application: Student A obtained 780 marks out of 1100.
The minimum required percentage is 60%.
Assume the mark sheet and ID document are available.
Review the provided information and give an admission summary.
```

**Important limitation:** No actual document-reader, uploaded-file extractor or admissions database is attached to these specialist agents. The prompts tell the supervisor to **assume** documents are uploaded. Consequently, the workflow can *reason about supplied text*, but cannot independently verify a real ID or document. To turn it into a verification service, add document intake, parsing, source-of-truth checks and human review. Aadhaar is an India-specific identifier; adjust this document requirement to the local context (for example, CNIC/B-form) if appropriate.

**Learning point:** Unlike a fixed sequential chain, the supervisor dynamically delegates through AI Agent Tool connections.

## 7. Exercise 5 — MCP Server (tools for external services)

**File:** `MCP Server(2).json`  
**Workflow name inside file:** `MCP Calander` (original spelling retained).  
**Purpose:** Make Google Calendar, Gmail, and Google Sheets actions available to compatible MCP clients.

```text
                          ┌─ create_event
                          ├─ get_events
                          ├─ reschedule_event
MCP Server Trigger ────────┼─ delete_event
                          ├─ send_email
                          ├─ get_contacts
                          └─ save_contact
```

**What the seven tools do**

| MCP tool | Backing service | Purpose |
| --- | --- | --- |
| `create_event` | Google Calendar | Create an event; check conflicts with `get_events` first |
| `get_events` | Google Calendar | List events for a date/time window and retrieve real event IDs |
| `reschedule_event` | Google Calendar | Change an existing event's start/end |
| `delete_event` | Google Calendar | Delete a specified event by its verified ID |
| `send_email` | Gmail | Send an invitation or notification |
| `get_contacts` | Google Sheets | Read name/email contact records |
| `save_contact` | Google Sheets | Insert or update contacts by name |

**Configure the server first:**

1. Import `MCP Server(2).json` into your n8n workspace.
2. Open all **Google Calendar** nodes and reconnect Google Calendar OAuth credentials. Select **your own calendar** rather than relying on the imported calendar selection.
3. Open the **Gmail** node and reconnect Gmail OAuth credentials; use a test mailbox if possible.
4. Open both **Google Sheets** nodes. Connect Google Sheets OAuth credentials and select **your own contact spreadsheet and worksheet**.
5. Create a worksheet with the **exact headers `NAME` and `EMAILS`** (matching this JSON's contact mapping). Add a few fake testing contacts.
6. Verify the MCP Server Trigger is correctly connected to the tools. Save, and **publish/activate** the workflow so its production endpoint is accessible.
7. Open the MCP Server Trigger node to find the endpoint generated for **your deployment**. Use that endpoint in the client workflows. Treat it as sensitive access information and protect the service appropriately.
8. Test `get_events` / `get_contacts` (read-only operations) before permitting creates, updates, deletions or emails.

**Important:** The uploaded JSON contains original account/calendar/spreadsheet selections and old n8n endpoint references. These are **examples, not credentials you should reuse**. For public distribution, remove personal identifiers and regenerate/update service endpoints. The workflow's exported `active` state is not proof it will be reachable after import.

## 8. Exercise 6 — Single Agent + MCP Client

**File:** `auto scheduling and emailing agent.json`  
**Purpose:** Connect one AI agent directly to an MCP server.

```text
Chat Trigger ──▶ AI Agent ──▶ Response
                     ├── OpenAI Chat Model
                     ├── Simple Memory
                     └── MCP Client ──▶ MCP Server (Exercise 5)
```

**Steps:** Import the file, reselect OpenAI credentials, open the **MCP Client** node, replace the original endpoint with the server endpoint from Exercise 5, confirm tools can be discovered, and try a **read-only** chat query:

```text
Show my calendar events for tomorrow.
```

For a **write operation**, start with a harmless event in your test calendar. Check the calendar afterward rather than trusting only the model's response.

**Learning point:** The agent can use remote tools through MCP without connecting each Google integration directly to the AI Agent node. The system prompt in this example is relatively simple; for real use, add explicit confirmation and authorization rules for email sending, event deletion and changes.

## 9. Exercise 7 — Advanced Multi-Agent Scheduling Assistant

**File:** `Multi-agent scheduling assistant(2).json`  
**Purpose:** Combine specialized agent roles with MCP integrations for scheduling and messaging.

```text
Chat Trigger
     │
     ▼
Coordinator Agent ─── Conversation Memory
     │
     ├── calendar_agent ── Calendar MCP ──┐
     │                                    │
     ├── contacts_agent ── Contacts MCP ──┼── MCP Server (Exercise 5)
     │                                    │
     └── email_agent ──── Email MCP ──────┘
```

**Specialists**

- **Coordinator:** interprets the request and enforces scheduling order.
- **Calendar agent:** finds existing events, checks collisions, creates/reschedules/deletes calendar events.
- **Contacts agent:** finds names in the contact spreadsheet and adds/updates contacts if asked.
- **Email agent:** writes and sends event invitations, cancellations and reschedule notices.
- **MCP clients:** expose only selected Calendar, Contacts or Email tools to the corresponding agent.

**Intended schedule-and-invite logic**

```text
User asks to schedule an event
            │
            ▼
Calendar agent checks for clashes
            │
     ┌──────┴──────┐
   Clash         Free slot
     │               │
 Ask user      Create calendar event
                     │
              Obtain real event ID
                     │
              Look up guest email
                     │
              Send invitation email
                     │
               Confirm outcome
```

**To configure:**

1. Finish and test the MCP Server in Exercise 5 **before** importing this client.
2. Import `Multi-agent scheduling assistant(2).json`.
3. Configure credentials for the **Coordinator, Calendar, Contacts and Email** model nodes.
4. Replace the imported URLs inside **Calendar MCP**, **Contacts MCP** and **Email MCP** with your functioning MCP Server endpoint.
5. Confirm tool names exposed by the server match the agent prompts (`get_events`, `create_event`, `reschedule_event`, `delete_event`, `get_contacts`, `save_contact`, `send_email`).
6. Use a safe test calendar, mailbox and contact sheet. Verify the MCP clients discover the tools.
7. Ask for a read-only query, then create a test event with no guest, then add a known test contact and test an invitation (with authorization).
8. Inspect execution logs after each test, especially the `calendar_agent`, `contacts_agent`, and `email_agent` tool calls.

**Suggested test conversation**

```text
1. What events do I have tomorrow?
2. Add contact Test Guest with email test-guest@example.com.
3. Schedule a test meeting with Test Guest for tomorrow at 4 PM for 30 minutes.
4. Show me the meeting details and tell me whether an invitation was actually sent.
```

> `test-guest@example.com` is a placeholder; use an inbox you control for any real sending tests.

**Safety and reliability:** The imported agent prompts instruct the workflow to check for clashes, never guess email addresses, retrieve real event IDs, and seek confirmation before cancellation. These are prompt-based rules, **not technical guarantees**. In production, add deterministic validation and approval steps outside the LLM for high-impact actions, plus audit logging and error-handling. Never report an email as sent without a successful Gmail tool result.

## 10. Comparison: Which architecture should I use?

| Pattern | Structure | Best for | Main drawback |
| --- | --- | --- | --- |
| Simple agent | One agent + optional tools | Chatbot, quick demo | Limited specialization |
| Sequential | A → B → C | Tasks with ordered dependencies | Slow/failure-prone chains; context may be lost |
| Parallel | A, B, C branches → aggregate | Independent tasks | Needs correct data merging |
| Supervisor | Coordinator selects specialist tools | Dynamic delegation | More orchestration and validation needed |
| MCP-enabled agent | Agent → standardized tools | External system integrations | Requires endpoints, permissions and secure credentials |
| MCP-enabled multi-agent | Coordinator → specialists → MCP tools | End-to-end operational processes | Most complex to debug and secure |

## 11. Troubleshooting checklist

| Symptom | What to check |
| --- | --- |
| Model node shows a credential error | Create/reselect OpenAI credentials in your own workspace; verify API access and model availability |
| Chat doesn't start | Make sure the workflow has a Chat Trigger, then open the test chat and inspect executions |
| Agent isn't using a tool | Verify the tool's **AI tool** connection, tool instructions and available tool names |
| Sequential agent forgets the flight | Inspect `output` from the previous agent; explicitly preserve structured fields across steps |
| Parallel summary includes only one agent | Add an explicit merge/aggregation step before the summary agent |
| Supervisor claims it verified an unseen document | It has no document-reading tool: supply the data or integrate a document parser and validation |
| MCP client cannot find tools | Check server workflow publication, endpoint URL, authentication/access rules and tool selection |
| Contacts agent says `NOT_FOUND` or `MISSING_EMAIL` | Confirm spreadsheet ID, `NAME` / `EMAILS` headers, rows, OAuth permission and exact recipient information |
| Email says sent but Gmail has no message | Inspect the `send_email` tool execution; require the actual Gmail success result |
| Invitation reports `NO_EVENT_ID` | Check that Calendar actually created an event and returned a valid event ID before email delegation |
| Calendar times are wrong | Check timezone; these advanced prompts use **Asia/Karachi (UTC+05:00)** |
| Delete or reschedule fails | Run `get_events` to retrieve the correct event ID; verify permissions and approval |

## 12. Suggested student practical assignments

1. **Simple Agent:** Modify its system prompt to become a programming tutor; test memory and calculator.
2. **Sequential:** Add a fourth agent for a travel budget summary. Preserve data between stages.
3. **Parallel:** Fix result aggregation; demonstrate that the summary receives all three branch outputs.
4. **Supervisor:** Add an eligibility threshold and explain which agent is responsible for each check.
5. **MCP Server:** Configure a disposable calendar and fake contacts spreadsheet; show a read-only tool call.
6. **MCP Client:** Query tomorrow's appointments through the MCP server.
7. **Scheduling Assistant:** Test clash handling, missing email resolution, event-ID validation and cancellation approval.

## 13. Sharing and production-readiness notes

- The files are **workflow exports**, not a complete hosted service. Each student needs their own working n8n environment, model access and suitable Google accounts to perform real integrations.
- These exports contain account references, workflow identifiers, spreadsheet selections, and MCP endpoint addresses. **Review and sanitize** before publishing publicly; never commit private API keys, tokens or OAuth secrets.
- For demo travel workflows, flight/hotel recommendations are illustrative **AI outputs**, not verified real-time booking information.
- The parallel example should be updated to merge results explicitly.
- The supervisor example does not actually read uploaded documents.
- Use read-only tests first and human approval for potentially consequential actions.
- The setup here documents the **uploaded workflow designs**; it does not claim they were successfully executed in your environment.

## Official learning resources

- [n8n website](https://n8n.io/)
- [n8n documentation](https://docs.n8n.io/)
- [n8n AI guides](https://docs.n8n.io/advanced-ai/)
- [n8n workflow concepts](https://docs.n8n.io/workflows/)
- [n8n integrations documentation](https://docs.n8n.io/integrations/)

---

**Recommended sequence:** First import and run the Simple Agent. Then open sequential → parallel → supervisor orchestration. Finally set up the MCP Server **before** running either MCP client workflow. Inspect every execution, not just the final chat response.


---
## Additional introductory resource: `n8n Nodes.json`

This eighth export is a **node demonstration canvas**, not one fully linked executable workflow. It contains 32 nodes demonstrating trigger types (manual, schedule, Typeform), storage (Sheets, n8n Data Table, Notion, Airtable), processing (Edit Fields, Split Out, Aggregate, If, Switch, Code, Merge, Wait), APIs (HTTP Request, Webhook and Respond), and AI tools (Chat Trigger, AI Agent, Anthropic model, OpenAI model, Tavily MCP, PostgreSQL chat memory).

**Setup:** Import `Workflow/n8n Nodes.json`, read its sticky-note sections, inspect connections, then configure one sample branch at a time and execute it using non-sensitive dummy data. Nodes shown without connections are references rather than one end-to-end runnable process. Credentials and document/database selections must be recreated in each learner's account.

**Safety:** The original upload included an embedded third-party weather API key and personal demonstration contacts. The `Workflow/n8n Nodes.json` inside this submission package is a **sanitized copy** with the key replaced by a placeholder and credentials/resource identifiers cleared. Before running the weather request, configure your own valid API credential; do not publish or reuse the key from the original file.

**Practice checks:** Inspect the Edit Fields → Split Out example for one item per lead; experiment with If/Switch conditions; test a Chat Trigger → AI Agent with an attached model and compare the node-level results.
