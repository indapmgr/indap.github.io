# 🏗️ Part 8 — Backend Capstone Project

> **Goal:** Build a complete, working backend API system from scratch: FastAPI backend, PostgreSQL database, Docker Compose orchestration, and a Python test script to verify end-to-end behavior.

---

## Navigation

[← Part 7: Advanced Git](part-7-advanced-git) | [Part 9: Final Evaluation →](part-9-final-evaluation)

---

## 1. System Overview

| Layer | Technology |
|-------|-----------|
| API | FastAPI (Python) |
| Database | PostgreSQL |
| Orchestration | Docker Compose |
| Testing | Python `requests` script |

### Final System Flow

```text
test_api.py → HTTP request → FastAPI Backend → PostgreSQL → Response → test_api.py
```

---

## 2. Required Features

### Backend

- `POST /users` — create a user with `name`, `email`, `age`; return `201` with the new user
- `GET /users` — return all users as a JSON array
- `GET /users/{user_id}` — return one user by id; return `404` if not found
- `GET /` — health check returning `{"status": "ok"}`

### Database

- Users are stored persistently in PostgreSQL
- Data must survive a container restart

### Docker

- Full system starts with one command: `docker compose up --build`

---

## 3. Project Structure

```text
project/
├── backend/
│   ├── app/
│   │   ├── main.py            # FastAPI app entry point
│   │   ├── routes/
│   │   │   └── users.py       # User API endpoints
│   │   ├── services/
│   │   │   └── user_service.py  # Business logic
│   │   ├── schemas/
│   │   │   └── user.py        # Pydantic request/response models
│   │   └── db/
│   │       └── connection.py  # Database connection helper
│   ├── init.sql               # SQL to create tables on first startup
│   ├── requirements.txt       # Python dependencies
│   └── Dockerfile             # Container build instructions
├── test_api.py                # Python script to verify end-to-end behavior
└── docker-compose.yml         # Orchestrates backend + database containers
```

---

## 4. Backend Implementation

### Pydantic Schema

**`backend/app/schemas/user.py`**

```python
from pydantic import BaseModel   # Base class for all data validation schemas

# Schema for incoming requests — what the client must send to create a user
class UserCreate(BaseModel):
    name: str      # Required; any string
    email: str     # Required; ideally unique in the database
    age: int       # Required; must be an integer

# Schema for outgoing responses — what the API returns to the client
class UserResponse(BaseModel):
    id: int        # Assigned by the database when the row is inserted
    name: str
    email: str
    age: int

    class Config:
        from_attributes = True   # Allows Pydantic to read from DB row objects
```

### Database Connection

**`backend/app/db/connection.py`**

```python
import psycopg2   # PostgreSQL driver for Python
import os         # Read environment variables (credentials, hostnames)

def get_connection():
    # All connection details come from environment variables set in docker-compose.yml
    # Never hardcode passwords in source code
    return psycopg2.connect(
        host=os.getenv("DB_HOST", "db"),          # "db" is the Docker service name
        database=os.getenv("DB_NAME", "appdb"),
        user=os.getenv("DB_USER", "postgres"),
        password=os.getenv("DB_PASSWORD", "password"),
    )
```

### Service Layer

**`backend/app/services/user_service.py`**

```python
from app.db.connection import get_connection
from app.schemas.user import UserCreate

def create_user(user: UserCreate) -> dict:
    conn = get_connection()   # Open a connection to the database
    try:
        with conn.cursor() as cur:
            # Parameterised query — %s placeholders are filled safely by psycopg2
            # RETURNING id gives us back the auto-generated id
            cur.execute(
                "INSERT INTO users (name, email, age) VALUES (%s, %s, %s) RETURNING id",
                (user.name, user.email, user.age),
            )
            user_id = cur.fetchone()[0]   # Extract the new id from the result
            conn.commit()                 # Make the insert permanent
            return {"id": user_id, "name": user.name, "email": user.email, "age": user.age}
    finally:
        conn.close()   # Always close the connection

def get_users() -> list:
    conn = get_connection()
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT id, name, email, age FROM users ORDER BY id")
            rows = cur.fetchall()   # List of tuples: [(1, "Alice", ...), ...]
            # Convert each tuple to a dict for JSON serialisation
            return [{"id": r[0], "name": r[1], "email": r[2], "age": r[3]} for r in rows]
    finally:
        conn.close()

def find_user(user_id: int) -> dict | None:
    conn = get_connection()
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT id, name, email, age FROM users WHERE id = %s", (user_id,))
            row = cur.fetchone()   # Returns None if no row matches
            if row is None:
                return None        # Caller will raise HTTPException 404
            return {"id": row[0], "name": row[1], "email": row[2], "age": row[3]}
    finally:
        conn.close()
```

### Routes

**`backend/app/routes/users.py`**

```python
from fastapi import APIRouter, HTTPException
from app.schemas.user import UserCreate, UserResponse
from app.services import user_service

# All routes in this file share the /users prefix and "users" tag in /docs
router = APIRouter(prefix="/users", tags=["users"])

# POST /users — create a new user; returns 201 on success
@router.post("/", response_model=UserResponse, status_code=201)
def create_user(user: UserCreate):
    return user_service.create_user(user)

# GET /users — return all users as a list
@router.get("/", response_model=list[UserResponse])
def get_users():
    return user_service.get_users()

# GET /users/{user_id} — return one user by id
@router.get("/{user_id}", response_model=UserResponse)
def get_user(user_id: int):
    user = user_service.find_user(user_id)
    if user is None:
        # HTTPException tells FastAPI to return a JSON error response
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

### Application Entry Point

**`backend/app/main.py`**

```python
from fastapi import FastAPI                         # The main FastAPI class
from fastapi.middleware.cors import CORSMiddleware  # Allows cross-origin requests
from app.routes import users                        # Import the users router

app = FastAPI(title="User API")   # Title appears in /docs

# CORS middleware lets external clients (browsers, test scripts) call the API
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],       # Allow any origin in development
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(users.router)   # Register all /users routes

# Health check — a quick way to confirm the server is up and responding
@app.get("/")
def health_check():
    return {"status": "ok"}
```

---

## 5. Database Initialisation

Before the backend can insert users, the table must exist. Create an SQL init script:

**`backend/init.sql`**

```sql
-- Create the users table on first database startup
-- IF NOT EXISTS prevents an error if the table already exists
CREATE TABLE IF NOT EXISTS users (
    id    SERIAL PRIMARY KEY,        -- Auto-incrementing integer ID
    name  VARCHAR(100) NOT NULL,     -- User's display name; required
    email VARCHAR(255) UNIQUE NOT NULL,  -- Must be unique; required
    age   INTEGER                    -- Optional age field
);
```

Mount it in `docker-compose.yml` under the `db` service so PostgreSQL runs it automatically on first start:

```yaml
db:
  image: postgres:15
  environment:
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: password
    POSTGRES_DB: appdb
  volumes:
    - postgres_data:/var/lib/postgresql/data
    - ./backend/init.sql:/docker-entrypoint-initdb.d/init.sql
```

---

## 6. Docker Setup

### Backend Dockerfile

**`backend/Dockerfile`**

```dockerfile
# Start from the official Python 3.11 slim image
FROM python:3.11-slim

# All commands run from /app inside the container
WORKDIR /app

# Copy requirements first — lets Docker cache the pip install layer
COPY requirements.txt .

# Install dependencies; --no-cache-dir keeps the image smaller
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Document that this container serves traffic on port 8000
EXPOSE 8000

# Start the FastAPI server; --host 0.0.0.0 makes it reachable from outside
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose

**`docker-compose.yml`**

```yaml
version: "3.9"   # Docker Compose file format version

services:

  # ── Backend ──────────────────────────────────────────────────────────────
  backend:
    build: ./backend             # Build from backend/Dockerfile
    ports:
      - "8000:8000"              # Host port 8000 → Container port 8000
    environment:
      - DB_HOST=db               # Must be the service name "db", not "localhost"
      - DB_NAME=appdb
      - DB_USER=postgres
      - DB_PASSWORD=password
    depends_on:
      - db                       # Start db container before backend

  # ── Database ─────────────────────────────────────────────────────────────
  db:
    image: postgres:15           # Official PostgreSQL 15 image
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: appdb         # Create this database on first run
    volumes:
      - postgres_data:/var/lib/postgresql/data          # Persist data
      - ./backend/init.sql:/docker-entrypoint-initdb.d/init.sql  # Run on first start

# Named volume managed by Docker — data persists across container restarts
volumes:
  postgres_data:
```

---

## 7. Python Test Script

**`test_api.py`**

```python
import requests   # pip install requests

BASE_URL = "http://localhost:8000"

def test_health():
    """Verify the server is running and responding."""
    response = requests.get(f"{BASE_URL}/")
    assert response.status_code == 200, f"Expected 200, got {response.status_code}"
    assert response.json()["status"] == "ok"
    print("✅ Health check passed")

def test_create_user():
    """Create a user and verify the response."""
    response = requests.post(
        f"{BASE_URL}/users",
        json={"name": "Alice", "email": "alice@example.com", "age": 28}
    )
    assert response.status_code == 201, f"Expected 201, got {response.status_code}"
    data = response.json()
    assert "id" in data, "Response should contain an id"
    assert data["name"] == "Alice"
    print(f"✅ Create user passed — id={data['id']}")
    return data["id"]   # Return the id for use in later tests

def test_get_users():
    """Fetch all users and verify at least one is returned."""
    response = requests.get(f"{BASE_URL}/users")
    assert response.status_code == 200, f"Expected 200, got {response.status_code}"
    users = response.json()
    assert isinstance(users, list), "Response should be a list"
    assert len(users) > 0, "There should be at least one user"
    print(f"✅ Get users passed — {len(users)} user(s) found")

def test_get_user_not_found():
    """Request a non-existent user and verify 404 is returned."""
    response = requests.get(f"{BASE_URL}/users/99999")
    assert response.status_code == 404, f"Expected 404, got {response.status_code}"
    print("✅ Get unknown user returns 404 — passed")

def test_invalid_data():
    """Send invalid data and verify 422 is returned."""
    response = requests.post(
        f"{BASE_URL}/users",
        json={"name": "Bob", "email": "bob@example.com", "age": "not-a-number"}
    )
    assert response.status_code == 422, f"Expected 422, got {response.status_code}"
    print("✅ Invalid data returns 422 — passed")

# Run all tests when the script is executed directly
if __name__ == "__main__":
    try:
        test_health()
        test_create_user()
        test_get_users()
        test_get_user_not_found()
        test_invalid_data()
        print("\n🎉 All tests passed!")
    except AssertionError as e:
        print(f"\n❌ Test failed: {e}")
    except requests.exceptions.ConnectionError:
        print("\n❌ Could not connect to the server.")
        print("   Make sure the server is running: docker compose up --build")
```

Run the tests after starting the system:

```bash
docker compose up --build -d
python test_api.py
```

---

## 8. Running the System

```bash
docker compose up --build
```

Verify:

| URL | Expected result |
|-----|----------------|
| `http://localhost:8000` | `{"status": "ok"}` |
| `http://localhost:8000/docs` | FastAPI interactive API documentation |
| `http://localhost:8000/users` | JSON array (empty on first run) |

---

## 9. Debugging System-Level Failures

| Symptom | What to check |
|---------|--------------|
| Backend can't reach DB | Is `DB_HOST=db`? Is the `db` container healthy? (`docker compose ps`) |
| Users not persisting | Is `conn.commit()` called? Does the `users` table exist? |
| Container crashes on start | `docker compose logs <service>` — read the stack trace |
| `init.sql` didn't run | Volume already exists from a previous run — `docker compose down -v`, then restart |

---

## 10. Delivery Checklist

Before opening the PR, verify the system from a clean checkout:

- [ ] `docker compose up --build` starts all services without errors
- [ ] `GET /` returns `{"status": "ok"}`
- [ ] `POST /users` creates a user and returns `201`
- [ ] `GET /users` returns persisted users from PostgreSQL
- [ ] `GET /users/{id}` returns the user; `GET /users/99999` returns `404`
- [ ] Restarting containers does not delete user data
- [ ] Invalid input returns `422` with a clear error
- [ ] `python test_api.py` runs and all tests pass
- [ ] Logs do not expose secrets

---

## 11. Deliverables

You must deliver:

- [ ] Backend with `POST /users`, `GET /users`, and `GET /users/{user_id}` connected to PostgreSQL
- [ ] Database that persists users across restarts
- [ ] Full system runs with `docker compose up --build`
- [ ] `test_api.py` that exercises all endpoints
- [ ] Code in a Git repository with a feature branch and PR
- [ ] PR description includes test evidence and any known limitations

---

## 12. Evaluation Criteria

| Area | What is assessed |
|------|-----------------|
| **Functionality** | Do all endpoints work correctly? |
| **Persistence** | Does data survive a container restart? |
| **Code structure** | Are routes, services, schemas, and DB separated correctly? |
| **Docker** | Does `docker compose up --build` start everything cleanly? |
| **Testing** | Does `test_api.py` pass? Were error cases tested? |
| **Git** | Was the work done in a branch? Is there a PR? Are commit messages meaningful? |

---

## Part 8 Summary

This part brings everything together. Building the full system requires every skill from Parts 1–7:

| Part | Contribution |
|------|-------------|
| Part 1 | Environment setup, Git, HTTP understanding |
| Part 2 | FastAPI routes, schemas, services |
| Part 3 | Virtual environments, pip, Python test scripts |
| Part 4 | Dockerfile and Docker Compose |
| Part 5 | PostgreSQL queries with psycopg2 |
| Part 6 | Debugging tools and methodology |
| Part 7 | Branch workflow, PR, commit messages |

---

## Navigation

[← Part 7: Advanced Git](part-7-advanced-git) | [Part 9: Final Evaluation →](part-9-final-evaluation)
