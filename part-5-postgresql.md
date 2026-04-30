# 🗄️ Part 5 — PostgreSQL + Database Integration

> **Key idea:** A database stores structured, queryable, persistent data. Without it, all data disappears when the server restarts.

---

## Navigation

[← Part 4: Docker](part-4-docker) | [Part 6: Debugging →](part-6-debugging)

---

## 1. Why a Database?

| Storage type | Survives restart? | Queryable? | Structured? |
|--------------|-------------------|------------|-------------|
| In-memory list (Python) | ❌ No | Limited | No |
| File on disk | ✅ Yes | Limited | No |
| PostgreSQL | ✅ Yes | ✅ Yes | ✅ Yes |

### Database Request Flow

```text
Backend → SQL Query → Database → Result rows → Backend → JSON Response
```

---

## 2. Core Concepts

| Concept | Description | Analogy |
|---------|-------------|---------|
| **Table** | A structured collection of rows | Spreadsheet tab |
| **Column** | A field with a defined data type | Column header |
| **Row** | One record in a table | One row in a spreadsheet |
| **Primary key** | A unique identifier for each row | ID number that never repeats |

---

## 3. Basic SQL

SQL (Structured Query Language) is how you talk to a relational database. You write SQL statements; the database executes them and returns results.

### Create a Table

```sql
-- Create a users table if it doesn't already exist
CREATE TABLE IF NOT EXISTS users (
    id    SERIAL PRIMARY KEY,        -- SERIAL auto-increments: 1, 2, 3 …
    name  VARCHAR(100) NOT NULL,     -- Text up to 100 chars; required
    email VARCHAR(255) UNIQUE NOT NULL,  -- Must be unique across all rows; required
    age   INTEGER                    -- Whole number; optional (can be NULL)
);
```

### Insert Data

```sql
-- Add a new row to the users table
-- Always list columns explicitly so the order doesn't matter
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@example.com', 28);
```

### Query All Rows

```sql
-- Return every row and every column from the users table
SELECT * FROM users;
```

### Query with a Condition

```sql
-- Return only users older than 20
-- WHERE filters rows; only matching rows are returned
SELECT * FROM users WHERE age > 20;
```

### Update a Row

```sql
-- Change the name for the user with id = 1
-- Always use WHERE with UPDATE or you'll change every row!
UPDATE users SET name = 'Alice Smith' WHERE id = 1;
```

### Delete a Row

```sql
-- Remove the row where id = 1
-- Always use WHERE with DELETE or you'll delete every row!
DELETE FROM users WHERE id = 1;
```

---

## 4. Connecting FastAPI to PostgreSQL

Install the PostgreSQL driver:

```bash
pip install psycopg2-binary
```

Add it to `requirements.txt`:

```text
psycopg2-binary
```

### Database Connection Helper

**`app/db/connection.py`**

```python
import psycopg2    # The PostgreSQL driver for Python
import os          # For reading environment variables

def get_connection():
    # Read connection details from environment variables (not hardcoded values)
    # The defaults here are for local development only
    return psycopg2.connect(
        host=os.getenv("DB_HOST", "db"),          # "db" is the Docker service name
        database=os.getenv("DB_NAME", "appdb"),
        user=os.getenv("DB_USER", "postgres"),
        password=os.getenv("DB_PASSWORD", "password"),
    )
```

> **Reminder:** When running inside Docker Compose, use `db` (the service name) as the host — **not** `localhost`. See Part 4 for why.

---

## 5. Inserting Data from the Backend

```python
from app.db.connection import get_connection

def create_user(name: str, email: str, age: int) -> dict:
    conn = get_connection()   # Open a database connection
    try:
        with conn.cursor() as cur:
            # Use %s placeholders — NEVER build SQL by joining strings with user data
            # This is called a parameterised query; it prevents SQL injection attacks
            cur.execute(
                "INSERT INTO users (name, email, age) VALUES (%s, %s, %s) RETURNING id",
                (name, email, age),    # Values are passed separately, not concatenated
            )
            user_id = cur.fetchone()[0]   # RETURNING id gives us the new row's id
            conn.commit()                 # Write the change permanently to the database
            return {"id": user_id, "name": name, "email": email, "age": age}
    finally:
        conn.close()   # Always close the connection, even if an exception occurred
```

---

## 6. Fetching Data and Converting to JSON

```python
def get_users() -> list:
    conn = get_connection()
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT id, name, email, age FROM users")
            rows = cur.fetchall()   # Returns a list of tuples: [(1, 'Alice', ...), ...]
            # Convert each tuple to a dict so FastAPI can serialise it to JSON
            return [
                {"id": row[0], "name": row[1], "email": row[2], "age": row[3]}
                for row in rows
            ]
    finally:
        conn.close()
```

---

## 7. Transactions

A **transaction** groups one or more operations. If any step fails, the whole transaction is rolled back — no partial changes.

```python
conn = get_connection()
try:
    with conn.cursor() as cur:
        # Both inserts happen together or not at all
        cur.execute(
            "INSERT INTO users (name, email) VALUES (%s, %s)",
            ("Alice", "alice@example.com")
        )
        cur.execute(
            "INSERT INTO accounts (user_id, balance) VALUES (currval('users_id_seq'), 0)"
        )
    conn.commit()    # Commit: both inserts are written permanently
except Exception:
    conn.rollback()  # Rollback: neither insert is saved if anything went wrong
finally:
    conn.close()
```

---

## 8. Common Database Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `connection refused` | PostgreSQL is not running or wrong host | Check the Docker service is up; use `db` as host in Compose |
| `relation "users" does not exist` | Table has not been created | Run the `CREATE TABLE` SQL first |
| `too many connections` | Connections are opened but not closed | Always call `conn.close()` in a `finally` block |
| `invalid input syntax for type integer` | Passing a string where an integer is expected | Validate and cast types before inserting |
| `duplicate key value violates unique constraint` | Inserting a value that already exists in a `UNIQUE` column | Check for existence first, or catch the exception and return `409 Conflict` |
| `password authentication failed` | Wrong DB password in env vars | Check the `DB_PASSWORD` env var matches `POSTGRES_PASSWORD` in Compose |

---

## 9. Database Safety Rules

These rules apply to every backend service that writes to a database:

- **Use parameterised queries** — never build SQL by concatenating user input (`f"WHERE id={user_input}"` is dangerous)
- **Commit successful writes** and roll back failed transactions
- **Close connections** in a `finally` block or use a connection pool/context manager
- **Keep credentials in environment variables** — never in source code
- **Return safe API errors** — never expose raw database exceptions to the client
- **Verify persistence** by restarting containers and querying the table directly

---

## 10. Debugging the Database

Access the database shell inside Docker:

```bash
docker exec -it <postgres_container_id> psql -U postgres -d appdb
```

You can find the container id with `docker compose ps`.

Useful psql commands:

| Command | What it does |
|---------|-------------|
| `\dt` | List all tables |
| `\d users` | Show the structure of the `users` table |
| `SELECT * FROM users;` | See all rows in the users table |
| `\q` | Quit psql |

---

## 11. Mini Project

1. Create the `users` table in PostgreSQL using the `CREATE TABLE` SQL above.
2. Update `create_user` in your service to insert into the real database.
3. Update `get_users` to query from the database.
4. Verify data persists: stop and restart the containers, confirm users are still there.

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
| SQL safety | Use parameterised queries — never concatenate user input into SQL |

---

## Navigation

[← Part 4: Docker](part-4-docker) | [Part 6: Debugging →](part-6-debugging)
