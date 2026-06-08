# PDF Q&A Chatbot with LangChain, FAISS, Hugging Face, and Databricks

## Project Overview

This project builds a beginner-friendly Retrieval-Augmented Generation system that allows a user to upload a PDF document and ask questions about its content.

The project was built fully in the cloud using Databricks Free Edition and GitHub. No local installation was required.

The chatbot reads an uploaded PDF, extracts the text, splits the document into smaller chunks, creates vector embeddings, stores the chunks in a FAISS vector index, retrieves the most relevant chunks for a user question, and generates an answer using a free Hugging Face model.

The goal of this project is to learn how a practical Q&A chatbot works using the core ideas behind LangChain and RAG.

---

## Project Versions

This repository contains two implementations of the PDF Q&A chatbot.

### Version 1: Free Open-Source RAG Implementation

Located in:

```text
notebooks/

This version uses:

Hugging Face embeddings
FAISS vector search
Free FLAN-T5 answer generation
Retrieval scoring and guardrails

It was built to avoid paid APIs and demonstrate the RAG workflow using free tools.

Version 2: OpenAI, Chroma, LCEL, and Streaming Implementation

Located in:

notebooks_v2_openAI/

This version uses:

OpenAI embeddings
GPT-based text formatting
Token-based splitting
Chroma vector database
MMR retrieval
LangChain prompt templates
LCEL chain composition
Source-aware answers
Streaming chatbot output

See the dedicated V2 README here:

notebooks_v2_openAI/README.md



## Problem Statement

Many documents contain useful information, but reading through the entire document manually can be time-consuming.

This project solves that problem by creating a simple PDF assistant that can answer questions from an uploaded document.

Example questions:

```text
What is Tableau used for?
What are dimensions and measures in Tableau?
How can Tableau help with data visualization?
```

The chatbot answers only from the uploaded PDF and also shows the source chunks used to generate the answer.

---

## Architecture

```text
User uploads PDF
        ↓
PDF text extraction
        ↓
Text chunking
        ↓
Save chunks to Databricks table
        ↓
Create embeddings
        ↓
Store vectors in FAISS
        ↓
User asks a question
        ↓
Retrieve relevant chunks
        ↓
Apply relevance threshold
        ↓
Generate answer with free Hugging Face model
        ↓
Return answer with sources
```

---

## Tools and Technologies

| Tool                      | Purpose                                    |
| ------------------------- | ------------------------------------------ |
| Databricks Free Edition   | Cloud notebook environment                 |
| GitHub                    | Version control and project repository     |
| Python                    | Main programming language                  |
| PySpark                   | Saving and querying document chunks        |
| pypdf                     | PDF text extraction                        |
| LangChain                 | Text splitting and RAG workflow components |
| sentence-transformers     | Free embedding model                       |
| FAISS                     | Local vector search index                  |
| Hugging Face Transformers | Free LLM-style answer generation           |
| FLAN-T5                   | Small open-source text generation model    |

---

## Project Structure

```text
pdf-rag-chatbot-databricks/
│
├── README.md
├── notebooks/
│   ├── 01_pdf_upload_and_text_extraction
│   ├── 02_embeddings_and_vector_search
│   ├── 03_pdf_qa_chatbot
│   ├── 04_pdf_qa_chatbot_with_scores
│   └── 05_pdf_qa_with_free_llm
│
├── src/
│   └── .gitkeep
│
├── docs/
│   └── .gitkeep
│
└── data/
    └── .gitkeep
```

---

## Notebook Summary

### 01_pdf_upload_and_text_extraction

This notebook loads the uploaded PDF from Databricks storage and extracts text from each page.

Main tasks:

* Define the PDF path
* Load the PDF using `pypdf`
* Extract text from all pages
* Preview the extracted content
* Split the PDF text into chunks
* Save the chunks into a Databricks table

Output table:

```text
pdf_rag_chunks
```

---

### 02_embeddings_and_vector_search

This notebook converts the PDF chunks into embeddings and builds a FAISS vector index.

Main tasks:

* Read chunks from the `pdf_rag_chunks` table
* Load the free embedding model
* Convert text chunks into vectors
* Store vectors in FAISS
* Test semantic search
* Save the FAISS index for later use

Embedding model used:

```text
sentence-transformers/all-MiniLM-L6-v2
```

FAISS index path used:

```text
/Volumes/workspace/365pdf/365pdfupload/faiss_intro_tableau_index
```

---

### 03_pdf_qa_chatbot

This notebook creates the first simple PDF Q&A chatbot.

Main tasks:

* Load the saved FAISS index
* Create a retriever
* Ask questions against the PDF
* Return the most relevant chunks as the answer
* Show source metadata

This version is retrieval-based and does not use an LLM yet.

---

### 04_pdf_qa_chatbot_with_scores

This notebook improves the chatbot by adding relevance scores.

Main tasks:

* Retrieve chunks with FAISS similarity scores
* Compare the best score against a relevance threshold
* Reject unrelated questions
* Return only confident answers from the PDF

This prevents the chatbot from answering unrelated questions such as:

```text
What is the capital of Japan?
```

If the question is not related to the uploaded PDF, the chatbot responds:

```text
I could not find a confident answer in the uploaded PDF.
Please ask a question related to the document.
```

---

### 05_pdf_qa_with_free_llm

This notebook adds a free Hugging Face model to generate cleaner answers.

Main tasks:

* Load the FAISS vector index
* Retrieve the most relevant PDF chunks
* Build a RAG prompt
* Generate an answer using FLAN-T5
* Return the final answer with sources

Model used:

```text
google/flan-t5-small
```

This version creates a more natural Q&A experience while still grounding the answer in the uploaded PDF.

---

## RAG Workflow Explained

RAG stands for Retrieval-Augmented Generation.

In this project, RAG works as follows:

1. The PDF is converted into text.
2. The text is split into smaller chunks.
3. Each chunk is converted into a numerical vector called an embedding.
4. The embeddings are stored in a FAISS vector index.
5. When a user asks a question, the question is also converted into an embedding.
6. FAISS finds the most similar document chunks.
7. The retrieved chunks are passed into a prompt.
8. The model generates an answer using the retrieved context.

This helps the chatbot answer based on the uploaded PDF instead of relying only on the model's general knowledge.

---

## Example Questions

```text
What is Tableau used for?
```

```text
What are dimensions and measures in Tableau?
```

```text
How can Tableau help with data visualization?
```

```text
How do you create a visualization in Tableau?
```

---

## Example Output Format

```text
Answer:
Tableau is used for data visualization and business intelligence. It helps users connect to data, explore information, and create visual dashboards.

Sources:
- Introduction_to_Tableau.pdf, chunk_id=2, score=0.8421
- Introduction_to_Tableau.pdf, chunk_id=5, score=0.9174
- Introduction_to_Tableau.pdf, chunk_id=7, score=1.0328
```

---

## Key Concepts Learned

This project demonstrates the following concepts:

* PDF ingestion
* Text extraction
* Document chunking
* Chunk overlap
* Embeddings
* Vector search
* Semantic retrieval
* FAISS indexing
* Retrieval guardrails
* Similarity scoring
* Prompt construction
* Hugging Face model usage
* RAG-based Q&A
* Databricks notebook development
* GitHub version control

---

## Why Databricks Was Used

Databricks was used as the cloud development environment for this project.

Benefits:

* No local setup required
* Browser-based notebook development
* Ability to store document chunks as tables
* Easy experimentation with Python, PySpark, and ML libraries
* GitHub integration through Databricks Git folders
* Suitable for learning lakehouse and AI workflows together

---

## Cost Consideration

This project was designed to avoid paid APIs.

Free components used:

* Databricks Free Edition
* Open-source Python libraries
* Free embedding model from Sentence Transformers
* FAISS vector search
* Hugging Face FLAN-T5 small model

No OpenAI API key was required for the current implementation.

---

## Limitations

This is a learning project and has some limitations:

* The chatbot currently works with one uploaded PDF at a time.
* FAISS index storage is local to the Databricks workspace path.
* The free FLAN-T5 small model may produce short or simple answers.
* It is not deployed as a web application yet.
* The chatbot does not currently support user authentication.
* The project does not yet include a production-grade vector database.
* Evaluation metrics for answer quality have not yet been added.

---

## Future Improvements

Possible next steps:

* Add support for multiple PDFs
* Store document metadata in a managed Delta table
* Add a Streamlit or Gradio user interface
* Add a Databricks app or dashboard interface
* Use Databricks Vector Search if available
* Add MLflow tracking for RAG experiments
* Add RAG evaluation metrics
* Add document-level filters
* Add chat history
* Add better prompt templates
* Add a stronger open-source LLM
* Deploy the chatbot as an API endpoint

---

## Current Status

Completed:

* PDF uploaded to Databricks
* Text extracted from PDF
* Text split into chunks
* Chunks saved to Databricks table
* Embeddings created
* FAISS vector index built
* Semantic search tested
* Basic PDF Q&A chatbot created
* Relevance scoring added
* Unrelated-question guardrail added
* Free Hugging Face answer generation added

---

## Project Goal

The goal of this project is not only to build a chatbot, but to understand the full RAG pipeline from scratch.

This project shows how a PDF document can be transformed into a searchable knowledge base and used to power a Q&A assistant using open-source tools and Databricks.
