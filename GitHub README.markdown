# 🚀 RUSH E-Commerce Customer Support RAG System

An end-to-end Retrieval-Augmented Generation (RAG) solution built with **n8n**, **Google Gemini**, and **Supabase Vector Store**. This system enables an intelligent AI Customer Support Agent to query product catalogs, shipping policies, and store details for the **RUSH** clothing brand in real-time.

---

## 🏗️ System Architecture & Workflow Overview

This repository consists of two interconnected n8n workflows that form a complete RAG pipeline:

```
[ Unstructured Text / Product Data ]
                 │
                 ▼
    [ Work1: Data Ingestion Pipeline ] ──► (Generates Embeddings via Gemini) ──► [ Supabase Vector Database ]
                                                                                            │
                                                                                            ▼
[ User Chat Query ] ──► [ Work2: AI Agent Workflow ] ──► (Retrieves Context) ───────────────┘
                                 │
                                 ▼
                     [ Accurate Response to User ]
```

### 1. `work1.json` — Data Ingestion Pipeline
* **Purpose:** Preprocesses, embeds, and stores brand data (products, prices, shipping policy) into the vector database.
* **Nodes Used:**
  * `When clicking 'Execute workflow'` (Manual Trigger)
  * `Edit Fields` (Data Input / Context definition)
  * `Default Data Loader` (JSON parser)
  * `Embeddings Google Gemini` (`models/gemini-embedding-001` - 3072 dimensions)
  * `Supabase Vector Store` (Vector storage)

### 2. `work2.json` — RAG Chatbot & AI Agent Pipeline
* **Purpose:** Serves as the interactive conversational agent that queries Supabase to answer user requests with grounded context.
* **Nodes Used:**
  * `n8n Chat Trigger` (User Interface)
  * `AI Agent` (Conversational logic)
  * `Google Gemini Chat Model` (LLM Engine)
  * `Vector Store Retriever` / `Supabase Vector Store` (Knowledge retrieval)
  * `Window Buffer Memory` (Conversation state)

---

## 🗄️ Database Setup (Supabase)

Before running the workflows, execute the following SQL in your Supabase **SQL Editor** to enable vector search and set up the `documents` table:

```sql
-- Enable the pgvector extension to work with embeddings
CREATE EXTENSION IF NOT EXISTS vector;

-- Create the documents table matching 3072 embedding dimensions
CREATE TABLE IF NOT EXISTS documents (
  id BIGSERIAL PRIMARY KEY,
  content TEXT,
  metadata JSONB,
  embedding VECTOR(3072)
);

-- Create a match_documents function for vector similarity searches
CREATE OR REPLACE FUNCTION match_documents (
  query_embedding VECTOR(3072),
  match_count INT DEFAULT 5,
  filter JSONB DEFAULT '{}'
)
RETURNS TABLE (
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
```

---

## 🛠️ Installation & Setup

### Prerequisites
1. An active **n8n** instance (Self-hosted or Cloud).
2. A **Google Gemini API Key** (from Google AI Studio).
3. A **Supabase Account** with an active project.

### Step 1: Import Workflows to n8n
1. Open n8n and click **Workflows** -> **Import from File**.
2. Select `work1.json` to import the Data Ingestion workflow.
3. Repeat for `work2.json` to import the AI Agent workflow.

### Step 2: Configure Credentials in n8n
1. **Google Gemini Credentials:** Add your Gemini API Key under `Google Gemini API` credentials.
2. **Supabase Credentials:** Add your Supabase `URL` and `Service Role Key` (or `Anon Key`).

### Step 3: Run Ingestion
1. Open **`work1.json`** in n8n.
2. Click **Execute Workflow**.
3. Ensure all nodes return green checkmarks ✅. This inserts the embedded RUSH store information into Supabase.

### Step 4: Test the Chatbot
1. Open **`work2.json`** in n8n.
2. Click **Open Chat** at the bottom of the interface.
3. Test with queries such as:
   * *"ما هي ألوان الهودي المتاحة وسعره؟"* (What hoodie colors are available and what is the price?)
   * *"ما هي شروط التوصيل المجاني في RUSH؟"* (What are the free shipping conditions for RUSH?)

---

## 📋 Features & Capabilities

* **Zero Hallucination:** Answers are anchored directly to stored brand data via Supabase Vector Store.
* **Arabic Language Native Support:** Pre-configured and optimized for Arabic e-commerce customer support.
* **Scalable Pipeline:** Easily update store items in `work1.json` to refresh knowledge instantly.

---

## 📄 License
This project is open-source and available under the [MIT License](LICENSE).