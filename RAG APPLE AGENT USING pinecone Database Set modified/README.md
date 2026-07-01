# RAG Apple Agent — Pinecone Knowledge Base

A Retrieval-Augmented Generation (RAG) agent that answers questions about Apple Inc. (AAPL) quarterly earnings using a Pinecone vector database — with a fully automated ingestion pipeline that keeps the knowledge base in sync with new financial reports.

## What it does

The workflow is split into two independent pipelines:

### 1. Data ingestion (keeps the knowledge base current)
- Triggered on a schedule (hourly) or instantly via a **Google Drive trigger** watching a shared folder of Apple earnings reports.
- Cross-references incoming files against a Google Sheets index to detect which reports haven't been processed yet.
- New files are downloaded from Google Drive, split into chunks with a recursive character text splitter, embedded via OpenAI embeddings, and upserted into a **Pinecone** vector store — namespaced per report (e.g. `Q1.pdf`, `Q2.pdf`).
- The Google Sheets index is updated so the same file is never re-processed.

### 2. RAG chat agent (answers questions)
- A chat trigger receives a user question (e.g. "What was Apple's Services revenue in Q3?").
- The question is parsed to detect which quarter is being referenced, scoping the Pinecone namespace lookup to that specific report.
- An AI agent, equipped with a **vector store retrieval tool**, searches Pinecone for the top-5 most relevant chunks and answers strictly from that retrieved context — never from memory or fabricated figures.
- The system prompt constrains the agent to Apple's quarterly financial metrics only: revenue by segment, gross margin, operating/net income, EPS, YoY/QoQ comparisons, guidance, and earnings call commentary.

## Stack

| Layer | Tool |
|---|---|
| Orchestration | n8n |
| Vector database | Pinecone |
| Embeddings | OpenAI (`text-embedding-3-small`) |
| LLM | OpenAI / OpenRouter (`gpt-4o-mini`) |
| Document source | Google Drive |
| Ingestion index | Google Sheets |
| Chat interface | n8n Chat Trigger |

## Why this matters

This is a production pattern for building a domain-specific AI analyst: instead of a general-purpose chatbot guessing at numbers, the agent is grounded entirely in retrieved source documents, with an automated pipeline that keeps that knowledge base fresh as new reports land — no manual re-indexing required.

## Setup notes

- Requires n8n credentials for Pinecone, OpenAI/OpenRouter, and Google Drive/Sheets configured in your instance.
- Pinecone index name used: `applee` (rename to match your own index).
- The Google Sheet used as the ingestion index needs `Id` and `Name` columns to track processed files.
- The namespace-detection logic expects quarter references in the format `Q1`–`Q9` in the user's question and file names like `Q1.pdf`.
