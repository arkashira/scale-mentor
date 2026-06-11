# TECH_SPEC.md – scale‑mentor
**Version:** 1.0.0 **Last Updated:** 2026‑06‑11  
**Repository:** https://github.com/axentx/scale-mentor  

---  

## 1. Overview  

scale‑mentor is a SaaS platform that delivers data‑driven scaling playbooks, KPI dashboards, and automation for fast‑growing businesses. It ingests financial, operational, and HR data from third‑party services (Stripe, Xero, HubSpot, etc.), maps the data to a canonical model, runs a rule‑engine powered by case‑study‑derived heuristics, and surfaces actionable recommendations through a web UI and REST API.  

The system is built as a modular, container‑native Python service that can be deployed on Kubernetes (or any OCI‑compatible platform). All components are stateless except for the PostgreSQL data store and Redis cache, enabling horizontal scaling.

---  

## 2. High‑Level Architecture  

```
+-------------------+          +-------------------+          +-------------------+
|   Front‑End (SPA) |  HTTPS   |   API Gateway     |  gRPC/   |   Business Logic  |
|   React + Vite   | <------> | (FastAPI + Auth)  |  HTTP    |   Service (Python)|
+-------------------+          +-------------------+          +-------------------+
                                   |   ^   |                     |
                                   |   |   |                     |
                                   v   |   v                     v
                         +-------------------+          +-------------------+
                         |   Integration Hub |          |   Playbook Engine |
                         |   (AsyncIO)       |          |   (Rule Engine)   |
                         +-------------------+          +-------------------+
                                   |                         |
                                   |                         |
                                   v                         v
                         +-------------------+   +-------------------+
                         |   PostgreSQL DB   |   |   Redis Cache      |
                         +-------------------+   +-------------------+
```

* **Front‑End** – React SPA (outside repo scope, consumes public API).  
* **API Gateway** – FastAPI application (`scale_mentor/api`). Handles authentication (OAuth2 + JWT), request validation, rate limiting, and routing to internal services.  
* **Integration Hub** – Async workers that pull/push data from external SaaS APIs (Stripe, Xero, HubSpot, etc.) using `httpx` + `aiohttp`. Normalizes data into the **Canonical Data Model** (CDM).  
* **Playbook Engine** – Core rule‑engine that matches CDM snapshots against a library of scaling playbooks (YAML/JSON). Uses `pandas` for KPI calculations and `Jinja2` for templated recommendations.  
* **Data Store** – PostgreSQL 15 (primary) for persistent entities (users, companies, playbooks, audit logs).  
* **Cache** – Redis 7 (session store, KPI cache, short‑lived playbook results).  

All services run in Docker containers orchestrated by Helm charts (see `deploy/helm`).  

---  

## 3. Component Breakdown  

| Component | Package | Responsibility | Key Classes / Modules |
|-----------|---------|----------------|-----------------------|
| **API Gateway** | `scale_mentor/api` | HTTP entry point, auth, routing | `main.py`, `router.py`, `auth.py`, `dependencies.py` |
| **Integration Hub** | `scale_mentor/integrations` | Async connectors, data normalization | `stripe.py`, `xero.py`, `hubspot.py`, `sync_manager.py` |
| **Playbook Engine** | `scale_mentor/engine` | KPI calc, rule matching, recommendation rendering | `engine.py`, `playbook_loader.py`, `kpi_calculator.py`, `renderer.py` |
| **Data Access Layer** | `scale_mentor/db` | ORM models, repository pattern | `models.py`, `schemas.py`, `repository.py` |
| **Cache Layer** | `scale_mentor/cache` | Redis client wrappers | `session.py`, `kpi_cache.py` |
| **Background Workers** | `scale_mentor/tasks` | Celery beat + tasks for periodic syncs | `celery_app.py`, `tasks.py` |
| **Domain** | `scale_mentor/domain` | Core business objects (Company, Playbook, KPI) | `entities.py`, `value_objects.py` |
| **Tests** | `tests/` | Unit & integration tests (pytest) | `test_api.py`, `test_engine.py`, … |

---  

## 4. Data Model  

### 4.1 Canonical Data Model (CDM)

| Entity | Fields (excerpt) | Source Mapping |
|--------|------------------|----------------|
| **Company** | `id`, `name`, `industry`, `created_at` | Internal |
| **FinancialSnapshot** | `company_id`, `period_start`, `period_end`, `revenue`, `expenses`, `ARR`, `MRR` | Stripe, Xero |
| **OpsSnapshot** | `company_id`, `period_start`, `period_end`, `active_users`, `churn_rate`, `support_tickets` | HubSpot, internal |
| **HiringSnapshot** | `company_id`, `period_start`, `period_end`, `headcount`, `open_positions`, `time_to_fill` | Greenhouse (future) |
| **KPI** | `company_id`, `name`, `value`, `timestamp` | Calculated by engine |
| **Playbook** | `id`, `title`, `category`, `steps` (JSON/YAML), `applicability_rules` | Static assets (`playbooks/`) |

All tables are defined via SQLAlchemy 2.0 declarative models (`scale_mentor/db/models.py`).  

### 4.2 Example Table DDL (PostgreSQL)

```sql
CREATE TABLE company (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    industry TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE financial_snapshot (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id UUID REFERENCES company(id),
    period_start DATE NOT NULL,
    period_end DATE NOT NULL,
    revenue NUMERIC(15,2),
    expenses NUMERIC(15,2),
    arr NUMERIC(15,2),
    mrr NUMERIC(15,2),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---  

## 5. Key APIs / Interfaces  

All endpoints are versioned under `/api/v1`. OpenAPI schema is auto‑generated (`/docs`).  

### 5.1 Authentication  

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/login` | Returns JWT (access + refresh). |
| POST | `/auth/refresh` | Refreshes access token. |
| POST | `/auth/logout` | Revokes refresh token (stored in Redis). |

### 5.2 Company Management  

| Method | Endpoint | Request | Response |
|--------|----------|---------|----------|
| GET | `/companies/{company_id}` | – | `CompanyDetail` |
| POST | `/companies` | `CompanyCreate` | `CompanyDetail` |
| PATCH | `/companies/{company_id}` | `CompanyUpdate` | `CompanyDetail` |

### 5.3 KPI Dashboard  

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/companies/{company_id}/kpis?period=last_30d` | Returns latest KPI values (cached). |
| GET | `/companies/{company_id}/kpis/history?metric=revenue&start=2024-01-01&end=2024-12-31` | Time‑series data for charting. |

### 5.4 Playbook Execution  

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/playbooks` | List all available playbooks (filtered by company context). |
| POST | `/companies/{company_id}/playbooks/{playbook_id}/run` | Triggers engine run; returns `RunId`. |
| GET | `/runs/{run_id}` | Polls status and fetches rendered recommendations. |

### 5.5 Integration Sync  

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/integrations/{provider}/sync` | Starts async sync for a provider (Stripe, Xero, HubSpot). |
| GET | `/integrations/{provider}/status` | Returns last sync timestamp & health. |

All request/response models are defined with Pydantic (`scale_mentor/api/schemas.py`).  

---  

## 6. Technology Stack  

| Layer | Technology | Version (as of 2026‑06) | Rationale |
|-------|-------------|------------------------|-----------|
| Language | Python | 3.11 | Modern syntax, async support |
| Web Framework | FastAPI | 0.115 | High performance, auto‑docs |
| ASGI Server | Uvicorn | 0.30 | Lightweight, production‑ready |
| ORM | SQLAlchemy 2.0 | 2.0.30 | Declarative, async support |
| DB | PostgreSQL | 15 | ACID, JSONB for playbook storage |
| Cache | Redis | 7.2 | Fast key‑value store, pub/sub |
| Task Queue | Celery + RabbitMQ | Celery 5.4, RabbitMQ 3.12 | Reliable background jobs |
| HTTP Clients | httpx (async) | 0.27 | Async external API calls |
| Data Processing | pandas | 2.2 | KPI calculations |
| Templating | Jinja2 | 3.1 | Playbook recommendation rendering |
| Containerisation | Docker | 24.0 | Consistent builds |
| Orchestration | Kubernetes (Helm) | 1.28 | Auto‑scaling, rollout |
| CI/CD | GitHub Actions | – | Lint, test, build, push images |
| Testing | pytest + pytest‑asyncio | 8.2 | Unit & integration coverage |
| Code Quality | ruff, mypy | – | Linting & static typing |
| Secrets Management | HashiCorp Vault (or K8s Secrets) | – | Secure storage of API keys |

---  

## 7. Dependencies  

* `fastapi>=0.115,<0.116`
* `uvicorn[standard]>=0.30,<0.31`
* `sqlalchemy[asyncio]>=2.0.30`
* `psycopg2-binary>=2.9`
* `redis>=5.0`
* `celery[redis]>=5.4`
* `httpx>=0.27`
* `pandas>=2.2`
* `jinja2>=3.1`
* `pydantic>=2.7`
* `python‑jwt>=2.8`
* `aiofiles`, `python‑dotenv` for config loading  

All pinned in `requirements.txt` and mirrored in `pyproject.toml`.  

---  

## 8. Deployment Architecture  

### 8.1 Container Images  

| Service | Dockerfile location | Image name (example) |
|---------|--------------------|----------------------|
| API Gateway | `Dockerfile.api` | `ghcr.io/axentx/scale-mentor/api:{{git_sha}}` |
| Integration Hub | `Dockerfile.worker` | `ghcr.io/axentx/scale-mentor/worker:{{git_sha}}` |
| Celery Beat | `Dockerfile.beat` | `ghcr.io/axentx/scale-mentor/beat:{{git_sha}}` |

Images are built on `python:3.11-slim` with multi‑stage builds to keep size < 120 MB.

### 8.2 Helm Chart (see `deploy/helm/scale-mentor`)  

Key values:

```yaml
replicaCount: 3
image:
  repository: ghcr.io/axentx/scale-mentor/api
  tag: "{{ .Chart.AppVersion }}"
service:
  type: ClusterIP
  port: 80
resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
  requests:
    cpu: "250m"
    memory: "256Mi"
envFrom:
  - secretRef:
      name: scale-mentor-secrets
postgresql:
  enabled: true
redis:
  enabled: true
celery:
  concurrency: 4
```

### 8.3 CI/CD Pipeline  

1. **Push** → GitHub Actions lint (`ruff`), type‑check (`mypy`), unit tests (`pytest`).  
2. **On success** → Build Docker images, push to GitHub Container Registry.  
3. **Deploy** → Helm upgrade to `prod` namespace (manual approval gate).  

---  

## 9. Security & Compliance  

| Concern | Mitigation |
|---------|------------|
| **Authentication** | OAuth2 Authorization Code flow with PKCE for SPA; JWT signed with RS256 (keys stored in Vault). |
| **Authorization** | Role‑based access control (RBAC) enforced in FastAPI dependencies (`admin`, `company_owner`). |
| **Data at Rest** | PostgreSQL encrypted with `pgcrypto`; Redis TLS enabled. |
| **Data in Transit** | All endpoints served over HTTPS (TLS 1.3). |
| **Secret Management** | API keys and DB credentials injected via K8s secrets or Vault; never committed. |
| **Audit Logging** | Every write operation logs user, timestamp, and payload to `audit_log` table. |
| **Compliance** | GDPR‑compatible data deletion endpoint (`/companies/{id}/purge`). |

---  

## 10. Observability  

* **Metrics** – Prometheus exporter (`fastapi_prometheus`) exposing request latency, error rates, DB pool usage.  
* **Tracing** – OpenTelemetry SDK with Jaeger backend for end‑to‑end request tracing (API → Integration Hub → DB).  
* **Logging** – Structured JSON logs via `structlog`; shipped to Loki/ELK stack.  

---  

## 11. Testing Strategy  

| Test Type | Scope | Tools |
|-----------|-------|-------|
| Unit | Individual functions, models, utils | pytest, pytest‑mock |
| Integration | API routes + DB (test container) | pytest‑asyncio, testcontainers |
| End‑to‑End | Full flow (sync → engine → dashboard) | Playwright (for SPA) + API calls |
| Load | KPI dashboard under concurrent users | locust.io (target 200 RPS) |
| Security | Auth/role enforcement, secret leakage | bandit, OWASP ZAP |

Coverage target: **≥85 %** for core modules (`engine`, `integrations`, `api`).  

---  

## 12. Future Enhancements (Roadmap)  

| Milestone | Feature | Description |
|-----------|---------|-------------|
| v1.1 | **AI‑augmented Recommendations** | Plug‑in LLM (vLLM) to generate natural‑language suggestions based on KPI trends. |
| v1.2 | **Marketplace for Playbooks** | Allow third‑party consultants to publish playbooks (sandboxed execution). |
| v1.3 | **Real‑time Streaming** | WebSocket KPI push using `fastapi-sockets` for live dashboards. |
| v2.0 | **Multi‑tenant Isolation** | Separate PostgreSQL schemas per tenant for stricter data isolation. |

---  

## 13. Glossary  

* **KPI** – Key Performance Indicator.  
* **CDM** – Canonical Data Model, unified schema for external SaaS data.  
* **Playbook** – Structured, step‑by‑step scaling guide stored as YAML/JSON.  
* **Sync** – Periodic pull of external data into the CDM.  

---  

*Prepared by the Scale‑Mentor Engineering Lead*  
*All specifications are subject to change with stakeholder review.*
