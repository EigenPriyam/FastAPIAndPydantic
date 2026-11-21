# 05 — CRUD in FastAPI (Patient Data API project)

Data is stored in a JSON file (`patients.json`) via helper functions `load_data()` / `save_data(data)`.

| Operation | Method | Endpoint | Inputs |
|---|---|---|---|
| Read all | GET | `/view` | — |
| Read one | GET | `/patient/{patient_id}` | path param |
| Sort | GET | `/sort` | query params `sort_by`, `order` |
| Create | POST | `/create` | request body (Patient) |
| Update | PUT | `/edit/{patient_id}` | path param + body (PatientUpdate) |
| Delete | DELETE | `/delete/{patient_id}` | path param |

(See 03 for GET details.)

## POST — Create
- Body = JSON from the client -> must be validated -> Pydantic model -> then saved.
```python
from typing import Literal
from fastapi.responses import JSONResponse

class Patient(BaseModel):
    id: str
    name: str
    gender: Literal['male', 'female', 'others']   # only these options
    ...

@app.post('/create')
def create_patient(patient: Patient):            # body auto-validated
    data = load_data()
    if patient.id in data:
        raise HTTPException(status_code=400, detail='Patient already exists')
    data[patient.id] = patient.model_dump(exclude={'id'})
    save_data(data)
    return JSONResponse(status_code=201, content={'message': 'Patient created successfully'})
```
- `Literal['Y','N','O']` -> only these values allowed.
- Duplicate id -> `HTTPException` 400. Success -> **201**.
- `exclude={'id'}` because the id is the dict key.

## PUT — Update
- Needs `patient_id` (path) **+** request body.
- A separate model **`PatientUpdate`** where *every field is optional* (client may change one or all).
```python
class PatientUpdate(BaseModel):
    name:   Optional[str]   = None
    city:   Optional[str]   = None
    height: Optional[float] = Field(default=None, gt=0)
    weight: Optional[float] = Field(default=None, gt=0)

@app.put('/edit/{patient_id}')
def update_patient(patient_id: str, patient_update: PatientUpdate):
    data = load_data()
    if patient_id not in data:
        raise HTTPException(status_code=404, detail='Patient not found')
    existing = data[patient_id]
    updates = patient_update.model_dump(exclude_unset=True)   # only what client sent
    existing.update(updates)
    # TRICK: dict -> Pydantic -> dict so computed fields are recalculated
    existing['id'] = patient_id
    patient_obj = Patient(**existing)
    data[patient_id] = patient_obj.model_dump(exclude={'id'})
    save_data(data)
    return JSONResponse(status_code=200, content={'message': 'Patient updated'})
```
- **`exclude_unset=True`** -> dict has only the fields in the request body.
- **Round-trip trick**: after editing height/weight, stored `bmi` & `verdict` are stale. Rebuilding a `Patient` (which has `computed_field`s) and dumping again recalculates them.
- Technically PATCH semantics, implemented with PUT.

## DELETE
```python
@app.delete('/delete/{patient_id}')
def delete_patient(patient_id: str):
    data = load_data()
    if patient_id not in data:
        raise HTTPException(status_code=404, detail='Patient not found')
    del data[patient_id]
    save_data(data)
    return JSONResponse(status_code=200, content={'message': 'Patient deleted'})
```
- 204 No Content is also a valid choice for delete.

## Flow summary
Client request -> FastAPI route -> Pydantic validates (422 on failure) -> logic (load data, check existence -> 400/404) -> save -> `JSONResponse` with 200/201.

## Next steps (from notes)
- Serve an ML model as an API.
- Dockerise (see 06) -> run locally -> deploy: AWS EC2, or Google Cloud Run.
- Revise the code along with the notes.

## Quick interview Q&A
- **Q: Why a separate `PatientUpdate` model?** All fields optional for partial updates.
- **Q: Why `exclude_unset=True`?** To update only fields the client sent, not overwrite with None defaults.
- **Q: Why rebuild the Patient on update?** To recompute computed fields (bmi, verdict).
- **Q: Which status for create / duplicate / missing?** 201 / 400 / 404.
- **Q: How is the request body validated?** Annotate the parameter with a Pydantic model.
