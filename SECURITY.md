# Security Policy

## Credentials & Secrets

All credentials in this workflow are managed through n8n's built-in credential store:

- **OpenAI API key** — stored as named n8n credential, never hardcoded
- **Pinecone API key** — stored as named n8n credential
- **Google Drive OAuth2** — stored as named n8n credential
- **Email credentials** — stored as named n8n credential

No API keys, tokens, or secrets appear in workflow node logic or this repository.

## Data Handling

- This workflow processes **synthetic/demo financial data** for academic purposes only
- No real financial data, PII, or proprietary business data is stored in this repository
- Pinecone vector store contains embeddings of demo data only
- Google Drive folder referenced contains assignment data only

## Reporting a Vulnerability

1. **Do not** open a public GitHub issue
2. Email: lamontesmithpmp@gmail.com with subject line `[SECURITY] novacart-finance-rag`
3. Expected response time: 72 hours

## Disclaimer

This workflow was built for academic demonstration purposes as part of the Interview Kickstart Applied Agentic AI program. Before deploying to production with real financial data, conduct a full security and compliance review appropriate to your organization's requirements.
