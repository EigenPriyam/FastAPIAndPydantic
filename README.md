# FastAPI Fundamentals

Learning project covering FastAPI and Pydantic basics, built around a small **Patient Data API** (CRUD over a JSON file), with Docker notes to follow.

## Tech stack
- Python 3.12+
- [FastAPI](https://fastapi.tiangolo.com/) (built on Starlette + Pydantic)
- Uvicorn (ASGI server)
- [uv](https://docs.astral.sh/uv/) for dependency management

## Project structure
```
FASTAPIFUNDAMENTALS/
├── src/app/
│   ├── get_method.py                    # GET endpoints (path & query params)
│   └── pydantic_and_other_methods.py    # Pydantic models + POST / PUT / DELETE
├── patients.json                        # JSON "database"
├── InterviewNotes/                      # Topic-wise revision notes (Markdown + PDF)
├── pyproject.toml / uv.lock
└── requirements.txt
```

## Setup
```bash
uv sync
# or: pip install -r requirements.txt
```

## Run
From the project root:
```bash
uv run uvicorn src.app.get_method:app --reload
# full CRUD version:
uv run uvicorn src.app.pydantic_and_other_methods:app --reload
```
Then open:
- http://localhost:8000/docs — Swagger UI
- http://localhost:8000/redoc — ReDoc
- http://localhost:8000/openapi.json — raw schema

## Endpoints
| Method | Path | Description |
|---|---|---|
| GET | `/` | Hello |
| GET | `/about` | About the API |
| GET | `/view` | All patients |
| GET | `/patient/{patient_id}` | One patient (path param) |
| GET | `/sort?sort_by=&order=` | Sorted patients (query params) |
| POST | `/create` | Create a patient (validated by Pydantic) |
| PUT | `/edit/{patient_id}` | Update fields of a patient |
| DELETE | `/delete/{patient_id}` | Delete a patient |

## Concepts covered
- API basics, WSGI vs ASGI, why FastAPI is fast
- HTTP methods and status codes, `HTTPException`
- Path and query parameters (`Path()`, `Query()`)
- Pydantic: `Field`, `field_validator`, `model_validator`, `computed_field`, nested models, `model_dump`
- CRUD operations with a JSON store
- Docker fundamentals (notes)

## Revision notes
Topic-wise notes live in [`InterviewNotes/`](InterviewNotes/README.md):
1. [API & FastAPI Basics](InterviewNotes/01_API_and_FastAPI_Basics.md)
2. [HTTP Methods & Status Codes](InterviewNotes/02_HTTP_Methods_and_Status_Codes.md)
3. [Path & Query Parameters](InterviewNotes/03_Path_and_Query_Parameters.md)
4. [Pydantic](InterviewNotes/04_Pydantic.md)
5. [CRUD Patient API](InterviewNotes/05_CRUD_Patient_API.md)
6. [Docker](InterviewNotes/06_Docker.md)

## Roadmap
- Serve an ML model through the API
- Dockerise the app and run it locally
- Deploy on AWS EC2 / Google Cloud Run
