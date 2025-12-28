---
hide:
    - toc
---

# Chapter 17: Monitoring and Analytics for Chatbots

> *“If you can’t measure it, you can’t improve it. And if you can’t see it, you can’t fix it.”*

Behind every successful chatbot is a dashboard full of charts, logs, and metrics. Why? Because **observability is the only way to scale reliably.**

When users complain about “slowness,” you need to know whether the delay is in the LLM, the vector DB, or the frontend. When your token costs spike, you need to trace the endpoint and user responsible. And when your chatbot’s accuracy dips, analytics may show a pattern—wrong prompt, bad input, or drift in the document index.

In this chapter, you'll learn how to monitor the **health, performance, and behavior** of your chatbot using modern tools—so you can act fast, optimize smartly, and keep your users happy.

---

## What Should You Monitor?

| Metric Category       | Examples                                   | Tools                               |
| --------------------- | ------------------------------------------ | ----------------------------------- |
| **Latency**           | Response time per endpoint/model           | Prometheus, Grafana, PostHog        |
| **Throughput**        | Requests per minute / concurrent sessions  | Cloud metrics, FastAPI middleware   |
| **Failures**          | 4xx, 5xx errors, timeouts, exceptions      | Sentry, OpenTelemetry, Rollbar      |
| **Token Usage**       | Total tokens used per user/model           | Custom logging + DB                 |
| **Vector Search**     | Similarity scores, chunk retrieval latency | Supabase logs, Redis traces         |
| **Frontend Behavior** | Clicks, message sends, bounce rate         | PostHog, Mixpanel, Google Analytics |


**Advanced Monitoring Tip:**

For distributed systems, add correlation IDs to every request and propagate them through all services (frontend, backend, vector DB, LLM). This enables end-to-end tracing and root cause analysis.

## 1. Prometheus + Grafana (Infra + Backend Metrics)

### Use When:

* You’re running FastAPI/Docker on your own servers
* You want deep insights into **CPU, memory, response time, endpoint load**

### Setup Steps:

1. Add `prometheus_fastapi_instrumentator`:

```bash
pip install prometheus-fastapi-instrumentator
```

2. Add to your FastAPI app:

```python
from prometheus_fastapi_instrumentator import Instrumentator

Instrumentator().instrument(app).expose(app)
```

3. Run **Prometheus** to scrape metrics:

```yaml
scrape_configs:
  - job_name: 'chatbot-backend'
    static_configs:
      - targets: ['localhost:8000']
```

4. Visualize in **Grafana** (build custom dashboards)

**Kubernetes/Cloud-Native:**

- Use Prometheus Operator or managed Prometheus (AWS, GCP) for auto-discovery and scaling.
- Export custom app metrics (e.g., token usage, RAG latency) using `/metrics` endpoints.

---

## Key Backend Metrics to Track

| Metric                      | Description                                 |
| --------------------------- | ------------------------------------------- |
| `http_request_duration`     | API response time                           |
| `inference_latency`         | Model generation speed (esp. large prompts) |
| `embedding_lookup_latency`  | Time spent in vector DB retrieval           |
| `token_usage_total`         | Per-user and per-model token consumption    |
| `queue_length_celery_tasks` | If using async task queues                  |

| `rag_retrieval_latency`    | Time to fetch docs for RAG                  |
| `frontend_latency`         | Time from user click to response            |

---

## 2. PostHog or Mixpanel (User Analytics)

**Product analytics** tools like PostHog or Mixpanel track:

* User flows and friction points
* Retention and active users
* Message frequency and drop-offs
* Feature usage (e.g., summarize, upload, dark mode)

### Example: PostHog for React Chat UI

```bash
npm install posthog-js
```

```js
import posthog from 'posthog-js';
posthog.init('phc_xxx', { api_host: 'https://app.posthog.com' });

posthog.capture('chat_started', { model: 'Mistral' });
posthog.capture('doc_uploaded', { fileSize: 13200 });
```

You can segment by **auth ID**, **tenant**, or **model used** for powerful filtering.

**Best Practice:**

- Anonymize user data before sending to analytics tools to comply with privacy laws (GDPR, CCPA).
- Use feature flags to track adoption of new chatbot features.

---

## 3. Error Logging with Sentry

* Instantly catch **exceptions** and **API failures**
* View **stack traces**, environment data, and user metadata
* Track frequency of specific error types

### Setup in FastAPI

```bash
pip install sentry-sdk
```

```python
import sentry_sdk
sentry_sdk.init(dsn="your_sentry_dsn")
```

Sentry will now log:

* Python exceptions
* HTTP errors
* Custom events (`sentry_sdk.capture_message()`)

**Advanced Error Handling:**

- Tag errors with user ID, tenant, and endpoint for rapid triage.
- Use Sentry Performance to trace slow transactions across async tasks and external APIs.

---

## 4. Custom Token Usage & Cost Tracker

Monitoring tokens is critical for:

* Cost estimation (OpenAI, Anthropic, Mistral-hosted)
* Rate limiting and billing users fairly

### Track token usage per request:

```python
usage = response['usage']
tokens = usage['total_tokens']
store_token_log(user_id, tokens, endpoint, timestamp)
```

Save logs in a **PostgreSQL table** or use **Supabase functions** to aggregate usage.

**Example: Token Usage Table (PostgreSQL)**

```sql
CREATE TABLE token_usage (
  id SERIAL PRIMARY KEY,
  user_id TEXT,
  endpoint TEXT,
  tokens INT,
  timestamp TIMESTAMPTZ DEFAULT now()
);
```

**Cloud-Native:**
- Use BigQuery, AWS Athena, or Supabase for large-scale aggregation and cost dashboards.

---

## 5. Log Everything That Matters

| Log Type               | Use Case                               |
| ---------------------- | -------------------------------------- |
| Input prompts          | Debug unexpected behavior              |
| Response time + source | Distinguish LLM vs. retrieval delays   |
| Retrieved docs         | Debug RAG mismatches or hallucinations |
| User metadata          | Multi-tenant auditing                  |

Use `logging` module in Python or ship logs to:

* Logstash + Kibana (ELK stack)
* Google Cloud Logging
* AWS CloudWatch

**Log Enrichment:**

- Add request IDs, user/tenant info, and latency to every log entry.
- Use structured logging (JSON) for easier parsing and search.

---

## Real-World Dashboard Example

A production chatbot dashboard might include:

* **Avg response time** (by endpoint, model)
* **Live users online**
* **Token burn rate** (hourly/daily)
* **Top queries / intents**
* **Error rate trend (last 24h)**
* **Model response latency histogram**
* **Most active tenants/users**

These dashboards are **not just for engineers**—they’re valuable for product teams, business leaders, and support agents.

**Example: Grafana Dashboard Panels**

- API latency heatmap by endpoint
- Token usage by user and model
- Error rate by tenant
- RAG retrieval time histogram

---

## Summary

| Tool        | Focus                            | Ideal Use Case                      |
| ----------- | -------------------------------- | ----------------------------------- |
| Prometheus  | Infra metrics, response latency  | Self-hosted backend observability   |
| Grafana     | Dashboards for metrics           | Visual monitoring for DevOps teams  |
| PostHog     | User behavior analytics          | Understanding feature adoption & UX |
| Sentry      | Error and exception tracking     | Debugging and log tracing           |
| Custom logs | Token usage, prompt traceability | Cost optimization and audit trails  |

> *You can’t improve what you don’t monitor. And you can’t scale what you don’t understand.*

