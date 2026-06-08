# RAG Pipeline Explanation

## Overview

This document explains the Retrieval-Augmented Generation pipeline used in this project.

The project allows a user to upload a PDF document and ask questions about the content of that document. The system retrieves relevant sections from the PDF and uses them to generate an answer.

This project is built for learning purposes using Databricks Free Edition, LangChain, FAISS, Sentence Transformers, and a free Hugging Face model.

---

## What is RAG?

RAG stands for Retrieval-Augmented Generation.

It combines two main steps:

1. **Retrieval**: Find relevant information from a document or knowledge base.
2. **Generation**: Use a language model to generate an answer from the retrieved information.

A normal language model answers from what it already learned during training. A RAG system adds an extra knowledge source, such as a PDF, database, website, or internal document collection.

In this project, the knowledge source is an uploaded PDF.

---

## Why RAG is Useful

Large language models do not automatically know the contents of a newly uploaded document.

For example, if a user uploads a PDF called:

```text
Introduction_to_Tableau.pdf
```

the model cannot answer detailed questions from that PDF unless the document content is first provided as context.

RAG solves this problem by retrieving the most relevant parts of the document and sending those parts to the model together with the user's question.

This makes the chatbot more grounded, more useful, and easier to inspect.

---

## High-Level Pipeline

```text
PDF Upload
    ↓
PDF Text Extraction
    ↓
Text Chunking
    ↓
Chunk Metadata Creation
    ↓
Databricks Table Storage
    ↓
Embedding Generation
    ↓
FAISS Vector Index Creation
    ↓
User Question
    ↓
Question Embedding
    ↓
Semantic Search
    ↓
Relevant Chunk Retrieval
    ↓
Relevance Score Check
    ↓
Prompt Construction
    ↓
Answer Generation
    ↓
Answer with Sources
```

---

## Step 1: Upload the PDF

The user uploads a PDF document into Databricks storage.

In this project, the sample course PDF was uploaded to a Databricks Volume path similar to:

```text
/Volumes/workspace/365pdf/365pdfupload/Introduction_to_Tableau.pdf
```

This keeps the file available inside Databricks notebooks without requiring anything to be installed locally.

---

## Step 2: Extract Text from the PDF

The PDF is read using the `pypdf` Python library.

Each page is processed one by one. If text is found on a page, it is added to a single combined text variable.

Example logic:

```python
from pypdf import PdfReader

reader = PdfReader(pdf_path)

all_text = ""

for page_number, page in enumerate(reader.pages, start=1):
    page_text = page.extract_text()

    if page_text:
        all_text += f"\n\n--- Page {page_number} ---\n"
        all_text += page_text
```

The result is one large text string containing the extracted content from the PDF.

---

## Step 3: Split Text into Chunks

A full PDF can be too large to search or send into a model at once.

To solve this, the extracted text is split into smaller overlapping chunks using LangChain's `RecursiveCharacterTextSplitter`.

The project uses:

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=800,
    chunk_overlap=150
)

chunks = text_splitter.split_text(all_text)
```

### Why Chunking Matters

Chunking is important because:

* It makes document search easier.
* It keeps retrieved context small enough for a model.
* It improves answer accuracy.
* It allows source chunks to be shown with the answer.

### Why Chunk Overlap Matters

Chunk overlap means part of one chunk is repeated in the next chunk.

Example:

```text
Chunk 1: characters 1 to 800
Chunk 2: characters 650 to 1450
Chunk 3: characters 1300 to 2100
```

This prevents important information from being lost at chunk boundaries.

---

## Step 4: Add Metadata to Each Chunk

Each chunk is stored with metadata.

Metadata helps trace answers back to the original document.

In this project, each chunk stores:

```text
chunk_id
source file name
text
```

Example:

```python
documents = []

for i, chunk in enumerate(chunks):
    documents.append({
        "chunk_id": i,
        "source": "Introduction_to_Tableau.pdf",
        "text": chunk
    })
```

This makes the answer more transparent because the chatbot can show which chunks were used.

---

## Step 5: Save Chunks to a Databricks Table

The chunks are saved to a Databricks table named:

```text
pdf_rag_chunks
```

This makes the chunked document reusable across different notebooks.

Example logic:

```python
from pyspark.sql import Row

rows = [
    Row(
        chunk_id=doc["chunk_id"],
        source=doc["source"],
        text=doc["text"]
    )
    for doc in documents
]

chunks_df = spark.createDataFrame(rows)

chunks_df.write.mode("overwrite").saveAsTable("pdf_rag_chunks")
```

After this step, the PDF content is stored in a structured format.

---

## Step 6: Create Embeddings

Embeddings convert text into numbers.

Each text chunk is transformed into a vector that represents its meaning.

This project uses the free embedding model:

```text
sentence-transformers/all-MiniLM-L6-v2
```

Example:

```python
from langchain_community.embeddings import HuggingFaceEmbeddings

embedding_model = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"
)
```

When one chunk is embedded, it becomes a vector of numbers.

For this model, each vector has 384 dimensions.

---

## Step 7: Build the FAISS Vector Index

After creating embeddings, the vectors are stored in FAISS.

FAISS is used for fast similarity search.

Example:

```python
from langchain_community.vectorstores import FAISS

vector_store = FAISS.from_texts(
    texts=texts,
    embedding=embedding_model,
    metadatas=metadata
)
```

The FAISS index is then saved to Databricks storage:

```python
faiss_index_path = "/Volumes/workspace/365pdf/365pdfupload/faiss_intro_tableau_index"

vector_store.save_local(faiss_index_path)
```

This allows the index to be loaded later in a chatbot notebook.

---

## Step 8: Ask a User Question

When the user asks a question, the system does not search using simple keyword matching.

Instead, the question is converted into an embedding using the same embedding model.

Example question:

```text
What is Tableau used for?
```

The question embedding is compared against the PDF chunk embeddings.

---

## Step 9: Retrieve Relevant Chunks

FAISS returns the chunks that are most similar to the user question.

Example:

```python
results_with_scores = vector_store.similarity_search_with_score(
    query=question,
    k=4
)
```

The value `k=4` means the system retrieves the top 4 most relevant chunks.

Each result contains:

```text
document chunk
metadata
similarity score
```

---

## Step 10: Check Relevance Score

Vector search will always return the nearest chunks, even when the question is unrelated to the PDF.

For example:

```text
What is the capital of Japan?
```

Even though this question is unrelated to a Tableau PDF, FAISS may still return some chunks because it always finds the closest match.

To avoid giving bad answers, the project uses a relevance threshold.

Example:

```python
RELEVANCE_THRESHOLD = 1.3
```

For FAISS distance scores:

```text
Lower score = better match
Higher score = weaker match
```

If the best score is worse than the threshold, the chatbot responds:

```text
I could not find a confident answer in the uploaded PDF.
Please ask a question related to the document.
```

This is a basic retrieval guardrail.

---

## Step 11: Build the RAG Prompt

After relevant chunks are retrieved, they are combined into a context block.

The prompt contains:

```text
instruction
retrieved context
user question
answer placeholder
```

Example:

```python
def build_prompt(question, context):
    prompt = f"""
You are a helpful PDF question-answering assistant.

Answer the question using only the context below.
If the answer is not in the context, say:
"I could not find this information in the uploaded PDF."

Context:
{context}

Question:
{question}

Answer:
"""
    return prompt
```

The key instruction is:

```text
Answer the question using only the context below.
```

This helps keep the answer grounded in the uploaded PDF.

---

## Step 12: Generate the Answer

A free Hugging Face model is used to generate the final answer.

This project uses:

```text
google/flan-t5-small
```

The model is loaded directly using Hugging Face Transformers:

```python
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM

model_id = "google/flan-t5-small"

tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForSeq2SeqLM.from_pretrained(model_id)
```

The prompt is tokenized and passed to the model:

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

This creates a cleaner answer instead of returning only raw document chunks.

---

## Step 13: Return Answer with Sources

The final response includes:

1. The generated answer.
2. The source file name.
3. The chunk IDs used.
4. The similarity scores.

Example output:

```text
Answer:
Tableau is used for data visualization and business intelligence. It helps users connect to data, explore data, and create visual dashboards.

Sources:
- Introduction_to_Tableau.pdf, chunk_id=2, score=0.8421
- Introduction_to_Tableau.pdf, chunk_id=5, score=0.9174
- Introduction_to_Tableau.pdf, chunk_id=7, score=1.0328
```

Showing sources makes the chatbot more trustworthy and easier to debug.

---

## Complete RAG Flow in Simple Words

The system works like this:

```text
Take the PDF
Break it into smaller pieces
Turn each piece into numbers
Store those numbers in a searchable index
Turn the user's question into numbers
Find the most similar PDF pieces
Check whether the match is good enough
Send the question and matching pieces to a model
Generate an answer
Show the answer and sources
```

---

## What Makes This a RAG Project?

This project is a RAG project because the answer generation is supported by retrieved context.

Without RAG:

```text
Question → Model → Answer
```

With RAG:

```text
Question → Retrieve document chunks → Model → Answer grounded in document
```

The retrieval step is what makes the chatbot document-aware.

---

## Current Implementation

The current implementation includes:

* PDF upload in Databricks
* Text extraction with `pypdf`
* Chunking with LangChain
* Chunk storage in a Databricks table
* Embeddings using Sentence Transformers
* FAISS vector search
* Similarity score filtering
* Prompt creation
* Answer generation using FLAN-T5
* Source display

---

## Limitations

This version is designed for learning, so it has some limitations:

* It currently focuses on one uploaded PDF.
* The FAISS index is stored in a Databricks workspace path.
* The free FLAN-T5 small model may produce short answers.
* There is no web application interface yet.
* There is no chat memory yet.
* There is no formal RAG evaluation yet.
* There is no production authentication or access control.
* The relevance threshold may need tuning for different documents.

---

## Possible Improvements

Future improvements could include:

* Support for multiple PDF uploads
* Better metadata, such as page numbers
* A user interface with Streamlit or Gradio
* Databricks Vector Search
* MLflow experiment tracking
* RAG evaluation metrics
* Better open-source LLM
* Chat history
* More advanced prompt templates
* Document filters
* API deployment
* Unity Catalog governance

---

## Key Learning Outcomes

By building this project, the following concepts were practiced:

* How PDF documents are loaded into a RAG system
* Why chunking is needed
* Why chunk overlap is useful
* What embeddings are
* How semantic search works
* How FAISS stores and searches vectors
* Why retrieval scores matter
* How to reduce unrelated answers
* How prompts connect retrieved context with a model
* How a free LLM can generate answers from retrieved context
* Why sources are important in RAG systems

---

## Final Summary

This project demonstrates the full beginner RAG pipeline on Databricks.

It starts with a PDF document and ends with a chatbot that can answer questions using retrieved content from that PDF.

The project is intentionally simple, free, and cloud-based so that the main focus stays on understanding the RAG workflow from end to end.
