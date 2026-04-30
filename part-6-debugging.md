# 🔍 Part 6 — Debugging

> **Key idea:** Debugging is not guessing. It is the systematic identification of where the system broke.

---

## Navigation

[← Part 5: PostgreSQL](part-5-postgresql) | [Part 7: Advanced Git →](part-7-advanced-git)

---

## 1. Debugging Is a System

Every failure happens in a specific layer. Your job is to find which layer failed, not to randomly change code until it works.

### Stack Layers

```text
HTTP Client → FastAPI Backend → Database → Docker / Infrastructure
```

Debugging means walking through each layer until you find the one that failed. Start at the layer you can directly observe, then narrow down.

---

## 2. Universal Debugging Framework

Ask these questions in order:

| Step | Question | How to check |
|------|----------|-------------|
| 1 | Is the system running? | `docker compose ps`, check the terminal |
| 2 | Is the request being sent? | `test_api.py` output, or curl the endpoint |
| 3 | Is the backend receiving it? | Check uvicorn terminal logs |
| 4 | Is the backend processing it correctly? | Add `print()` statements, check logs |
| 5 | Is the DB working? | Connect with `psql`, run the query manually |
| 6 | Is the response correct? | Check the status code and response body |

---

## 3. Debugging Scenarios

### Scenario: API returns 500

1. Read the **uvicorn logs** in the terminal — the stack trace tells you exactly where the error is.
2. Find the line number in the stack trace.
3. Add `print()` before and after the failing line to inspect values.
4. If DB-related: connect to the database and run the query manually.

Example: check your server logs:

```bash
docker compose logs backend
```

### Scenario: API returns 422 Unprocessable Entity

FastAPI could not validate the request body against your Pydantic schema.

Check:
1. Is the request body JSON? (Content-Type: application/json)
2. Are all required fields present?
3. Are the field types correct? (e.g., `age` must be an integer, not a string)

The response body will tell you exactly which field failed:

```text
{
  "detail": [
    {
      "loc": ["body", "age"],
      "msg": "value is not a valid integer",
      "type": "type_error.integer"
    }
  ]
}
```

### Scenario: Database connection failed

```text
psycopg2.OperationalError: could not connect to server
```

Checks:
1. Is the `db` container running? → `docker compose ps`
2. Are you using `db` (the service name) as the host, **not** `localhost`?
3. Are the DB credentials in the environment variables correct?
4. Did the DB container fail to start? → `docker compose logs db`
5. Is PostgreSQL still starting up? It takes a few seconds after the container starts.

### Scenario: Data not saving

1. Is `conn.commit()` being called after the insert?
2. Did the insert throw an exception that was silently swallowed by a bare `except`?
3. Connect to psql and check directly:

```bash
docker exec -it <postgres_container_id> psql -U postgres -d appdb
```

Then run:

```sql
-- See all rows in the users table
SELECT * FROM users;
```

### Scenario: Container keeps restarting

```bash
docker compose logs backend
```

Read the error. Common causes:

- Missing environment variable (check your `docker-compose.yml`)
- Dependency not in `requirements.txt` (add it and rebuild)
- Python syntax error in your code
- Application crash on startup (usually a bad import or missing file)

### Scenario: Port already in use

```text
Error: bind: address already in use
```

Find what is using the port (replace 8000 with your port):

```bash
lsof -i :8000
```

Then stop the conflicting process:

```bash
kill <PID>
```

Or simply change the port mapping in `docker-compose.yml`:

```yaml
ports:
  - "8001:8000"   # map host port 8001 to container port 8000
```

### Scenario: Git push rejected

```text
error: failed to push some refs to 'origin'
hint: Updates were rejected because the remote contains work that you do not have locally.
```

Fix: pull the remote changes first, then push:

```bash
git pull origin <branch-name>
# Resolve any conflicts that appear
git push
```

### Scenario: Merge conflict

```text
CONFLICT (content): Merge conflict in app/services/user_service.py
```

Fix:

1. Open the conflicting file. Find the conflict markers:

```text
<<<<<<< HEAD
your version of the code
=======
the incoming version from main
>>>>>>> feature/other-branch
```

2. Edit the file: keep the correct code, remove the conflict markers.
3. Stage and commit:

```bash
git add app/services/user_service.py
git commit -m "Resolve merge conflict in user_service"
```

### Scenario: API works locally but not in Docker

The most common cause is using `localhost` instead of the service name inside Docker Compose.

| Context | Correct host |
|---------|-------------|
| Running locally (no Docker) | `localhost` |
| Inside Docker Compose | service name (e.g. `db`, `backend`) |

Check your environment variables in `docker-compose.yml`:

```yaml
environment:
  - DB_HOST=db       # must be "db", not "localhost"
```

### Scenario: `ModuleNotFoundError`

```text
ModuleNotFoundError: No module named 'fastapi'
```

This usually means:
1. Your virtual environment is not activated — run `source venv/bin/activate`
2. The package is not installed — run `pip install fastapi`
3. Inside Docker: the package is missing from `requirements.txt`

---

## 4. Debugging Mindset

```text
Observe → Isolate → Fix → Verify
```

| Step | What to do |
|------|-----------|
| **Observe** | Read the error message carefully. Check all logs. |
| **Isolate** | Identify which layer failed. Narrow down to the failing function or line. |
| **Fix** | Make **one** targeted change at a time. |
| **Verify** | Test the fix. Confirm the error is gone. Test adjacent functionality too. |

> **Never** randomly change multiple things at once — you won't know which change fixed the problem.

---

## 5. Debugging Tools Reference

| Tool | Use it for |
|------|-----------|
| `uvicorn` terminal output | FastAPI errors and stack traces |
| `docker compose logs <service>` | Container stdout/stderr output |
| `docker compose ps` | Check which containers are running or restarting |
| `docker exec -it <id> psql` | Run SQL queries directly in the database |
| `python test_api.py` | Test all API endpoints from a script |
| `print()` / `logging` | Trace code execution; print variable values |

---

## 6. Reading a Stack Trace

When FastAPI reports an error, the terminal shows a **stack trace**. Always read from the **bottom up** — the bottom line is the actual error:

```text
Traceback (most recent call last):
  File "/app/app/routes/users.py", line 12, in create_user     ← called here
    return user_service.create_user(user)
  File "/app/app/services/user_service.py", line 8, in create_user  ← and here
    users_db.appnd(new_user)                                    ← actual error line
AttributeError: 'list' object has no attribute 'appnd'         ← THE REAL ERROR
```

The bottom line says `appnd` — it's a typo. The fix is to change `appnd` to `append`.

---

## Exercises

1. Deliberately break your DB connection string. Find the error using `docker compose logs`.
2. Remove `conn.commit()` from an insert. Verify data does not save. Add it back and confirm it works.
3. Introduce a typo in a Python function name. Read the stack trace and fix it.

---

## Part 6 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Debugging approach | Systematic layer-by-layer isolation — never guessing |
| Universal framework | Running? Sending? Receiving? Processing? DB working? Responding? |
| Stack traces | Read from the bottom — the last line is the actual error |
| Docker networking | Use service names inside Compose, not `localhost` |
| Mindset | Observe → Isolate → Fix → Verify |

---

## Navigation

[← Part 5: PostgreSQL](part-5-postgresql) | [Part 7: Advanced Git →](part-7-advanced-git)
