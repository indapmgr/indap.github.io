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

> **Full setup instructions:** If you haven't set up your development environment yet, go through [Part 1 — Section 1.9: Create and Activate a Virtual Environment](part-1-development-environment) first. It covers creating, activating, and troubleshooting venvs on Windows, Mac, and Linux with detailed error guidance.

---

## 2. Quick Venv Reference

This is a quick-reference summary. For full details and troubleshooting, see [Part 1 — Development Environment](part-1-development-environment).

**Create (once per project):**

```bash
python3 -m venv venv    # Mac / Linux
python -m venv venv     # Windows
```

**Activate (every new terminal session):**

| OS | Command |
|----|---------|
| 🪟 Windows (PowerShell) | `venv\Scripts\Activate.ps1` |
| 🪟 Windows (Command Prompt) | `venv\Scripts\activate.bat` |
| 🪟 Windows (Git Bash) | `source venv/Scripts/activate` |
| 🍎 Mac / 🐧 Linux | `source venv/bin/activate` |

When active, `(venv)` appears at the start of your terminal prompt.

**Deactivate:**

```bash
deactivate
```

> **Important:** Add `venv/` to your `.gitignore` file — never commit your virtual environment to Git. It is large and machine-specific.

Your `.gitignore` should contain:

```text
venv/
__pycache__/
*.pyc
.env
```

---

## 3. Installing Packages

With your venv active, install a package:

```bash
pip install fastapi uvicorn requests    # Windows
pip3 install fastapi uvicorn requests   # Mac / Linux
```

Save all installed packages so teammates can recreate your exact environment:

```bash
pip freeze > requirements.txt    # Windows
pip3 freeze > requirements.txt   # Mac / Linux
```

The file will look like:

```text
fastapi==0.104.1
uvicorn==0.24.0
requests==2.31.0
```

Install from an existing `requirements.txt` (what teammates do when they clone your repo):

```bash
pip install -r requirements.txt    # Windows
pip3 install -r requirements.txt   # Mac / Linux
```

---

## 4. Full Project Setup Workflow

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
pip install fastapi uvicorn requests    # Windows
pip3 install fastapi uvicorn requests   # Mac / Linux

# 5. Save dependencies for others
pip freeze > requirements.txt    # Windows
pip3 freeze > requirements.txt   # Mac / Linux

# 6. Start coding!
```

---

## 5. Testing Your API with Python

Instead of opening a browser, you can test your FastAPI endpoints with a simple Python script using the `requests` library. This is a standard backend technique.

The `requests` library lets you make HTTP calls from Python code — the same kind of calls a browser makes, but from a script you control.

Install it (with your venv active):

```bash
pip install requests    # Windows
pip3 install requests   # Mac / Linux
```

**`test_api.py`** (exploratory demo — prints results for you to inspect)

> This demo-style script prints what the server returns so you can see and understand the output. Part 8 shows a full test script with assertions that automatically fail when a response is wrong.

```python
import requests   # HTTP client library — install with pip (Windows) or pip3 (Mac/Linux)

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

## 6. Testing Error Cases

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

## 7. Testing with a Connection-Error Guard

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

## 8. Weekly Demo Workflow

Every week you will build something new, test it, and demo it to the team.

### 📌 Push to GitHub Before Every Demo

**This is not optional.** Before your demo, your project must be pushed to your GitHub repository. The team reviews and evaluates your work from GitHub — code that exists only on your laptop does not count.

**Quick push checklist:**

```bash
# 1. Make sure you're in your project folder
cd my-project

# 2. Stage all changes
git add .

# 3. Commit with a meaningful message (what did you build this week?)
git commit -m "Add user creation endpoint with age validation"

# 4. Push to GitHub
git push
```

Then confirm at `github.com/<your-username>/<your-repo>` that your latest code is there.

> **Tip:** Push at least once a day during the week, not just before the demo. It gives you a backup, shows your progress, and makes the final push less stressful.

---

### Before the Demo

- [ ] Code runs without errors
- [ ] The feature does what it is supposed to do
- [ ] You can explain what you built and why
- [ ] You know which HTTP endpoints are involved
- [ ] You have tested at least one failure case (wrong input, missing data, etc.)
- [ ] **Your latest code is pushed to GitHub** ✅

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

## 9. Mini Project

1. Create a Python virtual environment for your FastAPI project.
2. Install `fastapi`, `uvicorn`, and `requests`.
3. Write a `test_api.py` that tests both `POST /users` and `GET /users`.
4. Run the tests and fix any failures.
5. Push the project to a new GitHub repository named `fastapi-mini-project`.

> **📌 Remember:** Push to GitHub when done. See [Part 1 — section 1.11](part-1-development-environment#111-initialize-your-demo-project-and-push-to-github) for the full push guide.

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
