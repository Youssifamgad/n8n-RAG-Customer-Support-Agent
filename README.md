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

<div align="center">

# 🤖 Enterprise RAG AI Customer Support Agent

<b>An Autonomous, Zero-Hallucination, Bilingual AI Support Automation Engine</b>

[![n8n](https://img.shields.io/badge/n8n-FF6D5A?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io)
[![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)](https://supabase.com)
[![Google Gemini](https://img.shields.io/badge/Google%20Gemini-8E75B2?style=for-the-badge&logo=google%20gemini&logoColor=white)](https://ai.google.dev)
[![Telegram](https://img.shields.io/badge/Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](https://telegram.org)

</div>

---

> [!IMPORTANT]
> **Production Guardrails Active:** Powered by **Google Gemini 3.1 Flash Lite** and **Supabase `pgvector`**. This agent is strictly grounded in official policy documentation from Google Drive, completely eliminating AI hallucinations and safely declining out-of-scope inquiries.

---

## 📐 System Architecture

Below is the automated data flow rendering both the **ETL Ingestion Pipeline** and the **Real-Time Support Agent**:

```mermaid
graph TD
    subgraph ETL_Pipeline ["1️⃣ Automated Knowledge Ingestion Pipeline"]
        GD[📁 Google Drive Folder] -->|File Upload / Update| DL[⬇️ Download PDF]
        DL --> Read[📄 Extract Text Chunks]
        Read --> Emb1[🧠 Gemini Embeddings]
        Emb1 -->|768-d Vectors| SupabaseInsert[(⚡ Supabase Vector Store)]
    end

    subgraph Agent_Pipeline ["2️⃣ Conversational RAG Support Agent"]
        TG[💬 Telegram Customer Query] --> Agent[🤖 AI Agent Engine]
        Agent <-->|Semantic Search| SupabaseTool[(⚡ Supabase Vector Tool)]
        SupabaseTool <--> Emb2[🧠 Gemini Embeddings]
        Agent <-->|Chat Context| Mem[🧠 Session Memory]
        Agent -->|Grounded Response| SendTG[📩 Send Telegram Message]
    end

    style ETL_Pipeline fill:#1e293b,stroke:#3b82f6,color:#fff
    style Agent_Pipeline fill:#0f172a,stroke:#10b981,color:#fff
