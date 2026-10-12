# 🤖 Enterprise RAG AI Customer Support Agent (n8n + Supabase + Gemini + Telegram)

An automated, production-ready, bilingual AI Customer Support Agent built using **n8n**, **Supabase Vector Store**, **Google Gemini AI**, and **Telegram**.

This project implements an end-to-end **Retrieval-Augmented Generation (RAG)** architecture that synchronizes business documentation from **Google Drive**, processes and stores vector embeddings in **Supabase**, and serves users via an intelligent **Telegram Bot** with strict hallucination guardrails.

---

## 📌 Table of Contents
- [Project Overview](#-project-overview)
- [Architecture & Data Flow](#-architecture--data-flow)
- [Detailed Node Breakdown](#-detailed-node-breakdown)
- [Key Features & Technical Highlights](#-key-features--technical-highlights)
- [Real-World Testing & Verification](#-real-world-testing--verification)
- [Supabase Database Setup](#-supabase-database-setup)
- [Installation & Setup Guide](#-installation--setup-guide)
- [System Guardrails & Prompt Engineering](#-system-guardrails--prompt-engineering)

---

## 🚀 Project Overview

Traditional customer support bots often struggle with rigid decision trees or hallucinate answers when powered by pure LLMs. This project addresses both issues by combining **n8n's low-code workflow orchestration** with **LangChain RAG agents**.

The workflow handles two autonomous cycles:
1. **Automated Document Processing (ETL Pipeline):** Automatically detects updated policy documents in Google Drive, extracts binary text, generates semantic vector embeddings using Google Gemini, and indexes them into Supabase.
2. **Conversational Support Agent (Retrieval Pipeline):** Listens to customer queries on Telegram, searches the vector database for relevant business context, and generates precise, grounded responses in both **Arabic** and **English**.

---

## 🏗️ Architecture & Data Flow

```text
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                1. DOCUMENT INGESTION PIPELINE                            │
└─────────────────────────────────────────────────────────────────────────────────────────┘
  [Google Drive Trigger] ──► [Download File] ──► [Default Data Loader]
                                                          │
                                                    (Text Chunks)
                                                          │
                                                          ▼
  [Google Gemini Embeddings] ───────────────► [Supabase Vector Store (Insert)]

┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                2. CONVERSATIONAL RAG PIPELINE                           │
└─────────────────────────────────────────────────────────────────────────────────────────┘
  [Telegram Trigger] ──► [AI Agent Node] ────────────────────────────────► [Send Telegram Msg]
                            │  ▲
      ┌─────────────────────┼──┴─────────────────────┐
      │                     │                        │
      ▼                     ▼                        ▼
  [Gemini LLM]      [Simple Memory]      [Supabase Vector Tool]
  (Gemini Flash)    (Chat ID Context)              │
                                                   ▼
                                       [Gemini Embeddings]

-- 1. Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- 2. Create documents table
CREATE TABLE IF NOT EXISTS documents (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  content TEXT,
  metadata JSONB,
  embedding VECTOR(768)
);

-- 3. Create match documents function
CREATE OR REPLACE FUNCTION match_documents (
  query_embedding VECTOR(768),
  match_count INT DEFAULT 5,
  filter JSONB DEFAULT '{}'
) RETURNS TABLE (
  id BIGINT,
  content TEXT,
  metadata JSONB,
  similarity FLOAT
)
LANGUAGE plpgsql
AS $$
BEGIN
  RETURN QUERY
  SELECT
    documents.id,
    documents.content,
    documents.metadata,
    1 - (documents.embedding <=> query_embedding) AS similarity
  FROM documents
  WHERE documents.metadata @> filter
  ORDER BY documents.embedding <=> query_embedding
  LIMIT match_count;
END;
$$;
