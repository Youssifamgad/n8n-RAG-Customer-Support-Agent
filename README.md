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


🔍 Detailed Node Breakdown
1. Ingestion Sub-System (ETL)
Google Drive Trigger (googleDriveTrigger): Polls for fileUpdated events every minute in a targeted Google Drive folder.

Download File (googleDrive): Fetches the actual PDF file from Google Drive when changes are detected.

Default Data Loader (documentDefaultDataLoader): Parses the PDF file into structured text chunks ready for vector processing.

Embeddings Google Gemini (embeddingsGoogleGemini): Converts text chunks into 768-dimensional mathematical vector embeddings representing semantic meaning.

Supabase Vector Store - Ingest (vectorStoreSupabase): Batches embeddings (batch size: 20) and inserts them into the Supabase documents table.

2. Conversational Sub-System (RAG Agent)
Telegram Trigger (telegramTrigger): Listens for incoming customer queries via webhooks.

Simple Memory (memoryBufferWindow): Maintains conversational context per user by isolating sessions using {{ $json.message.chat.id }}.

Google Gemini Chat Model (lmChatGoogleGemini): Powers the reasoning engine using gemini-3.1-flash-lite.

Supabase Vector Store - Tool (vectorStoreSupabase): Exposes vector search as a callable tool for the Agent to fetch pertinent document chunks.

AI Agent (agent): Connects the LLM, Memory, and Retrieval Tool under explicit system prompts.

Send Text Message (telegram): Sends formatted, clean responses back to the user via Telegram.

-- Enable the pgvector extension
create extension if not exists vector;

-- Create the documents table
create table if not exists documents (
  id bigint primary key generated always as identity,
  content text,
  metadata jsonb,
  embedding vector(768)
);

-- Create a match function for vector similarity search
create or replace function match_documents (
  query_embedding vector(768),
  match_count int DEFAULT 5,
  filter jsonb DEFAULT '{}'
) returns table (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
)
language plpgsql
as $$
begin
  return query
  select
    documents.id,
    documents.content,
    documents.metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where documents.metadata @> filter
  order by documents.embedding <=> query_embedding
  limit match_count;
end;
$$;
⚙️ Installation & Setup Guide
Import Workflow:

Download RAG AI AGENT (3).json from this repository.

Open your n8n instance ➔ Click Workflows ➔ Select Import from File.

Configure Credentials:

Google Drive OAuth2 API: For folder monitoring and PDF downloads.

Google Gemini API Key: For text embeddings and Chat LLM.

Supabase API: Project URL and Service Role Key.

Telegram Bot API: Token generated via Telegram's @BotFather.

Node Parameter Tweaks:

In Google Drive Trigger, update the folderToWatch ID to match your Drive folder.

In Download file, set the File ID dynamically to ={{ $json.id }}.

Activate & Run:

Toggle the workflow status switch in n8n to Active.
