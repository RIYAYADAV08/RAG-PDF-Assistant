# 📄 RAG PDF Assistant

A **local, privacy-friendly AI assistant** built with **FastAPI, Ollama and FAISS**. Upload a PDF, ask questions about it, and get answers grounded in the document with **page-level source citations**. It also includes a chat API with persistent conversation memory stored in SQLite.

Everything runs **locally**: no API keys and no data leaves your machine.

---

## 📌 Project Overview

The project implements a complete **Retrieval-Augmented Generation (RAG)** pipeline:

1. **Ingest**: extract text from a PDF page by page and split it into overlapping chunks.
2. **Embed**: convert each chunk into a vector using `nomic-embed-text` (via Ollama).
3. **Store**: save the vectors in a **FAISS** index on disk.
4. **Retrieve**: embed the question and fetch the most similar chunks (cosine similarity).
5. **Generate**: send the retrieved context to a local LLM and return an answer with citations like `(Source 1)`.

---

## ✨ Features

- Upload PDFs and index them for semantic search
- Question answering strictly from document content
- Source citations with **filename, page number and snippet**
- Local LLM and embeddings through **Ollama** (no external API)
- **FAISS** vector index (cosine similarity) persisted on disk
- Chat API with **conversation memory** stored in **SQLite**
- Optional **filename filter** and answer deduplication in the lightweight RAG service
- Input validation with Pydantic and clear error messages (for example, when Ollama is not running)
- Health-check endpoint and basic automated test

---

## 🛠️ Tech Stack

| Technology | Purpose |
|------------|---------|
| Python 3.10+ | Programming language |
| FastAPI | REST API framework |
| Ollama | Local LLM and embeddings runtime |
| FAISS | Vector similarity search |
| SQLAlchemy + SQLite | Chat history and chunk storage |
| pypdf | PDF text extraction |
| NumPy | Vector operations |
| Pydantic | Request/response validation |
| Pytest | Testing |

**Models used:** `llama3.2:3b` (chat, `main.py`), `qwen2.5:1.5b` (chat, `rag_service.py`), `nomic-embed-text` (embeddings).

---

## 📂 Project Structure

```text
RAG-PDF-Assistant/
├── main.py              # Full app: chat memory (SQLite) + PDF RAG (FAISS)
├── rag_service.py       # Lightweight RAG-only service with filename filter
├── tests/
│   └── test_health.py   # Health endpoint test
├── .gitignore
└── README.md
```

> **Note:** `main.py` and `rag_service.py` are two separate apps that both use the `./rag_store` folder with different metadata formats. Run **one at a time**, or point each to its own storage folder.

---

## 🔗 API Endpoints

### `main.py` (Chat + RAG)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| POST | `/chat/new` | Start a new conversation with a system prompt |
| POST | `/chat/{conversation_id}` | Send a message and get a reply (history is remembered) |
| GET | `/history/{conversation_id}` | Fetch full conversation history |
| POST | `/rag/ingest` | Upload and index a PDF |
| POST | `/rag/query` | Ask a question about indexed PDFs |

### `rag_service.py` (RAG only)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| POST | `/rag/ingest` | Upload and index a PDF |
| POST | `/rag/query` | Ask a question (optional `filename` filter) |

### Example

```bash
# Ingest a PDF
curl -X POST http://127.0.0.1:8000/rag/ingest -F "file=@document.pdf"

# Ask a question
curl -X POST http://127.0.0.1:8000/rag/query \
  -H "Content-Type: application/json" \
  -d '{"question": "What are the working hours?", "top_k": 5}'
```

Sample response:

```json
{
  "answer": "- Working hours are 9 AM to 6 PM (Source 1)",
  "sources": [
    { "filename": "document.pdf", "page_num": 1, "snippet": "..." }
  ],
  "model": "llama3.2:3b",
  "latency_ms": 2400
}
```

---

## ⚙️ Getting Started

### Prerequisites

- Python 3.10 or higher
- [Ollama](https://ollama.com) installed and running

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/RAG-PDF-Assistant.git
cd RAG-PDF-Assistant

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install fastapi "uvicorn[standard]" requests numpy pypdf faiss-cpu sqlalchemy python-multipart pytest httpx

# 4. Pull the Ollama models
ollama pull llama3.2:3b
ollama pull qwen2.5:1.5b
ollama pull nomic-embed-text

# 5. Run the app
uvicorn main:app --reload
# or the lightweight RAG-only service:
# uvicorn rag_service:app --reload
```

Open **http://127.0.0.1:8000/docs** for the interactive Swagger UI.

### Run Tests

```bash
pytest
```

---

## 🧠 How Chunking Works

- Page text is cleaned and split into chunks of **900 characters** with **180 characters of overlap**, so context is not lost at chunk boundaries.
- Each chunk keeps its **filename and page number**, which powers the citations.

---

## 🚀 Future Improvements

- Add a `requirements.txt` and Docker support
- Frontend UI (React) for upload and chat
- Streaming responses
- Support for DOCX and TXT files
- Delete or re-index documents
- Hybrid search (BM25 + vectors) and re-ranking
- More tests for ingest and query endpoints

---

## 👩‍💻 Author

**Riya Yadav**


🔗 [LinkedIn](https://www.linkedin.com/in/riyayadav0803/)
