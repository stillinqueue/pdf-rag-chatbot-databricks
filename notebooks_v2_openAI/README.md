# Version 2: OpenAI, Chroma, LCEL, Streaming, and Multi-PDF RAG Chatbot

## Overview

This folder contains Version 2 of the PDF Q&A chatbot project.

Version 2 upgrades the original free implementation by using OpenAI models, Chroma vector storage, token-based splitting, LangChain prompt templates, LCEL chain composition, source-aware responses, and streaming chatbot output.

It has also been extended from a single-PDF chatbot into a multi-PDF RAG assistant that can search across multiple uploaded PDF documents and return answers with file-level and page-level source references.

The goal is to build a more structured and production-like RAG workflow while keeping the same practical use case:

```text
Upload one or more PDF documents
        ↓
Ask questions about the uploaded documents
        ↓
Retrieve relevant chunks across the document collection
        ↓
Generate grounded answers
        ↓
Return answers with sources
```

---

## Current Capabilities

Version 2 currently supports:

* Single-PDF Q&A
* Multi-PDF Q&A
* PDF loading from Databricks Volumes
* Header-based document structuring
* GPT-based text formatting
* Token-based splitting
* OpenAI embeddings
* Chroma vector database
* MMR retrieval
* LangChain prompt templates
* LCEL-style Q&A chains
* Source-aware answer formatting
* Streaming chatbot responses

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
* Multi-PDF indexing and retrieval

---

## Architecture

### Single-PDF Flow

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

### Multi-PDF Flow

```text
Multiple uploaded PDFs
        ↓
Load all PDFs from Databricks folder
        ↓
Preserve source file, path, page, and document type metadata
        ↓
Token-based splitting across all PDFs
        ↓
OpenAI embeddings
        ↓
Shared Chroma vector database
        ↓
MMR retriever across all PDF chunks
        ↓
GPT answer generation
        ↓
Source-aware response with PDF file, page, and chunk ID
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
├── 08_streaming_chatbot
├── 09_multi_pdf_loading
├── 10_multi_pdf_token_splitting
├── 11_multi_pdf_chroma_index
├── 12_multi_pdf_qa_with_sources
└── 13_multi_pdf_streaming_chatbot
```

---

# Single-PDF Notebooks

## 01 - PDF Upload and Transcript Loading

Loads one uploaded PDF from Databricks storage and extracts page-level text.

Main outputs:

```text
loader_pdf
docs_list
string_list_concat
```

This notebook prepares the raw document transcript for the later RAG pipeline.

---

## 02 - Header-Based Document Splitting

Creates a structured transcript with markdown-style headers and applies `MarkdownHeaderTextSplitter`.

Main output:

```text
docs_list_md_split
```

This adds section and page metadata to the document chunks.

---

## 03 - GPT Text Formatting

Uses GPT to clean the extracted PDF text before embedding.

This step fixes broken line breaks, awkward spacing, capitalization, and formatting issues while preserving the original meaning.

Main outputs:

```text
formatted_docs_list
string_list_formatted
```

---

## 04 - Token-Based Splitting

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

## 05 - OpenAI Embeddings, Chroma Vector Store, and Retriever

Creates the vector search layer for the single-PDF workflow.

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

## 06 - Prompt Templates and LCEL Q&A Chain

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

## 07 - Improved Context Formatting and Source-Aware Answers

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

## 08 - Streaming PDF Q&A Chatbot

Adds streaming output for a more interactive chatbot experience.

Main output:

```text
stream_pdf_answer
```

The answer is printed progressively as GPT generates it.

This notebook also includes an interactive question loop.

---

# Multi-PDF Extension Notebooks

## 09 - Multi-PDF Loading

Loads every PDF file from a Databricks folder.

Input folder:

```text
/Volumes/workspace/365pdf/365pdfupload/multi_pdf_docs
```

Main outputs:

```text
pdf_folder_path
pdf_files
all_pdf_docs
```

Each loaded page receives metadata:

```text
source_file
source_path
page_number
document_type
```

This metadata allows the chatbot to cite the original PDF file and page later.

---

## 10 - Multi-PDF Token Splitting

Splits all loaded PDF pages into token-sized chunks while preserving metadata.

Main outputs:

```text
all_pdf_docs
multi_pdf_token_docs
```

Each chunk receives a `chunk_id` so retrieved results can be traced more clearly.

---

## 11 - Multi-PDF Chroma Index

Creates one shared Chroma vector database for all uploaded PDFs.

Main outputs:

```text
multi_pdf_token_docs
embedding
multi_pdf_vectorstore
multi_pdf_retriever
```

The multi-PDF Chroma index is stored during development at:

```text
/tmp/chroma_multi_pdf_v1
```

This notebook enables semantic search across the complete PDF collection.

---

## 12 - Multi-PDF Q&A with Sources

Creates a source-aware Q&A assistant across all uploaded PDFs.

Main outputs:

```text
multi_pdf_retriever
format_multi_pdf_context
ask_multi_pdf
```

The final response includes:

```text
Answer:
...

Sources used:
- Document 1: file_name.pdf, page=..., chunk_id=...
- Document 2: another_file.pdf, page=..., chunk_id=...
```

---

## 13 - Multi-PDF Streaming Chatbot

Adds streaming chatbot output to the multi-PDF assistant.

Main output:

```text
stream_multi_pdf_answer
```

This notebook includes an interactive chatbot loop that can answer questions across all uploaded PDFs and stream the answer progressively.

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

During development, Chroma indexes are stored in `/tmp` because Chroma may run into disk I/O issues when writing its internal database files directly to a Databricks Volume path.

Single-PDF index:

```text
/tmp/chroma_intro_tableau_v2
```

Multi-PDF index:

```text
/tmp/chroma_multi_pdf_v1
```

These paths work well in Databricks for testing, but they may be cleared when the compute session restarts.

For production use, vector storage should be upgraded to a more durable managed option.

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

## Example Single-PDF Questions

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

## Example Multi-PDF Questions

```text
What is Agentic RAG?
```

```text
What is multimodal retrieval?
```

```text
What are the challenges of question answering over rich documents?
```

```text
Compare agentic RAG and multimodal RAG.
```

---

## Example Single-PDF Output

```text
Answer:

Tableau is used to help users connect to data, analyze information, and create visualizations such as charts and dashboards.

Sources used:
- Document 1: Section Title='Introduction to Tableau', Page Title='Page 1'
- Document 2: Section Title='Introduction to Tableau', Page Title='Page 2'
```

---

## Example Multi-PDF Output

```text
Answer:

Agentic RAG extends retrieval-augmented generation by allowing an AI system to make decisions about retrieval steps, tool usage, and reasoning paths before generating an answer.

Sources used:
- Document 1: Agentic RAG.pdf, page=2, chunk_id=14
- Document 2: Question-Answering.pdf, page=5, chunk_id=37
```

---

## Current Status

Completed:

* Single-PDF loading
* Header-based document structuring
* GPT text formatting
* Token-based splitting
* OpenAI embeddings
* Chroma vector indexing
* MMR retrieval
* Prompt-template-based Q&A
* LCEL chain
* Source-aware response formatting
* Single-PDF streaming chatbot
* Multi-PDF folder loading
* Multi-PDF token splitting
* Shared multi-PDF Chroma indexing
* Multi-PDF source-aware Q&A
* Multi-PDF streaming chatbot

---

## Limitations

Current limitations:

* The implementation currently supports PDFs only.
* The Chroma indexes are stored in `/tmp`, so they may need to be rebuilt after session restarts.
* API keys must be handled carefully before committing notebooks.
* There is no standalone web interface yet.
* There is no formal RAG evaluation notebook yet.
* There is no user authentication or document-level permission system.
* There is no persistent user workspace layer yet.
* There is no upload UI outside Databricks yet.

---

## Possible Future Improvements

Recommended next improvements:

* Add support for DOCX files
* Add support for PPTX files
* Add support for YouTube URLs
* Add support for images using OCR and vision models
* Add support for videos using transcription
* Add Microsoft Teams connector support
* Add SQL database connector support
* Save intermediate processed documents to avoid repeated parsing
* Use a durable vector database path or managed vector search
* Add RAG evaluation metrics
* Add a web application interface
* Add MLflow experiment tracking
* Add chat history
* Add stronger source citations
* Add document upload automation
* Add deployment as an API
* Integrate with a user workspace experience on stillinqueue.com

---

## Planned Source Expansion Roadmap

The next planned source types are:

```text
1. PDFs - completed for single and multiple PDF files
2. DOCX
3. PPTX
4. YouTube URLs
5. Images
6. Videos
7. Microsoft Teams
8. SQL databases
```

Each source type will be converted into a standard internal document format:

```text
Document(
    page_content="...",
    metadata={
        "source_type": "...",
        "source_name": "...",
        "source_path": "...",
        "page_number": "...",
        "chunk_id": "..."
    }
)
```

This standard format will allow all future source types to share the same downstream RAG pipeline:

```text
Load source
        ↓
Extract text
        ↓
Attach metadata
        ↓
Token split
        ↓
Embed
        ↓
Store in vector database
        ↓
Retrieve
        ↓
Answer with sources
```
