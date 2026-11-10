---
title: "A fully local RAG service with Kotlin, Ktor, Ollama and pgvector"
date: 2025-12-03
tags: [kotlin, ktor, llm, rag, postgres]
summary: "Upload PDFs, ask questions, keep everything on your machine: a small retrieval-augmented generation service with query expansion and no external API calls."
---

Most retrieval-augmented generation (RAG) examples start by sending your documents to someone else's API. I wanted the
opposite: a service I could point at private PDFs, where nothing leaves the machine. The result is
[mohaqeq/local-rag](https://github.com/mohaqeq/local-rag): a Kotlin service built on Ktor, using Ollama for the
models and PostgreSQL with pgvector for the embeddings.

## The pieces

| Piece | Role |
|---|---|
| **Ktor** | The HTTP service: `POST /embed` to ingest a PDF, a query endpoint to ask questions |
| **Ollama** | Local inference: `mxbai-embed-large` for embeddings, `qwen3` for answers (both configurable) |
| **PostgreSQL + pgvector** | Stores chunks with their embeddings; similarity search in SQL |
| **Docker Compose** | Starts the database, and optionally Ollama with the models pre-pulled |

The code is split along those lines: `clients/` for Ollama, the PDF processor and pgvector, and `services/` for the
embedding and query flows. Ktor modules hold routing, serialization, monitoring and the API docs.

## Ingestion

`POST /embed` takes a PDF upload. The text is extracted and cut into fixed-size chunks with a small overlap (7,500
characters with 100 characters of overlap by default, both in `application.yaml`). The overlap keeps a sentence that
straddles a boundary from being lost to both chunks. Each chunk is embedded and stored in pgvector.

## Answering: query expansion

A single embedding of the user's question is a fragile search key. If the document phrases things differently, distance
search misses it. So the query service first asks the local model for **five rephrasings** of the question, from
different perspectives. It embeds each one, takes the top 3 chunks for each, and merges them into one de-duplicated
context. Only then does it ask the model to answer from that context.

If nothing relevant comes back, the service says so ("I couldn't find any relevant information…") instead of letting
the model improvise. For a document Q&A tool, that is the most important line of code.

## Running it

```bash
docker compose --profile ollama up -d   # Postgres + Ollama (pulls the models on first start)
./gradlew run                           # the service on http://localhost:8080
curl -X POST http://localhost:8080/embed -F "file=@document.pdf"
```

If Ollama already runs elsewhere, start only the database and point `ollamaBaseUrl` at it.

## What I learned

- **Local is good enough for many document tasks.** A mid-size local model with decent retrieval answers factual
  questions about your own documents well. The quality ceiling is retrieval, not the model.
- **Query expansion is cheap and effective.** One extra generation call buys noticeably better recall than tuning
  chunk sizes.
- **Postgres is a fine vector store.** pgvector keeps embeddings next to everything else you already know how to back up
  and query.
- **The boring parts matter:** long request timeouts for big PDFs, a temp folder for uploads, and clear errors for
  non-PDF files.
