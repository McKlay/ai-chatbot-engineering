---
hide:
    - toc
---

# Chapter 6: Prompt Engineering Foundations

---

At the core of every effective chatbot powered by a Large Language Model lies the art and science of **prompt engineering**. Prompts are not mere input strings—they are carefully crafted instructions that guide the LLM's behavior, tone, and depth of reasoning. As you build ClayBot or any other chatbot, understanding how to design, structure, and optimize prompts is key to unlocking the full potential of your model.

This chapter explores prompt structure fundamentals, showcases real-world templates, and introduces advanced techniques like few-shot learning and Chain-of-Thought (CoT) prompting. You'll also learn how to debug and iterate prompts to improve output quality.

---

## 6.1 What Is a Prompt?

A prompt is the **input text** sent to a language model to elicit a response. For chatbots, prompts often simulate conversations through a sequence of roles and messages. They may include system instructions (guiding behavior), user queries, assistant replies, and retrieved content from RAG pipelines.

In essence, your prompt becomes the “brain” of the chatbot—defining not only *what* it says, but *how* it thinks.

---

## 6.2 Structure of a Chat-based Prompt

In OpenAI’s `chat/completions` format (used by `gpt-3.5-turbo` and `gpt-4`), prompts are structured as a **list of messages**, each with a role:

```json
[
  {"role": "system", "content": "You are a helpful assistant."},
  {"role": "user", "content": "What's the weather in Tokyo?"},
  {"role": "assistant", "content": "The weather in Tokyo is sunny and 25°C."}
]
```

### Prompt Roles:

* **System**: Sets the behavior and persona of the chatbot.
* **User**: Represents the end-user's input.
* **Assistant**: Represents the AI’s response.

> 💡 **Best Practice**: Think of the `system` message as "priming" the personality or rules of the assistant. Keep it concise but directive.

---

## 6.3 Crafting Effective System Prompts

System prompts can drastically influence tone, accuracy, and domain adherence. They’re essential in business chatbots for consistency, safety, and alignment with company voice.

### Examples:

**Customer Support Bot**

```text
You are a professional customer support agent for an e-commerce brand. Be polite, concise, and helpful. Answer based on the company’s return policy.
```

**Medical Assistant Bot**

```text
You are an AI medical assistant trained to provide general health information. Do not offer diagnoses or prescriptions. Advise users to consult a licensed medical professional.
```

**Friendly Tutor Bot**

```text
You are a friendly coding tutor helping students learn Python. Use simple examples, and encourage users to try code themselves.
```

---

## 6.4 Prompt Engineering Patterns

### 🔹 Zero-shot Prompting

No examples are provided. The model is expected to respond based only on instruction.

```text
Translate the following English sentence to Spanish: "I need to buy a ticket."
```

### 🔹 Few-shot Prompting

You provide examples to demonstrate the desired output format.

```text
Q: What’s the capital of France?
A: Paris

Q: What’s the capital of Japan?
A: Tokyo

Q: What’s the capital of Italy?
A:
```

### 🔹 Chain-of-Thought (CoT) Prompting

You encourage the model to "think aloud" by providing intermediate reasoning steps.

```text
Q: If Mary has 5 apples and gives 2 to John, how many apples does she have left?
A: First, Mary starts with 5 apples. She gives 2 away. 5 - 2 = 3. So the answer is 3.
```

> CoT is especially helpful for logical reasoning, multi-step math, or decision-making.

---

## 6.5 Prompting for RAG-Enhanced Bots

When using RAG (Retrieval-Augmented Generation), your prompt includes context fetched from vector search.

### Template with Context Injection:

```text
You are an assistant that answers based on the following document:

[Start of Document Context]
<insert retrieved content here>
[End of Document Context]

Question: <user question here>
Answer:
```

You can format the document section clearly using delimiters or markdown-style cues. This helps the model distinguish between the retrieved data and the user input.

> ⚠️ Avoid prompt bloat! The more tokens you inject, the higher the cost and the risk of truncation. Use summarization or token limit checks.

---

## 6.6 Techniques for Improving Prompt Quality

* ✅ **Be Specific**: Avoid vague prompts like “Help me.” Use: “Help me write a polite follow-up email to a client.”
* ✅ **Use Role-play**: Tell the model who it is and who it’s speaking to.
* ✅ **Control Style**: Use phrases like “Explain like I’m 10 years old” or “Use bullet points.”
* ✅ **Set Constraints**: Add rules like “Respond in under 100 words” or “Don’t mention external links.”
* ✅ **Iterate**: Test multiple variants and adjust based on actual outputs.

---

## 6.7 Debugging Prompt Failures

Even with good prompts, you may encounter failures such as:

* Hallucinations (confidently wrong answers).
* Inconsistent tone.
* Overly verbose or overly short answers.
* Ignoring instructions.

### Debugging Checklist:

* Is your system prompt too vague?
* Are you exceeding the context window?
* Can you simplify or reformat your examples?
* Are conflicting instructions being given?

Use logging on the backend to track prompt content vs. actual output, and experiment systematically.

---

## Conclusion: The Heartbeat of Every Chatbot

Prompt engineering is not a one-time setup—it’s an ongoing craft. Your prompts define your bot’s capabilities, tone, and reliability. Small changes can produce dramatically different outcomes.

By mastering prompt design and testing strategies, you’ve unlocked the most important layer of control over LLM-based bots. In the next chapter, we’ll go deeper into embedding memory and **RAG techniques**, enabling your bot to ground its answers in knowledge bases, documentation, or custom datasets.

---

