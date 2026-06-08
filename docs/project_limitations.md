# Project Limitations and Future Improvements

## Overview

This project is designed as a beginner-friendly learning implementation of a PDF Q&A chatbot using Retrieval-Augmented Generation.

It successfully demonstrates the full pipeline from PDF upload to answer generation, but it is not yet a production system.

The project focuses on learning the core RAG building blocks:

```text
PDF ingestion
text extraction
chunking
embedding
vector indexing
retrieval
guardrails
prompting
answer generation
source display
```

---

## Current Limitations

### 1. Single PDF Focus

The current implementation is designed around one uploaded PDF document.

The example document used is:

```text
Introduction_to_Tableau.pdf
```

To support multiple documents, the project would need stronger document management logic, including:

* Unique document IDs
* File-level metadata
* Upload timestamps
* Document filters
* User-selected document search
* Separate indexes or a shared multi-document index

---

### 2. Local FAISS Index

The project uses a FAISS index saved to a Databricks workspace or volume path.

This is suitable for learning, but it is not ideal for production because:

* It is not a managed vector database.
* It is not automatically scalable.
* It does not provide advanced governance features.
* It may need to be rebuilt when documents change.
* It does not provide built-in access control for different users or documents.

A production upgrade could use a managed vector search service such as Databricks Vector Search or another enterprise vector database.

---

### 3. Small Free LLM

The project uses:

```text
google/flan-t5-small
```

This model is lightweight and free, which makes it useful for learning.

However, it has limited answer quality compared to larger language models.

Common limitations include:

* Short answers
* Weak reasoning
* Limited summarization ability
* Occasional incomplete responses
* Limited ability to follow complex instructions
* Lower quality compared with modern hosted LLMs

For a production chatbot, a stronger model would usually be required.

---

### 4. Basic Relevance Threshold

The chatbot uses a simple FAISS score threshold to reject unrelated questions.

This is useful for learning, but it is not a complete relevance validation system.

A better production system would include:

* More robust similarity score calibration
* Retrieval evaluation
* Reranking
* Query rewriting
* Better fallback behavior
* Confidence scoring
* Human review for low-confidence answers

---

### 5. No Web Interface Yet

The chatbot currently runs inside Databricks notebooks.

There is no standalone user interface yet.

Possible future options include:

* Streamlit app
* Gradio app
* Databricks App
* Flask or FastAPI backend
* React frontend
* Simple internal web portal

A web interface would make the project easier for non-technical users to test.

---

### 6. No Chat History

The current chatbot does not maintain multi-turn conversation memory.

Each question is treated independently.

For example, the system may not understand follow-up questions such as:

```text
Can you explain that in simpler terms?
```

or:

```text
What about the second point?
```

A future version could add:

* Chat history
* Conversation-aware retrieval
* Follow-up question handling
* Session-level context
* Memory window management

---

### 7. No Automated Evaluation

The project does not yet include formal RAG evaluation metrics.

Future evaluation could include:

* Retrieval precision
* Context relevance
* Answer relevance
* Faithfulness
* Hallucination checks
* Manual test questions
* LLM-as-judge evaluation
* Regression tests for known questions

Evaluation would make it easier to measure whether changes improve or harm the chatbot.

---

### 8. No Production Security Layer

The current project is for learning and does not include a production security layer.

It does not currently include:

* User authentication
* Role-based access control
* Document-level permissions
* Audit logging
* PII detection
* Data masking
* Secret management
* Environment separation

These would be required for enterprise use.

---

### 9. No Automated Ingestion Pipeline

PDF processing is currently done manually through notebooks.

A production version would likely need an automated ingestion process.

Possible improvements include:

* Scheduled ingestion jobs
* File arrival triggers
* Automatic chunking
* Automatic embedding generation
* Index refresh logic
* Failed-file handling
* Ingestion status tracking

---

### 10. Limited Observability

The current project does not include detailed monitoring.

A production system should track:

* Number of uploaded documents
* Number of chunks created
* Embedding generation success or failure
* Retrieval latency
* Answer generation latency
* User questions
* Failed queries
* Low-confidence responses
* Model errors

This would help debug and improve the system over time.

---

## Future Improvements

Recommended future improvements:

1. Add support for multiple PDFs.
2. Add a simple user interface.
3. Add document upload automation.
4. Store document metadata in Delta tables.
5. Use Databricks Vector Search.
6. Add a stronger open-source or hosted LLM.
7. Add MLflow tracking for RAG experiments.
8. Add RAG evaluation notebooks.
9. Add chat history.
10. Deploy the chatbot as an API or app.
11. Add authentication and document-level access control.
12. Add logging and monitoring.
13. Add CI checks for notebook and Python code quality.
14. Add sample questions and expected answers.
15. Add screenshots of the working chatbot.

---

## Recommended Next Phase

The next logical phase is to make the project easier to use by adding:

```text
multi-PDF support
document metadata table
simple chat interface
RAG evaluation notebook
```

This would turn the project from a notebook-based learning demo into a more complete portfolio project.

---

## Conclusion

This project is a strong learning foundation for understanding PDF-based RAG chatbots.

It demonstrates how an uploaded PDF can be transformed into a searchable knowledge base and used to generate source-grounded answers.

The project is intentionally simple, free, and beginner-friendly, but it can be extended into a more production-ready architecture with managed vector search, stronger models, evaluation, monitoring, and a proper user interface.
