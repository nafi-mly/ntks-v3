# Brochoco it aint that DDD 🥀

```
your-fav-backend/
├── src/
│   ├── api/                        # LAYER: INTERFACES (Gatekeepers)
│   │   ├── v1/
│   │   │   ├── identity/           # Endpoint for User/Auth
│   │   │   │   ├── router.py
│   │   │   │   └── schemas.py      # Pydantic DTOs
│   │   │   ├── assistant/          # Main Agent Chat Endpoint
│   │   │   │   ├── router.py
│   │   │   │   └── schemas.py
│   │   │   └── knowledge/          # Endpoint for document upload (RAG)
│   │   │       ├── router.py
│   │   │       └── schemas.py
│   │   └── dependencies.py         # Global DI (Auth, DB Session)
│   │
│   ├── modules/                    # LAYER: THE HEART (Feature-First)
│   │   ├── identity/               # -- Feature: User Management --
│   │   │   ├── domain/             # Pure Logic (Entities, Repo Interface)
│   │   │   ├── application/
│   │   │   │   ├── use_cases/      # RegisterUser, LoginUser
│   │   │   │   ├── agents/         # AI as a "Power User"
│   │   │   │   │   ├── identity_agent.py
│   │   │   │   │   └── tools.py    # List tools (get_user_profile, etc)
│   │   │   └── infrastructure/
│   │   │       ├── persistence/    # SQLAlchemy Models & Repos
│   │   │       └── ai_tools/       # Impl: Tools that call Repo
│   │   │
│   │   ├── assistant/              # -- Feature: AI Orchestration --
│   │   │   ├── domain/
│   │   │   │   ├── prompts/        # Persona & System Prompt Templates
│   │   │   │   │   ├── agent_persona.py
│   │   │   │   │   └── tool_instructions.py
│   │   │   ├── application/
│   │   │   │   └── chat_manager.py # Manages chat flow & memory strategy
│   │   │   └── infrastructure/     
│   │   │       └── memory_store.py # Impl: save chat to Redis
│   │
│   │   └── knowledge/              # -- Feature: RAG & Documents --
│   │       ├── domain/
│   │       │   └── entities.py     # Document Entity
│   │       ├── application/
│   │       │   └── ingest_doc.py   # Use Case: File -> Embedding -> VectorStore
│   │       └── infrastructure/
│   │           └── vector_repo.py  # Impl: Calls core/ai/vector_store.py
│   │
│   ├── core/                       # LAYER: SHARED ENGINE (Cross-Cutting)
│   │   ├── ai/                     # -- AI AGENCY BASE ENGINE --
│   │   │   ├── base_agent.py       # Main class extended by all agents
│   │   │   ├── llm_client.py       # Wrapper for OpenAI/Claude (Singleton)
│   │   │   ├── vector_store.py     # Wrapper for PGVector/Pinecone (RAG)
│   │   │   └── memory/             # -- MEMORY & SUMMARIZATION --
│   │   │       ├── base.py
│   │   │       ├── strategies.py   # (The "Compact" logic is here!)
│   │   │       └── window.py       # Sliding window memory
│   │   ├── database.py             # SQLAlchemy Engine setup
│   │   ├── config.py               # Env vars (API_KEYS, DB_URL)
│   │   └── exceptions.py           # Global Error Handling
│   │
│   ├── main.py                     # App Entry Point
│   └── alembic/                    # DB Migrations
│
├── tests/                          # Mirrored Test Suite
│   ├── unit/
│   ├── integration/
│   └── agents/                     # Specifically for testing tool-calling accuracy
├── .env
├── pyproject.toml
└── README.md
```

### 📋 WhatsApp Integration Checklist (Meta Cloud API)

#### 1. Administrative Setup (Meta Developer Dashboard)
* [ ] **Create Account:** Sign up at [developers.facebook.com](https://developers.facebook.com).
* [ ] **Create App:** Choose type "Other" -> "Business".
* [ ] **Add Product:** Click "Set up" on the **WhatsApp** section.
* [ ] **Test Number:** Use the "Test Number" Meta provides to send messages to your own personal phone (so it's free during *development*).
* [ ] **Permanent Token:** This is what people often forget. Meta's default token only lasts 24 hours. You need to create a **System User** in Business Manager to get a *Permanent Access Token*.

#### 2. Environment Configuration (`.env`)
You'll need 3 new variables:
* [ ] `WHATSAPP_TOKEN`: Access token from Meta.
* [ ] `PHONE_NUMBER_ID`: Unique ID for your sender number.
* [ ] `WABA_ID`: WhatsApp Business Account ID (usually for *billing/logging* purposes).

#### 3. Technical Components (Python Logic)
* [ ] **`MessengerService`:** Create a new *class* in `app/core/messenger.py` with just one function: `send_text_message(to_phone, message)`.
* [ ] **HTTP Client:** Use `httpx` to hit the Meta endpoint with a `POST`: `https://graph.facebook.com/v20.0/{phone_number_id}/messages`.
* [ ] **Webhook Verification:** Add a `GET /webhook` endpoint in FastAPI. Meta will hit this with a *challenge* (a puzzle string) to verify your server is actually live before they start sending chat data.

#### 4. Final Wiring
* [ ] **Service Integration:** In `main.py` or the `router`, after getting the `ai_response` from `PaymentService`, call `MessengerService.send_text_message()`.
