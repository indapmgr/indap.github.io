---
title: "Part 1 — Foundations"
layout: page
---

# Part 1 — Foundations

> **Key idea:** Learn systems, not commands. Understand *why* a tool exists before memorising its syntax.

---

## Navigation

[← Handbook Home](index.md) | [Part 2: Backend →](part-2-backend-fastapi.md)

---

## 1. How to Approach Learning

The goal is not to memorise commands. The goal is to understand the system.

Every tool exists to solve a problem. Before using a tool, ask:

- What problem does this solve?
- How does it fit into the system?
- What would break without it?

---

## 2. Development Environment Setup

### Required Tools

| Tool | Purpose |
|------|---------|
| Python | Backend scripting and FastAPI |
| Node.js | Frontend runtime (Next.js) |
| npm | Node package manager |
| Git | Version control |
| Docker | Containerised environments |
| VS Code | Code editor |

### Verification Commands

Run each command to confirm the tool is installed:

```bash
python --version
node --version
npm --version
git --version
docker --version
docker compose version
```

Expected output — you should see version numbers, not errors:

```text
Python 3.11.x
v20.x.x
10.x.x
git version 2.x.x
Docker version 24.x.x
Docker Compose version v2.x.x
```

### Why Each Tool Exists

| Tool | Why it exists |
|------|--------------|
| **Python** | Readable, beginner-friendly language. Powers FastAPI for backend APIs |
| **Node.js** | JavaScript runtime outside the browser. Powers Next.js |
| **Git** | Tracks changes, enables team collaboration, prevents data loss |
| **Docker** | Guarantees the same environment on every machine. Eliminates "works on my machine" |
| **VS Code** | Full-featured editor with extensions for every language in this stack |

---

## 3. Common Setup Failures

| Error | Cause | Fix |
|-------|-------|-----|
| `command not found: python` | Python not installed or not on PATH | Install Python and add to PATH |
| `command not found: docker` | Docker Desktop not installed | Install Docker Desktop |
| Docker commands fail | Docker Desktop is not running | Launch Docker Desktop app |
| Version mismatch | Old version installed | Uninstall and reinstall the correct version |
| `npm: command not found` | Node.js not installed | Install Node.js (npm comes with it) |

---

## 4. Git Mental Model

Git tracks your code across four states:

```text
Working Directory → Staging → Commit → Remote
```

| State | What it means |
|-------|--------------|
| **Working Directory** | Files on your machine that you are editing |
| **Staging** | Files marked to include in the next commit (`git add`) |
| **Commit** | A saved snapshot stored in local Git history |
| **Remote** | The commit pushed to GitHub |

### Git Basics

Initialise a repository:

```bash
git init
```

Stage all changes:

```bash
git add .
```

Create a commit:

```bash
git commit -m "Describe what you changed"
```

Push to remote:

```bash
git push
```

### Branching Workflow

**Rules (non-negotiable):**

- Never work directly on `main`
- Create a new branch for every task
- Push your branch to GitHub when done

```bash
# Create and switch to a new branch
git checkout -b feature/my-task

# Do your work, then stage and commit
git add .
git commit -m "Add my feature"

# Push the branch to GitHub
git push -u origin feature/my-task
```

### Common Git Problems

| Problem | Cause | Fix |
|---------|-------|-----|
| Push rejected | Remote has commits you don't have | `git pull` first, then push |
| Merge conflict | Two branches changed the same lines | Manually resolve conflicts, then commit |
| Working on wrong branch | Forgot to create a branch | Stash changes, create branch, pop stash |

---

## 5. Internet Flow and HTTP Basics

### How the Internet Works

```text
Browser → Request → Server → Response
```

1. You type a URL or click a button
2. Your browser sends an HTTP request to a server
3. The server processes the request
4. The server sends back an HTTP response
5. Your browser renders the response

### HTTP Methods

| Method | Purpose | Example |
|--------|---------|---------|
| `GET` | Retrieve data | Get a list of users |
| `POST` | Create new data | Create a new user |
| `PUT` | Update existing data | Update a user's name |
| `DELETE` | Delete data | Delete a user |

### HTTP Status Codes

| Code | Meaning | When you see it |
|------|---------|-----------------|
| `200` | OK | Request succeeded |
| `201` | Created | New resource was created |
| `400` | Bad Request | Client sent invalid data |
| `404` | Not Found | Resource does not exist |
| `500` | Internal Server Error | Something broke on the server |

---

## Part 1 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Learning approach | Understand systems, not just commands |
| Environment | Verify all tools are installed and at the right version |
| Git model | Working Directory → Staging → Commit → Remote |
| Branching | Always branch, never commit directly to main |
| HTTP | Every web interaction is a request/response cycle |

---

## Navigation

[← Handbook Home](index.md) | [Part 2: Backend →](part-2-backend-fastapi.md)
