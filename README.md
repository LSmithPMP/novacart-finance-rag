# NovaCart AI Finance RAG System

> **Applied Agentic AI Program · Week 2 Assignment · Interview Kickstart · Spring 2026**  
> Lamonte Smith

A dual-pipeline **RAG (Retrieval-Augmented Generation)** system built in n8n that ingests internal financial data and external documents into Pinecone, then uses an AI Finance Reporting Agent to generate structured financial reports delivered via email.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  PIPELINE 1 — DATA INGESTION                 │
│                                                              │
│  Schedule Trigger                                            │
│       │                                                      │
│       ▼                                                      │
│  Google Drive (List) ──► Google Drive (Download)             │
│                                │                             │
│                                ▼                             │
│                    Switch (Filter by Type)                   │
│                    ┌───────────┴──────────┐                  │
│                    ▼                      ▼                  │
│              XLSX Branch           PDF/DOCX/TXT Branch       │
│                    │                      │                  │
│            Extract from File      Default Data Loader        │
│                    │                      │                  │
│         Format Rows to Text    Recursive Text Splitter       │
│                    │                      │                  │
│           Convert to File                 │                  │
│                    │                      │                  │
│                    ▼                      ▼                  │
│         Pinecone Vector Store    Pinecone Vector Store       │
│            (Excel — internal)      (Docs — internal)         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│               PIPELINE 2 — EXTERNAL DATA INGESTION           │
│                                                              │
│  Schedule Trigger                                            │
│       │                                                      │
│       ▼                                                      │
│  Google Drive (List) ──► Google Drive (Download)             │
│                                │                             │
│                                ▼                             │
│                       Default Data Loader                    │
│                                │                             │
│                    Recursive Text Splitter                   │
│                                │                             │
│                                ▼                             │
│                  Pinecone Vector Store (external)            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│               PIPELINE 3 — AI REPORTING AGENT                │
│                                                              │
│  Schedule Trigger                                            │
│       │                                                      │
│       ▼                                                      │
│  Set Reporting Period                                        │
│       │                                                      │
│       ▼                                                      │
│  Build Report Request                                        │
│       │                                                      │
│       ▼                                                      │
│  AI Finance Reporting Agent (GPT-4o)                         │
│  ├── Pinecone Tool — Internal data                           │
│  └── Pinecone Tool — External data                           │
│       │                                                      │
│       ▼                                                      │
│  Prepare Email                                               │
│       │                                                      │
│       ▼                                                      │
│  Send Report via Email                                       │
└─────────────────────────────────────────────────────────────┘
```

---

## How It Works

**Pipeline 1 — Internal Data Ingestion**
Scheduled ingestion pulls files from the `NovaCart_Assignment_Internal_Data` Google Drive folder. A Switch node routes by file type — Excel files are extracted, formatted row-by-row, converted to binary, and embedded into Pinecone under the `internal` namespace. PDF/DOCX/TXT files are chunked and embedded directly.

**Pipeline 2 — External Data Ingestion**
A parallel scheduled pipeline ingests external market and reference documents from a separate Google Drive folder into Pinecone's `external` namespace.

**Pipeline 3 — AI Finance Reporting Agent**
On schedule, a reporting period is set, a structured report request is built, and an AI Finance Reporting Agent (GPT-4o) queries both Pinecone namespaces (internal metrics + external context) to generate a comprehensive financial report — then delivers it via email.

---

## Key Design Decisions

- **Dual namespace strategy** — internal financial metrics and external market data stored in separate Pinecone namespaces, enabling the agent to query each independently or combined
- **Excel row formatting** — each row serialized as key-value text and base64-encoded before embedding, preserving structured data fidelity in the vector store
- **Metadata tagging** — embeddings tagged with `source_type`, `week_start_date`, `product`, `file_name`, and `drive_url` for precise retrieval filtering
- **text-embedding-3-large** — 1024-dimension embeddings for high-fidelity semantic search
- **Recursive character text splitting** — 200-token chunks with 20-token overlap for coherent retrieval

---

## Workflow Nodes

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | Schedule Trigger | Schedule | Trigger ingestion pipeline |
| 2 | Google Drive (List) | Google Drive | List files in target folder |
| 3 | Google Drive (Download) | Google Drive | Download each file |
| 4 | Switch (Filter Types) | Switch | Route by file extension (xlsx vs pdf/docx/txt) |
| 5 | Extract from File | Extract | Parse Excel rows |
| 6 | Format Rows to Text | Code | Serialize rows as key-value text + base64 encode |
| 7 | Convert to File | Convert | Convert formatted text to binary |
| 8 | Pinecone Vector Store (Excel) | Pinecone | Embed Excel rows into internal namespace |
| 9 | Data Loader (Excel) | Document Loader | Load Excel binary for embedding |
| 10 | Embeddings OpenAI | OpenAI | text-embedding-3-large (1024 dims) |
| 11 | Pinecone Vector Store (Docs) | Pinecone | Embed documents into internal namespace |
| 12 | Default Data Loader | Document Loader | Load PDF/DOCX/TXT for embedding |
| 13 | Recursive Text Splitter | Text Splitter | 200-token chunks, 20-token overlap |
| 14 | Set Reporting Period | Code | Define report date range |
| 15 | Build Report Request | Code | Structure prompt for AI agent |
| 16 | AI Finance Reporting Agent | LLM Agent | GPT-4o queries Pinecone + generates report |
| 17 | Pinecone Tool — Internal | Pinecone | Agent tool for internal data retrieval |
| 18 | Pinecone Tool — External | Pinecone | Agent tool for external data retrieval |
| 19 | Prepare Email | Code | Format report for email delivery |
| 20 | Send Report via Email | Email | Deliver report to recipient |

---

## Tech Stack

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat&logo=n8n&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI_GPT--4o-412991?style=flat&logo=openai&logoColor=white)
![Pinecone](https://img.shields.io/badge/Pinecone-000000?style=flat&logo=pinecone&logoColor=white)
![Google Drive](https://img.shields.io/badge/Google_Drive-4285F4?style=flat&logo=googledrive&logoColor=white)

**Models:** GPT-4o (reporting agent) · text-embedding-3-large (embeddings)  
**Vector Store:** Pinecone — dual namespace (internal + external)  
**Data Sources:** Google Drive (Excel, PDF, DOCX, TXT)  
**Patterns:** RAG · Dual-pipeline ingestion · Scheduled automation · AI agent reporting

---

## Environment Variables

```bash
OPENAI_API_KEY=       # Required: OpenAI API key for embeddings and GPT-4o
PINECONE_API_KEY=     # Required: Pinecone API key
GOOGLE_DRIVE_CREDS=   # Required: Google Drive OAuth2 credentials (managed via n8n)
EMAIL_CREDS=          # Required: Email credentials for report delivery
```

> ⚠️ Never commit actual values. Use n8n credential store for all secrets.

---

## Academic Context

| Field | Detail |
|-------|--------|
| Program | Applied Agentic AI |
| Institution | Interview Kickstart |
| Week | Week 2 — RAG Systems |
| Semester | Spring 2026 |

---

## Repository Structure

```
novacart-finance-rag/
├── workflow/
│   └── novacart-finance-rag.json     # n8n workflow export
├── SECURITY.md
└── README.md
```

---

## Security Notes

- All API keys and credentials managed via n8n credential store — never hardcoded
- Google Drive OAuth2 used for file access — no service account keys in workflow
- Pinecone API key stored as n8n environment variable
- See [SECURITY.md](SECURITY.md) for full policy

---

## License

MIT License — see [LICENSE](LICENSE)

---

<div align="center">
<sub>Built by <a href="https://github.com/LSmithPMP">Lamonte Smith</a> · Applied Agentic AI · Interview Kickstart · Spring 2026</sub>
</div>
