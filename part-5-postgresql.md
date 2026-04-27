---
title: "Part 5 — PostgreSQL + Database Integration"
layout: page
---

# Part 5 — PostgreSQL + Database Integration

> **Key idea:** A database stores structured, queryable, persistent data. Without it, all data disappears when the server restarts.

---

## Navigation

[← Part 4: Docker](part-4-docker.md) | [Part 6: Debugging →](part-6-debugging.md)

---

## 1. Why a Database?

| Storage type | Survives restart? | Queryable? | Structured? |
|--------------|-------------------|------------|-------------|
| In-memory list (Python) | ❌ No | Limited | No |
| File on disk | ✅ Yes | Limited | No |
| PostgreSQL | ✅ Yes | ✅ Yes | ✅ Yes |

### Database Request Flow

```text
Backend → Query → Database → Result → Backend → Response
```

---

## 2. Core Concepts

| Concept | Description | Analogy |
|---------|-------------|---------|
| **Table** | A structured collection of data | Spreadsheet |
| **Column** | A field with a defined type | Column header |
| **Row** | One record | One row in a spreadsheet |
| **Primary key** | Unique identifier for each row | ID number |

---

## 3. Basic SQL

### Create a Table

```sql
CREATE TABLE users (
    id   SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age  INTEGER
);
```

### Insert Data

```sql
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@example.com', 28);
```

### Query All Rows

```sql
SELECT * FROM users;
```

### Query with a Condition

```sql
SELECT * FROM users WHERE age > 20;
```

### Update a Row

```sql
UPDATE users SET name = 'Alice Smith' WHERE id = 1;
```

### Delete a Row

```sql
DELETE FROM users WHERE id = 1;
```

---

## 4. Connecting FastAPI to PostgreSQL

Install the driver:

```bash
pip install psycopg2-binary
```

Add to `requirements.txt`:

```text
psycopg2-binary
```

### Database Connection Helper

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

> **Note:** When running inside Docker Compose, use `db` (the service name) as the host — not `localhost`.

---

## 5. Inserting Data from the Backend

```python
from app.db.connection import get_connection

def create_user(name: str, email: str, age: int) -> dict:
    conn = get_connection()
    try:
        with conn.cursor() as cur:
            cur.execute(
                "INSERT INTO users (name, email, age) VALUES (%s, %s, %s) RETURNING id",
                (name, email, age),
            )
            user_id = cur.fetchone()[0]
            conn.commit()
            return {"id": user_id, "name": name, "email": email, "age": age}
    finally:
        conn.close()
```

---

## 6. Fetching Data and Converting to JSON

```python
def get_users() -> list:
    conn = get_connection()
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT id, name, email, age FROM users")
            rows = cur.fetchall()
            return [
                {"id": row[0], "name": row[1], "email": row[2], "age": row[3]}
                for row in rows
            ]
    finally:
        conn.close()
```

---

## 7. Transactions

A **transaction** groups one or more operations. If any step fails, the whole transaction is rolled back.

```python
conn = get_connection()
try:
    with conn.cursor() as cur:
        cur.execute("INSERT INTO users (name, email) VALUES (%s, %s)", ("Alice", "alice@example.com"))
        cur.execute("INSERT INTO accounts (user_id, balance) VALUES (currval('users_id_seq'), 0)")
    conn.commit()   # Both inserts succeed together
except Exception:
    conn.rollback() # Neither insert happens if one fails
finally:
    conn.close()
```

---

## 8. Common Database Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `connection refused` | PostgreSQL is not running or wrong host | Check Docker service is up; use `db` as host in Compose |
| `relation "users" does not exist` | Table has not been created | Run the `CREATE TABLE` SQL first |
| `too many connections` | Connections are not being closed | Always call `conn.close()` in a `finally` block |
| `invalid input syntax for type integer` | Passing a string where an integer is expected | Validate and cast types before inserting |
| `duplicate key value violates unique constraint` | Inserting a value that already exists in a `UNIQUE` column | Check for existence first or catch the exception |

---

## 9. Debugging the Database

Access the database shell inside Docker:

```bash
docker exec -it <postgres_container_id> psql -U postgres -d appdb
```

Useful psql commands:

| Command | What it does |
|---------|-------------|
| `\dt` | List all tables |
| `\d users` | Show the structure of the `users` table |
| `SELECT * FROM users;` | See all rows |
| `\q` | Quit psql |

---

## 10. Mini Project

1. Create the `users` table in PostgreSQL.
2. Update `create_user` in your service to insert into the real database.
3. Update `get_users` to query from the database.
4. Verify data persists: stop and restart the backend, confirm users are still there.

---

## Exercises

1. Add a `GET /users/{user_id}` endpoint that queries by `id`. Return `404` if not found.
2. Add a `DELETE /users/{user_id}` endpoint that deletes by `id`.
3. Handle the `duplicate key` error in `create_user` and return a `409 Conflict` response.

---

## Part 5 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Database purpose | Structured, queryable, persistent storage |
| SQL basics | `CREATE TABLE`, `INSERT`, `SELECT`, `WHERE`, `UPDATE`, `DELETE` |
| psycopg2 | Python library for connecting to PostgreSQL |
| Docker host | Use service name `db`, not `localhost`, inside Docker Compose |
| Connection handling | Always close connections in a `finally` block |

---

## Navigation

[← Part 4: Docker](part-4-docker.md) | [Part 6: Debugging →](part-6-debugging.md)
