# Personal Wishes Document Intake Assistant

An engineering demonstration of reliable, decoupled application design around Large Language Models (LLMs). The application conducts a conversational interview to compile a fictional **Personal Wishes Document** while enforcing a strict separation between unstructured conversation history and canonical structured state.

The project guarantees that the LLM acts purely as an extraction and reasoning engine—it never modifies the database directly, cannot invent unsupplied facts, and cannot corrupt application state due to strict runtime schema validation.

---

## Key Architectural Principles

1. **Decoupled Canonical State:** Unstructured conversational history (`Message`) and ground-truth data (`DocumentState`) live in separate collections. Conversation history is never used as the source of truth for compilation.
2. **Zero-Trust LLM Boundary:** The model only *proposes* updates matching a defined JSON contract. All responses pass through strict Zod schema validation before touching application memory or storage.
3. **Deterministic Document Generation:** The draft document is generated deterministically from confirmed canonical state via code templates, not synthesized freeform by an LLM.
4. **Resilient Provider Layer:** Includes both a live Gemini integration with automatic fallback/retry logic and a deterministic `MockLLMService` allowing full offline execution and testing without an active API key.
5. **Stateful Inquiries & Corrections:** If a user corrects a field (e.g., changing executor to Satvik) or queries previously recorded information (e.g., "who is my executor?"), the engine updates canonical state without overwriting unrelated fields and answers directly from the confirmed state.

---

## Technology Stack

- **Frontend:** React.js (v18), Vite, Vanilla CSS, Native Fetch API (No heavy UI frameworks)
- **Backend:** Node.js, Express.js (REST API architecture), CORS, dotenv
- **Database:** MongoDB Atlas with Mongoose ODM
- **LLM SDK:** Official Google Gen AI JavaScript SDK (`@google/generative-ai`)
- **Runtime Validation:** Zod
- **Testing:** Vitest, Supertest

---

## 3. Architecture

Document Intake Assistant
|
|-- Frontend (React + Vite)
|      |
|      |-- App.jsx
|      |      (session management, application state, API coordination)
|      |
|      |-- components/Chat.jsx
|      |      (chat messages, user input, loading and errors)
|      |
|      |-- components/StructuredState.jsx
|      |      (captured fields, progress, direct editing)
|      |
|      |-- components/DocumentPreview.jsx
|      |      (document preview, copy text, PDF download)
|      |
|      |-- services/api.js
|             (HTTP requests to backend using fetch)
|
|                 REST API (JSON)
|                         |
|                         v
|-- Backend (Node.js + Express)
|      |
|      |-- src/server.js
|      |      (starts server and database connection)
|      |
|      |-- src/app.js
|      |      (Express setup, middleware, routes)
|      |
|      |-- routes/conversation.routes.js
|      |      (session, message and state endpoints)
|      |
|      |-- controllers/conversation.controller.js
|      |      (handles HTTP requests and responses)
|      |
|      |-- services/conversation.service.js
|      |      (orchestrates one conversation turn)
|      |             |
|      |             |-------------------------------|
|      |             v                               v
|      |      services/llm/                    models/
|      |      |                                |
|      |      |-- llm.factory.js               |-- Session.js
|      |      |      (selects LLM service)      |-- Message.js
|      |      |                                |-- DocumentState.js
|      |      |-- gemini.service.js             |
|      |      |      (Gemini API integration)   v
|      |      |                           config/db.js
|      |      |-- groq.service.js               (MongoDB connection)
|      |      |      (Groq API integration)          |
|      |      |                                     v
|      |      |-- mock-llm.service.js        MongoDB
|      |             (mock responses)        (sessions, messages,
|      |                                      structured state)
|      |
|      |-- prompts/intake.prompt.js
|      |      (instructions for extracting user information)
|      |
|      |-- schemas/llm-response.schema.js
|      |      (validates expected LLM response structure)
|      |
|      |-- schemas/document.schema.js
|      |      (validates structured document data)
|      |
|      |-- services/state.service.js
|      |      (merges updates into structured state)
|      |
|      |-- services/document.service.js
|      |      (structured state -> document text)
|      |
|      |-- middleware/error.middleware.js
|      |      (centralized HTTP error handling)
|
|-- Document Generation
|      |
|      |-- Structured State
|      |      (validated user information)
|      |
|      |-- Document Service
|      |      (generates readable document content)
|      |
|      |-- DocumentPreview.jsx
|             (live preview -> copy text / PDF export)
|
|-- Testing
       |
       |-- tests/conversation.api.test.js
       |      (API behavior)
       |
       |-- tests/state.service.test.js
       |      (state merging)
       |
       |-- tests/document.service.test.js
       |      (document generation)
       |
       |-- tests/validation.test.js
       |      (schema validation)
       |
       |-- tests/fixtures/llm-fixtures.js
       |      (reusable test data)
       |
       |-- vitest.config.js
              (test runner configuration)


**Who decides what?**

| Component | Responsibility |
|---|---|
| React frontend | Collects input and displays results |
| Express routes | Maps HTTP requests to handlers |
| Controller | Handles requests and responses |
| Conversation service | Coordinates the chat workflow |
| LLM service | Interprets messages and proposes updates |
| Zod schemas | Validate expected data structures |
| State service | Merges new information with existing state |
| MongoDB models | Define stored data structures |
| Document service | Generates document text from state |
| DocumentPreview.jsx | Displays document and exports PDF |
| Error middleware | Handles backend errors |
| Vitest tests | Check expected application behavior |
```



## imp for api
generate api key at https://console.groq.com/keys

and put the  key in the backend in the .env.example file

rename the .env.example file to .env before executing for coverting the llm from mock to groq



## Repository Structure

```text
├── backend/
│   ├── src/
│   │   ├── config/          # Database connection and environment parsing
│   │   ├── controllers/     # Express route handlers
│   │   ├── middleware/      # Global error and exception handling
│   │   ├── models/          # Session, Message, and DocumentState schemas
│   │   ├── prompts/         # Anti-hallucination intake prompt templates
│   │   ├── routes/          # RESTful conversation and state endpoints
│   │   ├── schemas/         # Zod schemas for LLM contracts and state
│   │   ├── services/
│   │   │   ├── llm/         # GeminiService, MockLLMService, LLMFactory
│   │   │   ├── conversation.service.js
│   │   │   ├── document.service.js
│   │   │   └── state.service.js
│   │   ├── app.js           # Express app setup
│   │   └── server.js        # Server bootstrap
│   └── tests/               # 15 automated unit and integration tests
├── frontend/
│   ├── src/
│   │   ├── components/      # Chat, Message, StructuredState, DocumentPreview
│   │   ├── services/        # Backend API client
│   │   ├── App.jsx          # Root component with local session resumption
│   │   ├── App.css          # Professional 3-pane layout styling
│   │   └── main.jsx
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
├── AI_LOG.md                # Development log, prompt iterations, and decisions
├── PRODUCTION_NOTES.md      # Scaling, security, and production considerations
├── .env.example             # Safe template for environment variables
└── README.md

## Quick Start (Windows PowerShell)

Run the backend and frontend in two separate terminals. Do not start the backend twice: it uses port `5000`.

### 1. Configure the backend

From the repository root:

```powershell
cd .\backend
npm install
```

Make sure `backend/.env` contains a MongoDB connection string, a valid `GROQ_API_KEY`, and:

```dotenv
LLM_PROVIDER=groq
```

### 2. Start the backend

In the first terminal, from `document-intake-assistant\backend`:

```powershell
npm run dev
```

Wait for these messages before opening the frontend:

```text
[Database] MongoDB connected successfully
[Server] Document Intake Backend running on http://localhost:5000
```

Verify the backend in a browser or PowerShell:

```powershell
Invoke-RestMethod http://localhost:5000/health
```

Keep this terminal running. Stop it with `Ctrl+C` when finished.

### 3. Start the frontend

Open a second terminal. From the repository root, run:

```powershell
cd .\frontend
npm install
npm run dev
```

Open the URL printed by Vite, normally `http://localhost:5173`.

### Port 5000 error

If the backend reports `EADDRINUSE`, another backend process is already using port `5000`. Check it:

```powershell
Get-NetTCPConnection -LocalPort 5000 -State Listen
```

If it is an old backend process, stop it using its `OwningProcess` value:

```powershell
Stop-Process -Id <OwningProcess> -Force
```

Then run `npm run dev` once from the `backend` folder. If `/health` returns `status: ok`, the backend is already running and you should not start another copy.

### Run backend tests

From `document-intake-assistant\backend`:

```powershell
npm test
```

generate api key at https://console.groq.com/keys

at put the   key in the backend in the .env file 
