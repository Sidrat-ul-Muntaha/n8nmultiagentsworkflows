# Sample Inputs and Expected Outputs — n8n Practical Workflows

**Audience:** Beginner-to-advanced n8n workshop students  
**Companion:** `README.md` and the seven exported `.json` workflows  
**Recommended order:** Simple Agent → Sequential → Parallel → Supervisor → MCP Server → Simple MCP Client → Advanced Scheduling Assistant

> **Important:** The outputs shown below are **illustrative expectations, not recorded successful runs**. Language-model wording can differ. Dates, prices, venues, event IDs, and tool results must come from actual connected services when the workflow makes real-world claims. Use test calendars, test contacts, and addresses you control. **Never send a real invitation or delete a real event just to complete an assignment.**

## Before testing

1. Import each JSON file separately into n8n (**Workflows → Add workflow → Import from file**; menu names may vary).
2. Configure and select valid OpenAI credentials in every OpenAI Chat Model node. Several exported workflows have credentials that will not transfer across n8n accounts.
3. For Chat Trigger workflows, open the chat/test interface and send the input shown below.
4. For the MCP workflows, configure and activate the MCP server first, connect your own Google Calendar / Gmail / Google Sheets credentials, then update the MCP Client endpoint URL if your n8n host or webhook path differs.
5. Watch the **Executions** view and inspect individual node inputs/outputs; do not rely solely on the final chat text.
6. Use clearly fictional example names and test email addresses under your control. Replace example dates with a future date when testing calendar operations.

---

## 1. Simple AI Agent

**File:** `Simple_Agent.json`  
**Architecture:** Chat Trigger → AI Agent, with OpenAI Chat Model + Simple Memory + Calculator tool.

### Test 1 — General question

**Input:**
```text
What is artificial intelligence? Explain in three simple sentences.
```

**Expected output (example):**
```text
Artificial intelligence (AI) allows computers to perform tasks that usually need human intelligence.
It can learn from data, recognize patterns, and answer questions.
Examples include voice assistants, translation apps, and recommendation systems.
```

**Check:** A short, understandable explanation appears in chat.

### Test 2 — Calculator tool

**Input:**
```text
Calculate (75 + 85 + 90 + 80) / 4. Show the result.
```

**Expected output:**
```text
Average = 82.5
```

**Check:** The answer is **82.5**. Inspect the AI Agent execution to confirm that the **Calculator** tool was called; a numerically correct response by itself does not prove tool usage.

### Test 3 — Conversation memory (send as two messages in the same chat session)

**Input A:**
```text
My favorite programming language is Python.
```

**Input B:**
```text
What programming language did I say is my favorite?
```

**Expected output to B:**
```text
You said your favorite programming language is Python.
```

**Check:** Chat session continuity works. Try a fresh session to see whether the memory is session-specific.

---

## 2. Sequential Multi-Agent Travel Assistant

**File:** `Sequential_Multiagent.json`  
**Architecture:** Chat Trigger → Flight Recommendation Agent → Hotel Recommendation Agent → Travel Summary Agent. Each agent has its own OpenAI model connection.

### Test 1 — Basic trip planning

**Input:**
```text
Plan a three-day trip from Karachi to Lahore for one person with a modest budget.
Suggest a flight and a hotel. This is a classroom demo; do not book anything.
```

**Expected output (illustrative):**
```text
Travel Plan: Karachi → Lahore (3 days)
Flight: Look for an economy flight on the requested travel dates; exact fare not verified.
Hotel: Look for an affordable hotel in a convenient Lahore location; availability not verified.
Summary: Compare transport and accommodation costs before booking.
```

**Check:** The execution visits all **three agents in sequence**. The Flight Agent's text is passed to the Hotel Agent, and the Hotel Agent's output reaches the Summary Agent.

### Test 2 — Budget constraints

**Input:**
```text
I want a two-night trip to Islamabad. My accommodation budget is PKR 15,000 in total.
Suggest a suitable plan, and explain if exact prices cannot be checked.
```

**Expected output (illustrative):**
```text
Suggested destination: Islamabad
Accommodation target: PKR 7,500 per night or less for two nights.
Flight suggestion: Economy option; exact fare requires a live booking source.
Final note: Hotel rate and availability must be verified before purchase.
```

**Check:** The workflow preserves the budget constraint across the chain. **Known limitation:** the later agents receive the previous agent's `output` text; they may not retain the original user request unless preceding agents include its relevant constraints.

---

## 3. Parallel Multi-Agent Travel Assistant

**File:** `Multiagent_Parallel.json`  
**Architecture:** Chat Trigger branches into Flight Recommendation Agent, Tourist Place Recommendation Agent, and Hotel Recommendation Agent; all three are connected onward to the Travel Summary Agent.

### Test 1 — Independent recommendations

**Input:**
```text
Suggest a three-day trip to Dubai. Include a flight idea, a budget hotel option,
and three places to visit. This is for testing only.
```

**Expected outputs from separate branches (illustrative):**
```text
Flight Agent: Suggest an economy flight to Dubai; dates and price not verified.
Hotel Agent: Suggest a budget-friendly area or hotel category; availability not verified.
Tourist Place Agent: Burj Khalifa, Dubai Museum/Al Fahidi area, and Dubai Marina.
```

**Desired final combined output (after fixing aggregation):**
```text
Dubai Trip Plan
1. Flight: Economy flight suggestion (unverified)
2. Hotel: Budget-friendly accommodation suggestion (unverified)
3. Attractions: Burj Khalifa, Al Fahidi area, Dubai Marina
```

**Check:** All three independent AI Agents execute. **Important:** The exported workflow connects each branch directly to the Travel Summary Agent without an explicit consolidation/aggregation step. n8n may run the summary separately for incoming items rather than receiving all three recommendations as one complete input. A single complete summary is the **target behavior after adding a Merge/aggregation step**, not a guaranteed result from the unmodified file.

### Test 2 — Different destination

**Input:**
```text
Plan a short trip to Murree. Suggest travel, accommodation and attractions separately.
```

**Expected checks:** Distinct outputs appear from flight/travel, hotel and attraction branches. The Flight Agent might suggest a flight even where road travel is more practical because it is specifically instructed to recommend flights; discuss this limitation in class.

---

## 4. Supervisor Multi-Agent Admission Assistant

**File:** `Multiagent Supervisor.json`  
**Architecture:** Chat Trigger → Supervisor AI Agent → Marks Verification Agent / Document Verification Agent / Admission Summary Agent (as AI Agent Tools).

### Test 1 — Marks eligibility demonstration

**Input:**
```text
Admission test: Applicant Sara obtained 720 out of 900 marks in the previous examination.
For this classroom example, eligibility requires at least 60%.
She reports having submitted her mark sheet and identity document.
Please review the information and prepare a short admission summary.
```

**Expected output (illustrative):**
```text
Applicant: Sara
Marks: 720/900
Percentage: 80%
Eligibility under the stated 60% rule: Meets marks requirement
Documents: Reported as submitted; not independently verified
Overall: Preliminary eligible based on stated marks, subject to document verification.
```

**Check:** The Supervisor delegates to the specialized tools and produces a structured summary. The **60% threshold is supplied in the input**; it is not an established university policy in the workflow.

### Test 2 — Incomplete documents

**Input:**
```text
Applicant Ahmed obtained 450 out of 900 marks.
For this classroom example, the minimum percentage is 60%.
He says his mark sheet is available but his identity document is missing.
Generate an admission status summary without assuming the missing document is present.
```

**Expected output (illustrative):**
```text
Applicant: Ahmed
Percentage: 50%
Eligibility under the stated 60% rule: Does not meet marks requirement
Identity document: Reported missing
Status: Needs review / does not meet example marks criterion.
```

**Check and limitation:** The exported Document Verification Agent has **no actual PDF/file inspection tool**; it cannot verify uploaded documents simply because its system prompt says to. Treat document status as **student-reported** until connected to a real document source. One instruction mentions an Aadhaar card; adapt the document type to the actual institution's requirements.

---

## 5. MCP Server — Calendar, Contacts and Email Tools

**File:** `MCP Server(2).json`  
**Architecture:** MCP Server Trigger exposes these seven tool names: `get_events`, `create_event`, `reschedule_event`, `delete_event`, `get_contacts`, `save_contact`, `send_email`.

**This is a tool server, not a standalone chat agent.** Run tests through a compatible MCP Client/AI Agent (for example, workflow 6 or 7), or another MCP client that can invoke its tools. The results below describe **tool-level** behavior.

### Test 1 — Read calendar events

**Tool:** `get_events`  
**Illustrative input:**
```json
{
  "time_min": "2026-12-10T09:00:00+05:00",
  "time_max": "2026-12-10T17:00:00+05:00"
}
```

**Expected result:** A real list of events from the connected calendar in that interval, each with event details (such as ID, title, start and end). If none exist, expect an empty result. **Do not fabricate event IDs.**

### Test 2 — Create an event

**Tool:** `create_event`  
**Illustrative input:**
```json
{
  "Start": "2026-12-10T14:00:00+05:00",
  "End": "2026-12-10T15:00:00+05:00",
  "Summary": "n8n Demo Workshop",
  "Description": "Test event for n8n class",
  "location": "Test classroom",
  "Use_Default_Reminders": true
}
```

**Expected result:** Google Calendar confirms creation and returns its **real event ID**, start and end. This server tool's description instructs the calling agent to check `get_events` for clashes first; the `create_event` tool itself is not a standalone clash-prevention mechanism.

### Test 3 — Read or save a contact

**Tool:** `get_contacts` or `save_contact`  
**Illustrative input for `save_contact`:**
```json
{
  "name": "Demo Student",
  "email": "demo.student@example.com"
}
```

**Expected result:** A contact row is appended or updated in the connected Google Sheet (depending on whether the name already exists). The sheet's configured columns are `NAME` and `EMAILS`. The example address is a placeholder; use an address you control if testing email delivery.

### Test 4 — Send a test email

**Tool:** `send_email`  
**Illustrative input:**
```json
{
  "to_email": "YOUR_OWN_TEST_ADDRESS",
  "subject": "n8n MCP Test",
  "message": "<p>This is a test message from our n8n practical.</p>"
}
```

**Expected result:** Gmail returns a successful send result/message identifier, and the message arrives in your controlled test inbox. Never assume delivery solely because an AI Agent says it sent an email.

### Test 5 — Reschedule or delete

**Tool:** `reschedule_event` / `delete_event`  
**Expected result:** The operation succeeds **only with the real event ID** retrieved from `get_events`. In the classroom, use a newly created disposable event and confirm deletion before performing it. For deletion, the tool expects `Event_ID`; for rescheduling, it expects `event_id`, `new_start`, `new_end`.

**Setup caution:** The exported server has source-account calendar and spreadsheet selections. Replace these with your own sandbox/test resources and reconnect OAuth credentials before enabling writes.

---

## 6. Simple MCP Client — Automatic Scheduling and Emailing Agent

**File:** `auto scheduling and emailing agent.json`  
**Architecture:** Chat Trigger → AI Agent, with OpenAI Chat Model + Simple Memory + MCP Client. The MCP Client connects to the MCP server above.

### Test 1 — Read events through MCP

**Input:**
```text
Using the calendar MCP tool, list my events on December 10, 2026.
```

**Expected output:** A factual list of events from the connected calendar, or a message such as:
```text
No events found on December 10, 2026.
```

**Check:** The execution shows the **MCP Client** calling `get_events`. The empty-events response is only correct if the live calendar actually has no events.

### Test 2 — Create a demo event

**Input:**
```text
Check my calendar and create a test event named "n8n Practice" on December 11, 2026,
from 3:00 PM to 4:00 PM Pakistan time, if the slot is available.
```

**Expected output (only after success):**
```text
Created: n8n Practice
Date: December 11, 2026
Time: 3:00–4:00 PM PKT
Event ID: [actual ID returned by Google Calendar]
```

**Check:** The event appears in your calendar. If the slot is occupied, the assistant should report the clash rather than claim creation. The simple agent's system prompt is minimal, so strict clash checking or confirmation behavior is **not guaranteed** by this export; evaluate the tool trace and improve its instructions if needed.

### Test 3 — Contact lookup

**Input:**
```text
Look up the email address of Demo Student from the contact sheet.
```

**Expected output:** Return the actual matching address if present; otherwise report that it was not found. **Never invent an email address.**

---

## 7. Advanced Multi-Agent Scheduling Assistant

**File:** `Multi-agent scheduling assistant(2).json`  
**Architecture:** Chat Trigger → Coordinator Agent; its specialist tools are `calendar_agent`, `contacts_agent`, and `email_agent`, each with its own model and restricted MCP tool selection.

**Special behavior in the exported prompt:**
- Coordinator first delegates calendar clash checking and event creation.
- If a conflict is reported, stop and **do not send the invitation**.
- Once an event is actually created and a real event ID returned, look up the contact and ask the Email Agent to send the invitation.
- For cancellation, find the event first and ask for confirmation before deleting.
- For rescheduling, update the calendar and then notify the guest.

### Test 1 — Schedule and invite (successful path)

**Preparation:** Put a test contact named `Demo Student` with an email address you control into the configured Google Sheet. Make sure the selected calendar slot is free.

**Input:**
```text
Schedule "n8n Group Discussion" with Demo Student on December 15, 2026,
from 2:00 PM to 3:00 PM at the university computer lab.
Send an invitation email to Demo Student.
```

**Expected output (only if tools confirm success):**
```text
Event created: n8n Group Discussion
Date/time: December 15, 2026, 2:00–3:00 PM PKT
Venue: University computer lab
Calendar event ID: [real event ID]
Invitation sent to: [email from contact sheet]
```

**Expected tool sequence:** `calendar_agent` → `get_events` → `create_event` → `contacts_agent` → `get_contacts` → `email_agent` → `send_email`.

**Check:** Verify the event in Calendar **and** the email in the sent-mail record or controlled inbox.

### Test 2 — Clash detection

**Preparation:** Create a separate calendar event that occupies December 16, 2026, 11:00 AM–12:00 PM.

**Input:**
```text
Schedule a meeting with Demo Student on December 16, 2026,
11:30 AM to 12:30 PM. Send an invitation.
```

**Expected output (illustrative):**
```text
The requested time conflicts with an existing event.
I have not created the new meeting or sent an invitation.
Would you like another time, or do you want to book anyway?
```

**Check:** No `send_email` call, no new event creation, and the existing calendar appointment is unchanged.

### Test 3 — Contact not found

**Input:**
```text
Schedule "Project Review" with Unknown Guest on December 17, 2026,
10:00–11:00 AM at the computer lab, and send an invitation.
```

**Expected output:** If the slot is free, the calendar may be created first, but the assistant should say the guest email was **not found** and request the correct address. **It must not claim the email was sent.**

**Check:** A contact lookup occurs, `send_email` does not run, and the response clearly distinguishes **event created** from **invitation not sent**.

### Test 4 — Save contact

**Input:**
```text
Add a contact named Demo Student with email YOUR_OWN_TEST_ADDRESS.
```

**Expected output (only after Sheet write):**
```text
Contact saved: Demo Student — [the email address you provided]
```

**Check:** `contacts_agent` calls `save_contact` and the Google Sheet actually contains the contact.

### Test 5 — Cancellation requiring confirmation (two messages)

**Input A:**
```text
Cancel my n8n Group Discussion meeting on December 15, 2026.
```

**Expected response A:** The assistant finds the matching event and asks for confirmation **without deleting it**.

**Input B:**
```text
Yes, cancel that meeting.
```

**Expected response B (only after success):**
```text
The selected meeting has been cancelled.
```

**Check:** The event is deleted after confirmation, not before; if a guest is associated and found, a cancellation notice should also be attempted and its actual send status reported. This test assumes the event from Test 1 still exists.

### Test 6 — Reschedule

**Input:**
```text
Move my Project Review event from December 17, 2026, 10:00 AM
 to December 18, 2026, 10:00–11:00 AM, if the new time is free.
Notify the guest if their email is available.
```

**Expected output:** The assistant identifies the real event ID, checks the new time, updates the existing event, and sends a reschedule notification only when a recipient email is known. If the new slot is busy, it should stop and report the conflict.

**Check:** The existing event's time changes, rather than a duplicate event being created; the result is confirmed using the Calendar tool.

---

## Classroom validation checklist

For **every** workflow, students should demonstrate:

- [ ] The intended trigger starts the workflow.
- [ ] The correct AI agent(s) execute.
- [ ] Prompts preserve the user's essential constraints.
- [ ] The required tool(s) are called and show successful responses (where relevant).
- [ ] The final answer matches **actual** tool results, not invented claims.
- [ ] Errors and missing information are reported clearly.
- [ ] Screenshots show both the node execution trace and final output.
- [ ] The exported JSON has **no API keys, passwords, personal contact lists, or secret tokens**.

## Suggested student submission

```text
Student_Name/
  01_Simple_Agent.json
  02_Sequential_Multiagent.json
  03_Multiagent_Parallel.json
  04_Multiagent_Supervisor.json
  05_MCP_Server.json
  06_Auto_Scheduling_and_Emailing_Agent.json
  07_Multiagent_Scheduling_Assistant.json
  screenshots/
  test-results.md
```

**Instructor note:** For an imported workflow that has not yet been fixed/configured, the expected output is an **acceptance target**, not proof that the workflow currently works. In particular, add proper fan-in/aggregation for the Parallel workflow, add real document inspection for genuine admission verification, and connect actual Google/MCP credentials for the scheduling workflows.
