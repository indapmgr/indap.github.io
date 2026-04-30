# 🐍 Part 3 — Virtual Environments & API Testing

> **Key idea:** A virtual environment isolates your Python packages per project. Testing your API from Python confirms it works correctly — no browser required.

---

## Navigation

[← Part 2: Backend](part-2-backend-fastapi) | [Part 4: Docker →](part-4-docker)

---

## 1. Why Virtual Environments?

Imagine you have two Python projects:

- Project A needs `fastapi==0.95`
- Project B needs `fastapi==0.104`

If you install packages globally (without a virtual environment), they conflict — you can only have one version installed at a time. A **virtual environment** (`venv`) creates an isolated folder of packages for each project.

```text
your-machine/
├── project-a/
│   └── venv/       ← Project A's packages (fastapi 0.95, etc.)
└── project-b/
    └── venv/       ← Project B's packages (fastapi 0.104, etc.)
```

No conflicts. Each project stays clean and independent.

---

## 2. Creating a Virtual Environment

Navigate into your project folder, then run:

**Mac / Linux:**

```bash
python3 -m venv venv
```

**Windows:**

```bash
python -m venv venv
```

This creates a folder called `venv/` inside your project. You only need to do this **once** per project.

> **Important:** Add `venv/` to your `.gitignore` file — never commit your virtual environment to Git. It is large and machine-specific.

Your `.gitignore` should contain:

```text
venv/
__pycache__/
*.pyc
.env
```

---

## 3. Activating the Virtual Environment

You must activate the venv **every time you open a new terminal window**. Without activating, you're using the global Python and your project packages won't be found.

**Mac / Linux:**

```bash
source venv/bin/activate
```

**Windows (PowerShell):**

```bash
venv\Scripts\Activate.ps1
```

**Windows (Command Prompt / Git Bash):**

```bash
venv\Scripts\activate
```

When active, your terminal prompt changes to show `(venv)` at the start:

```text
(venv) user@machine:~/my-project$
```

To deactivate (return to global Python):

```bash
deactivate
```

---

## 4. Common Virtualenv Problems

| Problem | Cause | Fix |
|---------|-------|-----|
| `No such file or directory: venv/bin/activate` | venv not created yet | Run `python3 -m venv venv` first |
| `python: command not found` | Python not installed or wrong command | Try `python3` on Mac/Linux, or install Python on Windows |
| `Activate.ps1 cannot be loaded` (Windows PowerShell) | Execution policy blocked scripts | Run: `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` |
| Packages not found after install | venv not activated | Activate it first: `source venv/bin/activate` |
| Wrong Python version in venv | System Python is 2.x or old | Specify: `python3.11 -m venv venv` |
| `pip: command not found` | pip not available | Try `python -m pip install ...` instead |

---

## 5. Installing Packages

With your venv active, install a package:

```bash
pip install fastapi uvicorn requests
```

Save all installed packages so teammates can recreate your exact environment:

```bash
pip freeze > requirements.txt
```

The file will look like:

```text
fastapi==0.104.1
uvicorn==0.24.0
requests==2.31.0
```

Install from an existing `requirements.txt` (what teammates do when they clone your repo):

```bash
pip install -r requirements.txt
```

---

## 6. Full Project Setup Workflow

Every time you start a new project, follow this sequence:

```bash
# 1. Create the project directory
mkdir my-project
cd my-project

# 2. Create a virtual environment (one-time setup)
python3 -m venv venv

# 3. Activate it (every new terminal session)
source venv/bin/activate          # Mac/Linux
# venv\Scripts\activate           # Windows

# 4. Install dependencies
pip install fastapi uvicorn requests

# 5. Save dependencies for others
pip freeze > requirements.txt

# 6. Start coding!
```

---

## 7. Testing Your API with Python

Instead of opening a browser, you can test your FastAPI endpoints with a simple Python script using the `requests` library. This is a standard backend technique.

The `requests` library lets you make HTTP calls from Python code — the same kind of calls a browser makes, but from a script you control.

Install it (with your venv active):

```bash
pip install requests
```

**`test_api.py`**

```python
import requests   # HTTP client library — pip install requests

BASE_URL = "http://localhost:8000"   # Address where your FastAPI server is running

# ── Test 1: Create a new user ──────────────────────────────────────────────
print("Testing POST /users...")

response = requests.post(
    f"{BASE_URL}/users",              # URL to call
    json={                            # Data to send; requests serialises it to JSON
        "name": "Alice",
        "email": "alice@example.com",
        "age": 28
    }
)

print(f"  Status: {response.status_code}")   # Expect 201 Created
print(f"  Body:   {response.json()}")        # Expect the new user with an id

# ── Test 2: List all users ─────────────────────────────────────────────────
print("\nTesting GET /users...")

response = requests.get(f"{BASE_URL}/users")

print(f"  Status: {response.status_code}")   # Expect 200 OK
print(f"  Body:   {response.json()}")        # Expect a list containing Alice
```

Run it (with your FastAPI server running in a separate terminal):

```bash
python test_api.py
```

Expected output:

```text
Testing POST /users...
  Status: 201
  Body:   {'id': 1, 'name': 'Alice', 'email': 'alice@example.com', 'age': 28}

Testing GET /users...
  Status: 200
  Body:   [{'id': 1, 'name': 'Alice', 'email': 'alice@example.com', 'age': 28}]
```

---

## 8. Testing Error Cases

A good test also checks what happens when things go wrong:

```python
import requests

BASE_URL = "http://localhost:8000"

# ── Test: Send wrong data type ─────────────────────────────────────────────
print("Testing POST /users with invalid age...")

response = requests.post(
    f"{BASE_URL}/users",
    json={
        "name": "Bob",
        "email": "bob@example.com",
        "age": "not-a-number"    # age should be an int — FastAPI should reject this
    }
)

# FastAPI validates input automatically; invalid data returns 422
print(f"  Status: {response.status_code}")   # Expect 422 Unprocessable Entity
print(f"  Body:   {response.json()}")        # Expect a validation error message
```

---

## 9. Testing with a Connection-Error Guard

If the server is not running, your test script will crash. Add a guard:

```python
import requests

BASE_URL = "http://localhost:8000"

try:
    response = requests.get(f"{BASE_URL}/users", timeout=5)   # timeout=5 prevents hanging
    print(f"Status: {response.status_code}")
    print(f"Users:  {response.json()}")
except requests.exceptions.ConnectionError:
    # This happens when the server is not running at all
    print("ERROR: Could not connect to the server.")
    print("Make sure the FastAPI server is running: uvicorn app.main:app --reload")
except requests.exceptions.Timeout:
    # This happens when the server is running but takes too long to respond
    print("ERROR: The server took too long to respond.")
```

---

## 10. Weekly Demo Workflow

Every week you will build something new, test it, and demo it to the team.

### Before the Demo

- [ ] Code runs without errors
- [ ] The feature does what it is supposed to do
- [ ] You can explain what you built and why
- [ ] You know which HTTP endpoints are involved
- [ ] You have tested at least one failure case (wrong input, missing data, etc.)

### During the Demo

1. Start the server: `uvicorn app.main:app --reload`
2. Open `http://localhost:8000/docs` — show the auto-generated docs
3. Walk through the endpoint(s) you built
4. Send a request from `/docs` or your test script
5. Show the response
6. Explain what each part does (route, schema, service, database)

### After the Demo

Write a short reflection (a few sentences is fine):

- What did you build?
- What was hard or confusing?
- What would you do differently next time?

---

## 11. Mini Project

1. Create a Python virtual environment for your FastAPI project.
2. Install `fastapi`, `uvicorn`, and `requests`.
3. Write a `test_api.py` that tests both `POST /users` and `GET /users`.
4. Run the tests and fix any failures.

---

## Exercises

1. Add a test for `GET /users/{user_id}` — verify it returns `404` when the user doesn't exist.
2. Add a connection-error guard to `test_api.py` so it prints a helpful message if the server is not running.
3. Prepare a 3-minute demo of your working API.

---

## Part 3 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Virtual environment | Isolates packages per project; prevents version conflicts |
| Activation | Must activate in every new terminal session |
| `requirements.txt` | Records all dependencies so anyone can recreate the environment |
| `requests` library | Test your API from Python — no browser needed |
| Weekly demo | Build → Test → Demo → Reflect each week |

---

## Navigation

[← Part 2: Backend](part-2-backend-fastapi) | [Part 4: Docker →](part-4-docker)
