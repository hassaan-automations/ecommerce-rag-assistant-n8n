# ecommerce-rag-assistant-n8n
Production-grade E-Commerce Customer Support RAG Assistant built with n8n, Google Gemini, and Supabase pgvector.
# 🛒 E-Commerce Customer Support RAG Assistant (n8n + Gemini + Supabase)

A production-grade, zero-hallucination Retrieval-Augmented Generation (RAG) assistant built inside **n8n**. It automates customer support by strictly referencing official store policies for shipping, refunds, and cancellations.

---

## ⚡ Tech Stack & Architecture
- **Orchestration:** n8n
- **Vector Database:** Supabase (`pgvector` with 3072 dimensions)
- **Embeddings:** Google Gemini (`text-embedding-004`)
- **LLM / Chat Model:** Google Gemini (`gemini-1.5-flash`)
- **Data Source:** Velocity Retail Store Policies (PDF)

---

## 📸 Workflow & Live Implementation

### 1. Ingestion Pipeline
Monitors policy documents, parses binary PDF data, creates recursive chunks, and stores dense vector embeddings in Supabase.
![Ingestion Pipeline](ss-ingestion-flow.png)

### 2. Supabase pgvector Store
Structured vector storage with embedded text chunks and metadata for semantic similarity search.
![Supabase Vector Records](ss-supabase-vectors.png)

### 3. RAG Retrieval & Guardrail Chat Agent
Processes natural language queries, performs cosine similarity retrieval, and responds strictly using policy context with mandatory section citations.
![Chat Agent Live Test](ss-chat-rag.png)

---

## 🛡️ Production Guardrails
1. **Strict Context Grounding:** The model only generates responses from retrieved vectors.
2. **Zero-Hallucination Fallback:** If an inquiry falls outside documented policy, it routes the customer to human support without guessing.
3. **Mandatory Citations:** Every response concludes with the referenced policy section tag.

---

## 🚀 Setup & Usage
1. Import the workflow JSON from the repository into your n8n instance.
2. Configure your Google Gemini API and Supabase credentials in n8n.
3. Execute the Ingestion workflow to vectorize your policy PDF into Supabase.
4. Test and deploy the Chat Agent via webhook or chat trigger.
