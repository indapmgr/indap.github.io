---
title: "Part 3 — Frontend: Next.js + API Integration"
layout: page
---

# Part 3 — Frontend: Next.js + API Integration

> **Golden rule:** The frontend never talks to the database directly. It only talks to backend APIs.

---

## Navigation

[← Part 2: Backend](part-2-backend-fastapi.md) | [Part 4: Docker →](part-4-docker.md)

---

## 1. Frontend Thinking

The frontend is responsible for what the user sees and interacts with.

**Responsibilities:**

| Responsibility | What it means |
|----------------|--------------|
| **State** | Track data that changes over time (user input, API responses) |
| **Interaction** | Handle button clicks, form submissions, and user events |
| **API communication** | Fetch data from and send data to the backend |
| **Rendering** | Display data to the user |

### Frontend Request Flow

```text
User clicks → Event handler → API call → Response → State update → UI re-render
```

---

## 2. Next.js Project Structure

```text
frontend/
├── pages/
│   ├── index.tsx        # Home page
│   └── _app.tsx         # App wrapper
├── components/
│   ├── UserForm.tsx     # Form to create a user
│   └── UserList.tsx     # Component to display users
├── services/
│   └── api.ts           # All API calls live here
├── public/              # Static assets
├── package.json
└── Dockerfile
```

---

## 3. Creating a Next.js Project

```bash
npx create-next-app@latest frontend --typescript
cd frontend
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to see the app.

---

## 4. API Service Layer

All API calls belong in a single service file. Components call the service; they never call `fetch` directly.

**`services/api.ts`**

```typescript
const BASE_URL = process.env.NEXT_PUBLIC_API_URL || "http://localhost:8000";

export async function createUser(data: {
  name: string;
  email: string;
  age: number;
}) {
  const response = await fetch(`${BASE_URL}/users`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });

  if (!response.ok) {
    throw new Error(`Failed to create user: ${response.status}`);
  }

  return response.json();
}

export async function getUsers() {
  const response = await fetch(`${BASE_URL}/users`);

  if (!response.ok) {
    throw new Error(`Failed to fetch users: ${response.status}`);
  }

  return response.json();
}
```

---

## 5. UserForm Component

**`components/UserForm.tsx`**

```typescript
import { useState } from "react";
import { createUser } from "../services/api";

interface Props {
  onUserCreated: () => void;
}

export default function UserForm({ onUserCreated }: Props) {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [age, setAge] = useState("");
  const [error, setError] = useState("");

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");

    try {
      await createUser({ name, email, age: parseInt(age) });
      setName("");
      setEmail("");
      setAge("");
      onUserCreated();
    } catch (err) {
      setError("Failed to create user. Check the backend is running.");
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
        required
      />
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
        type="email"
        required
      />
      <input
        value={age}
        onChange={(e) => setAge(e.target.value)}
        placeholder="Age"
        type="number"
        required
      />
      <button type="submit">Create User</button>
      {error && <p style={{ color: "red" }}>{error}</p>}
    </form>
  );
}
```

---

## 6. Displaying Users

**`pages/index.tsx`**

```typescript
import { useEffect, useState } from "react";
import { getUsers } from "../services/api";
import UserForm from "../components/UserForm";

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

export default function Home() {
  const [users, setUsers] = useState<User[]>([]);

  async function loadUsers() {
    try {
      const data = await getUsers();
      setUsers(data);
    } catch {
      console.error("Could not load users");
    }
  }

  useEffect(() => {
    loadUsers();
  }, []);

  return (
    <main>
      <h1>Users</h1>
      <UserForm onUserCreated={loadUsers} />

      <ul>
        {users.map((user) => (
          <li key={user.id}>
            {user.name} — {user.email} — Age: {user.age}
          </li>
        ))}
      </ul>
    </main>
  );
}
```

---

## 7. Async/Await and Promises

Every API call is asynchronous — the result is not available immediately.

```typescript
// Wrong — this does NOT wait for the result
const users = getUsers();  // users is a Promise, not data

// Correct — await pauses until the Promise resolves
const users = await getUsers();  // users is the actual data
```

Always use `async/await` inside an `async` function.

---

## 8. Common Frontend Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Wrong API URL | Network error in DevTools | Check `BASE_URL` and the backend port |
| Backend not running | `ERR_CONNECTION_REFUSED` | Start the backend with `uvicorn` or `docker compose up` |
| CORS error | Error in console mentioning CORS | Add CORS middleware to FastAPI (see below) |
| Mutating state directly | UI does not update | Always create a new array/object with spread (`[...arr]`) |
| Forgetting `await` | Variables contain Promises, not data | Add `await` before every API call |

### Adding CORS to FastAPI

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 9. Debugging Frontend

| Tool | How to use it |
|------|--------------|
| Browser DevTools → Network tab | See every API request, its URL, method, status, and response body |
| Browser DevTools → Console | See JavaScript errors and `console.log` output |
| React state | Add `console.log(users)` to confirm state is updating |
| Check response body | Even a failed request may return a helpful error message |

---

## 10. Mini Project

Build a user management page:

1. Form with `name`, `email`, `age` fields
2. Submit sends `POST /users` to the backend
3. After submit, reload the user list
4. Display all users in a list from `GET /users`

---

## Exercises

1. Add an error message that shows when the API call fails.
2. Disable the submit button while the request is in progress.
3. Add a "Refresh" button that reloads the user list.

---

## Part 3 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Golden rule | Frontend never talks to DB — only to backend APIs |
| State | Use `useState` to track data that changes |
| API service | Put all `fetch` calls in one service file |
| Async/await | Always `await` API calls inside `async` functions |
| CORS | Add CORS middleware to FastAPI when frontend and backend run on different ports |

---

## Navigation

[← Part 2: Backend](part-2-backend-fastapi.md) | [Part 4: Docker →](part-4-docker.md)
