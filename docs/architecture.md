# Architecture: PDF Q&A Chatbot on Databricks

## 1. Overview

This project implements a Retrieval-Augmented Generation pipeline for answering questions from an uploaded PDF document.

The system runs fully in Databricks Free Edition and uses open-source Python libraries for PDF parsing, text chunking, embeddings, vector search, and answer generation.

The main goal of the project is to understand how a practical PDF-based Q&A chatbot works from end to end.

The chatbot can:

* Read an uploaded PDF
* Extract text from the document
* Split the text into smaller chunks
* Store the chunks in a Databricks table
* Convert chunks into embeddings
* Build a FAISS vector index
* Retrieve relevant chunks for a user question
* Generate an answer using a free Hugging Face model
* Return the answer with source chunk references

---

## 2. High-Level Architecture

```text
User uploads PDF
        ↓
Databricks Volume / Workspace storage
        ↓
PDF text extraction with pypdf
        ↓
Text chunking with LangChain
        ↓
Chunks saved to Databricks table
        ↓
Embeddings generated with Sentence Transformers
        ↓
FAISS vector index
        ↓
User asks a question
        ↓
Question converted into embedding
        ↓
Similarity search in FAISS
        ↓
Relevant PDF chunks retrieved
        ↓
Relevance score checked
        ↓
Prompt constructed with question + retrieved context
        ↓
Free Hugging Face FLAN-T5 model generates answer
        ↓
Answer returned with source chunks
```

---

## 3. Architecture Layers

This project can be understood in four main layers:

```text
1. Ingestion Layer
2. Processing Layer
3. Retrieval Layer
4. Answer Generation Layer
```

---

## 4. Ingestion Layer

The ingestion layer is responsible for making the PDF available inside Databricks.

### Input

The input is a PDF document uploaded to a Databricks Volume or workspace storage path.

Example path used in this project:

```text
/Volumes/workspace/365pdf/365pdfupload/Introduction_to_Tableau.pdf
```

### Purpose

The purpose of this layer is to provide a document source that can be processed by the pipeline.

### Current Implementation

The PDF is uploaded manually through the Databricks UI.

This keeps the project beginner-friendly and avoids any local laptop setup.

### Future Improvement

In a more advanced version, this layer could support:

* Multiple PDF uploads
* Automated file ingestion
* Cloud storage ingestion
* Metadata capture for each file
* Document versioning

---

## 5. Processing Layer

The processing layer converts the PDF into clean, searchable text chunks.

This layer includes:

```text
PDF loading
Text extraction
Text chunking
Metadata creation
Databricks table storage
```

---

## 6. PDF Text Extraction

The project uses the `pypdf` library to read the PDF file.

Each page is processed one by one, and the extracted text is combined into one larger text object.

### Output

The output of this step is plain text extracted from the PDF.

Example structure:

```text
--- Page 1 ---
Extracted text from page 1

--- Page 2 ---
Extracted text from page 2
```

### Why This Step Is Needed

PDFs are not directly searchable by a vector database or language model.

They must first be converted into text.

---

## 7. Text Chunking

After extracting the text, the document is split into smaller pieces called chunks.

The project uses LangChain's `RecursiveCharacterTextSplitter`.

### Chunking Configuration

```text
chunk_size = 800
chunk_overlap = 150
```

### What This Means

`chunk_size = 800` means each chunk is approximately 800 characters long.

`chunk_overlap = 150` means each chunk shares around 150 characters with the next chunk.

### Why Chunking Is Important

Chunking is one of the most important parts of RAG.

A full PDF can be too large to search efficiently or pass into a model prompt. Smaller chunks make retrieval more accurate.

### Why Overlap Is Used

Overlap helps preserve meaning across chunk boundaries.

For example, if an important explanation starts at the end of one chunk and continues in the next chunk, overlap reduces the chance that the context is split incorrectly.

---

## 8. Metadata Creation

Each chunk is stored with metadata.

Metadata used in this project:

```text
chunk_id
source
text
```

### Example

```text
chunk_id: 5
source: Introduction_to_Tableau.pdf
text: <chunk text>
```

### Why Metadata Matters

Metadata allows the chatbot to show where an answer came from.

This improves transparency and makes the output easier to validate.

---

## 9. Databricks Table Storage

The processed chunks are saved into a Databricks table.

Table name:

```text
pdf_rag_chunks
```

### Table Purpose

The table stores the chunked PDF content so it can be reused in later notebooks.

### Example Columns

```text
chunk_id
source
text
```

### Why Store Chunks in a Table

Saving chunks in a table makes the project more structured than a simple notebook-only implementation.

It also demonstrates how document data can be managed inside a lakehouse-style environment.

---

## 10. Retrieval Layer

The retrieval layer makes the PDF searchable by meaning.

This layer includes:

```text
Embedding generation
FAISS vector index creation
Question embedding
Similarity search
Relevance score checking
```

---

## 11. Embedding Generation

Each text chunk is converted into an embedding.

An embedding is a numerical representation of text.

Texts with similar meaning have embeddings that are close to each other in vector space.

### Embedding Model

This project uses:

```text
sentence-transformers/all-MiniLM-L6-v2
```

### Embedding Size

This model creates embeddings with 384 dimensions.

### Why Embeddings Are Needed

Embeddings allow semantic search.

This means the chatbot can find relevant chunks even when the user's question does not use the exact same words as the PDF.

Example:

```text
Question:
What is Tableau used for?

Relevant PDF text might say:
Tableau helps users create visual analytics and dashboards.
```

Even though the wording is different, embeddings help connect the meaning.

---

## 12. FAISS Vector Index

The project uses FAISS as the vector search index.

FAISS stores the embeddings and allows fast similarity search.

### FAISS Index Path

```text
/Volumes/workspace/365pdf/365pdfupload/faiss_intro_tableau_index
```

### Why FAISS Was Used

FAISS is:

* Free
* Open source
* Good for learning projects
* Easy to use with LangChain
* Suitable for small document collections

### Current Limitation

The FAISS index is saved to a workspace path.

For a production system, a managed vector database or Databricks Vector Search would be more appropriate.

---

## 13. User Question Processing

When the user asks a question, the question is also converted into an embedding using the same embedding model.

The system then compares the question embedding with the stored chunk embeddings.

This finds the chunks that are most semantically similar to the user's question.

---

## 14. Similarity Search

The chatbot retrieves the top matching chunks from FAISS.

Example configuration:

```text
k = 4
```

This means the system retrieves the top 4 most relevant chunks.

### Output

Each result includes:

```text
retrieved chunk text
chunk metadata
similarity score
```

---

## 15. Relevance Score Guardrail

Vector search will always return the closest chunks, even when the question is unrelated.

For example:

```text
What is the capital of Japan?
```

Even though this question is unrelated to a Tableau PDF, FAISS may still return the nearest available chunks.

To reduce this problem, the chatbot uses a relevance threshold.

### Threshold Used

```text
RELEVANCE_THRESHOLD = 1.3
```

### Logic

```text
If best score is lower than or equal to threshold:
    answer the question

If best score is higher than threshold:
    reject the question
```

### Rejection Message

```text
I could not find a confident answer in the uploaded PDF.
Please ask a question related to the document.
```

### Why This Is Important

This guardrail helps prevent the chatbot from answering questions that are not supported by the uploaded PDF.

---

## 16. Answer Generation Layer

The answer generation layer creates a clean natural-language response using the retrieved context.

This layer includes:

```text
Prompt construction
Free LLM-style generation
Answer formatting
Source display
```

---

## 17. Prompt Construction

The prompt combines:

```text
system instruction
retrieved context
user question
answer instruction
```

### Prompt Behavior

The prompt tells the model:

```text
Answer the question using only the context below.
If the answer is not in the context, say:
"I could not find this information in the uploaded PDF."
```

### Why Prompting Matters

The prompt helps keep the answer grounded in the retrieved PDF chunks.

Without this instruction, the model may use general knowledge instead of the uploaded document.

---

## 18. Free Hugging Face Model

The project uses a small open-source model from Hugging Face:

```text
google/flan-t5-small
```

### Why This Model Was Used

It was selected because:

* It is free
* It is small enough for a beginner project
* It can run in a notebook environment
* It avoids paid API usage
* It is suitable for simple question answering experiments

### Current Limitation

Because this is a small model, answers may be short or less detailed than responses from larger commercial LLMs.

---

## 19. Answer Output

The chatbot returns:

```text
generated answer
source file name
chunk IDs
retrieval scores
```

### Example Output

```text
Answer:
Tableau is used for data visualization and business intelligence. It helps users connect to data, explore information, and create dashboards.

Sources:
- Introduction_to_Tableau.pdf, chunk_id=2, score=0.8421
- Introduction_to_Tableau.pdf, chunk_id=5, score=0.9174
- Introduction_to_Tableau.pdf, chunk_id=7, score=1.0328
```

---

## 20. End-to-End Data Flow

```text
PDF file
   ↓
pypdf extracts text
   ↓
LangChain splits text into chunks
   ↓
Chunks stored in pdf_rag_chunks table
   ↓
Sentence Transformers converts chunks to embeddings
   ↓
FAISS stores vector embeddings
   ↓
User question is embedded
   ↓
FAISS retrieves similar chunks
   ↓
Threshold checks relevance
   ↓
Retrieved chunks are inserted into prompt
   ↓
FLAN-T5 generates answer
   ↓
Answer and sources are displayed
```

---

## 21. Current Implementation Summary

### Completed

* PDF uploaded to Databricks
* Text extracted from the PDF
* PDF text split into chunks
* Chunk metadata created
* Chunks saved to Databricks table
* Embeddings created using Sentence Transformers
* FAISS vector index created
* Semantic search tested
* Basic retrieval-based chatbot created
* Similarity score guardrail added
* Free Hugging Face model added
* Final answer returned with sources

### Not Yet Implemented

* Multi-PDF support
* Web application interface
* Chat history
* Production vector database
* RAG evaluation metrics
* User authentication
* API deployment
* Automated ingestion pipeline

---

## 22. Design Decisions

### Why Databricks?

Databricks was used because it provides a cloud-based notebook environment where the full project can be built without installing anything locally.

It also allows document chunks to be saved as tables, which makes the project closer to a real data platform workflow.

### Why FAISS?

FAISS was used because it is free, simple, and suitable for learning vector search.

### Why Hugging Face Instead of OpenAI?

The project avoids paid APIs.

Using a free Hugging Face model keeps the project cost-free and suitable for learning.

### Why Store Chunks in a Databricks Table?

Saving chunks to a table makes the project more organized and demonstrates how unstructured document data can be converted into structured, reusable data.

---

## 23. Production Architecture Upgrade Path

The current architecture is suitable for learning.

A more production-ready architecture could look like this:

```text
User uploads documents
        ↓
Cloud object storage
        ↓
Databricks Auto Loader
        ↓
Bronze document metadata table
        ↓
Silver parsed text table
        ↓
Gold chunk table
        ↓
Databricks Vector Search index
        ↓
RAG API endpoint
        ↓
Model Serving endpoint
        ↓
Web app chatbot interface
        ↓
Monitoring and evaluation
```

### Potential Production Additions

* Unity Catalog governance
* Databricks Volumes for document storage
* Databricks Vector Search
* MLflow experiment tracking
* Model Serving
* Lakehouse Monitoring
* CI/CD with GitHub Actions
* Secrets management
* Access control
* Audit logging
* Evaluation dataset
* Human feedback loop

---

## 24. Final Architecture Summary

This project demonstrates a complete beginner RAG architecture using Databricks and open-source tools.

It shows how a PDF can be transformed into a searchable knowledge base and used to power a question-answering chatbot.

The architecture is intentionally simple, free, and beginner-friendly, while still covering the core concepts used in real RAG systems:

```text
ingestion
extraction
chunking
embedding
indexing
retrieval
guardrailing
prompting
generation
source citation
```
