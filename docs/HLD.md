# High-Level Design (HLD)

This document describes the high-level architecture of the **Experimental AI Agent** built with LangChain, OpenAI, and FAISS.  

## 🔹 Key Features
- Natural-language user queries  
- Local context retrieval from FAISS (PDF embeddings)  
- LLM-based summarization & response generation (OpenAI through LangChain)  
- Web-search fallback when local context is insufficient  
- Final, consolidated, user-friendly answer  
- Modular design for easy swap-in of components (vector DB, LLM, search)  

---

## 🔹 HLD Diagram

```mermaid
flowchart LR
    Q[Natural-language User Queries]
    F["Local Context Retrieval (FAISS - PDF Embeddings)"]
    L["LLM Summarization & Response (OpenAI via LangChain)"]
    W["Web Search Fallback (if local context insufficient)"]
    A["Final Consolidated Answer (User-friendly Output)"]
    M["["Modular Design (Swap DB, LLM, Search)"]]

    Q -->|retrieve context| F
    F -->|summarize & answer| L
    L -->|if not sufficient| W
    W -->|refined context| L
    L -->|final response| A

    M -.-> F
    M -.-> L
    M -.-> W