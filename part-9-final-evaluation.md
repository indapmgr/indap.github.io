---
title: "Part 9 — Final Evaluation, Readiness, and Expectations"
layout: page
---

# Part 9 — Final Evaluation, Readiness, and Expectations

> This section defines what "ready" actually means, how interns will be evaluated, and how to behave in a real engineering team.

---

## Navigation

[← Part 8: Full Project](part-8-full-project.md) | [Handbook Home →](index.md)

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

- Run the full system locally (frontend + backend + database)
- Create API endpoints
- Connect frontend to backend
- Persist data in the database
- Use Docker Compose
- Debug failures

### Git Skills

You **must** be able to:

- Create feature branches
- Push code to GitHub
- Open pull requests
- Resolve merge conflicts

### Debugging Skills

Debugging is the **most important skill**.

You **must** be able to:

- Identify which layer failed
- Use logs effectively
- Fix issues without guessing

---

## 2. Evaluation Criteria

### Project Completion

| Area | Expectation |
|------|-------------|
| API | Works correctly |
| Frontend | Connected properly to backend |
| Database | Persists data |
| Docker | Runs the full system |

### Code Quality

| Good | Bad |
|------|-----|
| Clean structure | Everything in one file |
| Readable code | Messy, tangled logic |
| Proper validation | No input validation |
| Meaningful variable names | `x`, `temp`, `data2` |

### Debugging Ability

You will be tested on:

- Fixing a broken API
- Fixing a database connection failure
- Fixing Docker issues

### Git Workflow

You must:

- Not push directly to `main`
- Use meaningful commit messages
- Follow the pull request flow

---

## 3. Final Test (Mandatory)

Build the following from scratch:

### Backend

- `POST /users` — accept `name`, `email`, `age`; return the created user
- `GET /users` — return all users from the database

### Frontend

- Input form with `name`, `email`, `age` fields
- List that displays all users

### Database

- Store users persistently in PostgreSQL
- Data must survive a container restart

### Docker

- Full system starts and runs with `docker compose up --build`
- Frontend, backend, and database work together

### Constraint

You must not copy blindly. You must understand each step well enough to explain it.

---

## 4. Real-World Scenarios

You will encounter situations like these. Know how to handle them.

### Scenario 1: API Works Locally but Fails in Docker

Expected approach:
- Identify environment differences (host names, ports, env vars)
- Fix configuration for the Docker context
- Use `docker compose logs` to read the error

### Scenario 2: Database Is Not Saving Data

Expected approach:
- Check that `conn.commit()` is called
- Verify the query runs without errors
- Connect to psql and check the table directly

### Scenario 3: UI Is Not Updating

Expected approach:
- Check frontend state with `console.log`
- Inspect the API response in DevTools → Network
- Confirm the component re-renders when state changes

### Scenario 4: Git Conflict During Pull Request

Expected approach:
- Pull the target branch locally
- Open conflicting files and resolve manually
- Test after resolving
- Push the updated branch

---

## 5. Engineering Expectations

### Ownership

You are responsible for your code.

That means:

- You understand what you wrote
- You can explain it
- You can debug it
- You can improve it

### Consistency

Follow:

- Project structure conventions
- Naming conventions
- Team workflow

### Thinking Before Coding

Before writing code, ask:

- What is the input?
- What is the output?
- Where does this fit in the system?

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
- Read the error
- Check logs
- Isolate the issue
- Make one targeted change at a time

### Copy-Paste Coding

Do not copy code without understanding it.

If you use code, you should be able to explain:
- What it does
- Why it works
- Where it fits in the system

### Ignoring Errors

Errors are clues. Do not skip them.

Read every error message carefully. The answer is usually in the first line of the stack trace.

### Skipping Git Workflow

Skipping Git workflow breaks team flow.

Always:
- Use branches
- Make meaningful commits
- Open pull requests
- Resolve conflicts properly

---

## 7. Self-Check Before Saying "Done"

Ask yourself:

### System

- Does the frontend call the backend correctly?
- Does the backend return the correct data?
- Does the database persist data after a restart?

### Code

- Is the structure clean and properly separated?
- Is validation present?
- Is the logic readable?

### Debugging

- Did I test failure cases (wrong input, missing data)?
- Did I check logs?
- Did I verify the fix works end-to-end?

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

- Learn API design patterns more deeply
- Improve SQL query efficiency
- Study system design fundamentals
- Learn to handle production incidents
- Improve debugging speed and accuracy
- Read and review other engineers' code

---

## Final Summary

If you can:

- Build a full-stack system
- Debug issues independently
- Use Git correctly with branches and PRs
- Run a Docker Compose setup

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

[← Part 8: Full Project](part-8-full-project.md) | [Handbook Home →](index.md)
