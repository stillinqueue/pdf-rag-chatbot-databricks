# Version Comparison

## Overview

This project contains two implementations of a PDF Q&A chatbot.

Both versions solve the same practical problem:

```text
Upload a PDF document and ask questions about its contents.
```

The difference is in the implementation approach.

Version 1 focuses on a free open-source RAG pipeline.

Version 2 upgrades the project with OpenAI models, Chroma vector storage, LCEL chain composition, and streaming chatbot output.

## Summary

| Area            | Version 1                                | Version 2                              |
| --------------- | ---------------------------------------- | -------------------------------------- |
| Folder          | `notebooks/`                             | `notebooks_v2_openAI/`                 |
| Embeddings      | Hugging Face embeddings                  | OpenAI embeddings                      |
| Embedding model | `sentence-transformers/all-MiniLM-L6-v2` | `text-embedding-3-small`               |
| Vector store    | FAISS                                    | Chroma                                 |
| Text splitting  | Character-based splitting                | Header-based and token-based splitting |
| LLM             | FLAN-T5 small                            | GPT                                    |
| Prompting       | Python prompt function                   | LangChain prompt templates             |
| Chain logic     | Manual Python functions                  | LCEL chain                             |
| Retrieval       | Similarity search                        | MMR retrieval                          |
| Output          | Full answer after generation             | Streaming response                     |
| Cost approach   | Free/open-source                         | OpenAI API usage                       |

## Version 1: Free Open-Source Implementation

Version 1 was designed to avoid paid APIs.

It uses open-source tools to demonstrate the basic RAG workflow.

Main components:

```text
PDF upload
        ↓
Text extraction
        ↓
Character-based chunking
        ↓
Hugging Face embeddings
        ↓
FAISS vector index
        ↓
Similarity retrieval
        ↓
FLAN-T5 answer generation
        ↓
Sources shown
```

Version 1 is useful for learning the core RAG pipeline without using API credits.

## Version 2: OpenAI, Chroma, LCEL, and Streaming Implementation

Version 2 is a more advanced implementation.

It uses OpenAI models and a more structured LangChain workflow.

Main components:

```text
PDF upload
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
LangChain prompt template
        ↓
LCEL Q&A chain
        ↓
Source-aware answer
        ↓
Streaming chatbot output
```

Version 2 is closer to a modern RAG application pattern.

## Why Both Versions Are Useful

Having both versions makes the project stronger.

Version 1 shows that the pipeline can be built with free tools.

Version 2 shows how the same use case can be upgraded with stronger models and more advanced LangChain features.

Together, they demonstrate:

* Basic RAG understanding
* Open-source implementation skills
* OpenAI integration
* Vector database usage
* Prompt engineering
* LCEL chain composition
* Streaming chatbot behavior
* Source-aware answer generation

## Recommended Usage

Use Version 1 when:

* Avoiding paid APIs
* Learning the basics of RAG
* Testing simple retrieval and answer generation
* Working in a free environment

Use Version 2 when:

* Better answer quality is required
* OpenAI API tokens are available
* Prompt templates and LCEL are needed
* Streaming chatbot output is desired
* A more advanced portfolio implementation is preferred

## Conclusion

The two-version structure shows the project evolution clearly.

Version 1 proves the core idea.

Version 2 upgrades the implementation into a stronger OpenAI-powered RAG chatbot with Chroma, MMR retrieval, LCEL chaining, source-aware answers, and streaming output.
