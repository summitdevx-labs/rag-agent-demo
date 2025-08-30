# 📌 AI Retrieval & Web-Augmented Agent

An experimental AI agent built with **LangChain + OpenAI + FAISS**, designed to:
1. Take user input (queries).
2. Retrieve relevant context from a **local FAISS vector database** (PDF embeddings).
3. Summarize results via LLM.
4. If the answer is not satisfactory, call a **web search tool** to fetch missing context.
5. Return a final, refined answer.