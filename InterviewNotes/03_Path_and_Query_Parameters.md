# 03 — Path & Query Parameters

## Path parameters
- Dynamic part of the URL: `localhost:8000/patient/P001` (`localhost` = `127.0.0.1`).
- Used to **retrieve / update / delete a particular resource** by id.

```python
@app.get('/patient/{patient_id}')
def view_patient(patient_id: str):
    ...
```

### `Path()` — improve a path parameter
Adds metadata, validation rules and docs.
```python
from fastapi import FastAPI, Path

@app.get('/patient/{patient_id}')
def view_patient(
    patient_id: str = Path(..., description='ID of the patient', examples=['P001'])
):
    ...
```
- `...` (Ellipsis) = value is **required**.

## Query parameters
- Format: `/view?key1=value1&key2=value2`
  - `?` starts the query string, `&` joins pairs. e.g. `?city=Delhi&sort_by=age`
- Used for **filtering, sorting, pagination** etc. — typically **optional**.
- Sorting needs two things: *on what basis* (height/weight/bmi) and *how* (asc/desc).

### `Query()` — improve a query parameter
```python
from fastapi import Query

@app.get('/sort')
def sort_patients(
    sort_by: str = Query(..., description='Sort on basis of height, weight or bmi'),
    order:   str = Query('asc', description='Sort in asc or desc order'),
):
    ...
# sort_by -> required (...), order -> optional (default 'asc')
```

### Validate allowed values
- Don't trust the client: `if sort_by not in ['height','weight','bmi']: raise HTTPException(400, ...)`
- Or use `Literal['asc','desc']` as the type -> FastAPI rejects anything else automatically (422).

## Path vs Query — cheat sheet
| | Path | Query |
|---|---|---|
| Location | in the URL path `/patient/P001` | after `?` |
| Typical use | identify a specific resource | filter / sort / options |
| Required? | always | usually optional |
| Declared | name appears in `{}` in route | any function arg not in the path |

## GET endpoints in the project
- `/about` -> info, `/view` -> all patients, `/patient/{id}` -> path param, `/sort` -> query params.

## Quick interview Q&A
- **Q: How does FastAPI tell path from query params?** If the name is in the route `{}` it's a path param; otherwise (simple types) it's a query param.
- **Q: What does `...` mean in `Path(...)`/`Query(...)`?** Required.
- **Q: Why use `Path()`/`Query()`?** Metadata, validation constraints, better Swagger docs.
- **Q: How to restrict a query param to fixed values?** `Literal[...]` type or manual check + HTTPException.
