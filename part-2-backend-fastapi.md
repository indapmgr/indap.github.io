# ⚡ Part 2 — Backend: FastAPI

> **Key idea:** The backend is responsible for validation, business logic, data handling, and error handling. The frontend never touches the database.

---

## Navigation

[← Part 1: Foundations](part-1-foundations) | [Part 3: Virtualenv & Testing →](part-3-frontend-nextjs)

---

## 1. Backend Thinking

The backend sits between the client and the database. Every request must go through the backend.

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
│   ├── main.py          # FastAPI app entry point — where the server starts
│   ├── routes/          # API route handlers — one file per resource
│   ├── services/        # Business logic — keeps routes thin
│   ├── models/          # Database models
│   ├── schemas/         # Pydantic request/response schemas — define data shapes
│   └── db/              # Database connection helpers
├── requirements.txt     # Python dependencies (every library your app needs)
└── Dockerfile           # Instructions to build a Docker container
```

---

## 3. FastAPI Setup

First, activate your virtual environment (see Part 3 for details), then install dependencies:

```bash
pip install fastapi uvicorn psycopg2-binary
```

Save all installed packages to `requirements.txt` so others can recreate your environment:

```bash
pip freeze > requirements.txt
```

The file will contain entries like:

```text
fastapi
uvicorn
psycopg2-binary
```

---

## 4. Application Entry Point

**`app/main.py`**

```python
from fastapi import FastAPI       # FastAPI is the main framework class
from app.routes import users      # Import our users router module

app = FastAPI()                   # Create the application instance

# Register the users router — all /users routes come from here
app.include_router(users.router)

# Health-check endpoint: a quick way to confirm the server is alive
@app.get("/")
def health_check():
    return {"status": "ok"}
```

---

## 5. Routes

Routes define the API endpoints. They receive requests and return responses.

**`app/routes/users.py`**

```python
from fastapi import APIRouter
from app.schemas.user import UserCreate, UserResponse
from app.services import user_service

# APIRouter groups related endpoints; prefix means every route starts with /users
router = APIRouter(prefix="/users", tags=["users"])

# POST /users — create a new user; returns 201 Created on success
@router.post("/", response_model=UserResponse, status_code=201)
def create_user(user: UserCreate):
    # Delegate the actual work to the service layer
    return user_service.create_user(user)

# GET /users — return a list of all users
@router.get("/", response_model=list[UserResponse])
def get_users():
    return user_service.get_users()
```

---

## 6. Pydantic Schemas

Schemas define the exact shape of request and response data. FastAPI uses them for automatic validation — if the client sends the wrong type, FastAPI rejects the request immediately.

**`app/schemas/user.py`**

```python
from pydantic import BaseModel    # BaseModel is the base class for all schemas

# Schema for creating a user — defines what the client must send
class UserCreate(BaseModel):
    name: str       # Required string
    email: str      # Required string (add EmailStr for strict validation)
    age: int        # Required integer — FastAPI rejects strings automatically

# Schema for returning a user — includes the id assigned by the database
class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    age: int

    class Config:
        from_attributes = True  # Lets Pydantic read data from database row objects
```

---

## 7. Service Layer

The service layer contains business logic. Routes call services; services do not call routes. This separation makes code easier to test and maintain.

**`app/services/user_service.py`**

```python
from app.schemas.user import UserCreate

# In-memory store for the mini project (replace with DB in Part 5)
users_db = []
next_id = 1   # Simple counter to give each user a unique id

def create_user(user: UserCreate) -> dict:
    global next_id
    # Build a new user dict by unpacking the Pydantic model
    new_user = {"id": next_id, **user.dict()}
    users_db.append(new_user)   # Add to our in-memory list
    next_id += 1                # Increment so the next user gets a different id
    return new_user

def get_users() -> list:
    return users_db             # Return the full in-memory list
```

---

## 8. Database Connection

**`app/db/connection.py`**

```python
import psycopg2    # PostgreSQL driver for Python
import os          # os.getenv reads environment variables safely

def get_connection():
    # Read connection details from environment variables — never hardcode passwords!
    return psycopg2.connect(
        host=os.getenv("DB_HOST", "db"),          # "db" is the Docker service name
        database=os.getenv("DB_NAME", "appdb"),
        user=os.getenv("DB_USER", "postgres"),
        password=os.getenv("DB_PASSWORD", "password"),
    )
```

---

## 9. Running the Server

Start the development server with auto-reload (restarts on code changes):

```bash
uvicorn app.main:app --reload
```

Open the interactive API docs at:

```text
http://localhost:8000/docs
```

> **Tip:** The `/docs` page is generated automatically by FastAPI. You can send test requests directly from the browser without writing any code.

---

## 10. Error Handling

Use `HTTPException` to return error responses with the correct HTTP status codes:

```python
from fastapi import HTTPException

# GET /users/{user_id} — returns a specific user, or 404 if not found
@router.get("/{user_id}")
def get_user(user_id: int):
    user = user_service.find_user(user_id)
    if user is None:
        # Raise an HTTP 404 error — FastAPI converts this into a proper JSON response
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

---

## 11. Backend Quality Checklist

Before you call an API endpoint done, verify:

- Inputs are validated with Pydantic schemas
- Successful creates return `201 Created`
- Not-found cases return `404 Not Found`
- Invalid requests return `400 Bad Request` or `422 Unprocessable Entity`
- Internal errors do not leak raw database details to the client
- Route handlers stay thin; business logic lives in services
- Configuration comes from environment variables, not hardcoded secrets

---

## 12. Mini Project

Build an in-memory user API:

1. `POST /users` — accept `name`, `email`, `age`; return the created user with an `id`
2. `GET /users` — return all users as a list

Start with an in-memory list. Replace with PostgreSQL in Part 5.

---

## 13. Common Backend Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Business logic in routes | Routes become hard to test and maintain | Move logic to services |
| No input validation | Bad data reaches the database | Use Pydantic schemas |
| Returning raw DB exceptions | Leaks internal details | Catch exceptions and return `HTTPException` |
| Hardcoded credentials | Security risk | Use environment variables |
| No `status_code` on POST | Returns `200` instead of `201` | Set `status_code=201` on the route decorator |

---

## 14. Debugging Backend

| Tool | How to use it |
|------|--------------|
| uvicorn terminal output | Read it for errors and stack traces — the line number is there |
| `/docs` endpoint | Test endpoints interactively without any client |
| HTTP status codes | Check the response code to understand what went wrong |
| `print()` / logging | Add temporary prints to trace execution through the code |

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
| Delivery quality | Validate inputs, hide internals, keep routes thin |

---

## Navigation

[← Part 1: Foundations](part-1-foundations) | [Part 3: Virtualenv & Testing →](part-3-frontend-nextjs)
