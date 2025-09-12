# 📌 Experimental AI Agent — LangChain + OpenAI + FAISS

Lightweight experimental agent that answers user queries by retrieving local PDF context (FAISS), summarizing with an LLM (OpenAI via LangChain), and — when needed — falling back to a web search to refine answers.

Key features

- Natural-language user queries

- Local context retrieval from FAISS (PDF embeddings)

- LLM-based summarization & response generation (OpenAI through LangChain)

- Web-search fallback when local context is insufficient

- Final, consolidated, user-friendly answer

- Modular design for easy swap-in of components (vector DB, LLM, search)

Table of contents

- [Why this project](#why-this-project)

- [Architecture](#architecture-high-level)

- [Quickstart](#quickstart-local-dev)

- [Configuration](#configuration-important-env-variables)

- [How it works (workflow)](#how-it-works-workflow)

- [Component responsibilities](#component-responsibilities)

- [Success criteria & SLAs](#success-criteria--slas)

- [Testing & validation](#testing--validation)

- [Monitoring & observability](#monitoring--observability)

- [Security & privacy](#security--privacy)

- [Roadmap & Future improvements](#roadmap--future-improvements)

- [License](#license)

## Why this project

Useful when you have a body of PDFs (internal docs, manuals, research papers) and want an agent that:

1. Retrieves relevant passages quickly (FAISS),

2. Produces concise, high-quality answers (OpenAI LLM),

3. Supplements knowledge from the web only when the local corpus is missing key context.

## Architecture (high level)

User → Agent API → FAISS (local) → LLM (OpenAI via LangChain) → (optional) Web Search → Final Answer

## Quickstart (local dev)

Assumes Python 3.10+, pip, and a local FAISS index of PDF embeddings.

1. Clone repo:

```bash
git clone https://github.com/summitdevx-labs/rag-agent-demo.git
cd rag-agent-demo
```
2. Create venv and install:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

3. Set environment variables (example):

```bash
export OPENAI_API_KEY="sk-..."
export FAISS_INDEX_PATH="./data/faiss.index"
export PDF_EMBEDDINGS_DIR="./data/pdf_embeddings"
export WEB_SEARCH_API_KEY="..."   # optional/fallback
```

4. Start (dev):

```bash
python app/main.py    # or `uvicorn app.main:app --reload`
```

5. Example curl:

```bash
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -d '{"query":"Generate a concise overview of the PDF content in under 100 words."}'
```

## Configuration (important env variables)

- OPENAI_API_KEY — OpenAI credentials

- FAISS_INDEX_PATH — path to FAISS index file

- PDF_EMBEDDINGS_DIR — directory containing chunk->metadata mapping (JSON/CSV)

- WEB_SEARCH_API_KEY — optional; required if web search fallback used

- MAX_FAISS_RESULTS — number of top-k vectors to retrieve (default: 3)

- SIMILARITY_THRESHOLD — threshold to judge "sufficient local context" (e.g., 0.75)

## How it works (workflow)

1. **Receive query** from user.

2. **Retrieve** top-K similar document chunks from FAISS (PDF embeddings).

3. **Judge sufficiency**: if retrieved context score(s) ≥ SIMILARITY_THRESHOLD, proceed.

    - If sufficient → pass retrieved chunks + user query to LLM.

    - If insufficient → call web search, collect results, merge with local context, then call LLM.

4. **LLM**: summarize and produce an initial answer.

5. **Refinement**: optionally re-run a shorter retrieval or chain-of-thought prompt to improve clarity.

6. **Return** final refined answer to user (plus provenance: source docs, URLs used).


## Component responsibilities

- **API Layer**: Accepts user queries, returns final answers + metadata.

- **FAISS Retriever**: Provides nearest-neighbor search over PDF embeddings.

- **Embeddings Pipeline**: Script to chunk PDFs and compute embeddings (OpenAI embeddings or other).

- **LangChain Orchestrator**: Chains retrieval → LLM → decision logic → fallback.

- **LLM (OpenAI)**: Summarization, answer generation, confidence scoring.

- **Web Search Adapter**: Optional; triggered on low-confidence / low-recall.

- **Provenance/Logging**: Tracks which sources (pdf id, page, web URL) contributed to answer.

## Example prompt chain (pseudo)

```csharp
[System] You are an assistant that uses supplied document excerpts and web snippets.
[User] Query: <user query>

[Context] Top FAISS chunks: {chunk1, chunk2, ...}
[Assistant Task] Summarize and answer using context. If missing details, say "INSUFFICIENT" and list missing facts.

-> If "INSUFFICIENT", call web search and repeat with new context.
```

## Success criteria & SLAs

- **Accuracy**: ≥ 80% of queries answered fully from FAISS context alone (measured via test corpus).

- **Performance**:

    - FAISS-only path: median response < 5s.

    - With web fallback: median response < 10s.

- **Reliability**: 99% uptime for the agent service (SLA target).

- **Traceability**: Every response includes provenance (document id/page, or web URL).

## Testing & validation

Unit tests for retriever, embedding loader, and LangChain prompt templates.

End-to-end tests using held-out PDF corpus + ground-truth Q&A.

Synthetic "insufficient context" tests to validate fallback to web search.

Performance tests (load testing for concurrent queries).

## Monitoring & observability

Request latency and error rates (LangSmith/Prometheus + Grafana)

Retrieval statistics (FAISS recall, average similarity)

LLM tokens used and cost per request

Alerts for web search failure or FAISS index corruption

## Security & privacy

Store API keys in secure vault (do not hardcode).

PDFs and embeddings stored on disk with proper FS permissions.

Redact PII before sending context to external web search (if policy requires).

Audit logs for queries and data access.

## Roadmap & Future improvements

Swap FAISS with cloud vector DB (e.g., Pinecone/Weaviate) when needed.

Add RAG caching for repeated queries.

Add human-in-the-loop feedback and retraining pipeline.

Add UI with conversational memory & session history.

## Contact / Maintainer

Maintainer: 

Repo: https://github.com/summitdevx-labs/rag-agent-demo/tree/main

## License

MIT (or choose your preferred OSS license)



