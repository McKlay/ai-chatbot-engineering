---
hide:
    - toc
---

# Chapter 23: Custom Tool Integration and Plugins

> “Give a bot a prompt, and it responds. Give it tools, and it becomes a system.”

## Introduction

LLMs are powerful generalists—but in the real world, users often need specialists. They don’t just want chatbots that **talk**; they want chatbots that **do**—fetch data, control services, browse knowledge bases, execute internal functions, or interact with their company's APIs.

This is where **custom tool integration** and **plugins** come in.

By equipping your chatbot with modular tools, you unlock new dimensions of capability. Your assistant becomes an **API orchestrator**, a **workflow trigger**, a **search engine**, a **calculator**, even a **backend command runner**—all driven by natural language.

In this chapter, we’ll explore how to build, connect, and secure tool-based extensions—from lightweight API bridges to OpenAI’s plugin framework. You’ll learn to give your bot hands, not just a mouth.

**Pro Tip:**
Design tools as stateless, idempotent functions with clear input/output schemas. This makes them easier to test, secure, and reuse across LLMs and agents.

---

## 23.1 What Are Tools and Plugins?

### Tools

These are **external capabilities** your chatbot can call via code, API, or SDK—e.g.,:

* Internal APIs (e.g., `get_employee_salary(id)`)
* Database queries
* Python functions (e.g., date diff, calculator)
* Web scraping utilities

### Plugins

These are **defined, declarative tool wrappers** registered within a plugin ecosystem (e.g., OpenAI plugins or LangChain tools). They follow a standardized format and can be dynamically loaded by LLMs.

**Plugin Example:**
OpenAI plugins use OpenAPI specs and a manifest file to describe capabilities. LangChain tools use Python decorators and type hints for registration.

---

## 23.2 When (and Why) to Use Tools

### 1. **Dynamic Data Access**

LLMs don’t know your current inventory or customer records. Tools fill the gap.

### 2. **Precision and Control**

LLMs hallucinate. Tools return exact results (from APIs, databases, logic).

* Use tools for all critical or regulated data access—never rely on LLMs alone for facts or calculations.

### 3. **Domain Specialization**

Offload niche tasks (e.g., weather lookup, currency conversion) to tools with consistent responses.

---

## 23.3 Common Use Cases for Tools

| Tool Type            | Example Use Cases                             |
| -------------------- | --------------------------------------------- |
| **Data Lookup**      | Product availability, HR records, student GPA |
| **Math/Utility**     | Date calculations, compound interest formula  |
| **Web/API Access**   | Crypto prices, GitHub issues, stock updates   |
| **Document Agents**  | Search inside uploaded PDF or Notion page     |
| **Function Calling** | Structured output via OpenAI function schemas |
| **Automation**       | Triggering workflows, creating Jira tickets   |

---

## 23.4 Implementing Custom Tools in a Chatbot Backend

Let’s say you want your chatbot to check the weather for a given city.

### 23.4.1 Step 1: Create the Tool Function (Python)

```python
import requests

def get_weather(city: str) -> str:
    api_key = "your_openweather_key"
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}&units=metric"
    data = requests.get(url).json()
    
    if data.get("main"):
        temp = data["main"]["temp"]
        return f"The temperature in {city} is {temp}°C."
    else:
        return "City not found."
```

**Security Note:**
- Store API keys in environment variables or secret managers, not in code.
- Add input validation to prevent injection or abuse (e.g., restrict city names to known values).

### 23.4.2 Step 2: Call Tool from LLM Pipeline

In OpenAI's function calling schema:

```python
functions = [
    {
        "name": "get_weather",
        "description": "Get current weather in a city",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {"type": "string"}
            },
            "required": ["city"]
        }
    }
]
```

**Tip:**
Log all tool invocations and responses for debugging and audit trails.

The LLM will detect user intent (“What’s the weather in Manila?”), call the tool, and weave the response into conversation.

---

## 23.5 Building an OpenAI Plugin (Standard Format)

### 23.5.1 Plugin Components

| File             | Purpose                                       |
| ---------------- | --------------------------------------------- |
| `ai-plugin.json` | Plugin manifest for OpenAI                    |
| `openapi.yaml`   | OpenAPI schema describing available endpoints |
| `logo.png`       | Branding for display in ChatGPT UI            |
| Hosted server    | Backend serving your APIs                     |

### 23.5.2 Sample `ai-plugin.json`

```json
{
  "schema_version": "v1",
  "name_for_model": "weather_tool",
  "name_for_human": "Weather Tool",
  "description_for_model": "Get current weather for any city",
  "auth": { "type": "none" },
  "api": {
    "type": "openapi",
    "url": "https://yourdomain.com/openapi.yaml"
  },
  "logo_url": "https://yourdomain.com/logo.png",
  "contact_email": "support@yourdomain.com"
}
```

**OpenAPI Schema Example:**
```yaml
openapi: 3.0.0
info:
    title: Weather Tool
    version: 1.0.0
paths:
    /weather:
        get:
            summary: Get weather for a city
            parameters:
                - in: query
                    name: city
                    schema:
                        type: string
                    required: true
            responses:
                '200':
                    description: Weather info
```

> Once uploaded and approved, users can install this plugin directly into ChatGPT.

---

## 23.6 Tool Integration with LangChain

LangChain makes tools first-class citizens.

### 23.6.1 Define a Tool

```python
from langchain.tools import tool

@tool
def get_joke():
    return "Why did the developer go broke? Because he used up all his cache."
```

### 23.6.2 Add Tool to Agent

```python
from langchain.agents import initialize_agent
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(temperature=0)
tools = [get_joke]

agent = initialize_agent(tools, llm, agent="zero-shot-react-description")
agent.run("Tell me a programming joke")
```

**Advanced:**
Use LangChain’s `Tool` class for more complex tools with input validation, async support, or multi-step workflows.

---

## 23.7 Multi-Tool Strategies

### 1. **Tool Router**

Route queries to the correct tool based on keywords or classification.

* Use intent classification or LLM-based routing to select the right tool for each query.

### 2. **Tool Chaining**

One tool calls another—e.g., PDF parser → keyword extractor → search API.

* Use dependency graphs or workflow engines for complex, multi-step tool chains.

### 3. **Memory + Tools**

Maintain session memory (e.g., user's name, preferences) to contextualize tool usage.

* Store tool results in session state for follow-up queries or undo actions.

---

## 23.8 UX Tips for Tool-Driven Chatbots

* Be transparent: “Let me check that for you…”
* Provide status/loading feedback for async tools.
* Allow users to *override* tool results if incorrect.
* Include “undo” or “cancel” options after tool-triggered actions.

* Show tool results with clear attribution (e.g., “According to OpenWeather…”).

---

## 23.9 Security Considerations

* **Rate limit** tool access to prevent abuse.
* **Validate inputs** strictly—never trust LLM-generated values blindly.
* **Scope permissions** if tools access private data or third-party services.
* **Audit logs**: track all tool calls and responses.

* **Timeouts**: Set reasonable timeouts for tool execution to avoid blocking the chat.
* **Error Handling**: Gracefully handle tool failures and inform the user.

---

## 23.10 Real-World Example: Notion Integration Plugin

Use Case: Your chatbot fetches meeting notes from Notion using a plugin.

### Workflow

1. User: “What were the key points from last Friday’s meeting?”
2. LLM identifies the intent → triggers `/notion/get_notes(date=…)`
3. Plugin returns notes from Notion API.
4. Chatbot summarizes and replies.

**Tech Stack**: FastAPI backend, Notion SDK, OpenAPI schema, ChatGPT plugin registration.

**Pattern:**
Use a plugin manifest and OpenAPI schema to make your tool discoverable and usable by LLMs and external agents.

---

## Conclusion

A chatbot with tools is no longer just an assistant—it’s a platform.

**Tool & Plugin Integration Checklist:**
- [ ] All tool functions are stateless and validated
- [ ] API keys and secrets managed securely
- [ ] Tool calls logged and auditable
- [ ] Plugins follow OpenAPI and manifest standards
- [ ] Rate limiting and error handling in place

Whether calling APIs, triggering automations, or fetching structured data, tool integrations enable your bot to operate within real-world systems. And with plugins, this capability becomes scalable, discoverable, and sharable across platforms.

In the next chapter, we’ll tie everything together into **business strategy**—showing how to turn your chatbot into a revenue-generating product through subscriptions, usage-based pricing, and SaaS models.

---