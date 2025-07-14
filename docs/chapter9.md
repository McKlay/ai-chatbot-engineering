---
hide:
    - toc
---

# Chapter 9: Initial Deployment Strategy

---

After building a functional chatbot with a modular backend, smart prompt logic, vector search, and a clean frontend interface, it’s time to put it all together and **deploy it to the cloud**.

This chapter walks you through deploying both frontend and backend components using **Render** and **Netlify**, setting up environment variables, and optionally containerizing your backend using **Docker** for better portability and scalability. You’ll walk away with a fully working MVP chatbot, accessible via the web.

---

## 9.1 Deployment Architecture Overview

Your deployment setup will follow a **frontend-backend split**:

```bash
┌────────────┐      HTTPS       ┌────────────┐       HTTPS/API       ┌──────────────┐
│  React UI  │ ───────────────▶ │ FastAPI App│ ─────────────────────▶│ OpenAI, DB   │
└────────────┘                 └────────────┘                        └──────────────┘
     (Netlify)                    (Render)                              (OpenAI API, Supabase)
```

---

## 9.2 Backend Deployment with Render

### Why Render?

* Free tier available.
* Easy FastAPI support.
* Built-in environment variable management.
* Automatic redeployment on push.

### Steps:

1. **Create a GitHub repo** with your FastAPI backend code.
   Folder structure:

```bash
   chatbot-backend/
     ├── app/
     │   ├── main.py
     │   ├── routes.py
     │   ├── openai_utils.py
     │   ├── vectorstore.py
     │   └── ...
     ├── requirements.txt
     └── Dockerfile (optional)
```

2. **Set up Render service**:

   * Go to [render.com](https://render.com)
   * Create a new **Web Service**
   * Connect your GitHub repo
   * Set build command: `pip install -r requirements.txt`
   * Set start command: `uvicorn app.main:app --host=0.0.0.0 --port=10000`

3. **Add environment variables** in Render’s dashboard:

   * `OPENAI_API_KEY`
   * `SUPABASE_URL`
   * `SUPABASE_KEY`
   * (any others your backend uses)

4. **Verify API endpoint**:
   You should be able to test:

   ```
   https://your-backend-name.onrender.com/chat
   ```

---

## 9.3 Frontend Deployment with Netlify

### Why Netlify?

* React-friendly and automatic from GitHub.
* Free tier with HTTPS.
* Supports custom domains, environment variables, redirects.

### Steps:

1. **Create a GitHub repo** with your React frontend code.
   Typical structure:

```bash
   chatbot-frontend/
     ├── public/
     ├── src/
     │   └── App.jsx
     ├── .env
     └── package.json
```

2. **Configure `.env` file** (optional):

```bash
   VITE_API_URL=https://your-backend-name.onrender.com
```

3. **Deploy to Netlify**:

   * Go to [netlify.com](https://netlify.com)
   * Connect your GitHub frontend repo
   * Set build command: `npm run build`
   * Set publish directory: `dist` (for Vite) or `build` (for CRA)
   * Add environment variables if needed

4. **Verify chatbot UI**:
   You should now be able to visit:

```bash
   https://your-chatbot-site.netlify.app
```

---

## 9.4 Optional: Backend Dockerization

Containerization ensures your backend works consistently across local, cloud, or production environments.

### Sample Dockerfile:

```Dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app/ ./app
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Run Locally:

```bash
docker build -t claybot-backend .
docker run -p 8000:8000 --env-file .env claybot-backend
```

You can later deploy this container to:

* **Render (via Docker tab)**
* **Railway**
* **AWS ECS / GCP Cloud Run / Azure Container Apps**
* **Kubernetes (for scaling Part 4)**

---

## 9.5 Deployment Checklist

| Task                           | Status ✅ |
| ------------------------------ | -------- |
| FastAPI app works locally      | ✅        |
| `.env` secrets are configured  | ✅        |
| Vector DB (Supabase) is live   | ✅        |
| OpenAI API tested successfully | ✅        |
| Render service up              | ✅        |
| Frontend connects to backend   | ✅        |
| Netlify deployed UI            | ✅        |
| CORS configured properly       | ✅        |
| Test: end-to-end conversation  | ✅        |

---

## 9.6 Tips for MVP Rollout

* **Collect Logs**: Use Render’s dashboard or integrate a logging service.
* **Add Feedback Button**: Let users report bugs or suggestions.
* **Soft Launch**: Share with a few testers before public launch.
* **Monitor Usage**: Track API token usage, cost, and errors.

> Remember: MVP doesn't mean minimal functionality. It means delivering value as early as possible with the least complexity.

---

## Conclusion: Your Chatbot Is Live 🎉

With your MVP chatbot deployed to the cloud, you’ve completed the full loop:

* Designed architecture
* Engineered prompts
* Embedded external knowledge
* Built frontend interactions
* Deployed a live chatbot

This makes you ready to scale, extend, and customize. In **Part 3**, we shift gears to gain full control over chatbot infrastructure—by learning how to **host your own LLM models**, optimize performance, and cut costs dramatically.

---

