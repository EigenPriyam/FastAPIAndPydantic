# 02 — HTTP Methods, Status Codes & HTTPException

## Static vs Dynamic
- Static -> little user interaction (blogs, calendar). Dynamic -> high interaction (Instagram, Facebook).
- Dynamic interactions = **CRUD**: Create, Retrieve, Update, Delete.
- Client ⟷ Server over HTTP; the request must say which kind of interaction it is (the **HTTP method**).

## HTTP methods <-> CRUD
| Method | CRUD | Notes |
|---|---|---|
| GET | Retrieve | no body |
| POST | Create | request body |
| PUT | Update | **replaces the whole resource** |
| PATCH | Partial update | change only some fields |
| DELETE | Delete | |
| HEAD / OPTIONS | metadata / allowed methods | |

- These aren't the *only* methods; they're the 4 that map to CRUD.
- The `/edit` endpoint in the project is really PATCH-style, though PUT is commonly used.
- See methods live: browser -> right-click -> Inspect -> **Network** tab.

## Request body
- The part of the HTTP request carrying client -> server data. Used in **POST / PUT** (PATCH). Format: JSON (or XML).

## Status codes
Three-digit numbers telling the client whether the request worked and, if not, why.

| Range | Meaning |
|---|---|
| 2xx | Success |
| 3xx | Redirect / further action needed |
| 4xx | Client error |
| 5xx | Server error |

| Code | Name | When |
|---|---|---|
| 200 | OK | Success (GET/POST) |
| 201 | Created | POST created a new row/resource |
| 204 | No Content | Success, nothing returned (e.g. DELETE) |
| 400 | Bad Request | Malformed/invalid request (e.g. duplicate id) |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Authenticated but not permitted |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable Entity | **FastAPI default when Pydantic validation fails** |
| 500 | Internal Server Error | General server failure |
| 502 | Bad Gateway | Gateway got an *invalid* response from backend |
| 503 | Service Unavailable | Server down/overloaded |
| 504 | Gateway Timeout | Gateway got *no response in time* |

## HTTPException
- **Built-in FastAPI class** — you `raise` it with your own status code + message.
- (Custom exception classes are a different feature: `@app.exception_handler`.)

```python
from fastapi import HTTPException

def view_patient(patient_id: str = Path(...)):
    data = load_data()
    if patient_id in data:
        return data[patient_id]
    raise HTTPException(status_code=404, detail='Patient Not Found')
```
- Use cases: duplicate id (400), not found (404), invalid query value (400).

## `JSONResponse`
```python
from fastapi.responses import JSONResponse
return JSONResponse(status_code=201, content={'message': 'Patient created successfully'})
```
Returns JSON with a chosen status code.

## Quick interview Q&A
- **Q: PUT vs PATCH?** PUT replaces the full resource; PATCH changes some fields.
- **Q: 401 vs 403?** 401 = not logged in; 403 = logged in but not allowed.
- **Q: 502 vs 504?** 502 invalid response from backend; 504 timeout.
- **Q: What status does FastAPI return on validation failure?** 422.
- **Q: How do you return a custom error?** `raise HTTPException(status_code=..., detail=...)`.
- **Q: 200 vs 201?** 201 specifically means a new resource was created.
