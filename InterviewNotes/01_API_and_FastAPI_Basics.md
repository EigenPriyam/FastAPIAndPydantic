# 01 — API & FastAPI Basics

## What is an API?
- **API = Application Programming Interface**: a fixed contract (URL + method + input/output format) one program uses to talk to another.
- Frontend (e.g. Streamlit) ⟷ **API** ⟷ Backend (logic). The API is the bridge; the frontend only needs the contract, not the backend internals.

## Why APIs? (Monolith vs Scalable architecture)
| Monolithic | API-based (scalable) |
|---|---|
| Frontend & backend tightly coupled | Backend exposes an API; frontend and 3rd parties consume it |
| One change touches many files | Frontend/backend change independently |
| 3rd parties can't use the data | 3rd parties can use it -> monetisation (paid APIs) |

- **Protocol: HTTP**; data exchanged as **JSON** (language-independent: Python, Java, PHP...).
- **One backend, many frontends**: Web, Android, iPhone all call the same backend via the API.

## AI/ML use case
- Frontend ⟷ Backend (API) ⟷ ML model (often a pickle).
- Flow: user submits -> backend loads model -> sends input -> gets prediction -> returns it to frontend.
- **Load the model once at app startup**, not inside every request (latency). Other formats: joblib, ONNX, `.pt`, `.keras`.

## What is FastAPI?
- Python framework for building APIs. Built on:
  - **Starlette** -> HTTP requests/responses (web layer)
  - **Pydantic** -> data validation
- **Fast to run** (low latency, async) and **fast to code**.

## Why is FastAPI fast to run? (WSGI vs ASGI)
Request flow: `Client -> Web Server -> (S)GI -> API code -> back`
- Web server listens on a port; the gateway interface converts requests to something Python understands and responses back to something the client understands.

| | Flask | FastAPI |
|---|---|---|
| Interface | **WSGI** (synchronous) | **ASGI** (asynchronous) |
| Server | Gunicorn (WSGI server) | Uvicorn (ASGI server) |
| Lib | Werkzeug | Starlette |
| Code style | sync `def` | `async def` supported |

**Gotchas / corrections**
- "Uvicorn is better than Gunicorn" is wrong — they serve different interfaces. In production they're often combined: **Gunicorn as process manager running Uvicorn workers**.
- WSGI: each *worker* handles one request at a time -> scale with more workers/threads. ASGI: one worker can juggle many requests while they wait on I/O.
- **Async only helps I/O-bound work** (DB, HTTP calls, file reads). A CPU-heavy ML prediction inside `async def` **blocks the event loop** -> use plain `def` (FastAPI runs it in a thread pool).

## Why is FastAPI fast to code? (3 features)
1. Automatic input validation (Pydantic)
2. Auto-generated interactive docs
3. Easy to deploy

## Running & docs
```bash
uvicorn main:app --reload      # main.py file, `app` object
```
- `/docs` -> Swagger UI (default), `/redoc` -> alternate docs, `/openapi.json` -> raw schema.

## Quick interview Q&A
- **Q: What is FastAPI built on?** Starlette (web) + Pydantic (validation).
- **Q: WSGI vs ASGI?** Sync vs async gateway interfaces; Flask=WSGI, FastAPI=ASGI.
- **Q: Gunicorn vs Uvicorn?** WSGI server vs ASGI server; used together in prod.
- **Q: When does async help?** I/O-bound waits, not CPU-bound work.
- **Q: Where do you load an ML model?** Once at startup, not per request.
- **Q: How do you see API docs?** `/docs` (Swagger), `/redoc`.
