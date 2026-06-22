# TECH_SPEC.md

## 1. Overview

**Project:** *thought-bridge*  
**Purpose:** Provide a two‑tier AI service that first generates high‑level, correct design artifacts (architecture diagrams, pseudo‑code, test plans) and then produces low‑level, production‑ready implementation code. The system guarantees correctness over raw inference speed by employing a *verification* pipeline that cross‑checks outputs against a curated knowledge base and static analysis.

---

## 2. Architecture

```
┌───────────────────────┐
│  1. Client API Layer  │
│  (REST / gRPC)        │
└─────────────┬─────────┘
              │
┌─────────────▼─────────┐
│ 2. Orchestration Layer│
│   (FastAPI + Celery)  │
└───────┬───────────────┘
        │
┌───────▼───────────────┐
│ 3. High‑Level Engine  │
│   (vLLM + Prompting)  │
└───────┬───────────────┘
        │
┌───────▼───────────────┐
│ 4. Verification Layer │
│   (LLM + Static‑Analysis) │
└───────┬───────────────┘
        │
┌───────▼───────────────┐
│ 5. Low‑Level Engine   │
│   (CodeGen + Templates) │
└───────┬───────────────┘
        │
┌───────▼───────────────┐
│ 6. Output Formatter   │
│   (Markdown / JSON)   │
└───────────────────────┘
```

### 2.1 Client API Layer
- **Endpoints**  
  - `POST /v1/think` – Submit a task description.  
  - `GET /v1/status/{task_id}` – Poll task status.  
  - `GET /v1/result/{task_id}` – Retrieve final artifacts.  
- **Transport** – HTTPS, JSON payloads.  
- **Auth** – API key + JWT (optional).

### 2.2 Orchestration Layer
- **Framework** – FastAPI for HTTP, Celery for async task queue.  
- **Broker** – Redis (dedicated cluster).  
- **Result Store** – PostgreSQL (metadata) + S3‑compatible object store (artifacts).

### 2.3 High‑Level Engine
- **Model** – vLLM serving `meta-llama/Meta-Llama-3.1-70B-Instruct`.  
- **Prompt** – Structured prompt template that asks for:  
  1. Problem statement.  
  2. Architecture diagram (PlantUML).  
  3. Pseudo‑code (Python‑like).  
  4. Test plan.  
- **Safety** – Prompt injection mitigated via input sanitisation and token‑limit enforcement.

### 2.4 Verification Layer
- **Components**  
  - **LLM verifier** (`gpt-4o-mini`) checks logical consistency against the knowledge base.  
  - **Static analyzer** (`pylint`, `bandit`) runs on pseudo‑code.  
- **Process** – Verifier receives high‑level output, generates a *verification report*. If any check fails, the task is retried with a new prompt or flagged for manual review.

### 2.5 Low‑Level Engine
- **Model** – vLLM serving `codellama/CodeLlama-34b-Instruct`.  
- **Template Engine** – Jinja2 templates for target language (Python, Go, Rust).  
- **Post‑processing** – Code formatting (`black`, `gofmt`, `rustfmt`) and linting.

### 2.6 Output Formatter
- **Formats** – Markdown (with embedded PlantUML), JSON (structured artifacts).  
- **Delivery** – Stored in object store; link returned to client.

---

## 3. Data Model

| Table | Columns | Description |
|-------|---------|-------------|
| `tasks` | `id PK`, `user_id`, `description`, `status`, `created_at`, `updated_at` | Task metadata |
| `results` | `id PK`, `task_id FK`, `artifact_type`, `artifact_url`, `created_at` | Links to stored artifacts |
| `verifications` | `id PK`, `task_id FK`, `passed`, `report_text`, `created_at` | Verification outcome |

Artifacts stored in S3:
- `thought-bridge/{task_id}/architecture.puml`
- `thought-bridge/{task_id}/pseudo.py`
- `thought-bridge/{task_id}/test_plan.md`
- `thought-bridge/{task_id}/implementation.py`

---

## 4. Key APIs / Interfaces

| API | Method | Path | Request | Response |
|-----|--------|------|---------|----------|
| Submit Task | POST | `/v1/think` | `{ "description": "..." }` | `{ "task_id": "...", "status": "queued" }` |
| Task Status | GET | `/v1/status/{task_id}` | N/A | `{ "status": "...", "progress": 0-100 }` |
| Get Result | GET | `/v1/result/{task_id}` | N/A | `{ "artifacts": { "architecture": "...", "pseudo": "...", "implementation": "..." } }` |

---

## 5. Technology Stack

| Layer | Technology | Version | Rationale |
|-------|------------|---------|-----------|
| API | FastAPI | 0.110+ | Modern async framework |
| Orchestration | Celery | 5.4+ | Reliable task queue |
| Broker | Redis | 7.x | Low‑latency pub/sub |
| DB | PostgreSQL | 16.x | ACID compliance |
| Object Store | MinIO (S3 API) | 2026 | Self‑hosted, cost‑effective |
| LLM Inference | vLLM | 0.4+ | GPU‑efficient, multi‑model |
| Models | Meta‑Llama‑3.1‑70B‑Instruct, CodeLlama‑34B‑Instruct, GPT‑4o‑Mini | N/A | High‑quality reasoning & code |
| Prompting | OpenAI Prompt Toolkit | N/A | Structured prompts |
| Static Analysis | pylint, bandit | 3.x | Security & style checks |
| Formatting | black, gofmt, rustfmt | N/A | Code style enforcement |
| Templates | Jinja2 | 3.x | Render code skeletons |
| Monitoring | Prometheus + Grafana | N/A | Metrics & dashboards |
| CI/CD | GitHub Actions | N/A | Automated testing & deployment |

---

## 6. Dependencies

| Category | Package | Version |
|----------|---------|---------|
| Python | fastapi | 0.110+ |
| | uvicorn | 0.29+ |
| | celery | 5.4+ |
| | redis | 5.0+ |
| | sqlalchemy | 2.0+ |
| | alembic | 1.13+ |
| | pydantic | 2.6+ |
| | jinja2 | 3.1+ |
| | vllm | 0.4+ |
| | openai | 1.12+ |
| | prometheus_client | 0.20+ |
| | black | 24.4+ |
| | pylint | 3.0+ |
| | bandit | 1.7+ |
| | minio | 7.0+ |
| | boto3 | 1.34+ |

All dependencies are pinned in `pyproject.toml` and `requirements.txt`. Docker images are built from `Dockerfile` using multi‑stage builds.

---

## 7. Deployment

### 7.1 Infrastructure

| Component | Host | Replicas | Notes |
|-----------|------|----------|-------|
| API | Kubernetes (EKS/GKE) | 2 | Autoscaled |
| Celery Workers | Kubernetes | 4 | Horizontal pod autoscaler |
| Redis | Managed Redis | 1 | Sentinel for HA |
| PostgreSQL | Managed Postgres | 1 | Replication |
| MinIO | Kubernetes | 1 | Erasure coding enabled |
| vLLM Servers | GPU nodes | 2 | Each hosts one model |
| Prometheus | Kubernetes | 1 | Scrape all pods |
| Grafana | Kubernetes | 1 | Dashboard for latency, throughput |

### 7.2 CI/CD Pipeline

1. **Build** – Docker image built from `Dockerfile`.  
2. **Test** – Unit tests (pytest), integration tests (FastAPI TestClient).  
3. **Lint** – flake8, black.  
4. **Deploy** – Helm chart applied to staging cluster.  
5. **Smoke Test** – End‑to‑end request to `/v1/think`.  
6. **Promote** – If all checks pass, merge to `main` and deploy to production.

### 7.3 Scaling Strategy

- **Model scaling** – Add GPU nodes; vLLM handles model sharding.  
- **Task queue** – Increase Celery workers based on queue depth.  
- **Stateless API** – Horizontal scaling via Kubernetes Service.  
- **Cache** – Redis for session & short‑term caching of verification results.

---

## 8. Security & Compliance

- **API Keys** – Stored in Vault; rotated quarterly.  
- **Transport** – TLS 1.3 enforced.  
- **Data** – All artifacts encrypted at rest (MinIO SSE‑S3).  
- **Audit** – All API calls logged to CloudWatch/Stackdriver.  
- **GDPR** – Data retention policy: 90 days; user can request deletion.

---

## 9. Monitoring & Observability

| Metric | Source | Alert |
|--------|--------|-------|
| Request latency (API) | Prometheus | > 2s |
| Task queue length | Celery | > 100 |
| Model GPU utilisation | NVIDIA DCGM | > 80% |
| Error rate | Prometheus | > 1% |
| Verification pass rate | Custom | < 95% |

Dashboards: latency, throughput, verification success, resource utilisation.

---

## 10. Future Enhancements

- **Multilingual support** – Add language‑specific LLMs.  
- **Interactive debugging** – WebSocket stream of verification steps.  
- **Self‑learning loop** – Store failed verifications to fine‑tune models.  
- **Marketplace** – Plug‑in architecture for custom code templates.

---

*Prepared by: Senior Product/Engineering Lead – Axentx*
