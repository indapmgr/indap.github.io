# 🎯 Part 9 — Final Evaluation, Readiness, and Expectations

> This section defines what "ready" actually means, how interns will be evaluated, and how to behave in a real engineering team.

---

## Navigation

[← Part 8: Full Project](part-8-full-project) | [Handbook Home →](index)

---

## 0. What "Ready" Actually Means

Being "ready" is **not**:

- Finishing tutorials
- Copying code
- Knowing syntax

Being ready means:

> You can take a task and complete it independently.

### Real Definition

```text
Understand problem → Break it down → Implement → Debug → Deliver
```

---

## 1. Core Skills Required

These are **non-negotiable**.

### Technical Skills

You **must** be able to:

- Set up a Python virtual environment and install dependencies
- Run the full backend system locally (API + database + Docker)
- Create FastAPI endpoints with correct routes, schemas, and services
- Persist data in PostgreSQL using parameterised queries
- Use Docker Compose to start the full system with one command
- Debug failures using logs, `print()`, and `psql`
- Explain how you verified the service works

### Git Skills

You **must** be able to:

- Create feature branches
- Push code to GitHub
- Open pull requests
- Resolve merge conflicts

### Debugging Skills

Debugging is the **most important skill**.

You **must** be able to:

- Identify which layer failed (API, database, Docker, network)
- Use logs effectively (`uvicorn` output, `docker compose logs`)
- Fix issues without guessing

---

## 2. Evaluation Criteria

### Project Completion

| Area | Expectation |
|------|-------------|
| API | All endpoints work correctly with proper status codes |
| Database | Data persists after container restarts |
| Docker | Full system starts with `docker compose up --build` |
| Testing | `test_api.py` passes; error cases are covered |

### Code Quality

| Good | Bad |
|------|-----|
| Clean structure (routes/services/schemas/db) | Everything in one file |
| Readable variable and function names | `x`, `temp`, `data2` |
| Proper input validation with Pydantic | No validation at all |
| Parameterised SQL queries | String-concatenated SQL |

### Debugging Ability

You will be tested on:

- Fixing a broken API endpoint
- Fixing a database connection failure
- Diagnosing a Docker container that won't start

### Git Workflow

You must:

- Not push directly to `main`
- Use meaningful commit messages (verb + what + why)
- Follow the pull request workflow

---

## 3. Final Test (Mandatory)

Build the following from scratch without copying blindly. You must understand each step well enough to explain it.

### API

- `POST /users` — accept `name`, `email`, `age`; return the created user with `201`
- `GET /users` — return all users from the database
- `GET /users/{user_id}` — return one user by id; return `404` if not found
- `GET /` — health check returning `{"status": "ok"}`

### Database

- Store users persistently in PostgreSQL
- Data must survive a container restart
- Use parameterised queries (no string concatenation)

### Docker

- Full system starts and runs with `docker compose up --build`
- Backend and database work together inside Docker Compose

### Testing

- Write a `test_api.py` using the `requests` library
- Test all endpoints including error cases
- All assertions must pass

---

## 4. Real-World Scenarios

You will encounter situations like these. Know how to handle them.

### Scenario 1: API Works Locally but Fails in Docker

Expected approach:
- Check environment variables in `docker-compose.yml` — especially `DB_HOST`
- Remember: use service names (`db`) not `localhost` inside Docker Compose
- Read `docker compose logs backend` for the actual error

### Scenario 2: Database Is Not Saving Data

Expected approach:
- Check that `conn.commit()` is called after the insert
- Verify the query runs without exceptions (`try/except` may be hiding the error)
- Connect to psql directly and check the table:

```bash
docker exec -it <postgres_id> psql -U postgres -d appdb
```

Then run:

```sql
-- Check if any rows exist
SELECT * FROM users;
```

### Scenario 3: Container Keeps Restarting

Expected approach:
- Run `docker compose logs backend` and read the full error
- Look at the bottom of the stack trace for the actual exception
- Common causes: missing env var, missing dependency in `requirements.txt`, Python syntax error

### Scenario 4: Git Conflict During Pull Request

Expected approach:
- Pull the target branch locally
- Open conflicting files, find the `<<<<<<<` markers
- Resolve by choosing or combining the correct code
- Remove all conflict markers, save, commit, push

---

## 5. Engineering Expectations

### Ownership

You are responsible for your code.

That means:

- You understand what you wrote
- You can explain it to another person
- You can debug it when it breaks
- You can improve it when asked

### Consistency

Follow:

- Project structure conventions (routes / services / schemas / db)
- Naming conventions (snake_case for Python functions, descriptive names)
- Team workflow (branch → PR → review → merge)

### Thinking Before Coding

Before writing code, ask:

- What is the input? What is the expected output?
- Where does this fit in the system? (route? service? schema?)
- What can go wrong? How should errors be handled?

### Debug Mindset

```text
Observe → Isolate → Fix → Verify
```

---

## 6. Common Failure Behaviors

Avoid these.

### Guessing

Do not randomly change code hoping something will fix the problem.

Instead:
- Read the error message completely
- Check the logs
- Isolate which layer failed
- Make one targeted change at a time

### Copy-Paste Coding

Do not copy code without understanding it.

If you use code from documentation or examples, you must be able to explain:
- What it does
- Why it is needed
- Where it fits in the system

### Ignoring Errors

Errors are clues. Do not skip or silence them.

Read every error message carefully. The first and last lines of a stack trace contain the most information.

### Skipping Git Workflow

Always:
- Use branches for every task
- Make meaningful commits
- Open pull requests before merging
- Resolve conflicts properly

---

## 7. Self-Check Before Saying "Done"

Ask yourself:

### System

- Does `docker compose up --build` start cleanly?
- Do all API endpoints return the expected status codes?
- Does the database persist data after a restart?

### Code

- Is the structure clean and properly separated?
- Is Pydantic validation present on all inputs?
- Are SQL queries parameterised?
- Are secrets and configuration read from environment variables?

### Debugging

- Did I test failure cases (wrong input, missing user, duplicate email)?
- Did I check logs?
- Did I verify the fix works end-to-end?

### Delivery

- Does the PR explain what changed and why?
- Does the PR include exact steps to verify the work?
- Are any limitations or follow-up tasks noted?

---

## 8. What Makes You Stand Out

| Level | Behaviour |
|-------|-----------|
| **Good intern** | Completes assigned tasks |
| **Great intern** | Understands the full system, not just their piece |
| **Exceptional intern** | Debugs independently, improves code quality, asks precise questions |

---

## 9. Growth Path

After completing this handbook, aim to:

- Learn API design patterns more deeply (REST best practices, versioning)
- Study SQL query optimisation (indexes, EXPLAIN)
- Learn about connection pooling (e.g. `psycopg2.pool` or `asyncpg`)
- Study system design fundamentals
- Learn to handle production incidents
- Improve debugging speed and accuracy
- Read and review other engineers' code

---

## Final Summary

If you can:

- Build a full backend API system (FastAPI + PostgreSQL + Docker)
- Debug issues independently using logs and systematic isolation
- Use Git correctly with branches and PRs
- Write a test script that verifies all endpoints
- Run a Docker Compose setup from scratch
- Show evidence that your work was tested and verified

Then:

> **You are ready to contribute.**

---

## Final Note

This handbook is not something to read once.

It is something to:

- Revisit when you face a problem
- Practice by building real things
- Apply every day

---

## Navigation

[← Part 8: Full Project](part-8-full-project) | [Handbook Home →](index)
