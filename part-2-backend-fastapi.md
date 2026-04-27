---
title: "Part 2 — Backend: FastAPI"
layout: page
---

# Part 2 — Backend: FastAPI

> **Key idea:** The backend is responsible for validation, business logic, data handling, and error handling. The frontend never touches the database.

---

## Navigation

[← Part 1: Foundations](part-1-foundations.md) | [Part 3: Frontend →](part-3-frontend-nextjs.md)

---

## 1. Backend Thinking

The backend sits between the frontend and the database. Every request must go through the backend.

**Responsibilities:**

| Responsibility | What it means |
|----------------|--------------|
| **Validation** | Check that incoming data is correct before processing it |
| **Business logic** | Apply rules (e.g. a user must be 18+) |
| **Data handling** | Read and write data to the database |
| **Error handling** | Return useful error messages when something goes wrong |

### Backend Request Flow

```text
Request → Validate → Process → DB → Response
```

---

## 2. Project Structure

A well-organised FastAPI project looks like this:

```text
backend/
├── app/
│   ├── main.py          # FastAPI app entry point
│   ├── routes/          # API route handlers
│   ├── services/        # Business logic
│   ├── models/          # Database models
│   ├── schemas/         # Pydantic request/response schemas
│   └── db/              # Database connection
├── requirements.txt     # Python dependencies
└── Dockerfile           # Container definition
```

---

## 3. FastAPI Setup

Install dependencies:

```bash
pip install fastapi uvicorn psycopg2-binary
```

Save to `requirements.txt`:

```text
fastapi
uvicorn
psycopg2-binary
```

---

## 4. Application Entry Point

**`app/main.py`**

```python
from fastapi import FastAPI
from app.routes import users

app = FastAPI()

app.include_router(users.router)

@app.get("/")
def health_check():
    return {"status": "ok"}
```

---

## 5. Routes

**`app/routes/users.py`**

```python
from fastapi import APIRouter
from app.schemas.user import UserCreate, UserResponse
from app.services import user_service

router = APIRouter(prefix="/users", tags=["users"])

@router.post("/", response_model=UserResponse, status_code=201)
def create_user(user: UserCreate):
    return user_service.create_user(user)

@router.get("/", response_model=list[UserResponse])
def get_users():
    return user_service.get_users()
```

---

## 6. Pydantic Schemas

Schemas define the shape of request and response data. FastAPI uses them for automatic validation.

**`app/schemas/user.py`**

```python
from pydantic import BaseModel

class UserCreate(BaseModel):
    name: str
    email: str
    age: int

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    age: int

    class Config:
        from_attributes = True
```

---

## 7. Service Layer

The service layer contains business logic. Routes call services; services do not call routes.

**`app/services/user_service.py`**

```python
from app.schemas.user import UserCreate

# In-memory store for the mini project (replace with DB later)
users_db = []
next_id = 1

def create_user(user: UserCreate) -> dict:
    global next_id
    new_user = {"id": next_id, **user.dict()}
    users_db.append(new_user)
    next_id += 1
    return new_user

def get_users() -> list:
    return users_db
```

---

## 8. Database Connection

**`app/db/connection.py`**

```python
import psycopg2
import os

def get_connection():
    return psycopg2.connect(
        host=os.getenv("DB_HOST", "db"),
        database=os.getenv("DB_NAME", "appdb"),
        user=os.getenv("DB_USER", "postgres"),
        password=os.getenv("DB_PASSWORD", "password"),
    )
```

---

## 9. Running the Server

```bash
uvicorn app.main:app --reload
```

Open the interactive API docs at:

```text
http://localhost:8000/docs
```

---

## 10. Error Handling

Use `HTTPException` to return error responses with correct status codes:

```python
from fastapi import HTTPException

@router.get("/{user_id}")
def get_user(user_id: int):
    user = user_service.find_user(user_id)
    if user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

---

## 11. Mini Project

Build an in-memory user API:

1. `POST /users` — accept `name`, `email`, `age`; return the created user with an `id`
2. `GET /users` — return all users as a list

Start with an in-memory list. Replace with PostgreSQL in Part 5.

---

## 12. Common Backend Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Business logic in routes | Routes become hard to test and maintain | Move logic to services |
| No input validation | Bad data reaches the database | Use Pydantic schemas |
| Returning raw DB exceptions | Leaks internal details | Catch exceptions and return `HTTPException` |
| Hardcoded credentials | Security risk | Use environment variables |
| No `status_code` on POST | Returns 200 instead of 201 | Set `status_code=201` |

---

## 13. Debugging Backend

| Tool | How to use it |
|------|--------------|
| uvicorn output | Read the terminal for errors and stack traces |
| `/docs` endpoint | Test endpoints interactively without a frontend |
| HTTP status codes | Check the response code to understand what went wrong |
| `print()` / logging | Add temporary prints to trace execution |

---

## Exercises

1. Add a `DELETE /users/{user_id}` endpoint that removes a user from the in-memory list.
2. Add validation: `age` must be between 1 and 120. Return a `400` error if it is not.
3. Add a `GET /users/{user_id}` endpoint. Return `404` if the user does not exist.

---

## Part 2 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Backend flow | Request → Validate → Process → DB → Response |
| Project structure | Separate routes, services, schemas, models, db |
| FastAPI | Automatic validation, auto-generated docs at `/docs` |
| Pydantic | Define input/output schemas with type hints |
| Error handling | Always use `HTTPException` with correct status codes |

---

## Navigation

[← Part 1: Foundations](part-1-foundations.md) | [Part 3: Frontend →](part-3-frontend-nextjs.md)
