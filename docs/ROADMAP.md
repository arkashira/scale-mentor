# ROADMAP.md – scale‑mentor
**Versioned product roadmap aligned with the current repo state (Python, pyproject, core modules under `axentx_product`, `business`, `src`).**  
All items are scoped to be **shippable**, **testable**, and **aligned with our validated market need** (data‑driven scaling playbooks + KPI automation).  

---  

## 📅 Milestones Overview
| Milestone | Target Release | Primary Goal | MVP‑Critical? |
|-----------|----------------|--------------|---------------|
| **MVP – “Launch Ready”** | **2026‑07‑31** | Deliver a usable, self‑service SaaS that a pilot customer can adopt and see a 30 % reduction in scaling trial‑and‑error. | ✅ |
| **v1 – “Growth Enablement”** | **2026‑11‑15** | Expand integrations, enrich playbooks, add collaborative features, and improve reliability. | — |
| **v2 – “Enterprise Scale”** | **2027‑04‑30** | Harden security, add role‑based access, AI‑augmented recommendations, and white‑label capabilities. | — |

---  

## 🚀 MVP – Launch Ready (Due 2026‑07‑31)

| # | Feature | Description | Acceptance Criteria | MVP‑Critical |
|---|---------|-------------|---------------------|--------------|
| 1 | **Core Playbook Engine** | Load, render, and navigate step‑by‑step scaling playbooks (JSON/YAML). | • Playbooks importable via `src/playbooks/` <br>• UI shows next/previous steps <br>• Progress persisted per user | ✅ |
| 2 | **KPI Dashboard (Beta)** | Real‑time view of key metrics (ARR, CAC, Burn, Headcount). | • Dashboard shows at least 4 default KPIs <br>• Data refreshed every 5 min from in‑memory store <br>• Export to CSV | ✅ |
| 3 | **Stripe & Xero Sync (Read‑only)** | Pull financial data to feed KPI dashboard. | • OAuth flow works for test accounts <br>• Daily sync jobs succeed > 95 % <br>• Errors logged & visible in UI | ✅ |
| 4 | **Automation Stub** | Trigger a simple webhook when a playbook step is marked complete. | • Configurable webhook URL <br>• Payload includes user‑id, step‑id, timestamp | ✅ |
| 5 | **User Management (Basic)** | Sign‑up, login, password reset (email only). | • JWT‑based auth <br>• Password hashing with Argon2 <br>• Unit tests ≥ 80 % coverage | ✅ |
| 6 | **CI/CD & Test Suite** | Automated build, lint, unit & integration tests. | • GitHub Actions pipeline passes on every PR <br>• Code coverage badge ≥ 85 % | ✅ |
| 7 | **Documentation & Onboarding** | Quick‑start guide, API docs (Swagger), and sample data. | • README updated with “Getting Started” <br>• Swagger UI reachable at `/docs` | ✅ |
| 8 | **Security Baseline** | HTTPS, secret management, input validation. | • No high‑severity OWASP findings in automated scan <br>• Secrets stored in GitHub Actions secrets | ✅ |

### MVP Success Metrics
- **Pilot adoption:** ≥ 5 beta customers within 30 days of launch.  
- **Time‑to‑value:** Users report ≥ 30 % reduction in scaling experimentation (survey).  
- **Stability:** < 1 % crash rate, < 2 % failed sync jobs.  

---  

## 🌱 v1 – Growth Enablement (Due 2026‑11‑15)

| Theme | Feature | Description | Shippable Scope |
|-------|---------|-------------|-----------------|
| **Integration Expansion** | HubSpot & QuickBooks sync (read/write) | Pull CRM pipelines, push invoice status. | Bi‑directional sync, UI mapping UI. |
| **Playbook Authoring** | Web‑based editor & versioning | Drag‑and‑drop step builder, markdown support, version history. | Save drafts, publish versions, rollback. |
| **Collaboration** | Team spaces & comments | Multiple users per org, comment threads on steps. | Role‑based view, notifications. |
| **Automation Enhancements** | Conditional workflow engine | IF‑THEN rules based on KPI thresholds (e.g., auto‑scale hiring). | Visual rule builder, test sandbox. |
| **Analytics & Reporting** | Custom KPI builder + trend charts | Users define own metrics, chart types, export PDF. | Save dashboards, schedule email reports. |
| **Reliability** | Horizontal scaling, Redis cache, DB migrations | Containerised deployment (Docker‑Compose + Helm). | Zero‑downtime deploy scripts. |
| **Compliance** | GDPR data‑subject tools | Data export/delete endpoints, consent logs. | Auditable logs, UI for request handling. |

---  

## 🏢 v2 – Enterprise Scale (Due 2027‑04‑30)

| Theme | Feature | Description | Shippable Scope |
|-------|---------|-------------|-----------------|
| **Security Hardened** | SSO (SAML/OIDC), RBAC, audit logging | Enterprise identity providers, granular permissions. | Admin console, exportable audit trail. |
| **AI‑augmented Guidance** | LLM‑driven recommendation engine (vLLM) | Suggest next playbook steps based on KPI trends. | API endpoint, UI hints, confidence score. |
| **White‑Label & Multi‑Tenant** | Custom branding, sub‑domains, tenant isolation | Separate DB schemas, theming per client. | Branding kit, tenant admin portal. |
| **Advanced Automation** | Full RPA integration (e.g., Zapier, n8n) | Trigger external workflows beyond webhooks. | Pre‑built connectors, custom script runner. |
| **Performance & Scale** | Distributed task queue (Celery + Redis), autoscaling | Support >10k concurrent users, sub‑second KPI refresh. | Autoscaling policies, load‑test report. |
| **Marketplace** | Playbook marketplace & revenue share | Community‑submitted playbooks, rating system. | Marketplace UI, payment integration. |
| **Compliance Extensions** | SOC 2, ISO 27001 readiness | Documentation, controls, third‑party audit checklist. | Compliance package, self‑assessment tool. |

---  

## 📌 How We Prioritize

1. **Customer Pain → Revenue** – Features that directly reduce scaling time or unlock new integration revenue are top priority.  
2. **Technical Foundations** – Security, CI/CD, and scalability are non‑negotiable before expanding the feature set.  
3. **Feedback Loop** – Each release includes a “Pilot Feedback” sprint to incorporate real‑world usage data back into the roadmap.  

---  

## 📈 Success Metrics per Phase  

| Phase | KPI | Target |
|-------|-----|--------|
| MVP | Pilot NPS | ≥ 45 |
| MVP | Monthly Active Users (MAU) | ≥ 200 by month 2 |
| v1 | Integration Adoption Rate | ≥ 60 % of customers using ≥ 2 external services |
| v1 | Churn (first 3 mo) | ≤ 5 % |
| v2 | Enterprise ARR | ≥ $500k ARR within 6 mo of v2 launch |
| v2 | SLA uptime | 99.9 % |

---  

## 📚 References  

* **Repo:** `github.com/axentx/scale-mentor` – current codebase, `pyproject.toml`, tests.  
* **Runbook:** AXENTX RUNBOOK + LOCATIONS (2026‑05‑23) – deployment standards, CI pipelines.  
* **Tech Stack:** Python, vLLM (future AI engine), existing integration libraries.  

---  

*Prepared by the Senior Product/Engineering Lead, 2026‑06‑11.*
