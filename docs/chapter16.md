---
hide:
    - toc
---

# Chapter 16: Multi-Tenancy and User Management

> *“A chatbot that talks to everyone the same way isn’t scalable—it’s generic.”*

As your chatbot grows, it won’t just serve one user—it’ll serve **hundreds, maybe thousands**, each with their own data, preferences, histories, and permissions. That’s where **multi-tenancy** and **user management** come in.

This chapter explores how to build **secure, isolated, personalized experiences** for every user (or team) that interacts with your chatbot—whether through a SaaS dashboard, an internal tool, or an enterprise integration.

You’ll learn how to:

* Authenticate users with **JWTs, OAuth, or API keys**
* Persist user sessions across chats
* Isolate tenant data (multi-user or multi-org)
* Add permission-based access control
* Handle timeouts, logout, and re-authentication flows

---

## What Is Multi-Tenancy?

**Multi-tenancy** means a **single chatbot backend** serves multiple distinct users or clients—while keeping their data and interactions isolated.

| Type              | Description                              | Use Case                             |
| ----------------- | ---------------------------------------- | ------------------------------------ |
| **Single-Tenant** | One instance per customer                | Dedicated deployments (e.g. on-prem) |
| **Multi-Tenant**  | One shared instance, many isolated users | SaaS chatbot apps                    |
| **Multi-Tier**    | Users within organizations/teams         | Enterprise teams with roles/ACLs     |

---

## User Authentication Options

### 🔹 1. JSON Web Tokens (JWT)

JWT is a lightweight, stateless auth method. Ideal for APIs and SPAs.

```python
import jwt

payload = {"user_id": 123, "exp": datetime.utcnow() + timedelta(hours=1)}
token = jwt.encode(payload, secret_key, algorithm="HS256")
```

* Fast and stateless
* No server-side revocation without blacklists

### 🔹 2. OAuth 2.0

OAuth lets users log in with Google, GitHub, Microsoft, etc.

* Requires redirect flow and token exchange
* Ideal for production SaaS logins

Use libraries like:

* `authlib` (Python)
* Firebase Auth (client + server support)
* `next-auth` (React + Next.js)

### 🔹 3. API Keys

For programmatic access to your chatbot backend (e.g., embedding in 3rd-party apps).

* Store keys in a DB with metadata (created\_at, usage\_count)
* Limit rate per key using Redis or middleware

---

## User Session Management

| Feature                 | Tool Options                  | Purpose                                |
| ----------------------- | ----------------------------- | -------------------------------------- |
| **Session storage**     | Redis, PostgreSQL, Supabase   | Persist chat history or metadata       |
| **State management**    | React Context, Redux, Zustand | Handle client-side user state          |
| **Token refresh**       | JWT refresh tokens            | Keep users logged in across sessions   |
| **Rate limit per user** | Redis + IP or token key       | Prevent abuse from individual accounts |

---

## Tenant Isolation Strategies

### Option 1: Tenant ID in Every DB Row

```sql
SELECT * FROM messages WHERE tenant_id = 'abc123';
```

* Simple, scalable
* Requires strict query enforcement

### Option 2: Separate Schemas or Databases

* Separate Postgres schemas per org (`public`, `tenant_abc`, etc.)

* Or deploy isolated DBs per customer (e.g., high-paying clients)

* Strong isolation

* More ops complexity

---

## 👥 Role-Based Access Control (RBAC)

You may want to assign roles like:

* `admin`: can view all chats, add team members
* `member`: can chat but not configure model
* `viewer`: read-only analytics

### Example Role Check (FastAPI)

```python
def require_admin(user: dict):
    if user["role"] != "admin":
        raise HTTPException(status_code=403, detail="Admin access required")
```

Store roles in your user table and embed in JWT payloads:

```json
{ "user_id": "123", "role": "admin" }
```

---

## Integrating User Context into Chat

In multi-user chatbots, it’s important to **inject user-specific context** during prompt construction or RAG:

```python
prompt = f"You are helping user {user.name}. Their company is {user.company}."
```

Use:

* **Dynamic system prompts**
* **User profile embeddings**
* **Personalized RAG document filters (via tenant ID)**

---

## Security Tips

| Issue                 | Mitigation                                    |
| --------------------- | --------------------------------------------- |
| Insecure tokens       | Use strong secrets + rotate keys              |
| Horizontal data leaks | Strict tenant filters in DB queries           |
| Session hijacking     | Use HTTPS + token expiration + fingerprinting |
| Unauthorized access   | Role checks, path-based ACLs                  |

---

## Summary

By adding authentication, user session state, and tenant isolation, your chatbot becomes more than just an app—it becomes a **platform**.

You’ve now built the foundation for:

* Serving multiple users safely
* Isolating chat and document data per tenant
* Supporting user roles and API access

> *Next: Let’s see how we can observe all of this at scale—with real-time analytics, logging, and performance tracking.*

---