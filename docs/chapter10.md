---
hide:
    - toc
---

# Chapter 10: Introduction to Self-Hosted LLMs

> *“At some point, the question is no longer ‘What can GPT do for us?’ but rather, ‘What could we do if we owned the brain?’”*

## Why Self-Host?


Most chatbot builders begin with API calls to OpenAI, Anthropic, or Cohere. It's fast, it works, and it scales—until it doesn't.

At a certain stage, product owners start to run into barriers:

* **Monthly bills skyrocket** due to token usage, especially as user adoption grows.
* **User data privacy** becomes a concern, especially in finance, healthcare, or enterprise SaaS, where regulations like GDPR or HIPAA apply.
* **Latency** becomes noticeable in regions far from OpenAI’s servers, impacting user experience.
* **Customization limits** emerge—your use case demands domain-specific knowledge, custom guardrails, or unique workflows that generic models don’t capture.

This is when teams start exploring **self-hosting**—running open-source LLMs on their own infrastructure to **cut costs, increase privacy, and tune behavior**. Self-hosting can also unlock new business models, such as on-premise deployments for clients or offline/edge AI.

But self-hosting isn’t just a technical switch. It’s a **philosophical one**. You’re not just building with AI anymore—you’re **operating** it. You become responsible for uptime, security, and model performance.

---

## Benefits of Self-Hosting


| Benefit          | Description                                                                    |
| ---------------- | ------------------------------------------------------------------------------ |
| Full Control  | You decide the model, tokenizer, prompt format, and fine-tuning setup.         |
| Data Privacy  | Sensitive user data never leaves your servers—ideal for compliance needs.      |
| Lower Latency  | Host the model near your users (edge or on-prem) to reduce response time.      |
| Cost Savings  | For high-volume apps, GPU hosting may be cheaper than pay-per-token APIs.      |
| Customization | Tune models to specific domains, add guardrails, chain with internal tools, or integrate with proprietary data. |
| Offline/Edge   | Run models in disconnected environments or on customer premises.              |

---

## Tradeoffs & Challenges


| Challenge               | Description                                                               |
| ----------------------- | ------------------------------------------------------------------------- |
| Hardware Complexity  | Requires knowledge of GPU/TPU setup and maintenance.                      |
| Model Management     | Loading, updating, and versioning models must be handled manually.        |
| Inferencing Overhead | Inference is compute-heavy and requires optimization to stay responsive.  |
| Tooling Ecosystem    | No built-in dashboards, logs, or error handling like OpenAI’s playground. |
| Security & Access    | You must secure APIs, model weights, and usage logs yourself.             |
| Scaling              | Need to plan for load balancing, failover, and monitoring.                |

---

## Hardware Considerations

To self-host LLMs, you’ll need the right compute environment. The table below summarizes key options:

| Hardware Type | Description                           | Ideal Use Case                           | Notes                                        |
| ------------- | ------------------------------------- | ---------------------------------------- | -------------------------------------------- |
| **GPU**       | Graphics Processing Unit              | Real-time chat, RAG systems, fine-tuning | Best for transformer models                  |
| **TPU**       | Tensor Processing Unit (Google Cloud) | Deep learning training workloads         | Limited framework support outside TensorFlow |
| **CPU**       | Standard processors                   | Batch inference, quantized small models  | Cheaper but much slower                      |


**Tip:** Use **A100, H100, or L4** GPUs for production-grade hosting. For local dev/testing, **RTX 3080/3090/4090** is often enough. Always check model requirements for VRAM and compatibility.

---

## On-Prem vs. Cloud Hosting

| Strategy    | Pros                                           | Cons                                              |
| ----------- | ---------------------------------------------- | ------------------------------------------------- |
| **Cloud**   | Easy to scale, managed GPUs, global access     | Can be expensive long-term, dependent on provider |
| **On-Prem** | Full data control, potentially cheaper in bulk | Requires hardware, cooling, maintenance           |
| **Hybrid**  | Cloud for burst workloads, on-prem for base    | Requires orchestration + monitoring               |

Popular cloud GPU providers include:

* 🔸 **AWS EC2 / SageMaker**
* 🔸 **Google Cloud / Vertex AI**
* 🔸 **Azure ML**
* 🔸 **RunPod**, **Lambda Labs**, **Paperspace** *(budget-friendly for hobbyists)*

---

## Choosing a Model to Host

Here are some popular open-source LLMs:


| Model       | Size (params) | Highlights                               | License                       |
| ----------- | ------------- | ---------------------------------------- | ----------------------------- |
| **LLaMA 2** | 7B–70B        | Strong general performance, fine-tunable | Meta (non-commercial for now) |
| **Mistral** | 7B            | Very fast, efficient, great for RAG      | Apache 2.0                    |
| **Falcon**  | 7B–180B       | Good open weights, multilingual support  | Apache 2.0                    |
| **Gemma**   | 2B–7B         | Google-backed, performant and compact    | Apache 2.0                    |
| **Phi-2**   | 2.7B          | Extremely small yet surprisingly capable | MIT                           |
| **Qwen**    | 7B–72B        | Multilingual, strong performance         | Apache 2.0                    |
| **Mixtral** | 8x7B (MoE)    | Mixture-of-Experts, high throughput      | Apache 2.0                    |


Start with **Mistral-7B** or **Phi-2** if you're deploying on a single GPU. These models are light enough to run on a 24–32GB GPU and fast enough for real-time inference. For edge or offline use, quantized versions (e.g., GGUF, GPTQ) can run on CPUs or even mobile devices.

---

## Hosting Options Preview

We’ll go deeper in the next chapters, but here’s a preview of what’s ahead:


| Method                              | Tech Stack                      | Use Case                            |
| ----------------------------------- | ------------------------------- | ----------------------------------- |
| **SageMaker Endpoint**              | AWS, Docker, PyTorch/TensorFlow | Enterprise-grade model hosting      |
| **Google Vertex AI**                | TF/ONNX + managed services      | Auto-scaled inference endpoints     |
| **Hugging Face Inference Endpoint** | Transformers + Web UI           | Easiest deployment, lower control   |
| **FastAPI + Docker**                | Open source infra               | DIY local or cloud hosting          |
| **llama.cpp / GGUF**                | CPU/embedded devices            | Offline or edge chatbot experiences |
| **vLLM / TGI**                      | High-throughput serving         | Multi-user, production inference    |

---

## When Should You Self-Host?


Use the checklist below:

✅ You need full control over model behavior  
✅ Your app handles sensitive data (health, legal, finance)  
✅ You're serving millions of tokens per day  
✅ You want to train or fine-tune on proprietary datasets  
✅ You want to run inference offline or on-prem  
✅ You want to avoid vendor lock-in or API rate limits  

---

## Summary


Self-hosting an LLM transforms you from a consumer of AI to an **operator of intelligence**. You get freedom, privacy, and performance—but you pay for it in complexity. The learning curve is real, but so are the rewards for teams that need control and flexibility.

The rest of this part will walk you through how to **host your own LLM** step-by-step—on AWS, GCP, Hugging Face, or entirely from scratch using open-source tools. You'll learn how to make it secure, fast, and production-ready, with tips for monitoring, scaling, and cost management.

> *Next stop: building on managed cloud platforms — where convenience meets configurability.*

---

