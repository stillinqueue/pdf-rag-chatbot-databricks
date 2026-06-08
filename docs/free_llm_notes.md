# Free LLM Notes

## Purpose

This project avoids paid APIs and uses a free open-source model for answer generation.

The goal is to learn the full Retrieval-Augmented Generation workflow without requiring:

* An OpenAI API key
* A paid model endpoint
* Local installation on a laptop
* Paid cloud infrastructure

The complete project runs inside Databricks Free Edition and uses open-source Python libraries.

---

## Model Used

The model used for answer generation is:

```text
google/flan-t5-small
```

This model is loaded through Hugging Face Transformers inside a Databricks notebook.

---

## Why FLAN-T5 Small Was Chosen

FLAN-T5 Small was selected because it is suitable for a beginner learning project.

Main reasons:

* It is free to use for learning.
* It is lightweight compared to larger open-source models.
* It can run in a notebook environment.
* It supports instruction-style prompts.
* It does not require paid API usage.
* It is simple enough to understand and debug.

This makes it a good option for demonstrating the basic RAG flow without adding cost or infrastructure complexity.

---

## How the Model Fits into the RAG Pipeline

The model is used only after the retrieval step.

The pipeline works like this:

```text
User question
        ↓
Question embedding
        ↓
FAISS similarity search
        ↓
Top relevant PDF chunks retrieved
        ↓
Retrieved chunks inserted into prompt
        ↓
FLAN-T5 generates answer
        ↓
Answer returned with sources
```

The model does not directly read the full PDF. It only receives the retrieved context chunks.

This is important because the answer is grounded in the uploaded document instead of relying only on the model's general knowledge.

---

## Prompt Design

The model receives a prompt that contains:

```text
retrieved PDF context
user question
answer instruction
```

Example prompt structure:

```text
You are a helpful PDF question-answering assistant.

Answer the question using only the context below.
If the answer is not in the context, say:
"I could not find this information in the uploaded PDF."

Context:
...

Question:
...

Answer:
```

This prompt is designed to reduce hallucination by instructing the model to answer only from the retrieved context.

---

## Important Implementation Note

The original Hugging Face `pipeline()` approach with the task name `text2text-generation` did not work in the Databricks environment used for this project.

The error showed that the installed runtime did not recognize the `text2text-generation` task.

Because of that, the project loads the model directly with:

```python
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM

model_id = "google/flan-t5-small"

tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForSeq2SeqLM.from_pretrained(model_id)

print("Free FLAN-T5 model loaded successfully")
```

This direct loading approach avoids the pipeline task compatibility issue.

---

## Answer Generation Helper Function

The project uses the following helper function to generate answers:

```python
def generate_answer_with_flan_t5(prompt, max_new_tokens=200):
    inputs = tokenizer(
        prompt,
        return_tensors="pt",
        truncation=True,
        max_length=1024
    )

    outputs = model.generate(
        **inputs,
        max_new_tokens=max_new_tokens
    )

    answer = tokenizer.decode(
        outputs[0],
        skip_special_tokens=True
    )

    return answer
```

This function:

1. Converts the prompt into model input tokens.
2. Truncates long prompts to fit within the model input limit.
3. Generates a response.
4. Converts the generated tokens back into readable text.

---

## RAG Function with Free LLM

The model is used inside the final RAG function after relevant chunks are retrieved from FAISS.

```python
RELEVANCE_THRESHOLD = 1.3

def ask_pdf_with_llm(question, k=4):
    results_with_scores = vector_store.similarity_search_with_score(
        query=question,
        k=k
    )

    if not results_with_scores:
        return "I could not find any relevant information in the uploaded PDF."

    best_doc, best_score = results_with_scores[0]

    print("Best retrieval score:", best_score)

    if best_score > RELEVANCE_THRESHOLD:
        return "I could not find a confident answer in the uploaded PDF. Please ask a question related to the document."

    context_parts = []
    source_lines = []

    for i, (doc, score) in enumerate(results_with_scores, start=1):
        context_parts.append(doc.page_content)
        source_lines.append(
            f"- {doc.metadata.get('source')}, chunk_id={doc.metadata.get('chunk_id')}, score={score:.4f}"
        )

    context = "\n\n".join(context_parts)
    context = context[:2500]

    prompt = build_prompt(question, context)

    answer = generate_answer_with_flan_t5(prompt)

    final_response = f"""
Answer:
{answer}

Sources:
{chr(10).join(source_lines)}
"""

    return final_response
```

---

## Relevance Threshold

The project uses a simple relevance threshold before sending context to the model.

```python
RELEVANCE_THRESHOLD = 1.3
```

FAISS returns similarity distance scores. In this project:

```text
Lower score = better match
Higher score = weaker match
```

If the best retrieved chunk has a score above the threshold, the chatbot refuses to answer.

This helps prevent unrelated questions from producing misleading answers.

Example unrelated question:

```text
What is the capital of Japan?
```

Expected response:

```text
I could not find a confident answer in the uploaded PDF.
Please ask a question related to the document.
```

---

## Benefits of This Approach

Using FLAN-T5 Small keeps the project:

* Free
* Cloud-based
* Beginner-friendly
* Independent of paid API keys
* Easy to run inside Databricks
* Suitable for learning how RAG works end to end

It also helps separate the main RAG components clearly:

```text
retrieval
prompting
generation
source display
```

---

## Limitations of FLAN-T5 Small

FLAN-T5 Small is useful for learning, but it has limitations.

Common limitations:

* Answers may be short.
* Reasoning quality is limited.
* It may not summarize long context perfectly.
* It may sometimes produce incomplete answers.
* It may copy wording from the retrieved context.
* It is not suitable for production-grade chatbot use.
* It may struggle with complex multi-step questions.

Because of these limitations, this model should be treated as a learning tool rather than an enterprise-ready assistant.

---

## Why Not Use OpenAI API?

The project intentionally avoids OpenAI API usage because the goal is to keep the project free.

Using OpenAI or another hosted LLM would likely improve answer quality, but it would require:

* API key setup
* Usage-based billing
* Secrets management
* Additional cost tracking

For this learning project, the priority is understanding the RAG workflow, not using the strongest possible model.

---

## Future Model Upgrade Options

Possible future upgrades include:

* A larger Hugging Face model
* Databricks Foundation Model APIs
* Databricks Model Serving
* OpenAI API
* Anthropic Claude API
* Meta Llama models
* Mistral models
* A locally hosted open-source model
* A managed enterprise LLM endpoint

These options can improve answer quality but may require more compute, paid access, or additional setup.

---

## Summary

This project uses `google/flan-t5-small` as a free answer generation model inside the RAG pipeline.

The model receives only the retrieved PDF context and the user question, then generates a short answer.

This approach is not production-grade, but it is effective for learning the main concepts behind a PDF Q&A chatbot:

```text
retrieve relevant chunks
build a grounded prompt
generate an answer
show sources
avoid paid APIs
```
