# Version 2: OpenAI, Chroma, LCEL, and Streaming PDF RAG Chatbot

## Overview

This folder contains Version 2 of the PDF Q&A chatbot project.

Version 2 upgrades the original free implementation by using OpenAI models, Chroma vector storage, token-based splitting, LangChain prompt templates, LCEL chain composition, source-aware responses, and streaming chatbot output.

The goal is to build a more structured and production-like RAG workflow while still using the same practical use case:

```text
Upload a PDF document
        ↓
Ask questions about the uploaded document
        ↓
Receive grounded answers with sources
```

The example document used in this project is:

```text
Introduction_to_Tableau.pdf
```

---

## Key Upgrades in Version 2

Compared with the first implementation, Version 2 adds:

* OpenAI embeddings instead of Hugging Face embeddings
* GPT-based text formatting before indexing
* Token-based splitting instead of only character-based splitting
* Chroma vector database instead of FAISS
* MMR retrieval for more diverse context
* LangChain prompt templates
* LCEL-style chain composition
* Source-aware answers
* Streaming chatbot responses

---

## Architecture

```text
Uploaded PDF
        ↓
PDF loading with PyPDFLoader
        ↓
Header-based document structuring
        ↓
GPT text formatting
        ↓
Token-based splitting
        ↓
OpenAI embeddings
        ↓
Chroma vector database
        ↓
MMR retriever
        ↓
Prompt template
        ↓
GPT answer generation
        ↓
Source-aware response
        ↓
Streaming chatbot output
```

---

## Folder Contents

```text
notebooks_v2_openAI/
│
├── README.md
├── 01_pdf_upload_and_transcript_loading
├── 02_markdown_header_splitting
├── 03_gpt_text_formatting
├── 04_token_splitting
├── 05_openai_embeddings_chroma_retriever
├── 06_prompt_templates_and_lcel_chain
├── 07_improved_context_formatting
└── 08_streaming_chatbot
```

---

## Notebook Descriptions

### 01 - PDF Upload and Transcript Loading

Loads the uploaded PDF from Databricks storage and extracts page-level text.

Main outputs:

```text
loader_pdf
docs_list
string_list_concat
```

This notebook prepares the raw document transcript for the later RAG pipeline.

---

### 02 - Header-Based Document Splitting

Creates a structured transcript with markdown-style headers and applies `MarkdownHeaderTextSplitter`.

Main output:

```text
docs_list_md_split
```

This adds section and page metadata to the document chunks.

---

### 03 - GPT Text Formatting

Uses GPT to clean the extracted PDF text before embedding.

This step fixes broken line breaks, awkward spacing, capitalization, and formatting issues while preserving the original meaning.

Main outputs:

```text
formatted_docs_list
string_list_formatted
```

---

### 04 - Token-Based Splitting

Splits the cleaned documents using `TokenTextSplitter`.

Configuration:

```text
encoding_name = cl100k_base
chunk_size = 500
chunk_overlap = 50
```

Main output:

```text
docs_list_tokens_split
```

Token-based splitting is more aligned with how language models process text.

---

### 05 - OpenAI Embeddings, Chroma Vector Store, and Retriever

Creates the vector search layer.

Main components:

```text
OpenAIEmbeddings(model="text-embedding-3-small")
Chroma vector database
MMR retriever
```

Retriever configuration:

```text
search_type = mmr
k = 2 or 3
fetch_k = 6 or 8
lambda_mult = 0.7
```

This notebook creates semantic search over the uploaded PDF.

---

### 06 - Prompt Templates and LCEL Q&A Chain

Builds the Q&A chain using LangChain prompt templates and LCEL.

The chain performs:

```text
question
   ↓
retrieve context
   ↓
format prompt
   ↓
send to GPT
   ↓
parse string output
```

Main output:

```text
chain_retrieving
```

---

### 07 - Improved Context Formatting and Source-Aware Answers

Improves the chatbot response by formatting retrieved context and returning source metadata.

Main outputs:

```text
format_context_with_sources
ask_pdf_v2_with_sources
```

The final response includes:

```text
Answer:
...

Sources used:
- Document 1: Section Title='...', Page Title='...'
- Document 2: Section Title='...', Page Title='...'
```

---

### 08 - Streaming PDF Q&A Chatbot

Adds streaming output for a more interactive chatbot experience.

Main output:

```text
stream_pdf_answer
```

The answer is printed progressively as GPT generates it.

This notebook also includes an interactive question loop.

---

## OpenAI Models Used

### Embedding Model

```text
text-embedding-3-small
```

Used to convert document chunks and questions into vector embeddings.

### Chat Model

```text
gpt-4o-mini
```

Used for:

* Text formatting
* Answer generation
* Streaming chatbot responses

The model can be changed to `gpt-4o` if stronger answer quality is required.

---

## Vector Database

Version 2 uses Chroma.

During development, the Chroma index is stored in:

```text
/tmp/chroma_intro_tableau_v2
```

This path works well in Databricks for testing, but it may be cleared when the compute session restarts.

For production use, the vector storage design should be upgraded to a more durable managed option.

---

## Important Security Note

Do not commit a real OpenAI API key to GitHub.

Notebook cells should use a placeholder like:

```python
import os

os.environ["OPENAI_API_KEY"] = "PASTE_YOUR_OPENAI_API_KEY_HERE"
```

Before committing notebooks, make sure the real key has been removed.

A better production approach would use Databricks secrets or another secure secret-management method.

---

## Example Questions

```text
What is Tableau used for?
```

```text
What are dimensions and measures in Tableau?
```

```text
How does Tableau help with data visualization?
```

```text
How do you create a visualization in Tableau?
```

---

## Example Output

```text
Answer:

Tableau is used to help users connect to data, analyze information, and create visualizations such as charts and dashboards.

Sources used:
- Document 1: Section Title='Introduction to Tableau', Page Title='Page 1'
- Document 2: Section Title='Introduction to Tableau', Page Title='Page 2'
```

---

## Current Status

Completed:

* PDF loading
* Header-based document structuring
* GPT text formatting
* Token-based splitting
* OpenAI embeddings
* Chroma vector indexing
* MMR retrieval
* Prompt-template-based Q&A
* LCEL chain
* Source-aware response formatting
* Streaming chatbot output

---

## Limitations

Current limitations:

* The implementation currently focuses on one uploaded PDF.
* The Chroma index is stored in `/tmp`, so it may need to be rebuilt after session restarts.
* API keys must be handled carefully before committing notebooks.
* There is no standalone web interface yet.
* There is no formal RAG evaluation notebook yet.
* There is no user authentication or document-level permission system.

---

## Possible Future Improvements

Recommended next improvements:

* Add support for multiple PDFs
* Save intermediate formatted documents to avoid repeated GPT formatting
* Use a durable vector database path or managed vector search
* Add RAG evaluation metrics
* Add a simple Streamlit, Gradio, or Databricks app interface
* Add MLflow experiment tracking
* Add chat history
* Add stronger source citations
* Add document upload automation
* Add deployment as an API
