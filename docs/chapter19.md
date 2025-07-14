---
hide:
    - toc
---

# Chapter 19: Security, Privacy, and Compliance

> *“Trust is earned. Security is engineered. Compliance is enforced.”*

No matter how smart your chatbot is—if it leaks data, mishandles PII, or violates user trust, it will be shut down faster than it responds to a prompt.

This chapter explores the **defensive side** of chatbot infrastructure: how to design for **security**, build for **privacy**, and operate within **compliance frameworks** like **GDPR**, **HIPAA**, and **SOC 2**.

You’ll learn how to:

* Secure APIs, databases, and inference endpoints
* Protect sensitive data during training and inference
* Handle user data rights (delete, anonymize, audit)
* Pass compliance audits and meet legal obligations

Whether you're working on an internal tool or a global SaaS chatbot, these practices are **non-negotiable.**

---

## Threats You Must Defend Against

| Threat Type             | Example Scenario                                     |
| ----------------------- | ---------------------------------------------------- |
| **Prompt Injection**    | Malicious prompt tricks model into revealing secrets |
| **Unauthorized Access** | Unauthenticated API access to documents or chats     |
| **Data Leakage**        | Logging PII or chats in plaintext                    |
| **Inference Hijack**    | Users running the model in unintended ways           |
| **Supply Chain Risk**   | Vulnerabilities in LLM models or libraries           |

---

## Securing Your Chatbot APIs

### 1. **API Authentication & Access Control**

| Method  | Use Case                            |
| ------- | ----------------------------------- |
| JWT     | Web login, session-bound access     |
| OAuth2  | Enterprise or 3rd-party login       |
| API Key | Programmatic access to backend APIs |

Example: API Key header check in FastAPI

```python
from fastapi import Header, HTTPException

def verify_key(x_api_key: str = Header(...)):
    if x_api_key != "SECRET_KEY":
        raise HTTPException(status_code=403, detail="Forbidden")
```

> **Best Practice:** Use HTTPS **everywhere**, even in dev.

---

## Data Privacy Best Practices

| Practice                             | Why It Matters                                        |
| ------------------------------------ | ----------------------------------------------------- |
| **Don’t log full prompts/responses** | Prevent PII or sensitive content leakage              |
| **Encrypt data at rest**             | Secure user files, vectors, and chat history          |
| **Encrypt data in transit**          | Prevent MITM attacks on API traffic                   |
| **Limit access**                     | Use IAM roles, RBAC, and principle of least privilege |
| **Audit logs**                       | Forensics after security incidents                    |

### Encrypting Vector DB Fields (e.g., Supabase)

While pgvector doesn’t natively encrypt embeddings, you can:

* Hash filenames or tenant IDs
* Store encrypted documents in S3/GCS with signed URLs
* Avoid embedding user secrets (strip during chunking)

---

## GDPR, HIPAA, and Other Compliance Rules

### 1. GDPR (Europe)

| Rule                      | Implication                             |
| ------------------------- | --------------------------------------- |
| **Right to be forgotten** | Delete all user data upon request       |
| **Data minimization**     | Only store what is strictly needed      |
| **Explicit consent**      | Must get user opt-in for processing PII |
| **Data export**           | Allow users to download their data      |

Implement:

* `DELETE /user_data/{user_id}`
* Export endpoint: `GET /user_data/export`

---

### 2. HIPAA (US - Healthcare)

For chatbots dealing with health info (PHI):

* **Encrypt everything (TLS, storage, logs)**
* **Avoid using 3rd-party APIs that don’t sign BAAs (Business Associate Agreements)**
* **Log access to health data**
* **Tokenize PHI before LLM inference, when possible**

---

### 3. SOC 2 (General SaaS Compliance)

Focuses on:

* Security
* Availability
* Confidentiality
* Privacy
* Processing Integrity

> Use **audit logs, environment separation, access reviews, backups**, and **incident response plans**.

---

## Inference-Time Safety

| Risk                             | Mitigation                                        |
| -------------------------------- | ------------------------------------------------- |
| Prompt injection                 | Pre-validate prompt format; system prompt filters |
| Jailbreak attempts               | Add guardrails + toxic content detection          |
| Bias, toxicity, or hallucination | Use moderation layer or reranker                  |
| Unsafe output (e.g., PII leak)   | Mask or scrub response before sending             |

### Tools:

* OpenAI Moderation API
* Google Perspective API
* Custom toxicity classifiers

---

## Secure Your Inference Pipeline

| Component         | Defense Strategy                         |
| ----------------- | ---------------------------------------- |
| FastAPI API       | Auth middleware, HTTPS, rate limits      |
| Vector Store      | Row-level security, field encryption     |
| LLM Inference     | Prompt templates + input sanitization    |
| Storage (S3, GCS) | Signed URLs, encryption, access controls |
| CI/CD Pipelines   | Secrets management, code scanning        |

---

## Summary

Security and compliance are **not afterthoughts**—they’re integral to trust, especially when handling real-world data.

| Layer        | Protection Strategy                     |
| ------------ | --------------------------------------- |
| API access   | JWT, OAuth2, API keys                   |
| Chat history | Tokenize, anonymize, encrypt            |
| Storage      | Field-level encryption, access controls |
| Monitoring   | Error logging + audit trails            |
| Compliance   | Implement GDPR, HIPAA, SOC 2 guardrails |

> *A chatbot isn’t just a product—it’s a data gateway. Keep it secure, private, and accountable.*

---