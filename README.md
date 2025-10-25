# Q-A RAG App

A Retrieval-Augmented Generation (RAG) Question & Answer application implemented in Python. The app indexes documents into a vector store, retrieves relevant passages for a user query, and uses a language model to generate answers grounded in those sources.

Repository: https://github.com/bibas06/Q-A-RAG-App

Table of contents
- Overview
- Features
- Requirements
- Installation
- Configuration
- Quick start
- CLI usage
- API usage
- Indexing / Ingest
- Testing
- Deployment
- Contributing
- Contact

Overview
--------
Q-A RAG App is designed to make factual, citeable answers by combining retrieval from a vector database with a generative model. The retrieval step provides context from your documents; the generator composes an answer using only that context (and cites sources).

Features
--------
- Document ingestion (PDF, TXT, Markdown, DOCX, HTML)
- Chunking with configurable size & overlap
- Pluggable embedding providers (OpenAI, Hugging Face, etc.)
- Pluggable vector stores (FAISS locally, Pinecone, Milvus)
- Retriever with k-NN and simple hybrid options
- LLM-based answer generation with citations
- CLI for ingestion and queries
- Small web API to query the index

Requirements
------------
- Python 3.9+
- pip
- Optional: Docker & Docker Compose (for containerized vector dbs)

Core Python packages (example)
- numpy
- faiss-cpu (optional)
- sentence-transformers or openai
- fastapi / uvicorn (for API)
- python-dotenv
- tika, pdfminer.six, python-docx (for parsing various doc types)
Install via requirements.txt provided in repo.

Installation
------------
Clone the repo:
```bash
git clone https://github.com/bibas06/Q-A-RAG-App.git
cd Q-A-RAG-App
```

Create and activate a virtual environment:
```bash
python -m venv .venv
source .venv/bin/activate    # macOS/Linux
.venv\Scripts\activate       # Windows (PowerShell)
pip install -r requirements.txt
```

Configuration
-------------
Create a `.env` file in the project root. Example variables:

```env
# LLM / embedding provider
OPENAI_API_KEY=sk-...
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_COMPLETION_MODEL=gpt-4o-mini

# Vector DB configuration (choose one)
VECTOR_DB=faiss           # or pinecone, milvus, etc.

# Pinecone example
PINECONE_API_KEY=
PINECONE_ENVIRONMENT=
PINECONE_INDEX_NAME=qa-index

# App settings
CHUNK_SIZE=1000
CHUNK_OVERLAP=200
TOP_K=5
N_WORKERS=4

# Web/API
PORT=8000
```

Quick start
-----------
1. Ingest (index) your documents:
```bash
python scripts/ingest.py --source ./data
```

2. Run the API:
```bash
uvicorn app:app --reload --port ${PORT:-8000}
```

3. Query via CLI:
```bash
python scripts/query.py --q "What is retrieval-augmented generation?"
```

Or query API:
```bash
curl -X POST "http://localhost:8000/query" -H "Content-Type: application/json" -d '{"query": "What is RAG?"}'
```

CLI usage
---------
- scripts/ingest.py
  - --source PATH : folder or file to ingest
  - --chunk-size N : chunk size override
  - --chunk-overlap N : overlap override

- scripts/query.py
  - --q "your question"
  - --top-k N : how many documents to retrieve

API usage
---------
- POST /query
  - Body: {"query": "Your question", "top_k": 5}
  - Response: {"answer": "...", "sources": [{"doc_id": "...", "score": 0.95, "excerpt": "..."}]}

Indexing / Ingest
-----------------
- Document parsing: repo includes parsers for common file types. New parsers can be added under `parsers/`.
- Chunking: Documents are broken into overlapping chunks using CHUNK_SIZE and CHUNK_OVERLAP.
- Embeddings: Each chunk is converted into an embedding via the configured provider.
- Vector store: Embeddings are inserted into the configured vector database.

Common ingestion command:
```bash
python scripts/ingest.py --source ./data --namespace my-docs
```

Testing
-------
- Unit tests: pytest
```bash
pytest
```
- Integration tests: require valid API keys and optionally a running vector DB; these tests are separated and marked accordingly.

Docker
------
A Dockerfile and docker-compose.yml can be added or updated to include:
- The application container
- Optional local FAISS service or a dockerized Pinecone-compatible service
Example build & run:
```bash
docker build -t qa-rag-app:latest .
docker run -e OPENAI_API_KEY="$OPENAI_API_KEY" -p 8000:8000 qa-rag-app:latest
```

Deployment
----------
- Small deployments: single container (Docker).
- Production: use Kubernetes, managed vector DB (Pinecone/Weaviate), and hosted LLMs for scalability.
- Securely store keys using your cloud provider's secret manager.

Security & privacy
------------------
- Never commit your API keys or .env to source control.
- If your documents contain sensitive data, configure access control and consider on-prem or private cloud for vector stores and LLM calls.

Contributing
------------
Contributions are welcome:
1. Fork the repo
2. Create a branch: git checkout -b feature/your-feature
3. Add tests for new behavior
4. Open a pull request describing the change

Follow the repository's code style and include tests for significant changes.

Contact
-------
Maintainer: bibas06
Repo: https://github.com/bibas06/Q-A-RAG-App

Notes
-----
This README is tailored for a Python-first implementation. If you'd like a version targeted at a different structure (for example, explicit examples for Pinecone or FAISS configurations, or a full Docker Compose with vector DB), tell me which stack you prefer and I will extend this README accordingly.
