# 🔧 Part 1 — Foundations

> **Key idea:** Learn systems, not commands. Understand *why* a tool exists before memorising its syntax.

---

## Navigation

[← Handbook Home](index) | [Part 2: Backend →](part-2-backend-fastapi)

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
| Python 3.11+ | Backend language that powers FastAPI |
| Git | Version control — tracks all your code changes |
| Docker | Runs services in containers; same environment everywhere |
| VS Code | Code editor with extensions for Python, Docker, Git |

> **Note:** You do not need Node.js, npm, or any JavaScript tools for this backend handbook.

### Installation Links

- **Python**: [python.org/downloads](https://www.python.org/downloads/) — choose Python 3.11 or newer
- **Git**: [git-scm.com](https://git-scm.com/) — includes Git Bash on Windows
- **Docker Desktop**: [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
- **VS Code**: [code.visualstudio.com](https://code.visualstudio.com/)

### Verification Commands

After installing, open a terminal and run each command. You should see version numbers, not errors:

```bash
python --version
git --version
docker --version
docker compose version
```

Expected output:

```text
Python 3.11.x
git version 2.x.x
Docker version 24.x.x
Docker Compose version v2.x.x
```

> **Windows tip:** If `python` is not found, try `python3`. If neither works, Python is not on your PATH — see the troubleshooting table below.

### Why Each Tool Exists

| Tool | Why it exists |
|------|--------------|
| **Python** | Readable, beginner-friendly language. Powers FastAPI for backend APIs |
| **Git** | Tracks changes, enables team collaboration, prevents data loss |
| **Docker** | Guarantees the same environment on every machine. Eliminates "works on my machine" |
| **VS Code** | Full-featured editor with extensions for Python, Docker, and Git |

---

## 3. Common Setup Failures

These are the most common problems beginners hit. Check here before asking for help.

| Error | Cause | Fix |
|-------|-------|-----|
| `command not found: python` | Python not installed or not on PATH | Install Python; on Windows, tick "Add Python to PATH" during install |
| `python` opens the Microsoft Store (Windows) | Windows App Execution Aliases intercepting | Disable in Settings → Apps → Advanced app settings → App execution aliases |
| `command not found: docker` | Docker Desktop not installed | Install Docker Desktop |
| Docker commands fail | Docker Desktop is not running | Launch the Docker Desktop app from your applications folder |
| `docker compose` not found | Old Docker version uses `docker-compose` (hyphen) | Update Docker Desktop, or use `docker-compose` (with hyphen) |
| `Permission denied` (Mac/Linux) | Running Docker without sudo or group membership | Add your user to the `docker` group: `sudo usermod -aG docker $USER` then log out and back in |
| `SSL: CERTIFICATE_VERIFY_FAILED` (Mac) | Python SSL certificates not installed | Run `/Applications/Python\ 3.x/Install\ Certificates.command` |
| VS Code can't find Python | Python not on PATH or wrong interpreter | Press `Ctrl+Shift+P` → "Python: Select Interpreter" and choose the correct one |

---

## 4. Git Mental Model

Git tracks your code across four states:

```text
Working Directory → Staging → Commit → Remote
```

| State | What it means |
|-------|--------------|
| **Working Directory** | Files on your machine that you are editing right now |
| **Staging** | Files you have marked to include in the next commit (`git add`) |
| **Commit** | A saved snapshot stored in your local Git history |
| **Remote** | The commit pushed to GitHub where others can see it |

### Git Basics

Before you start, tell Git who you are (do this once):

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Initialise a new repository:

```bash
git init
```

Check what has changed (run this often):

```bash
git status
```

Stage all changed files:

```bash
git add .
```

Create a commit with a description:

```bash
git commit -m "Describe what you changed and why"
```

Push to remote (GitHub):

```bash
git push
```

### Branching Workflow

**Rules (non-negotiable):**

- Never work directly on `main`
- Create a new branch for every task
- Push your branch to GitHub when done

```bash
# Start from the latest main branch
git checkout main
git pull

# Create and switch to a new branch for your task
git checkout -b feature/my-task

# Do your work, then stage and commit
git add .
git commit -m "Add my feature"

# Push the branch to GitHub so others can see it
git push -u origin feature/my-task
```

### Common Git Problems

| Problem | Cause | Fix |
|---------|-------|-----|
| Push rejected | Remote has commits you don't have locally | `git pull` first, resolve any conflicts, then `git push` |
| Merge conflict | Two branches changed the same lines | Manually resolve conflicts, remove the conflict markers, then commit |
| Working on wrong branch | Forgot to create a branch before coding | `git stash`, create the correct branch, then `git stash pop` |
| `fatal: not a git repository` | You're not in a git repo folder | Run `git init` or `cd` into the correct project folder |

---

## 5. Internet Flow and HTTP Basics

### How the Internet Works

```text
Client → HTTP Request → Server → HTTP Response → Client
```

1. You type a URL or click a button in a browser
2. Your browser (the **client**) sends an HTTP **request** to a server
3. The server processes the request (runs code, queries the database)
4. The server sends back an HTTP **response** with data
5. Your browser renders the response

### HTTP Methods

| Method | Purpose | Example |
|--------|---------|---------|
| `GET` | Retrieve data (no side effects) | Get a list of users |
| `POST` | Create new data | Create a new user |
| `PUT` | Replace existing data completely | Update all fields of a user |
| `PATCH` | Update part of existing data | Update just a user's name |
| `DELETE` | Delete data | Delete a user |

### HTTP Status Codes

| Code | Meaning | When you see it |
|------|---------|-----------------|
| `200` | OK | Request succeeded (GET, PUT, PATCH) |
| `201` | Created | A new resource was created (POST) |
| `400` | Bad Request | Client sent invalid or malformed data |
| `404` | Not Found | The resource you asked for does not exist |
| `422` | Unprocessable Entity | Data format is valid but fails validation rules |
| `500` | Internal Server Error | Something broke on the server side |

---

## 6. Terminal Basics

If you are new to the terminal, here are the most important commands:

| Command | What it does | Example |
|---------|-------------|---------|
| `pwd` | Print working directory (where am I?) | `pwd` |
| `ls` (Mac/Linux) / `dir` (Windows) | List files in current folder | `ls` |
| `cd <folder>` | Change directory | `cd my-project` |
| `cd ..` | Go up one directory | `cd ..` |
| `mkdir <name>` | Create a new folder | `mkdir backend` |
| `touch <file>` (Mac/Linux) | Create a new empty file | `touch main.py` |
| `cat <file>` | Print file contents | `cat requirements.txt` |
| `Ctrl+C` | Stop a running program | Use when the server is running and you want to stop it |

> **Windows users:** Use **Git Bash** (installed with Git) or **Windows Terminal** with PowerShell. Many Unix commands like `ls` and `touch` work in Git Bash.

---

## Part 1 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Learning approach | Understand systems, not just commands |
| Environment | Python, Git, Docker, VS Code — verify all are installed |
| Git model | Working Directory → Staging → Commit → Remote |
| Branching | Always branch, never commit directly to main |
| HTTP | Every web interaction is a request/response cycle |
| Terminal | Learn the basic navigation commands — you'll use them every day |

---

## Navigation

[← Handbook Home](index) | [Part 2: Backend →](part-2-backend-fastapi)
