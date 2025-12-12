---
hide:
    - toc
---

# Chapter 12: Open-Source Model Hosting (Local & Cloud)

> *"Cloud platforms give you the road—but open-source tools let you build your own vehicle."*


By now, you’ve seen how to host models using the big three clouds. But what if you want **complete flexibility**? What if you want to host **LLaMA**, **Mistral**, or **Falcon** yourself—without needing SageMaker, Vertex, or Azure?

This chapter is about full control. We’ll explore **open-source model hosting**, both locally and in the cloud. You’ll learn how to:

* Load and serve LLMs using Hugging Face Transformers or llama.cpp
* Deploy them via **FastAPI**, **Docker**, and **GPU instances**
* Use hosted solutions like Hugging Face Inference Endpoints for quick API access
* Optimize for latency and memory with tools like `bitsandbytes`, quantization, and GGUF
* Integrate with RAG, moderation, and custom pipelines

Whether you’re running on your own GPU, a cloud VM, or even a Raspberry Pi, this chapter is your path to **independent, scalable chatbot infrastructure**.

---

## Why Go Open-Source?


| Motivation           | Description                                                                          |
| -------------------- | ------------------------------------------------------------------------------------ |
| **Freedom**       | No API limits. No vendor lock-in. Your model, your rules.                            |
| **Transparency**  | See what the model is doing—inspect weights, logits, activations, and logs.          |
| **Customization** | Fine-tune, quantize, or layer with tools like RAG, rerankers, or moderation filters. |
| **Cost Control**  | Ideal for high-volume, edge, or offline deployments without ongoing API usage charges. |
| **Offline/Edge**  | Run LLMs in air-gapped, private, or embedded environments.                           |

---

## Hosting Methods

| Method                              | Deployment Type  | Skill Level  | Use Case                      |
| ----------------------------------- | ---------------- | ------------ | ----------------------------- |

| Hugging Face Inference Endpoints | Cloud-managed    | Beginner     | Fast, no infra setup          |
| Docker + FastAPI                 | Cloud or Local   | Intermediate | Production, full control      |
| llama.cpp + GGUF                 | CPU / Embedded   | Advanced     | On-device, offline inference  |
| vLLM + OpenLLM                   | Optimized Cloud  | Advanced     | Multi-model, high-concurrency |
| Text Generation WebUI            | Local (UI-based) | Beginner     | Testing, demos                |
| LM Studio / Ollama               | Desktop / Local  | Beginner     | GUI for local LLMs            |

---

## 🔹 Option 1: Hugging Face Inference Endpoints

This is the **simplest way** to host an open-source model:

* Choose a model on 🤗 [Hugging Face Hub](https://huggingface.co/models)
* Click "Deploy → Inference Endpoint"
* Select your region and instance type
* Get a **ready-to-use API URL**

### Sample API Call

```python
import requests

API_URL = "https://api-inference.huggingface.co/models/mistralai/Mistral-7B-Instruct-v0.1"
headers = {"Authorization": f"Bearer YOUR_HF_TOKEN"}

response = requests.post(API_URL, json={"inputs": "What is retrieval-augmented generation?"}, headers=headers)
print(response.json())
```

**Pros**

* Zero infra setup
* Supports private models
* Auto-scaling and HTTPS


**Cons**

* Limited concurrency (unless upgraded)
* Usage-based pricing after free tier
* Less control over hardware and scaling

---

## 🔹 Option 2: Docker + FastAPI (Custom Hosting)

Want full control over your endpoint? Build it yourself.

### Folder Structure

```bash
llm-server/
├── app/
│   ├── main.py
│   └── model_loader.py
├── Dockerfile
├── requirements.txt
```

### `model_loader.py`

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

MODEL_NAME = "TheBloke/Mistral-7B-Instruct-v0.1-GGUF"

tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME)
model = AutoModelForCausalLM.from_pretrained(MODEL_NAME, torch_dtype=torch.float16).to("cuda")
```

### `main.py`

```python
from fastapi import FastAPI, Request
from app.model_loader import model, tokenizer
import torch

app = FastAPI()

@app.post("/chat")
async def generate_response(request: Request):
    data = await request.json()
    prompt = data.get("prompt", "")
    inputs = tokenizer(prompt, return_tensors="pt").to("cuda")
    outputs = model.generate(**inputs, max_new_tokens=100)
    response = tokenizer.decode(outputs[0], skip_special_tokens=True)
    return {"response": response}
```

### `Dockerfile`

```Dockerfile
FROM pytorch/pytorch:2.0.0-cuda11.7-cudnn8-runtime

RUN pip install fastapi uvicorn transformers accelerate

COPY ./app /app
WORKDIR /app

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Run Locally

```bash
docker build -t llm-api .
docker run --gpus all -p 8000:8000 llm-api
```

**Pros**

* Full access to weights, prompt logic, memory control
* Can plug into vector search (RAG), moderation, reranking


**Cons**

* Requires GPU or paid cloud VM
* Manual optimization needed for concurrency and scaling
* You manage security, monitoring, and updates

---

## 🔹 Option 3: `llama.cpp` and GGUF for CPU Inference

Need **offline, lightweight LLM inference**? Use `llama.cpp`.

* Convert model to **GGUF** format (optimized for quantization)
* Run on CPU (MacBook, Raspberry Pi, Jetson Nano)
* Blazing fast with quantized (e.g., Q4\_0) versions


Tools like **text-generation-webui**, **LM Studio**, **Ollama**, and **llama-cpp-python** let you integrate these into your Python backend, desktop app, or web UI. Many support chat templates, quantization, and custom pipelines.

Ideal for:

* Embedded AI
* Private air-gapped deployments
* Running LLMs without GPUs

---

## Tips for Choosing a Model


| Model          | Ideal Use Case                                     |
| -------------- | -------------------------------------------------- |
| **Mistral-7B** | General-purpose RAG/chatbots (low latency)         |
| **LLaMA 2**    | Research, fine-tuning, secure deployments          |
| **Gemma**      | Efficient small models (2B/7B) with Apache license |
| **Phi-2**      | Edge-friendly, great for experimentation           |
| **Zephyr**     | Chat-finetuned small models (OpenChat, Alpaca)     |
| **Qwen**       | Multilingual, strong performance                   |
| **Mixtral**    | High throughput, Mixture-of-Experts                |

---

## Quantization Options

| Type  | Description                         | Benefit          |
| ----- | ----------------------------------- | ---------------- |
| 8-bit | Slight speedup, some memory savings | Good accuracy    |
| 4-bit | Massive memory reduction            | Great for CPUs   |
| GPTQ  | Post-training quantization          | Lower disk size  |
| AWQ   | Activation-aware quantization       | Better inference |
| GGUF  | Format for CPU-friendly LLMs        | llama.cpp ready  |


Use Hugging Face model cards to find GGUF or GPTQ versions of open-source models. Quantized models can dramatically reduce memory and hardware requirements, enabling LLMs on laptops or edge devices.

---

## Practical Deployment Options


| Deployment Method       | Use Case                 | Stack                 |
| ----------------------- | ------------------------ | --------------------- |
| **Render + Docker**     | Cheap GPU cloud host     | FastAPI + Docker      |
| **AWS EC2 Spot + tmux** | Cost-optimized inference | Python + HF + LLaMA   |
| **Paperspace / Lambda** | Dev testing and demos    | GPU Jupyter notebooks |
| **Hugging Face Space**  | Public chatbot frontend  | Gradio + Transformers |
| **Ollama / LM Studio**  | Local desktop chatbots   | GUI + local LLMs      |

---

## Summary


Open-source model hosting gives you **maximum control** and **cost-efficiency**, but it demands technical ownership. Whether you use Hugging Face’s one-click endpoints or build your own GPU-powered API with FastAPI and Docker, this path lets you customize every layer—tokenization, decoding, post-processing, and beyond. You can integrate RAG, moderation, analytics, and even build your own playground UI.

> *In the next chapter, we’ll go even further—customizing these open-source models through fine-tuning to specialize them for your domain.*

---
