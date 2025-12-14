---
hide:
    - toc
---

# Chapter 13: Fine-Tuning Your Own Models

> *"You can prompt a model to act smart. But when it needs to be fluent in your domain? That’s when you teach it."*


At some point, you’ll realize: no prompt is clever enough to fully overcome a model's limitations when it wasn’t trained for your context. If your chatbot has to speak like a lawyer, diagnose like a doctor, or respond like your company’s internal support rep, it’s time for **fine-tuning**.

This chapter teaches you how to **adapt a pretrained LLM**—like LLaMA, Mistral, or Falcon—to **specialized tasks and tones** using modern fine-tuning techniques. We'll focus especially on **LoRA** (Low-Rank Adaptation), the gold standard for low-cost, high-efficiency training, and its variants like QLoRA for memory efficiency.

Whether you're updating just the output layer, teaching a model your internal knowledge base, or aligning it with your brand’s voice, this is how you give your chatbot a true **voice of its own**.

---

## When Should You Fine-Tune?

✅ You want responses that match a **specific tone**, format, or persona  
✅ Your chatbot must operate in a **specialized domain** (legal, finance, medicine, etc.)  
✅ You have **structured Q\&A pairs, documents, or dialogues** for training  
✅ You want to reduce the need for heavy prompts at inference time  
✅ Prompt engineering **can’t reach** the level of fluency you need  

> If you’re mostly happy with the base model but want subtle shifts, consider *prompt tuning* or *embedding-based retrieval* (RAG). Otherwise—fine-tuning is the key.

---

## Fine-Tuning Methods Overview


| Method                 | Description                                | Use Case                               | Resource Need        |
| ---------------------- | ------------------------------------------ | -------------------------------------- | -------------------- |
| **LoRA**               | Injects learnable adapters into layers     | Most popular for open LLMs             | Moderate (1 GPU)     |
| **QLoRA**              | LoRA + 4-bit quantization                  | Memory-efficient fine-tuning           | Low (16GB GPU)       |
| **Full Fine-Tune**     | Retrains all model weights                 | Rare—used for large datasets           | Very high (A100s)    |
| **Instruction Tuning** | Fine-tunes with examples + instructions    | Great for chatbots                     | Common in OSS        |
| **PEFT**               | Parameter-Efficient Fine-Tuning (umbrella) | Includes LoRA, Adapters, Prefix Tuning | Modular & extensible |
| **Delta Tuning**       | Only updates a small subset of weights     | Fast, low-resource, quick experiments  | Very low             |

We’ll focus on **LoRA + QLoRA** with the **PEFT** library (Hugging Face), which allows you to fine-tune large models on **a single A100 or even a 3090**.

---

## Dataset Formats

| Format Type      | Example                                                   | Used In                   |
| ---------------- | --------------------------------------------------------- | ------------------------- |
| **Alpaca-style** | `{"instruction": "...", "input": "...", "output": "..."}` | Chat / Instruction tuning |
| **Plain Q\&A**   | `{"question": "...", "answer": "..."}`                    | FAQ bots, support bots    |
| **JSONL / CSV**  | Structured columns or key-value pairs                     | General fine-tuning       |
| **Dialogues**    | List of alternating user/assistant messages               | Multi-turn chatbots       |


Clean, well-formatted data is **more important than volume**. Even 1,000–10,000 examples of high-quality pairs can beat massive but noisy datasets. Always review, deduplicate, and balance your data for best results.

---

## Implementation (QLoRA + PEFT)

Let’s fine-tune `mistralai/Mistral-7B-Instruct-v0.1` on your data using Hugging Face tools.

### Install Dependencies

```bash
pip install transformers datasets accelerate peft bitsandbytes trl
```

### Load Model with 4-bit Quantization

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(load_in_4bit=True, bnb_4bit_compute_dtype="float16")
model = AutoModelForCausalLM.from_pretrained("mistralai/Mistral-7B-Instruct-v0.1", quantization_config=bnb_config, device_map="auto")
tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.1")
```

### Add LoRA Layers with PEFT

```python
from peft import prepare_model_for_kbit_training, LoraConfig, get_peft_model

model = prepare_model_for_kbit_training(model)

config = LoraConfig(
    r=8,
    lora_alpha=16,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, config)
```

### Load Dataset (Alpaca Format)

```python
from datasets import load_dataset

data = load_dataset("json", data_files="your_dataset.json")
```

### Training


Use `transformers.Trainer` or `trl.SFTTrainer` for simplicity. For more control, you can subclass these or use PyTorch Lightning.

```python
from transformers import TrainingArguments
from trl import SFTTrainer

training_args = TrainingArguments(
    output_dir="./mistral-finetuned",
    per_device_train_batch_size=4,
    num_train_epochs=3,
    learning_rate=2e-4,
    logging_steps=10,
    save_total_limit=2,
    save_strategy="epoch",
    bf16=True,
)

trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=data["train"],
    args=training_args,
)

trainer.train()
```

---

## Saving and Using the Model

After training:

```python
model.save_pretrained("./mistral-finetuned")
tokenizer.save_pretrained("./mistral-finetuned")
```


You can now deploy this using the same FastAPI/Docker method from [Chapter 12](chapter12.md#open-source-model-hosting-local-cloud), or convert to GGUF for CPU/edge inference.

---

## Common Pitfalls


| Issue                      | Solution                                            |
| -------------------------- | --------------------------------------------------- |
| OOM (Out of Memory) errors | Use QLoRA, smaller batch, or gradient checkpointing |
| Training but model forgets | Ensure consistent prompt formatting and padding     |
| Slow convergence           | Lower learning rate, cleaner data                   |
| Inference mismatch         | Match `prompt_template` in training and inference   |
| Overfitting                | Use validation set, early stopping, or regularization|
| Tokenizer mismatch         | Always use the same tokenizer for train/inference   |

---

## Summary


Fine-tuning is how you **embed your company’s brain** into an LLM. With LoRA and quantization, it’s now feasible to train on a laptop with a decent GPU or a low-cost cloud instance. You can steer tone, tighten reasoning, and build agents that go far beyond general-purpose capabilities. Fine-tuned models can be shared, versioned, and continually improved as your needs evolve.

> *Next: Once your model is trained, how do you make it faster, smaller, and cheaper to serve? Time for serious optimization.*

---