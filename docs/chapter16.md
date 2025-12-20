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

**Real-World Example:**

A SaaS chatbot platform serves 100 companies. Each company (tenant) has its own documents, users, and chat history. The backend uses a shared database with `tenant_id` filtering, ensuring Company A never sees Company B's data. Within each company, users have roles (admin, member, viewer) that control access to features.

**Key Design Decision:**

| Consideration        | Shared Infrastructure | Dedicated Per-Tenant |
|----------------------|-----------------------|----------------------|
| **Cost Efficiency**  | High (shared resources)| Low (separate deploys)|
| **Isolation**        | Logical (app-level)    | Physical (infra-level)|
| **Customization**    | Limited               | Full control         |
| **Ops Complexity**   | Lower                 | Higher               |

---

## User Authentication Options

### 🔹 1. JSON Web Tokens (JWT)

JWT is a lightweight, stateless auth method. Ideal for APIs and SPAs.

```python
import jwt
from datetime import datetime, timedelta

payload = {"user_id": 123, "exp": datetime.utcnow() + timedelta(hours=1)}
token = jwt.encode(payload, secret_key, algorithm="HS256")
```

* Fast and stateless
* No server-side revocation without blacklists

**Security Tip:** Use `HS256` for symmetric keys (shared secret) or `RS256` for asymmetric keys (public/private keypair). Store secrets in environment variables or secret managers (AWS Secrets Manager, Azure Key Vault).

**Token Validation Example:**

```python
from fastapi import HTTPException, Depends
from fastapi.security import HTTPBearer

security = HTTPBearer()

def verify_token(credentials: HTTPAuthorizationCredentials = Depends(security)):
    try:
        payload = jwt.decode(credentials.credentials, secret_key, algorithms=["HS256"])
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

### 🔹 2. OAuth 2.0

OAuth lets users log in with Google, GitHub, Microsoft, etc.

* Requires redirect flow and token exchange
* Ideal for production SaaS logins

Use libraries like:

* `authlib` (Python)
* Firebase Auth (client + server support)
* `next-auth` (React + Next.js)

**Example: Google OAuth with Authlib (FastAPI)**

```python
from authlib.integrations.starlette_client import OAuth

oauth = OAuth()
oauth.register(
    name='google',
    client_id='YOUR_CLIENT_ID',
    client_secret='YOUR_CLIENT_SECRET',
    server_metadata_url='https://accounts.google.com/.well-known/openid-configuration',
    client_kwargs={'scope': 'openid email profile'}
)

@app.get('/login')
async def login(request: Request):
    redirect_uri = request.url_for('auth')
    return await oauth.google.authorize_redirect(request, redirect_uri)

@app.get('/auth')
async def auth(request: Request):
    token = await oauth.google.authorize_access_token(request)
    user = token.get('userinfo')
    # Create session or JWT for user
    return {"email": user['email']}
```

### 🔹 3. API Keys

For programmatic access to your chatbot backend (e.g., embedding in 3rd-party apps).

* Store keys in a DB with metadata (created\_at, usage\_count)
* Limit rate per key using Redis or middleware

**Example: API Key Middleware (FastAPI)**

```python
from fastapi import Header, HTTPException

API_KEYS_DB = {"sk-abc123": {"user_id": 1, "tenant_id": "company_a"}}

def verify_api_key(x_api_key: str = Header()):
    if x_api_key not in API_KEYS_DB:
        raise HTTPException(status_code=403, detail="Invalid API key")
    return API_KEYS_DB[x_api_key]

@app.get("/chat", dependencies=[Depends(verify_api_key)])
async def chat_endpoint():
    return {"message": "Authorized"}
```

**Best Practice:** Hash API keys in the database (like passwords) and show the full key only once upon creation. Use prefixes (e.g., `sk-`, `pk-`) to identify key types.

---

## User Session Management

| Feature                 | Tool Options                  | Purpose                                |
| ----------------------- | ----------------------------- | -------------------------------------- |
| **Session storage**     | Redis, PostgreSQL, Supabase   | Persist chat history or metadata       |
| **State management**    | React Context, Redux, Zustand | Handle client-side user state          |
| **Token refresh**       | JWT refresh tokens            | Keep users logged in across sessions   |
| **Rate limit per user** | Redis + IP or token key       | Prevent abuse from individual accounts |

**Example: Session Storage with Redis**

```python
import redis
r = redis.Redis(host='localhost', port=6379)

def save_session(user_id, chat_history):
    r.setex(f"session:{user_id}", 3600, json.dumps(chat_history))  # 1 hour TTL

def get_session(user_id):
    data = r.get(f"session:{user_id}")
    return json.loads(data) if data else []
```

**Token Refresh Flow:**

1. Issue **access token** (short-lived, e.g., 15 min) + **refresh token** (long-lived, e.g., 7 days)
2. When access token expires, client sends refresh token to `/refresh` endpoint
3. Backend validates refresh token and issues new access token
4. Store refresh tokens in DB with user_id for revocation capability

---

## Tenant Isolation Strategies

### Option 1: Tenant ID in Every DB Row

```sql
SELECT * FROM messages WHERE tenant_id = 'abc123';
```

* Simple, scalable
* Requires strict query enforcement

**Implementation Pattern:**

```python
# Middleware to inject tenant_id into all queries
from fastapi import Request

@app.middleware("http")
async def add_tenant_context(request: Request, call_next):
    user = verify_token(request.headers.get("Authorization"))
    request.state.tenant_id = user["tenant_id"]
    response = await call_next(request)
    return response

# Use in queries
@app.get("/documents")
async def get_documents(request: Request):
    tenant_id = request.state.tenant_id
    docs = db.query("SELECT * FROM documents WHERE tenant_id = ?", [tenant_id])
    return docs
```

### Option 2: Separate Schemas or Databases

* Separate Postgres schemas per org (`public`, `tenant_abc`, etc.)

* Or deploy isolated DBs per customer (e.g., high-paying clients)

* Strong isolation

* More ops complexity

**Performance Note:** Row-level tenant filtering (Option 1) is typically sufficient for most SaaS apps. Use separate schemas/DBs only when:

- Regulatory requirements mandate physical isolation (e.g., healthcare, finance)
- Customers pay for dedicated infrastructure
- Performance requires isolated query loads

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

**Advanced RBAC: Permission-Based Access**

Instead of hardcoded roles, use **permissions** for flexibility:

```python
PERMISSIONS = {
    "admin": ["chat", "view_analytics", "manage_users", "configure_model"],
    "member": ["chat", "view_analytics"],
    "viewer": ["view_analytics"]
}

def require_permission(permission: str):
    def dependency(user: dict = Depends(verify_token)):
        user_permissions = PERMISSIONS.get(user["role"], [])
        if permission not in user_permissions:
            raise HTTPException(status_code=403, detail=f"Missing permission: {permission}")
        return user
    return dependency

@app.post("/configure", dependencies=[Depends(require_permission("configure_model"))])
async def configure_model():
    return {"status": "configured"}
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
**Example: Tenant-Aware RAG Query**

```python
def retrieve_context(query, tenant_id, user_id):
    # Filter vector search by tenant
    results = vector_db.search(
        query_embedding=embed(query),
        filter={"tenant_id": tenant_id},
        limit=5
    )
    # Optional: boost user's own documents
    user_docs = [r for r in results if r.metadata.get("uploaded_by") == user_id]
    return user_docs + [r for r in results if r not in user_docs]
```

**Personalization Ideas:**

- Store user preferences (language, tone, model settings) in user profile
- Track frequently asked questions per user for quick retrieval
- Use conversation history to maintain context across sessions
---

## Security Tips

| Issue                 | Mitigation                                    |
| --------------------- | --------------------------------------------- |
| Insecure tokens       | Use strong secrets + rotate keys              |
| Horizontal data leaks | Strict tenant filters in DB queries           |
| Session hijacking     | Use HTTPS + token expiration + fingerprinting |
| Unauthorized access   | Role checks, path-based ACLs                  |

**Additional Security Measures:**

| Technique                | Implementation                                                    |
|--------------------------|-------------------------------------------------------------------|
| **Token Fingerprinting** | Bind JWT to user agent or IP; reject if mismatch                  |
| **Audit Logging**        | Log all access attempts, failed auth, privilege escalations       |
| **Input Validation**     | Sanitize all user inputs to prevent injection attacks             |
| **Data Encryption**      | Encrypt sensitive fields (passwords, API keys) at rest using AES  |
| **CORS Configuration**   | Restrict API access to trusted domains only                       |

**Example: Audit Logging Middleware**

```python
@app.middleware("http")
async def audit_log(request: Request, call_next):
    user = getattr(request.state, "user_id", "anonymous")
    logger.info(f"User {user} accessed {request.method} {request.url.path}")
    response = await call_next(request)
    return response
```

---

## Summary

By adding authentication, user session state, and tenant isolation, your chatbot becomes more than just an app—it becomes a **platform**.

You’ve now built the foundation for:

* Serving multiple users safely
* Isolating chat and document data per tenant
* Supporting user roles and API access
**Production Checklist:**

- [ ] Authentication implemented (JWT, OAuth, or API keys)
- [ ] Tenant isolation enforced at DB query level
- [ ] User sessions persisted (Redis or DB)
- [ ] RBAC or permission system in place
- [ ] Security headers configured (CORS, HTTPS, CSP)
- [ ] Audit logging enabled for sensitive operations
- [ ] Token refresh and logout flows tested
> *Next: Let’s see how we can observe all of this at scale—with real-time analytics, logging, and performance tracking.*

---