# AI & IT Operations Assistant (Proof of Concept)

A grounded Microsoft Copilot Studio assistant combined with deterministic Power Automate workflows, built to handle internal IT/process self-service: answering repetitive policy questions and turning free-text access requests into structured, auditable approval workflows.

**Stack:** Copilot Studio · Power Automate · SharePoint · Dataverse · Microsoft 365 / Power Platform

**Status:** Proof of concept, not a production deployment. Core components built and tested in an isolated Microsoft 365 tenant with fictional policies and test data. One known integration limitation is documented below, with a diagnosis and fix plan.

Full narrative write-up (problem, design reasoning, test results): **[case study on my portfolio site](../../#ai-it-operations-assistant)** <!-- update once the site page is live -->

---

## Problem

Employees ask repetitive IT questions and submit access requests through unstructured messages (chat, email). This creates:

- Incomplete or vague requests
- Manual follow-up by IT
- Limited status transparency for the requester
- Inconsistent, hard-to-audit approval records
- Risk of over-broad permission requests
- Risk of an ungrounded AI assistant inventing internal policy details

## Design approach: AI where it helps, deterministic automation where it must be reliable

The core decision this project makes explicit — and the one I'd bring to a company — is that **not every step belongs to the AI**. Natural language is AI's job; anything that has to be predictable, auditable, and repeatable is a deterministic workflow's job.

| Area | Technology | Reason |
|---|---|---|
| Natural-language questions | Copilot Studio | Users express the same need in many different ways |
| Knowledge grounding | AI + controlled documents | Use approved sources only, avoid inventing internal policy |
| Request capture | AI extracts structured fields | Converts conversation into system, justification, priority, approver, description |
| Approval and status logic | Deterministic Power Automate | Business actions must be predictable, auditable, repeatable |
| Access decision | Human in the loop | Sensitive permissions are never granted autonomously by AI |

## Implemented components

- **Microsoft 365 / Power Platform setup** — tenant, licensing, Power Platform environment, Dataverse, security roles
- **SharePoint tracking** — an IT Requests list as the structured system of record
- **Approval workflow** — `New request → Pending Approval → approval → Approved/Rejected → update → email`
- **Copilot Studio assistant** — grounded on three fictional internal policy documents
- **Safety behaviour** — least privilege, no credential handling, escalation on unknown information, no autonomous access grants
- **Request-creation tool** — an agent tool that extracts six structured fields from conversation and is designed to create the SharePoint request

## Test plan and results

| Test case | Expected | Actual | Status |
|---|---|---|---|
| Valid access request | Approval completes | Approved + email sent | Passed |
| Excessive Global Admin request | Rejected on least-privilege grounds | Rejected + comments | Passed |
| Exact policy retrieval | Returns 2-business-day target | Refined and retested | Passed after iteration |
| Unknown internal question | Escalate, do not invent an answer | Escalated | Passed |
| Sensitive data / credentials request | Refuse unsafe handling | Refused / warned | Passed |
| Tool input extraction | Populate all six request fields | All six extracted | Passed |
| Agent → workflow invocation | Create SharePoint item | HTTP 403 before run | **Known limitation** |

## Known limitation and diagnosis

The Copilot agent correctly identifies the request-creation intent and populates all six workflow inputs. The tool call itself returns HTTP 403, and monitoring shows zero workflow runs — meaning the request is blocked at the agent-to-workflow authorization boundary, before SharePoint or workflow logic ever executes.

**Planned troubleshooting:**
- Validate runtime entitlement and licensing in a fully licensed tenant
- Validate environment ownership and runtime permissions
- Test the bridge using the classic Agent Flow experience
- Use a dedicated least-privilege service identity where appropriate

I'm documenting this rather than hiding it deliberately — diagnosing *why* an integration fails, not just that it does, is most of the job.

## Security and governance considerations

- **Least privilege** — broad administrator access is challenged by default; minimum permissions preferred
- **Human approval** — the AI never grants access autonomously
- **Prompt safety** — passwords, MFA codes, API keys, tokens, and private keys are never requested or stored
- **Data minimization** — only information required for the process is collected
- **Grounding and escalation** — unsupported internal questions are escalated, never answered from general knowledge
- **Production separation** — a real deployment would use controlled Dev/Test/Prod environments, DLP policies, monitoring, and formal ownership

## Production roadmap

1. Resolve the agent-to-workflow authorization issue in a fully licensed environment
2. Move demo knowledge files to permission-aware SharePoint sources
3. Add employee self-service status lookups
4. Add resilience: error handling, retries, owner notifications, logging, monitoring
5. Pilot with a small group; validate usefulness, answer quality, failure modes, support needs
6. Measure value: self-service rate, approval time, flow errors, escalation rate, user satisfaction
7. Productionize: permission model, DLP, GDPR review, support ownership, documentation, rollback plan

## Documentation

Full project documentation (PDF): [`docs/AI_IT_Operations_PoC_Project_Documentation.pdf`](docs/AI_IT_Operations_PoC_Project_Documentation.pdf)

---

*This is a proof of concept built in an isolated Microsoft 365 tenant with fictional policies and test data — not a production deployment and not affiliated with any employer.*
