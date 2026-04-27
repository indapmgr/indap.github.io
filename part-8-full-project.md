---
title: "Part 8 — Full End-to-End Project"
layout: page
---

# Part 8 — Full End-to-End Project

> **Goal:** Build a complete, working full-stack system from scratch: Next.js frontend, FastAPI backend, PostgreSQL database, and Docker Compose orchestration.

---

## Navigation

[← Part 7: Advanced Git](part-7-advanced-git.md) | [Part 9: Final Evaluation →](part-9-final-evaluation.md)

---

## 1. System Overview

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js (TypeScript) |
| Backend | FastAPI (Python) |
| Database | PostgreSQL |
| Orchestration | Docker Compose |

### Final System Flow

```text
UI → API call → Backend → DB → Backend → Response → UI
```

---

## 2. Required Features

### Backend

- `POST /users` — create a user with `name`, `email`, `age`
- `GET /users` — return all users

### Frontend

- Form to create a user
- List displaying all users

### Database

- Users are stored persistently in PostgreSQL

### Docker

- Full system starts with one command: `docker compose up --build`

---

## 3. Project Structure

```text
project/
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── routes/
│   │   │   └── users.py
│   │   ├── services/
│   │   │   └── user_service.py
│   │   ├── schemas/
│   │   │   └── user.py
│   │   └── db/
│   │       └── connection.py
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── pages/
│   │   └── index.tsx
│   ├── components/
│   │   └── UserForm.tsx
│   ├── services/
│   │   └── api.ts
│   ├── package.json
│   └── Dockerfile
└── docker-compose.yml
```

---

## 4. Backend Implementation

### Pydantic Schema

**`backend/app/schemas/user.py`**

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
```

### Database Connection

**`backend/app/db/connection.py`**

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

### Service Layer

**`backend/app/services/user_service.py`**

```python
from app.db.connection import get_connection
from app.schemas.user import UserCreate

def create_user(user: UserCreate) -> dict:
    conn = get_connection()
    try:
        with conn.cursor() as cur:
            cur.execute(
                "INSERT INTO users (name, email, age) VALUES (%s, %s, %s) RETURNING id",
                (user.name, user.email, user.age),
            )
            user_id = cur.fetchone()[0]
            conn.commit()
            return {"id": user_id, "name": user.name, "email": user.email, "age": user.age}
    finally:
        conn.close()

def get_users() -> list:
    conn = get_connection()
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT id, name, email, age FROM users ORDER BY id")
            rows = cur.fetchall()
            return [{"id": r[0], "name": r[1], "email": r[2], "age": r[3]} for r in rows]
    finally:
        conn.close()
```

### Routes

**`backend/app/routes/users.py`**

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

### Application Entry Point

**`backend/app/main.py`**

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routes import users

app = FastAPI(title="User API")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(users.router)

@app.get("/")
def health_check():
    return {"status": "ok"}
```

---

## 5. Frontend Implementation

### API Service

**`frontend/services/api.ts`**

```typescript
const BASE_URL = process.env.NEXT_PUBLIC_API_URL || "http://localhost:8000";

export async function createUser(data: { name: string; email: string; age: number }) {
  const res = await fetch(`${BASE_URL}/users`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });
  if (!res.ok) throw new Error(`Create failed: ${res.status}`);
  return res.json();
}

export async function getUsers() {
  const res = await fetch(`${BASE_URL}/users`);
  if (!res.ok) throw new Error(`Fetch failed: ${res.status}`);
  return res.json();
}
```

### Home Page

**`frontend/pages/index.tsx`**

```typescript
import { useEffect, useState } from "react";
import { createUser, getUsers } from "../services/api";

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

export default function Home() {
  const [users, setUsers] = useState<User[]>([]);
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [age, setAge] = useState("");

  async function loadUsers() {
    const data = await getUsers();
    setUsers(data);
  }

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    await createUser({ name, email, age: parseInt(age) });
    setName("");
    setEmail("");
    setAge("");
    await loadUsers();
  }

  useEffect(() => {
    loadUsers();
  }, []);

  return (
    <main style={{ padding: "2rem", fontFamily: "sans-serif" }}>
      <h1>User Management</h1>

      <h2>Add User</h2>
      <form onSubmit={handleSubmit}>
        <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Name" required />
        <input value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" type="email" required />
        <input value={age} onChange={(e) => setAge(e.target.value)} placeholder="Age" type="number" required />
        <button type="submit">Create</button>
      </form>

      <h2>Users</h2>
      <ul>
        {users.map((u) => (
          <li key={u.id}>{u.name} — {u.email} — Age: {u.age}</li>
        ))}
      </ul>
    </main>
  );
}
```

---

## 6. Docker Setup

### Backend Dockerfile

**`backend/Dockerfile`**

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Frontend Dockerfile

**`frontend/Dockerfile`**

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

### Docker Compose

**`docker-compose.yml`**

```yaml
version: "3.9"

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=db
      - DB_NAME=appdb
      - DB_USER=postgres
      - DB_PASSWORD=password
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:8000
    depends_on:
      - backend

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: appdb
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

## 7. Database Initialisation

Before the backend can insert users, the table must exist. Create an init script:

**`backend/init.sql`**

```sql
CREATE TABLE IF NOT EXISTS users (
    id    SERIAL PRIMARY KEY,
    name  VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age   INTEGER
);
```

Mount it in docker-compose.yml under the `db` service:

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

## 8. Running the System

```bash
docker compose up --build
```

Verify:

| URL | Expected result |
|-----|----------------|
| `http://localhost:3000` | Frontend user management page |
| `http://localhost:8000/docs` | FastAPI interactive API documentation |
| `http://localhost:8000/users` | JSON array of all users |

---

## 9. Debugging System-Level Failures

| Symptom | What to check |
|---------|--------------|
| Frontend can't reach backend | Is `NEXT_PUBLIC_API_URL` correct? Is backend container running? |
| Backend can't reach DB | Is `DB_HOST=db`? Is the `db` container healthy? |
| Users not persisting | Is `conn.commit()` called? Is the table created? |
| Container crashes on start | `docker compose logs <service>` — read the stack trace |

---

## 10. Deliverables

You must deliver:

- [ ] Backend with `POST /users` and `GET /users` connected to PostgreSQL
- [ ] Frontend with create-user form and user list
- [ ] Database that persists users across restarts
- [ ] Full system runs with `docker compose up --build`
- [ ] Code in a Git repository with a feature branch and PR

---

## 11. Evaluation Criteria

| Area | What is assessed |
|------|-----------------|
| **Functionality** | Does `POST /users` create? Does `GET /users` return data? |
| **Persistence** | Does data survive a container restart? |
| **Code structure** | Are routes, services, schemas, and DB separated correctly? |
| **Docker** | Does `docker compose up --build` start everything cleanly? |
| **Git** | Was the work done in a branch? Is there a PR? Are commit messages meaningful? |

---

## Part 8 Summary

This part brings everything together. Building the full system requires every skill from Parts 1–7:

| Part | Contribution |
|------|-------------|
| Part 1 | Environment, Git, HTTP understanding |
| Part 2 | FastAPI routes, schemas, services |
| Part 3 | Next.js components, state, API calls |
| Part 4 | Dockerfile and Docker Compose |
| Part 5 | PostgreSQL queries with psycopg2 |
| Part 6 | Debugging tools and methodology |
| Part 7 | Branch workflow, PR, commit messages |

---

## Navigation

[← Part 7: Advanced Git](part-7-advanced-git.md) | [Part 9: Final Evaluation →](part-9-final-evaluation.md)
