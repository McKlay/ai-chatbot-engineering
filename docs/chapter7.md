---
hide:
    - toc
---

# Chapter 7: Embeddings and Retrieval-Augmented Generation (RAG)

---

While prompts guide the LLM’s behavior, they alone can’t store long-term memory or domain-specific knowledge. What happens when your chatbot needs to reference company documents, FAQs, or proprietary knowledge? That’s where **RAG (Retrieval-Augmented Generation)** comes in.

This chapter explores how to give your chatbot memory—by using **embeddings**, **vector databases**, and a smart retrieval pipeline. You’ll learn how to embed documents, store them efficiently using `pgvector` in Supabase, and query relevant context at runtime to inject into your prompts.

---

## 7.1 What Is Retrieval-Augmented Generation (RAG)?

RAG is a hybrid technique combining information retrieval and language generation. The idea is simple:

> “Don’t make the model guess. Let it look up information first.”

### RAG Flow Overview:

1. **User sends a question.**
2. **Input is embedded** into a high-dimensional vector.
3. **Relevant documents are retrieved** via vector similarity search.
4. **Retrieved context is injected** into the prompt sent to the LLM.
5. **LLM responds** with an answer grounded in that context.

This creates a powerful feedback loop between storage and generation—minimizing hallucinations and enabling domain-specific accuracy.

---

## 7.2 Chunking: Preparing Text for Embedding

Before embedding your knowledge base, you must break it into digestible pieces. This is known as **chunking**.

### Common Chunking Strategies:

| Strategy                                | Pros                            | Cons                          |
| --------------------------------------- | ------------------------------- | ----------------------------- |
| Fixed-size (e.g. 500 tokens)            | Simple to implement             | May split sentences awkwardly |
| Sentence-based                          | Preserves grammatical structure | Varies in length              |
| Overlapping windows                     | Improves retrieval accuracy     | Increases storage size        |
| Recursive splitting (based on headings) | Hierarchical context            | More complex to implement     |

> 💡 Use a hybrid strategy: split by paragraph → add sliding window overlap of 10–20% for better semantic coverage.

---

## 7.3 Embedding with `text-embedding-3-small` (OpenAI)

Once you have chunks, each is converted into an embedding—a numerical vector representing its semantic meaning.

### Example Python Code (using OpenAI):

```python
from openai import OpenAI
import openai
openai.api_key = "your-api-key"

response = openai.embeddings.create(
    model="text-embedding-3-small",
    input="How to reset a printer to factory settings?"
)

embedding = response['data'][0]['embedding']
```

### Why `text-embedding-3-small`?

* Fast and affordable.
* High semantic performance.
* Compact vector size (1536 dims).

---

## 7.4 Setting Up Supabase with `pgvector`

Supabase is a developer-friendly PostgreSQL-as-a-service platform. When paired with the `pgvector` extension, it becomes a powerful vector store.

### Step-by-Step:

1. **Create a Supabase project.**

2. **Enable `pgvector` extension:**

   ```sql
   create extension if not exists vector;
   ```

3. **Create embeddings table:**

   ```sql
   create table documents (
     id uuid primary key,
     content text,
     embedding vector(1536)
   );
   ```

4. **Insert documents and embeddings:**

   ```sql
   insert into documents (id, content, embedding)
   values (
     gen_random_uuid(),
     'How to connect to WiFi?',
     '[0.021, 0.894, ...]'
   );
   ```

5. **Query by similarity:**

   ```sql
   select content
   from documents
   order by embedding <-> '[user_embedding]'
   limit 5;
   ```

> 📌 Use `embedding <-> vector` for cosine similarity in `pgvector`.

---

## 7.5 Vector Retrieval in the Backend

You’ll build a function that:

1. Embeds the user query.
2. Sends a similarity query to Supabase.
3. Returns the top-k matching chunks.

### Python Example:

```python
def retrieve_context(user_query: str):
    query_embedding = get_embedding(user_query)
    results = query_supabase_top_k(query_embedding)
    return "\n".join([r['content'] for r in results])
```

You can now inject this into the prompt:

```python
context = retrieve_context(user_input)
prompt = f"""
Use the following document context to answer the question.

[Start Context]
{context}
[End Context]

User: {user_input}
"""
```

---

## 7.6 Best Practices for RAG Bots

| Principle              | Recommendation                                              |
| ---------------------- | ----------------------------------------------------------- |
| Limit context length   | Keep injected context under 1500 tokens                     |
| Clean input            | Strip HTML, fix typos, remove irrelevant sections           |
| Cache embeddings       | Don’t re-embed the same query repeatedly                    |
| Handle missing results | Fallback to LLM-only response if retrieval fails            |
| Track provenance       | Show sources or titles alongside answers (for transparency) |

---

## 7.7 Debugging Retrieval Failures

When RAG doesn't perform as expected:

* Check if documents were properly chunked.
* Ensure correct vector dimensionality.
* Inspect similarity query—are the right chunks being returned?
* Log the actual injected prompt for inspection.

Use tools like Postgres query logs, vector visualizations (e.g., t-SNE), and similarity scoring to inspect RAG performance.

---

## Conclusion: Giving Your Chatbot a Brain

With RAG, your chatbot evolves from a generic assistant to a domain-aware knowledge bot. By embedding documents and retrieving relevant content during conversations, you build an **AI system with grounded knowledge, reduced hallucinations, and improved trustworthiness**.

In the next chapter, we’ll turn back to the frontend—bringing this intelligence to users with a sleek React-based chat interface and refined UX features.

---





