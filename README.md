# StudyMate AI

An AI-powered learning companion for students. Upload study notes (PDF/TXT), get AI-generated summaries and flashcards, take quizzes, chat with your documents using RAG, and track progress on an analytics dashboard.

## Stack

| Layer | Technology |
|-------|------------|
| Frontend | React (Vite), Tailwind CSS, React Router, Axios, Recharts |
| Backend | Node.js, Express |
| Database | MongoDB (Mongoose) |
| Auth | JWT in httpOnly cookies, bcrypt password hashing |
| AI | Google Gemini API (text generation + embeddings) |
| Vector search | Cosine similarity over MongoDB-stored embeddings |

## Features

- **Authentication** — Register, login, logout with protected routes
- **Document upload** — PDF/TXT parsing, chunking, embedding, vector indexing
- **AI summary & flashcards** — LLM-generated summary and 10 flip-card flashcards
- **AI quiz** — 10-question multiple choice with instant scoring
- **Chat with notes (RAG)** — Ask questions grounded in your uploaded documents
- **Analytics dashboard** — Quiz score history, weak topics, usage stats

## Project Structure

```
studymate-ai/
├── backend/
│   └── src/
│       ├── config/       # Database connection
│       ├── models/       # Mongoose schemas (User, Document, Chunk, etc.)
│       ├── routes/       # Express API routes
│       ├── services/     # AI, vector store, document processing
│       ├── middleware/   # Auth, validation, error handling
│       └── utils/        # Text chunking
├── frontend/
│   └── src/
│       ├── api/          # Axios API client
│       ├── components/   # Reusable UI components
│       ├── context/      # Auth & toast state
│       └── pages/        # Route pages
└── README.md
```

## Setup

### Prerequisites

- Node.js 18+
- MongoDB (local or [MongoDB Atlas](https://www.mongodb.com/atlas))
- [Google Gemini API key](https://aistudio.google.com/apikey)

### 1. Clone and configure

```bash
cd studymate-ai
cp .env.example backend/.env
# Edit backend/.env with your MONGO_URI, JWT_SECRET, and GEMINI_API_KEY
```

### 2. Backend

```bash
cd backend
npm install
npm run dev
```

Server runs at `http://localhost:5000`.

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

App runs at `http://localhost:5173` (API proxied to backend).

## API Routes

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/auth/register` | Register new user |
| POST | `/api/auth/login` | Login |
| POST | `/api/auth/logout` | Logout |
| GET | `/api/auth/me` | Current user |
| POST | `/api/documents/upload` | Upload PDF/TXT |
| GET | `/api/documents` | List user's documents |
| GET | `/api/documents/:id` | Document detail |
| POST | `/api/documents/:id/generate-summary` | Generate summary |
| POST | `/api/documents/:id/generate-flashcards` | Generate flashcards |
| POST | `/api/documents/:id/generate-quiz` | Generate quiz |
| POST | `/api/documents/:id/chat` | RAG chat `{ question }` |
| POST | `/api/documents/:id/quiz/:quizId/submit` | Submit quiz answers |
| GET | `/api/analytics/dashboard` | Analytics data |

## RAG Pipeline (Retrieval-Augmented Generation)

StudyMate AI uses RAG so chat answers are grounded in the user's uploaded notes, not generic LLM knowledge.

### Indexing (at document upload)

1. **Extract text** — `pdf-parse` for PDFs, plain read for `.txt`
2. **Chunk** — Split into ~500-token segments with overlap (`utils/chunkText.js`)
3. **Embed** — Each chunk is sent to Gemini's embedding API → vector (float array)
4. **Store** — Chunks + embeddings saved in MongoDB (`Chunk` collection)

### Retrieval (at chat query)

1. **Embed query** — User question → same embedding model → query vector
2. **Similarity search** — Cosine similarity between query vector and all document chunk vectors (`services/vectorStore.js`)
3. **Top-K selection** — Top 5 most similar chunks retrieved as context

### Generation

1. Retrieved chunks are formatted as context blocks
2. Context + question sent to Gemini with instruction to answer only from context
3. Response returned with source chunk indices and relevance scores

See inline comments in:

- `backend/src/services/vectorStore.js` — cosine similarity & retrieval
- `backend/src/services/documentService.js` — `chatWithDocument()` RAG orchestration
- `backend/src/models/Chunk.js` — embedding storage schema

### Why app-level vector search?

MongoDB Atlas Vector Search requires an Atlas cluster with a vector index. This project stores embeddings in MongoDB and computes cosine similarity in Node.js for simplicity and local development. For production at scale, migrate to Atlas Vector Search or a dedicated vector DB.

## Architecture Diagram

```
┌─────────────┐     httpOnly JWT      ┌─────────────┐
│   React     │ ◄──────────────────► │   Express   │
│   Frontend  │      REST API         │   Backend   │
└─────────────┘                       └──────┬──────┘
                                             │
                    ┌────────────────────────┼────────────────────────┐
                    ▼                        ▼                        ▼
              ┌──────────┐            ┌─────────────┐           ┌──────────┐
              │ MongoDB  │            │ Gemini API  │           │ pdf-parse│
              │ (docs,   │            │ (LLM +      │           │ (upload) │
              │ chunks,  │            │ embeddings) │           └──────────┘
              │ quizzes) │            └─────────────┘
              └──────────┘
```

## License

MIT
