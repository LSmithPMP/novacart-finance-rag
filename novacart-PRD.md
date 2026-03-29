# Product Requirements Document
## NovaCart AI Finance RAG System

| Field | Detail |
|-------|--------|
| Project Name | NovaCart AI Finance RAG System |
| Document Type | Product Requirements Document (PRD) |
| Version | 1.0 |
| Date | March 2026 |
| Program | Applied Agentic AI |
| Institution | Interview Kickstart |
| Week | Week 2 — RAG Systems |
| Architecture | Dual-Pipeline RAG · Scheduled Agent Reporting |
| Stack | n8n · Pinecone · OpenAI GPT-4o · text-embedding-3-large · Google Drive |
| Status | Production — Deployed |

---

## 1. Executive Summary

The NovaCart AI Finance RAG System is a dual-pipeline Retrieval-Augmented Generation (RAG) workflow built in n8n. It ingests internal financial metrics and external market documents from Google Drive into Pinecone vector storage, then deploys an AI Finance Reporting Agent that autonomously generates structured financial reports and delivers them via email on a scheduled basis.

The system implements a dual namespace strategy — internal company metrics and external market data are stored in separate Pinecone namespaces, enabling the reporting agent to query each independently or combine them for richer contextual analysis.

---

## 2. Problem Statement

Financial reporting requires aggregating information from multiple sources — internal metrics, market benchmarks, and external trend data — and synthesizing them into coherent, actionable reports. This process is time-consuming when done manually and inconsistent when delegated across teams.

The NovaCart AI Finance RAG System automates the full pipeline: from raw data ingestion and vector embedding to AI-driven report generation and email delivery.

---

## 3. Objectives

- Ingest internal financial metrics (Excel format) from Google Drive into Pinecone with row-level metadata tagging
- Ingest external market documents (PDF, DOCX, TXT) from Google Drive into a separate Pinecone namespace
- Generate structured financial reports using an AI Finance Reporting Agent (GPT-4o) querying both namespaces
- Deliver reports automatically via email on a configurable schedule
- Maintain semantic precision through dual namespace isolation and metadata-filtered retrieval

---

## 4. System Architecture — Three Pipelines

### Pipeline 1 — Internal Data Ingestion
- Schedule Trigger → Google Drive (List files) → Google Drive (Download each file)
- Switch node routes by file type: Excel → extract rows → format as key-value text → base64 encode → convert to binary → embed into Pinecone (internal namespace)
- PDF/DOCX/TXT → default data loader → recursive text splitter → embed into Pinecone (internal namespace)
- Metadata tags applied: source_type, week_start_date, product name, file name, MIME type

### Pipeline 2 — External Data Ingestion
- Parallel scheduled pipeline for external market data
- Google Drive (List) → Download → Default Data Loader → Recursive Text Splitter → Pinecone (external namespace)

### Pipeline 3 — AI Finance Reporting Agent
- Schedule Trigger → Set Reporting Period → Build Report Request → AI Finance Reporting Agent (GPT-4o)
- Agent equipped with two Pinecone tools: Pinecone Tool — Internal and Pinecone Tool — External
- Agent queries both namespaces to generate a comprehensive financial report
- Report formatted → delivered via email to configured recipient

---

## 5. Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Dual namespace strategy | Keeps internal metrics and external market data cleanly separated — agent can query each independently or combined |
| text-embedding-3-large at 1,024 dims | High-fidelity semantic retrieval balancing accuracy and performance |
| Row-level Excel serialization | Each row serialized as key-value text + base64 encoded, preserving structured data fidelity |
| Metadata tagging on all embeddings | source_type, week_start_date, product tags enable precise filtered retrieval |
| 200-token chunks / 20-token overlap | Preserves context coherence across chunk boundaries |
| Scheduled reporting trigger | Fully automated end-to-end — no manual intervention required |

---

## 6. Workflow Nodes (20)

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | Schedule Trigger | Schedule | Trigger Pipeline 1 ingestion |
| 2 | Google Drive (List) | Google Drive | List files in internal data folder |
| 3 | Google Drive (Download) | Google Drive | Download each file |
| 4 | Switch (Filter Types) | Switch | Route by file extension — xlsx vs pdf/docx/txt |
| 5 | Extract from File | Extract | Parse Excel rows |
| 6 | Format Rows to Text | Code | Serialize rows as key-value text + base64 encode |
| 7 | Convert to File | Convert | Convert formatted text to binary |
| 8 | Pinecone Vector Store (Excel) | Pinecone | Embed Excel rows into internal namespace |
| 9 | Data Loader (Excel) | Document Loader | Load Excel binary for embedding |
| 10 | Embeddings OpenAI | OpenAI | text-embedding-3-large — 1,024 dimensions |
| 11 | Pinecone Vector Store (Docs) | Pinecone | Embed documents into internal namespace |
| 12 | Default Data Loader | Document Loader | Load PDF/DOCX/TXT for embedding |
| 13 | Recursive Text Splitter | Text Splitter | 200-token chunks, 20-token overlap |
| 14 | Schedule Trigger 2 | Schedule | Trigger Pipeline 2 external ingestion |
| 15 | Pinecone Vector Store (External) | Pinecone | Embed external docs into external namespace |
| 16 | Schedule Trigger 3 | Schedule | Trigger Pipeline 3 reporting |
| 17 | Set Reporting Period | Code | Define report date range |
| 18 | Build Report Request | Code | Structure prompt for AI agent |
| 19 | AI Finance Reporting Agent | LLM Agent | GPT-4o queries Pinecone + generates report |
| 20 | Send Report via Email | Email | Deliver report to recipient |

---

## 7. Security Notes

- All API keys and credentials managed via n8n credential store — never hardcoded in workflow nodes
- Google Drive OAuth2 used for file access — no service account keys in workflow
- Pinecone API key stored as n8n environment variable
- All datasets contain synthetic/demo financial data — no real proprietary data in this repository

---

## 8. Academic Context

| Field | Detail |
|-------|--------|
| Program | Applied Agentic AI |
| Institution | Interview Kickstart |
| Week | Week 2 — RAG Systems |
| Semester | Spring 2026 |
| Author | Lamonte Smith |
| GitHub | github.com/LSmithPMP/novacart-finance-rag |

---

*Lamonte Smith · Interview Kickstart — Applied Agentic AI · March 2026 · Confidential*
