# AI & IT Operations Assistant

A grounded Microsoft Copilot Studio assistant paired with deterministic Power Automate workflows for internal IT self-service: answering repetitive policy questions and turning free-text access requests into structured, auditable approval workflows.

**Stack:** Copilot Studio · Power Automate · SharePoint · Dataverse · Microsoft 365 / Power Platform

**Status:** Core flow built and tested end-to-end in an isolated Microsoft 365 tenant with fictional policies and test data. One integration limitation is documented below, with a diagnosis and a fix plan.

Full write-up with screenshots: **[case study on my portfolio](https://kirillsadchikov.github.io/projects/ai-it-operations-assistant.html)**

---

## The problem

Most IT access requests follow the same shape: someone needs access to a system, for a reason, approved by someone. In practice they show up as Teams messages, half-written emails, or a hallway ask — incomplete information, no real audit trail, and IT chasing people for details that should've been there from the start. People also sometimes ask for more access than they need, just in case.

I wanted to see what that process looks like once it's actually built: a chat interface that asks the right questions, a workflow that routes it for approval, and a record of what happened and why.

## Design approach

The split that matters here: Copilot Studio handles the parts where language varies — understanding the request, answering policy questions, deciding what's missing. Power Automate handles everything that needs to be the same every time — who approves what, what happens on approval or rejection, what gets logged. AI is good at the first kind of problem and bad at the second. It shouldn't be making decisions that need to be predictable and auditable, and it should never be the one deciding who gets access to what.

| Where | Handled by |
|---|---|
| Understanding the request, answering policy questions | Copilot Studio |
| Approval routing, status updates, notifications | Power Automate (deterministic) |
| Whether access is actually granted | A human approver, always |

## What's built

- **SharePoint tracking** — an IT Requests list as the structured system of record (requester, system, justification, priority, approver, status)
- **Approval workflow** — new request → pending approval → approver notified by email → approved/rejected → status updated → requester notified
- **Copilot Studio assistant** — grounded on three policy documents (IT access policy, incident escalation guide, internal AI usage policy), with one tool wired up for creating a request
- **Guardrails** — pushes back on vague or over-privileged requests instead of forwarding them, never asks for passwords or MFA codes, escalates instead of guessing on unknown policy questions

## Test results

| Test case | Expected | Result |
|---|---|---|
| Valid access request | Approved, requester notified | Passed |
| Vague/over-privileged request (e.g. Global Admin "just in case") | Challenged, not forwarded | Passed |
| Exact policy retrieval | Correct figure, cited | Passed |
| Unknown policy question | Escalated, not guessed | Passed |
| Credential request | Refused | Passed |
| Field extraction from conversation | All six fields captured | Passed |
| Agent → workflow handoff | Request created in SharePoint | **Known limitation, see below** |

## Known limitation

The agent correctly identifies the request-creation intent and extracts all six fields cleanly from the conversation. Where it breaks: the call from the Copilot agent into the Power Automate flow comes back with an HTTP 403, and the flow's run history shows zero executions — it's getting blocked at the authorization boundary between the agent and the workflow, before anything in SharePoint happens. The same flow runs fine when triggered directly from SharePoint, which points to a licensing or runtime-permission gap between the agent and the Power Platform environment in this tenant rather than a problem with the flow itself.

Next step would be checking runtime entitlements in a fully licensed tenant, and trying the classic Agent Flow connection as a fallback if that doesn't resolve it.

## If I kept building this

Fix the agent-to-workflow permission issue, then move the knowledge base off uploaded files and onto permission-aware SharePoint sources. Add a status lookup so people can check their own request instead of asking. Add retries, owner notifications, and logging so failures don't go unnoticed. Then pilot it with a small group and measure the thing — self-service rate, approval time, how often it escalates instead of answering.

## Documentation

Full project documentation (PDF): [`docs/AI_IT_Operations_PoC_Project_Documentation.pdf`](docs/AI_IT_Operations_PoC_Project_Documentation.pdf)
