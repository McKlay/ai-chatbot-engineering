---
hide:
    - toc
---

# Chapter 11: Hosting Models on Cloud Platforms

> *“You don’t have to build the rocket—just rent the launchpad.”*


Self-hosting doesn’t always mean racking your own servers in a cold data center. In fact, **most developers start their self-hosting journey using cloud platforms**—where you get access to powerful GPUs, managed containers, autoscaling endpoints, and observability dashboards. Cloud hosting lets you focus on your application logic, while the provider handles hardware, scaling, and security.

This chapter will walk you through **three major cloud platforms** used for hosting LLMs:

* **Amazon SageMaker**
* **Google Vertex AI**
* **Azure Machine Learning**

Each platform has its own philosophy, pricing structure, and developer experience. We’ll compare them and then dive into hands-on hosting steps so you can launch a real chatbot backend from any of them. You’ll also learn how to secure endpoints and choose the right platform for your needs.

---

## When Should You Use Cloud Hosting?


Cloud platforms shine when:

* You want **scalable** LLM hosting with managed infrastructure and global reach.
* You need to integrate with cloud-native services (e.g., S3, Pub/Sub, Cloud Run, Azure Functions).
* You don't want to deal with GPU provisioning, driver updates, or model loading logistics.
* You want **endpoint-based inference** with SLAs, autoscaling, and built-in monitoring.

They’re perfect for MVPs, production deployments, and internal enterprise tools that require compliance, reliability, and rapid iteration. Cloud hosting also simplifies disaster recovery and multi-region deployments.

---

## Option 1: Hosting on AWS SageMaker


### 🔹 Key Features

* **Model Hosting as a Service** (inference endpoints with autoscaling and health checks)
* Integrated with **S3, CloudWatch, IAM** for storage, logging, and security
* Prebuilt **PyTorch, TensorFlow, Hugging Face containers** for easy deployment
* Offers **GPU** and **multi-model endpoints** (host several models on one endpoint)
* Built-in monitoring, logging, and scaling policies

### Basic Workflow

1. **Prepare your model**

   * Save as `.tar.gz` with `pytorch_model.bin`, `config.json`, etc.
2. **Upload to S3**
3. **Create a SageMaker Model**

   * Use prebuilt Hugging Face container image
4. **Deploy as Endpoint**

   * Set instance type (e.g., `ml.g5.xlarge` for single A10 GPU)
5. **Call via HTTPS API**

### Example

```python
from sagemaker.huggingface import HuggingFaceModel

model = HuggingFaceModel(
    model_data='s3://your-bucket/model.tar.gz',
    role='your-sagemaker-role',
    transformers_version='4.26',
    pytorch_version='1.13',
    py_version='py39'
)

predictor = model.deploy(instance_type='ml.g5.xlarge')
```


### Pros

* Best for **enterprise-scale** hosting and compliance
* Native support for **multiple models per endpoint**
* Deep **IAM + security integration** and audit trails
* Excellent documentation and community support

### Cons

* Can get **expensive** if not carefully managed (idle endpoints cost money)
* Steeper learning curve for setup and configuration
* Some features require AWS-specific knowledge

---

## Option 2: Hosting on Google Vertex AI


### 🔹 Key Features

* **Model Upload** and **Container-based Deployment** (bring your own container)
* Excellent for **TF/ONNX**, but supports PyTorch via custom containers
* Integrated with **BigQuery, GCS, Firebase, GKE** for data, storage, and orchestration
* Built-in **explainability**, **monitoring**, and **logging**
* Easy scaling and versioning of endpoints

### Basic Workflow

1. **Package Model**

   * SavedModel, ONNX, or PyTorch format
2. **Upload Model to Vertex**
3. **Create Endpoint**

   * Choose compute type (e.g., `n1-standard-4` + T4 GPU)
4. **Deploy Model**

   * Via Console or `gcloud` CLI

### Example (CLI)

```bash
gcloud ai models upload \
  --region=us-central1 \
  --display-name=mymodel \
  --artifact-uri=gs://your-model-dir \
  --container-image-uri=us-docker.pkg.dev/vertex-ai/prediction/pytorch-x.y:latest
```


### Pros

* **Tight GCP integration** (Cloud Run, Pub/Sub, BigQuery, etc.)
* **Cost-effective** for small-medium workloads and burst scaling
* Simplified UI for ML lifecycle and deployment
* Good support for explainability and monitoring

### Cons

* Less Hugging Face support out-of-the-box (workaround: custom containers)
* May require more **DevOps skill** for custom workflows and advanced scaling
* Some features are region-specific

---

## Option 3: Hosting on Azure ML


### 🔹 Key Features

* Supports **ML pipelines**, **AutoML**, and **LLMOps** (end-to-end ML lifecycle)
* Deployment via **ACI** (Azure Container Instances) or **AKS** (Kubernetes)
* Works well with **OpenAI on Azure** (hybrid solution for combining hosted and custom models)
* Deep integration with Azure Active Directory and enterprise security

### Basic Workflow

1. **Register Model**  
2. **Create Environment** (Python + PyTorch/HF)  
3. **Define Inference Script**  
4. **Deploy via CLI or SDK**  

### Example (Script)

```python
from azureml.core.model import Model
from azureml.core.webservice import AciWebservice, Webservice

model = Model.register(...)

inference_config = InferenceConfig(...)
deployment_config = AciWebservice.deploy_configuration(...)

service = Model.deploy(workspace, 'myservice', [model], inference_config, deployment_config)
```


### Pros

* Enterprise-grade security, **Active Directory integration**, and compliance
* Works well with **OpenAI hybrid models** (some on Azure, some self-hosted)
* Good for large organizations with existing Azure infrastructure

### Cons

* Verbose setup process and more steps for deployment
* May require **Azure-specific skills** and familiarity with Azure CLI/SDK
* Some features are only available in certain regions

---

## Pros & Cons Comparison Table


| Feature               | SageMaker             | Vertex AI           | Azure ML             |
| ----------------------| ----------------------|---------------------| ---------------------|
| LLM Container Support | Hugging Face, TF, PT  | Better for TF/ONNX  | Manual setup often   |
| Cost Management       | Fine-grained control  | Auto-scaling tiers  | Pay-as-you-go + AKS  |
| Autoscaling           | Yes (real-time)       | Yes                 | Yes                  |
| Security              | IAM, VPC              | IAM, VPC            | Azure AD, VNET       |
| Dev Experience        | CLI + SDK heavy       | UI + CLI friendly   | Heavier setup        |
| Multi-Model Support   | Yes                   | Not native          | Yes (AKS)            |
| Monitoring/Logging    | CloudWatch            | Stackdriver         | Azure Monitor        |

---

## Securing the Endpoints


Cloud providers help you enforce:

* **IAM-based access** (internal only, service accounts, role-based access)
* **Rate limits** and **quotas** to prevent abuse and control cost
* **HTTPS encryption** by default for all endpoints
* **Private VPC endpoints** for zero-exposure setups and compliance
* **Audit logging** for security and troubleshooting

> Pro tip: Always monitor your usage. Even idle endpoints can incur cost. Set up alerts for high usage or errors, and regularly review billing dashboards.

---

## When to Use Each


| Scenario                                          | Recommended Platform                   |
| ------------------------------------------------- | -------------------------------------- |
| Building enterprise chatbot for internal tools    | Azure ML (Active Directory)            |
| Public LLM-based app with real-time usage spikes  | AWS SageMaker                          |
| Startup-grade MVP or Google-native apps           | Google Vertex AI                       |
| You want full DevOps control over image/container | Use Docker + Render (see next chapter) |
| Need hybrid cloud/on-prem deployment              | Azure ML or custom Kubernetes          |

---

## Summary


Cloud platforms strike a powerful balance between **control and convenience**. You don’t need to worry about spinning up NVIDIA drivers or model loading quirks—just upload your model, deploy an endpoint, and call it from your chatbot. You get reliability, scalability, and security out of the box.

This chapter taught you the key workflows and tradeoffs of each major cloud platform. In the next chapter, we’ll **go deeper into open-source model hosting**—where you get even more flexibility by using your own FastAPI/Docker stack or tools like Hugging Face Inference Endpoints. You’ll learn how to optimize for cost, performance, and customization.

> *Ready to host your own brain—from scratch? Let's get building.*

---

