---
title: "Part 6 — Debugging"
layout: page
---

# Part 6 — Debugging

> **Key idea:** Debugging is not guessing. It is the systematic identification of where the system broke.

---

## Navigation

[← Part 5: PostgreSQL](part-5-postgresql.md) | [Part 7: Advanced Git →](part-7-advanced-git.md)

---

## 1. Debugging Is a System

Every failure happens in a specific layer. Your job is to find which layer failed, not to randomly change code until it works.

### Stack Layers

```text
Frontend → API call → Backend → Database → Infrastructure
```

Debugging means walking through each layer until you find the one that failed.

---

## 2. Universal Debugging Framework

Ask these questions in order:

| Step | Question | How to check |
|------|----------|-------------|
| 1 | Is the system running? | Check `docker compose ps`, terminal output |
| 2 | Is the request being sent? | Check DevTools → Network tab |
| 3 | Is the backend receiving it? | Check uvicorn logs |
| 4 | Is the backend processing it? | Add `print()` or check logs |
| 5 | Is the DB working? | Connect with `psql`, check query |
| 6 | Is the response returning? | Check DevTools → Network response body |

---

## 3. Debugging Scenarios

### Scenario: Frontend button does nothing

1. Open DevTools → Console. Is there a JavaScript error?
2. Open DevTools → Network tab. Did the request fire?
3. If no request: check the event handler is attached correctly.
4. If request fired: check the URL, method, and request body.

### Scenario: API returns 500

1. Read the uvicorn logs in the terminal — the stack trace tells you exactly where the error is.
2. Check the line number in the stack trace.
3. Add `print()` before and after the failing line to inspect values.
4. If DB-related: connect to the database and run the query manually.

### Scenario: CORS error

```text
Access to fetch at 'http://localhost:8000' from origin 'http://localhost:3000' has been blocked by CORS policy
```

Fix: Add CORS middleware to FastAPI:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Scenario: Database connection failed

```text
psycopg2.OperationalError: could not connect to server
```

Checks:
1. Is the `db` container running? → `docker compose ps`
2. Are you using `db` (service name) as the host, not `localhost`?
3. Are the DB credentials in the environment variables correct?
4. Did the DB container fail to start? → `docker compose logs db`

### Scenario: Data not saving

1. Is `conn.commit()` being called after the insert?
2. Did the insert throw an exception that was silently swallowed?
3. Connect to psql and check: `SELECT * FROM users;`

### Scenario: Container keeps restarting

```bash
docker compose logs backend
```

Read the error. Common causes:
- Missing environment variable
- Dependency not in `requirements.txt`
- Application crash on start

### Scenario: Port already in use

```text
Error: bind: address already in use
```

Find and kill the process:

```bash
# Find what's using port 8000
lsof -i :8000

# Kill it by PID
kill <PID>
```

Or change the port in `docker-compose.yml`.

### Scenario: Git push rejected

```text
error: failed to push some refs to 'origin'
hint: Updates were rejected because the remote contains work that you do not have locally.
```

Fix:

```bash
git pull origin <branch-name>
# Resolve any conflicts
git push
```

### Scenario: Merge conflict

```text
CONFLICT (content): Merge conflict in app/services/user_service.py
```

Fix:

1. Open the file. Find the conflict markers:

```text
<<<<<<< HEAD
your changes
=======
incoming changes
>>>>>>> feature/other-branch
```

2. Edit the file to keep the correct version.
3. Remove the conflict markers.
4. Stage and commit:

```bash
git add app/services/user_service.py
git commit -m "Resolve merge conflict in user_service"
```

### Scenario: API works locally but not in Docker

The most common cause is using `localhost` instead of the service name.

| Context | Correct host |
|---------|-------------|
| Local machine | `localhost` |
| Inside Docker Compose | service name (e.g. `db`, `backend`) |

Check your environment variables in `docker-compose.yml`.

---

## 4. Debugging Mindset

```text
Observe → Isolate → Fix → Verify
```

| Step | What to do |
|------|-----------|
| **Observe** | Read the error message. Check logs. Look at DevTools. |
| **Isolate** | Identify which layer failed. Narrow down to the failing function. |
| **Fix** | Make one targeted change. |
| **Verify** | Test the fix. Confirm the error is gone. Test related functionality too. |

---

## 5. Debugging Tools Reference

| Tool | Use it for |
|------|-----------|
| Browser DevTools → Network | See all HTTP requests and responses |
| Browser DevTools → Console | See JavaScript errors |
| uvicorn terminal output | See FastAPI errors and stack traces |
| `docker compose logs <service>` | See container stdout/stderr |
| `docker exec -it <id> psql` | Run queries directly in the database |
| `docker ps` | See which containers are running |
| `print()` / `logging` | Trace code execution |

---

## Exercises

1. Deliberately break your DB connection string. Find the error using `docker compose logs`.
2. Remove `conn.commit()` from an insert. Verify data does not save. Add it back.
3. Remove the CORS middleware. Observe the browser error. Re-add it.

---

## Part 6 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Debugging approach | Systematic layer-by-layer isolation, never guessing |
| Universal framework | Is it running? Is it sending? Is it receiving? Is it processing? Is the DB working? Is it responding? |
| CORS | Needs middleware in FastAPI when frontend/backend are on different origins |
| Docker networking | Use service names inside Compose, not `localhost` |
| Mindset | Observe → Isolate → Fix → Verify |

---

## Navigation

[← Part 5: PostgreSQL](part-5-postgresql.md) | [Part 7: Advanced Git →](part-7-advanced-git.md)
