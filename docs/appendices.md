---
hide:
    - toc
---

# Technical Appendices

> “The last 10% is where the polish lives. And sometimes, it’s where the magic happens.”

This section provides **practical, implementation-ready resources** to support every stage of chatbot development—from setup to deployment, from prompt design to cloud hosting. These appendices are designed for quick reference and rapid execution.

---

## Appendix A: Setting Up Local and Cloud-Hosted Environments for Inference

### Local Inference Setup (for Open-Source LLMs)

**Hardware Recommended**:

* GPU with ≥6 GB VRAM (NVIDIA RTX 3060 or higher)
* 16 GB RAM, SSD storage

**Software Stack**:

```bash
conda create -n chatbot python=3.10
conda activate chatbot
pip install torch transformers accelerate
```

**Run an Open LLM (e.g., Mistral-7B) with Hugging Face**:

```python
from transformers import AutoTokenizer, AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained("mistralai/Mistral-7B-Instruct-v0.1", device_map="auto")
tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.1")

input = tokenizer("Explain vector embeddings.", return_tensors="pt").to("cuda")
output = model.generate(**input, max_new_tokens=200)
print(tokenizer.decode(output[0], skip_special_tokens=True))
```

---

### Cloud Deployment Setup (OpenAI or HF-based APIs)

**Recommended Platforms**:

* **Backend**: Render, Railway, GCP Cloud Run
* **Frontend**: Vercel, Netlify
* **Storage**: Supabase, Firebase, S3

**Environment Variables to Configure**:

* `OPENAI_API_KEY`
* `SUPABASE_URL`
* `SUPABASE_SERVICE_ROLE_KEY`
* `ALLOWED_ORIGINS`

**Deployment Flow**:

1. Push to GitHub
2. Connect repo to platform (Render/Vercel)
3. Set env variables
4. Auto-deploy on main branch push

---

## Appendix B: Comprehensive Docker & Kubernetes Setup

### Dockerfile (FastAPI-based Chatbot Backend)

```Dockerfile
FROM python:3.10-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY ./app /app/app

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Kubernetes Manifest (Basic)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chatbot-backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: chatbot
  template:
    metadata:
      labels:
        app: chatbot
    spec:
      containers:
      - name: chatbot
        image: your-dockerhub/chatbot:latest
        ports:
        - containerPort: 8000
```

---

## Appendix C: Cloud Service Comparison (LLM Hosting & Vector DBs)

### LLM APIs

| Provider                   | Best For                    | Notes                          |
| -------------------------- | --------------------------- | ------------------------------ |
| OpenAI                     | Text + tool calling         | GPT-3.5/4.0, stable + scalable |
| Anthropic                  | Safer outputs, long context | Claude 3 models                |
| Mistral (via Hugging Face) | Open-source alternatives    | Fast & license-flexible        |

---

### Vector Databases

| Service  | Features                              | Hosted?         |
| -------- | ------------------------------------- | --------------- |
| Supabase | pgvector, open source, Postgres-based | Yes (free tier) |
| Pinecone | Fully managed, enterprise-ready       | Yes             |
| Weaviate | Schema-based search, hybrid support   | Yes             |
| Qdrant   | High performance, local + hosted      | Both            |

---

## Appendix D: Prompt Engineering Cookbook

### 1. System Role Prompts

```txt
You are a helpful assistant that explains programming concepts to junior developers using simple analogies and code examples.
```

### 2. Few-shot Prompt (Support Bot)

```txt
User: I want a refund.
Bot: Sure, I can help you with that. Can you provide your order number?

User: My tracking ID isn’t updating.
Bot: No problem. Can you share the tracking ID so I can check its status?
```

### 3. Function Call Format (OpenAI)

```json
{
  "name": "get_invoice_status",
  "parameters": {
    "invoice_id": "string"
  }
}
```

### 4. RAG Prompt (Document Q\&A)

```txt
You are a helpful assistant. Use only the provided context to answer.
If the answer is not in the context, say "I don't know."

Context:
{{retrieved_chunks}}

Question: {{user_question}}
```

---
