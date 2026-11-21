# 04 — Pydantic (Data Validation)

## Why Pydantic?
- Python is **dynamically typed**; type hints are **not enforced**.
```python
def insert_data(name: str, age: int): ...
insert_data('yash', 'Thirty')   # works, no error!
```
- Pydantic enforces types at runtime. Use **v2**.

## Basic steps
1. Create a class inheriting `BaseModel`.
2. Instantiate it -> data is validated automatically (`ValidationError` on failure).
3. Pass the validated object to functions.

```python
from pydantic import BaseModel

class Patient(BaseModel):
    name: str
    age: int

p1 = Patient(**{'name': 'Yash', 'age': 27})   # ** unpacks the dict

def insert_patient(patient: Patient):
    print(patient.name, patient.age)          # dot notation
```

## Collections
- `List[str]` -> a list **and** each item is str. Plain `list` -> only checks it's a list.
- `from typing import List, Dict`; on Python 3.9+ use `list[str]`, `dict[str, str]` directly.

## Required, Optional & defaults
- All fields are **required by default**.
- `age: int = 0` -> default value.
- `allergies: Optional[List[str]] = None` -> optional.
- **v2 gotcha:** `Optional[X]` only means *None is allowed*. Without `= None` the field is **still required**. The default is what makes it optional.

## Built-in special types
```python
from pydantic import EmailStr, AnyUrl
class Patient(BaseModel):
    email: EmailStr        # needs: pip install "pydantic[email]"
    linkedin_url: AnyUrl
```

## `Field` and `Annotated`
- `Annotated` is from **`typing`**; `Field` from `pydantic`.
- Constraints are kwargs: `gt, ge, lt, le, min_length, max_length` (`Field(age>18)` is invalid -> `Field(gt=18)`).
- `Field` does 3 things: **validation, metadata, default value**.

```python
from typing import Annotated, Optional
from pydantic import BaseModel, Field

class Patient(BaseModel):
    age: int = Field(gt=18)
    name: Annotated[str, Field(max_length=50, title='Name of the patient',
                               description='Give the name in < 50 chars',
                               examples=['Yash', 'Amit'])]
    married: Annotated[Optional[bool], Field(default=None, description='Is the patient married')]
    weight: Annotated[float, Field(gt=0, strict=True)]
```

### Type coercion & `strict`
- `'age': '72'` -> with `strict=False` (default) Pydantic **coerces** to `72`.
- `strict=True` -> `'72'` raises an error.

## `field_validator` — custom logic on ONE field
```python
from pydantic import BaseModel, EmailStr, field_validator

class Patient(BaseModel):
    email: EmailStr

    @field_validator('email')
    @classmethod
    def email_validator(cls, value):
        if value.split('@')[-1] not in ['hdfc.com', 'icici.com']:
            raise ValueError('Not a valid domain')
        return value          # MUST return the value
```
- `cls` is just the class. To read other fields add `info: ValidationInfo` -> `info.data['other']` (or use a model validator).
- **Always `raise ValueError(...)`, never `return ValueError(...)`.**

### `mode='before'` vs `'after'` (default `after`)
- **after** -> runs on the value *after* coercion (`'70'` is already `70`).
- **before** -> gets the *raw* input (`'70'` string). `0 < '70'` raises `TypeError` (not cleanly wrapped) -> convert first: `value = int(value)`.
- Only `ValueError`/`AssertionError` become a clean `ValidationError`.

```python
@field_validator('age', mode='after')
@classmethod
def check_age(cls, value):
    if 0 < value < 100:
        return value
    raise ValueError('Incorrect Age')
```

## `model_validator` — rules across 2+ fields
```python
from pydantic import BaseModel, model_validator

class Patient(BaseModel):
    age: int
    contact: dict[str, str]

    @model_validator(mode='after')
    def validate_emergency(self):
        if self.age > 60 and 'emergency' not in self.contact:
            raise ValueError('Patients older than 60 must have an emergency contact')
        return self           # must return self
```
- v2 `mode='after'` validators are **instance methods** (`self`), return `self`. (`(cls, model)` = old v1 `root_validator`.)
- `field_validator` -> one attribute; `model_validator` -> whole model.

## `computed_field` — derived, not sent by the user
```python
from pydantic import BaseModel, computed_field

class Patient(BaseModel):
    height: float   # metres
    weight: float   # kg

    @computed_field
    @property
    def bmi(self) -> float:
        return round(self.weight / (self.height ** 2), 2)
```
- The **function name becomes the field name** (`bmi`).
- Included automatically in `model_dump()` and JSON responses.

## Nested models
```python
class Address(BaseModel):
    city: str
    state: str
    pincode: str

class Patient(BaseModel):
    name: str
    address: Address

Patient(name='Yash', address={'city': 'Delhi', 'state': 'Delhi', 'pincode': '110001'})  # dict works too
patient.address.city
```

## Exporting (dumping)
```python
p.model_dump()                          # -> dict
p.model_dump_json()                     # -> JSON string
p.model_dump(include={'name', 'age'})   # only these
p.model_dump(exclude={'address'})       # all except these
p.model_dump(exclude_unset=True)        # only fields explicitly provided
```

## Learning path
Type validation -> Field/Annotated -> field_validator -> model_validator -> computed_field -> nested models -> dumping.

## Quick interview Q&A
- **Q: Do type hints enforce types in Python?** No; Pydantic does at runtime.
- **Q: Optional vs default in Pydantic v2?** Optional = None allowed; default makes it non-required.
- **Q: field_validator vs model_validator?** Single field vs cross-field/whole model.
- **Q: before vs after validators?** Raw input vs coerced value.
- **Q: What is coercion / strict?** Auto-converting `'72'` -> `72`; `strict=True` disables it.
- **Q: What is computed_field?** A derived property included in serialisation.
- **Q: `model_dump` vs `model_dump_json`?** dict vs JSON string.
- **Q: What does `exclude_unset` do?** Dumps only fields the client actually sent.
