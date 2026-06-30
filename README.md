# DocChat — Bilingual RAG Web App 🇪🇹
### አማርኛ / English Document Intelligence

A modern, minimal, production-ready document QA platform.  
**Stack:** FastAPI · Gemini 1.5 Flash · ChromaDB · Vanilla JS (no framework)

---

## Features

| Feature | Detail |
|---------|--------|
| 📄 Multi-format | PDF, DOCX, TXT, PPTX |
| 🌍 Bilingual | Amharic (አማርኛ) + English auto-detection |
| ⚡ Streaming | Token-by-token streaming like ChatGPT |
| 🧠 RAG Pipeline | Chunk → Embed → Vector Store → Retrieve → Generate |
| 🔍 Semantic Search | Gemini text-embedding-004 |
| 🗃 Vector DB | ChromaDB (in-memory / persistent) |
| 🐳 Docker | One-command deployment |

---

## Architecture

```
Browser (HTML/CSS/JS)
        │  Upload files          │  SSE stream
        ▼                        ▼
  ┌─────────────────────────────────────────┐
  │           FastAPI Backend               │
  │                                         │
  │  /upload                /chat/stream    │
  │     │                        │          │
  │  Extract text           Embed query     │
  │  Chunk (800 chars)      ChromaDB search │
  │  Embed (Gemini)         Top-5 chunks    │
  │  ChromaDB.add()         Gemini generate │
  │                         SSE yield       │
  └─────────────────────────────────────────┘
```

---

## Quick Start

### 1. Clone & Configure

```bash
git clone https://github.com/YOUR_USERNAME/doc-chat
cd doc-chat
```

Set your Gemini API key:
```bash
export GEMINI_API_KEY="your-key-here"
```
Or create `.env`:
```
GEMINI_API_KEY=your-key-here
```

### 2a. Docker (Recommended)

```bash
docker-compose up --build
```

- Frontend: http://localhost:3000  
- Backend API: http://localhost:8000  
- API Docs: http://localhost:8000/docs  

### 2b. Local Dev

**Backend:**
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

**Frontend:**
```bash
# Any static server works
cd frontend
npx serve . -p 3000
# or: python -m http.server 3000
```

---

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET`  | `/health` | Status + doc count |
| `POST` | `/upload` | Upload & index files |
| `POST` | `/chat/stream` | SSE streaming QA |
| `GET`  | `/documents` | List indexed docs |
| `DELETE` | `/documents` | Clear all docs |

### Chat request body
```json
{
  "question": "ዋናው ሀሳብ ምንድን ነው?",
  "language": "auto"   // "auto" | "en" | "am"
}
```

### SSE stream format
```
data: {"token": "The", "sources": ["doc.pdf"]}
data: {"token": " main", "sources": ["doc.pdf"]}
...
data: {"done": true, "sources": ["doc.pdf"]}
```

---

## RAG Pipeline Detail

```
1. UPLOAD
   └─ Extract text (pypdf / python-docx / pptx / txt)
   └─ Chunk: 800 chars, 150 overlap
   └─ Embed: Gemini text-embedding-004 (batches of 100)
   └─ Store: ChromaDB with source metadata

2. QUERY
   └─ Embed user question
   └─ ChromaDB cosine similarity → Top 5 chunks
   └─ Build prompt: system + context + question
   └─ Gemini 1.5 Flash stream → SSE tokens → browser
```

---

## Publish to GitHub

```bash
git init
git add .
git commit -m "feat: bilingual RAG doc chat (Amharic + English)"
git remote add origin https://github.com/YOUR_USERNAME/doc-chat.git
git push -u origin main
```

---

## Deploy to Production

### Railway / Render
1. Push to GitHub
2. Connect repo → set `GEMINI_API_KEY` env var
3. Deploy backend (FastAPI) and frontend (static site) separately

### VPS (Ubuntu)
```bash
# Install docker
curl -fsSL https://get.docker.com | sh
# Clone and run
git clone ... && cd doc-chat
GEMINI_API_KEY=your-key docker-compose up -d
```

---

## Language Detection

The backend prompt instructs Gemini to:
- Detect whether the question is in **Amharic** or **English**
- Respond in the **same language** automatically
- Manual override: click `አማርኛ` or `English` button in the UI

---

## Security Note

⚠️ Replace the hardcoded API key in `main.py` with an environment variable before deploying publicly:
```python
GEMINI_API_KEY = os.getenv("GEMINI_API_KEY")
```

---

*Built with ❤️ for Ethiopia 🇪🇹*
