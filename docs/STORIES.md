# STORIES.md – scale‑mentor

## Overview
This document captures the product backlog for **scale‑mentor**, organized into Epics that map to the core value pillars described in the README (Playbooks, KPI Dashboard, Automation, Integrations, and On‑Demand Guidance). Stories are ordered to deliver a Minimum Viable Product (MVP) first, then incremental enhancements that increase configurability, security, and scalability.

---

## Epics & Story Map

| Epic | Goal | MVP Stories |
|------|------|--------------|
| **E1 – Playbook Engine** | Deliver data‑driven, step‑by‑step scaling playbooks that can be browsed, customized, and marked as complete. | 1, 2, 3 |
| **E2 – KPI Dashboard** | Provide a real‑time, role‑based dashboard showing the most‑important financial, operations, and hiring metrics. | 4, 5 |
| **E3 – Automation Workflows** | Enable users to automate repetitive scaling tasks (e.g., invoicing, hiring pipeline triggers). | 6, 7 |
| **E4 – Third‑Party Integrations** | Connect to core SaaS tools (Stripe, Xero, HubSpot) for data ingestion and action execution. | 8, 9 |
| **E5 – Expert Guidance & Support** | Offer on‑demand access to scaling experts via chat and scheduled sessions. | 10, 11 |
| **E6 – Platform Foundations** | Harden security, testing, and deployment pipelines to support production use. | 12, 13, 14, 15 |

---

## Detailed User Stories

### **E1 – Playbook Engine**

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **1** | **As a** Business Owner, **I want** to browse a library of pre‑built scaling playbooks, **so that** I can quickly find a roadmap that matches my growth stage. | - Playbooks are listed with title, stage (e.g., “Early Revenue”, “Series A”), and brief summary.<br>- List can be filtered by stage and industry tag.<br>- Clicking a playbook opens a detailed view with all steps. |
| **2** | **As a** Business Owner, **I want** to clone a playbook into my own workspace, **so that** I can edit steps without affecting the master template. | - “Clone” button creates a private copy linked to the user’s account.<br>- Cloned playbook appears in “My Playbooks” section.<br>- Original template remains unchanged. |
| **3** | **As a** Business Owner, **I want** to mark individual steps as **Completed** and add free‑form notes, **so that** I can track progress and capture learnings. | - Each step has a checkbox and a “Add note” field.<br>- Completion status persists across sessions.<br>- A progress bar reflects % of steps completed. |

### **E2 – KPI Dashboard**

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **4** | **As a** Finance Manager, **I want** a dashboard showing Monthly Recurring Revenue (MRR), churn, and burn rate, **so that** I can monitor financial health at a glance. | - Dashboard tiles for MRR, churn %, and burn rate.<br>- Data refreshes automatically every 5 min.<br>- Values are pulled from integrated Stripe/Xero data sources (mocked for MVP). |
| **5** | **As an** Operations Lead, **I want** to view hiring velocity and fulfillment lead‑time metrics, **so that** I can spot bottlenecks in scaling staff. | - Two charts: “Candidates per week” and “Time‑to‑hire”.<br>- Hover shows raw numbers and trend arrows.<br>- Filters for department and seniority level. |

### **E3 – Automation Workflows**

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **6** | **As a** Business Owner, **I want** to create a “New Customer Onboard” workflow that automatically creates a HubSpot contact and sends a welcome email, **so that** onboarding is consistent and hands‑free. | - Workflow builder UI with “Trigger → Action” blocks.<br>- Pre‑built trigger: “Stripe payment succeeded”.<br>- Action: “Create HubSpot contact” + “Send email via SendGrid”.<br>- Test run shows contact created and email sent (mocked). |
| **7** | **As a** Finance Manager, **I want** a scheduled job that reconciles Stripe payouts with Xero invoices every night, **so that** our books stay up‑to‑date without manual effort. | - Scheduler UI to set “Daily at 02:00 UTC”.<br>- Job logs success/failure with error details.<br>- Reconciliation report downloadable as CSV. |

### **E4 – Third‑Party Integrations**

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **8** | **As a** Developer, **I want** a secure OAuth flow for connecting Stripe, Xero, and HubSpot accounts, **so that** data can be pulled without storing raw credentials. | - “Connect” button initiates provider‑specific OAuth.<br>- Tokens stored encrypted in DB (AES‑256).<br>- Revoking access removes tokens and stops data sync. |
| **9** | **As a** Business Owner, **I want** a data mapping UI to align external fields (e.g., Stripe “amount” → KPI “Revenue”), **so that** the dashboard reflects my terminology. | - Drag‑and‑drop mapping canvas.<br>- Validation that required fields are mapped.<br>- Mapping saved per‑account and applied to all downstream reports. |

### **E5 – Expert Guidance & Support**

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **10** | **As a** Startup Founder, **I want** an in‑app chat widget to ask quick scaling questions and receive AI‑augmented answers, **so that** I get immediate help. | - Chat widget accessible from any page.<br>- Backend calls `vLLM` inference endpoint with context (current playbook step).<br>- Response appears within 2 seconds; fallback to “Contact support” if confidence < 70 %. |
| **11** | **As a** Founder, **I want** to schedule a 30‑minute video call with a scaling expert, **so that** I can dive deeper into my specific challenges. | - Calendar integration (Google Calendar) to show expert availability.<br>- Booking creates a Zoom link and sends confirmation email.<br>- Booking appears in “My Sessions” list. |

### **E6 – Platform Foundations**

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **12** | **As a** Security Officer, **I want** role‑based access control (RBAC) so that only authorized users can view or edit sensitive data. | - Roles: Owner, Manager, Viewer.<br>- Permissions matrix documented.<br>- Unauthorized UI elements are hidden; API returns 403. |
| **13** | **As a** QA Engineer, **I want** unit and integration test coverage ≥ 80 % across core modules, **so that** regressions are caught early. | - `pytest` suite with coverage report.<br>- CI pipeline fails if coverage drops below 80 %. |
| **14** | **As a** DevOps Engineer, **I want** a Docker‑based production image and Helm chart, **so that** the service can be deployed to Kubernetes with zero‑downtime. | - `Dockerfile` builds a slim image (< 150 MB).<br>- Helm chart includes liveness/readiness probes.<br>- Rolling update strategy verified in staging. |
| **15** | **As a** Product Owner, **I want** analytics (feature usage, funnel conversion) collected via Snowplow, **so that** we can validate product‑market fit and prioritize future work. | - Event schema for “Playbook Opened”, “Step Completed”, “Workflow Executed”.<br>- Dashboard in Looker shows weekly active users and conversion rates.<br>- GDPR‑compliant opt‑out toggle in user settings. |

---

## MVP Release Plan (Sprint 1‑3)

| Sprint | Epics Covered | Stories Delivered |
|--------|----------------|-------------------|
| **Sprint 1 (2 weeks)** | E1, E2, E6 (foundations) | 1, 2, 3, 4, 5, 12, 13 |
| **Sprint 2 (2 weeks)** | E3, E4 (basic integration) | 6, 8, 9, 14 |
| **Sprint 3 (2 weeks)** | E5, E6 (security & analytics) | 10, 11, 15, 7 (optional if time) |

*All stories are **shippable** – they have clear acceptance criteria, no external dependencies beyond the listed integrations, and can be demonstrated to stakeholders at the end of each sprint.*

--- 

*Prepared by the senior product/engineering lead, 2026‑06‑11.*
