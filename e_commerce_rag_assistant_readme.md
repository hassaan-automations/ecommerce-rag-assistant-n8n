# AI-Powered E-Commerce Policy & Support Knowledge Assistant (RAG Pipeline)

[![n8n](https://img.shields.io/badge/Orchestration-n8n-EA4B71?logo=n8n)](https://n8n.io/)
[![Google Gemini](https://img.shields.io/badge/LLM%20%26%20Embeddings-Google%20Gemini-4285F4?logo=google)](https://ai.google.dev/)
[![Supabase](https://img.shields.io/badge/Vector%20Store-Supabase%20pgvector-3ECF8E?logo=supabase)](https://supabase.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An end-to-end Retrieval-Augmented Generation (RAG) assistant designed for retail and e-commerce customer support workflows. Built natively inside **n8n**, this system automates multi-document knowledge indexing, semantic vector search via **Supabase pgvector**, and bounded query generation with **Google Gemini**, enforcing zero-hallucination guardrails and automated section-level citations.

---

## Architecture Overview

```
                      INGESTION WORKFLOW
 [Google Drive / PDF] ──> [Data Loader] ──> [Text Splitter] 
                                                   │
                                                   ▼
                                         [Gemini Embeddings] (dim: 3072)
                                                   │
                                                   ▼
                                        [Supabase (pgvector)]

                     RETRIEVAL & INFERENCE
 [Customer Inquiry]   ──> [n8n AI Agent] ──> [Supabase Vector Tool]
                                │                     │
                                │ <── [Top-K Chunks] ─┘
                                ▼
                     [Google Gemini Model]
                                │
                                ▼
                 [Grounded Response + Citation]
```

---

## Key Features

- **Automated Cloud Document Ingestion:** Listens to cloud-hosted policy files (e.g., Google Drive), parses binary documents, and applies configurable text chunking (`chunk_size: 500`, `chunk_overlap: 50`).
- **High-Dimension Semantic Indexing:** Embeds unstructured operational guidelines using Google Gemini embedding models directly into PostgreSQL via Supabase `pgvector`.
- **Strict Hallucination Prevention:** Hardened system prompts prevent speculation; out-of-scope queries trigger automated fallback escalations to live human support teams.
- **Section-Level Attribution:** Every generated answer automatically cites its originating document section (e.g., `[Hawala / Reference: Section 2 - Return Policy]`).
- **Modular n8n Design:** Completely decoupled workflows for Document Ingestion and Agentic Q&A, allowing horizontal scaling and database swapping.

---

## Tech Stack

| Layer | Tool / Technology | Purpose |
| :--- | :--- | :--- |
| **Orchestration** | n8n (Self-hosted or Cloud) | Ingestion pipeline & conversational agent graph |
| **Vector Database** | Supabase (`pgvector`) | Relational document store & Cosine Similarity search |
| **Embeddings** | Google Gemini (`text-embedding-004` / Gemini Embed) | High-dimensional dense vector embeddings |
| **LLM Inference** | Google Gemini (`gemini-1.5-flash` / `gemini-1.5-pro`) | Context reasoning and bounded conversational response |
| **Input Data** | Velocity Retail Store Policies (v2.4) | Structured real-world operational guidelines |

---

## Database Schema & Vector Search (Supabase)

Execute this script in the Supabase SQL Editor to initialize the vector database table and RPC match function:

```sql
-- 1. Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- 2. Create documents table (configured for 3072-dimension Gemini embeddings)
CREATE TABLE IF NOT EXISTS documents (
  id BIGSERIAL PRIMARY KEY,
  content TEXT NOT NULL,
  metadata JSONB DEFAULT '{}'::jsonb,
  embedding VECTOR(3072)
);

-- 3. Create Cosine Similarity match function
CREATE OR REPLACE FUNCTION match_documents (
  query_embedding VECTOR(3072),
  match_count INT DEFAULT 4,
  filter JSONB DEFAULT '{}'::jsonb
) RETURNS TABLE (
  id BIGINT,
  content TEXT,
  metadata JSONB,
  similarity FLOAT
)
LANGUAGE plpgsql
AS $$
#variable_conflict use_column
BEGIN
  RETURN QUERY
  SELECT
    id,
    content,
    metadata,
    1 - (documents.embedding <=> query_embedding) AS similarity
  FROM documents
  WHERE metadata @> filter
  ORDER BY documents.embedding <=> query_embedding
  LIMIT match_count;
END;
$$;
```

---

## Setup & Deployment Instructions

### 1. Repository Setup
```bash
git clone https://github.com/<your-username>/ecommerce-rag-assistant-n8n.git
cd ecommerce-rag-assistant-n8n
```

### 2. Configure Credentials in n8n
- **Google Gemini API:** Add credential using your API key from Google AI Studio.
- **Supabase API:** Add `Host URL` and `service_role` secret key from `Project Settings > API`.
- **Google Drive (Optional):** Setup OAuth2 credentials to enable auto-download of documents.

### 3. Import n8n Workflows
1. Inside your n8n workspace, navigate to **Workflows > Import from File**.
2. Import `workflows/ingestion_workflow.json` to index the knowledge base.
3. Import `workflows/chat_agent_workflow.json` to activate the AI Agent.

### 4. Ingestion Run
- Place your operational policy PDF (e.g., `Velocity_Retail_Store_Policies.pdf`) into Google Drive or the local disk node.
- Click **Execute Step / Test Workflow** in the Ingestion workflow to vectorize and populate the Supabase `documents` table.

---

## Verification & Test Scenarios

The system has been evaluated against core e-commerce operational scenarios:

| Category | Test Question | Verification Criteria | Status |
| :--- | :--- | :--- | :--- |
| **Shipping Fees** | *"Free shipping kab milti hai aur standard charges kitne hain?"* | Correctly cites Rs. 250 fee & free delivery threshold of Rs. 3,500. | Passed |
| **Return Terms** | *"Exchange karwana ho to courier fees kaun dega?"* | Identifies customer responsibility for preference-based exchanges. | Passed |
| **Damaged Goods** | *"Parcel damaged nikla hai, kya proof chahiye?"* | Outlines mandatory 24-hr unboxing video with label visible. | Passed |
| **Guardrails** | *"Kya aap international shipping provide karte hain?"* | Refuses conjecture; outputs human support fallback email. | Passed |

---

## Repository Structure

```text
├── workflows/
│   ├── ingestion_workflow.json      # n8n Document processing & vectorization pipeline
│   └── chat_agent_workflow.json     # n8n Conversational agent graph with Supabase tool
├── data/
│   └── Velocity_Retail_Policies.pdf # Sample policy knowledge base document
├── sql/
│   └── setup_supabase.sql           # Database schema & pgvector match function
└── README.md
```

---

## Author
- **Portfolio / Project:** 6-Month AI Automation Engineering Roadmap (Month 3, Project 2)
- **Role:** AI Automation Engineer (n8n, Python, LLM & RAG Systems)