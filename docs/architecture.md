# Pre-Study: View21 ↔ Dynamics 365 Integration

**Author:** Luay Sakr
**Date:** 2026-02-11
**Status:** Draft – Pre-Study / Architectural Assessment
**Version:** 0.1

> **Disclaimer:** This document was refined using AI assistance for structure, formatting, and phrasing.
---

## 1. Purpose

This document is a **pre-study** to evaluate how View21 can be integrated with Microsoft Dynamics 365 (Dataverse). It is not a final design — it is a structured assessment meant to:

- Frame the problem clearly
- State assumptions transparently
- Compare realistic integration approaches
- Recommend an initial direction
- Define a concrete next step (Proof of Concept)


---

## 2. Problem Statement

View21 and Dynamics 365 are two systems that need to exchange data. The exact scope of this exchange is still being defined, but the core question is:

> **How should we connect View21 to Dynamics 365 so that data flows reliably between them, without over-engineering the solution or creating security risks?**

### What we know

- Dynamics 365 (Dataverse) has a well-documented **Web API** (OData v4)
- Authentication is handled via **Azure AD** (OAuth2)
- Both read and write operations are needed
- The solution must work in a corporate Azure environment

### What we don't yet know

- The exact entities and fields to synchronise
- Whether data flow is one-way, two-way, or event-driven
- The expected data volume and frequency
- Whether View21 has existing integration capabilities (webhooks, APIs, message queues)

---

## 3. Assumptions

These are clearly stated so they can be validated or corrected:

| # | Assumption | Impact if Wrong |
|---|-----------|-----------------|
| A1 | View21 is a web-based application that can make or receive HTTP calls | Would need an adapter or middleware |
| A2 | The integration will involve **CRM entities** (Accounts, Contacts, Opportunities) | Different entities may have different API patterns |
| A3 | Data volume is **moderate** (hundreds to low thousands of records per day, not millions) | High volume would require batch/async patterns |
| A4 | The solution will run on **Azure** (since D365 is already there) | On-prem hosting adds network/auth complexity |
| A5 | **Near real-time** sync is acceptable (seconds to minutes, not milliseconds) | Sub-second would require different architecture |
| A6 | There is **no existing middleware** (e.g., BizTalk, MuleSoft, Logic Apps) already in use | Existing middleware should be evaluated first |
| A7 | Budget is limited — we prefer **code-based solutions** over expensive iPaaS licenses | iPaaS (e.g., MuleSoft) could simplify but costs more |
| A8 | The team has **.NET** competency | Different stack would change implementation details |

---

## 4. Design Alternatives

### Overview

| Approach | Complexity | Cost | Flexibility | Maintenance |
|----------|-----------|------|-------------|-------------|
| **A. Direct API Integration** | Low–Medium | Low | High | Medium |
| **B. Azure Logic Apps / Power Automate** | Low | Medium–High | Medium | Low |
| **C. Azure Service Bus + Worker** | Medium | Low–Medium | Very High | Medium |
| **D. Third-party iPaaS** (MuleSoft, etc.) | Low | High | High | Low |

---

### A. Direct API Integration (Custom Middleware API)

```
┌──────────┐       HTTPS/REST         ┌──────────────┐        OAuth2 + OData       ┌──────────┐
│  View21  │ ◄──────────────────────► │  Integration │ ◄─────────────────────────► │   D365   │
│          │                          │   API (.NET) │                             │ Dataverse│
└──────────┘                          └──────────────┘                             └──────────┘
                                            │
                                      Azure AD Token
```

**How it works:** A lightweight .NET API sits between View21 and D365. It handles authentication, data mapping, error handling, and retry logic. Both systems talk to this API.

**Pros:**
- Full control over logic, mapping, and error handling
- Low cost (runs as an App Service or container)
- Easy to test, debug, and extend
- No vendor lock-in beyond Azure AD

**Cons:**
- Requires development effort
- Team must maintain the code
- No built-in visual monitoring (must be added)

---

### B. Azure Logic Apps / Power Automate

**How it works:** Low-code/visual workflows that connect D365 to View21 using pre-built connectors.

**Pros:**
- Fast to set up for simple scenarios
- Built-in D365 connectors
- No code to maintain

**Cons:**
- Gets expensive at scale (per-action pricing)
- Limited flexibility for complex mapping or error handling
- Difficult to version-control and test
- Vendor lock-in to Microsoft's low-code platform

---

### C. Azure Service Bus + Worker Services

**How it works:** Events are published to Azure Service Bus queues/topics. Worker services pick up messages and process them asynchronously.

**Pros:**
- Best for decoupled, event-driven architecture
- Handles spikes and failures gracefully
- Scales independently

**Cons:**
- More moving parts to set up and monitor
- Overkill if data volume is low
- Adds operational complexity

---

### D. Third-party iPaaS (MuleSoft, Dell Boomi, etc.)

**How it works:** A commercial integration platform handles connectors, mapping, and monitoring.

**Pros:**
- Pre-built connectors for many systems
- Visual monitoring and alerting
- Reduced development effort

**Cons:**
- **High licensing cost** (often per-connection or per-transaction)
- Vendor lock-in
- May not justify cost for a single integration

---
