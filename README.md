# DeskPilot

An internal IT support assistant that retrieves private company guidance first, checks whether the evidence is adequate, and uses Tavily web search only when internal evidence is weak. LangGraph coordinates the flow; Gemini provides chat and embedding models; Pinecone stores the knowledge base.

## Setup

```powershell
conda create -n project python=3.11 -y
conda activate project
pip install -r requirements.txt
Copy-Item .env.example .env
```

Fill in `.env` with `GEMINI_API_KEY`, `PINECONE_API_KEY`, and `TAVILY_API_KEY`. The Gemini chat and embedding models both use `GEMINI_API_KEY`. The default embedding model is `models/gemini-embedding-001` (768 dimensions), and the default Pinecone index is `fde-it-support-rag-gemini`.

Start the service with:

```powershell
python run.py
```

## Knowledge base migration

Changing embedding providers requires re-embedding all documents. Existing OpenAI vectors cannot be queried with Gemini embeddings. Create/use a fresh Pinecone index, then re-ingest the source documents. The app checks index dimensions and stops with an actionable error if they do not match; it does not delete an existing index.

## Latency note

Greetings and thanks are handled locally, and support questions go directly to private retrieval without an LLM routing call. For a successful private-KB answer, evidence assessment and answer drafting share one Gemini call. Empty retrieval results skip grading. These changes reduce avoidable model round trips; measure p50/p95 latency on a fixed question set before claiming a numeric improvement.
