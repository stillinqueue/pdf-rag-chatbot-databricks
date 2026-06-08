# OpenAI V2 RAG Architecture

## Overview

Version 2 of this project upgrades the original PDF Q&A chatbot by using OpenAI models, Chroma vector storage, token-based splitting, LangChain prompt templates, LCEL chain composition, source-aware answers, and streaming responses.

The goal is to create a more advanced RAG pipeline while keeping the same practical use case:

```text
Upload a PDF document
        ↓
Ask questions about the document
        ↓
Receive grounded answers with sources
```

## Architecture Flow

```text
Uploaded PDF
        ↓
PyPDFLoader
        ↓
Page-level LangChain documents
        ↓
Markdown-style header structuring
        ↓
MarkdownHeaderTextSplitter
        ↓
GPT-based text formatting
        ↓
TokenTextSplitter
        ↓
OpenAIEmbeddings
        ↓
Chroma vector database
        ↓
MMR retriever
        ↓
LangChain prompt template
        ↓
LCEL Q&A chain
        ↓
GPT answer generation
        ↓
Source-aware response
        ↓
Streaming chatbot output
```

## Main Components

### PDF Loading

The uploaded PDF is loaded from Databricks storage using `PyPDFLoader`.

The output is a list of LangChain `Document` objects.

Main variables:

```text
loader_pdf
docs_list
string_list_concat
```

### Header-Based Structuring

The extracted PDF text is converted into a markdown-style transcript so that header-based splitting can be applied.

This helps preserve metadata such as:

```text
Section Title
Page Title
```

### GPT Text Formatting

GPT is used to clean the extracted text before indexing.

This step helps fix:

* Broken line breaks
* Awkward spacing
* Poor punctuation
* Formatting issues caused by PDF extraction

The formatting prompt preserves the original meaning and does not add new information.

### Token-Based Splitting

The cleaned documents are split with `TokenTextSplitter`.

Configuration:

```text
encoding_name = cl100k_base
chunk_size = 500
chunk_overlap = 50
```

Token-based splitting is better aligned with how language models process text.

### OpenAI Embeddings

The project uses OpenAI embeddings to convert text chunks into vector representations.

Embedding model:

```text
text-embedding-3-small
```

These embeddings allow semantic search over the uploaded PDF content.

### Chroma Vector Database

The embedded document chunks are stored in Chroma.

Development path:

```text
/tmp/chroma_intro_tableau_v2
```

This path is suitable for Databricks testing, but it may be cleared when the compute session restarts.

### MMR Retriever

The retriever uses Maximal Marginal Relevance.

MMR helps retrieve chunks that are both relevant and diverse.

Example configuration:

```text
search_type = mmr
k = 3
fetch_k = 8
lambda_mult = 0.7
```

### Prompt Template

The chatbot uses a structured LangChain prompt template.

The prompt instructs GPT to:

* Answer only from the retrieved PDF context
* Avoid outside knowledge
* Return a fallback response when the answer is not found
* Keep the answer beginner-friendly
* Avoid inventing page numbers or facts

### LCEL Chain

The Q&A logic is built using LangChain Expression Language.

The chain performs:

```text
question
   ↓
retrieve documents
   ↓
format context
   ↓
apply prompt template
   ↓
send to GPT
   ↓
parse response
```

### Streaming Output

The final notebook streams the GPT response progressively.

This creates a more natural chatbot experience compared with waiting for the full answer to finish.

## Current Limitations

* The project currently focuses on one uploaded PDF.
* The Chroma index is stored in `/tmp`, so it may need to be rebuilt after session restart.
* The project does not yet include a web application interface.
* API keys must be removed before committing notebooks to GitHub.
* Formal RAG evaluation has not yet been added.

## Future Improvements

Possible improvements include:

* Add support for multiple PDFs
* Store formatted documents to avoid repeated GPT formatting
* Add durable vector storage
* Add RAG evaluation metrics
* Add a simple web UI
* Add chat history
* Add stronger citation formatting
* Add deployment as an API
