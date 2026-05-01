# 🔧 Part 1 — Foundations

> **Key idea:** Before you write a single line of backend code, your machine needs to be set up correctly. This section walks you through every step — from installing your IDE to running your first Python program. Future sections assume you have completed this setup.
>
> The goal is not to memorise commands. The goal is to understand the system. Every tool exists to solve a problem — understand *why* it exists, not just how to use it.

---

## Navigation

[← Handbook Home](index) | [Part 2: Backend →](part-2-backend-fastapi)

---

## 1. Full Development Environment Setup

This is the most important section in the handbook. Get everything here working before moving forward. When something goes wrong in a later section, the first question will always be: "Did you complete the setup checklist in Part 1?"

---

### 1.1 Choose Your IDE — VS Code or PyCharm?

An **IDE** (Integrated Development Environment) is the app you use to write, run, and debug code. Think of it as a text editor with superpowers — it highlights errors, lets you run code, shows you your files, and includes a terminal.

You have two great options for Python. Pick one and stick with it — do not install both and switch back and forth while you're learning.

#### Side-by-Side Comparison

| Feature | 🟦 VS Code | 🟩 PyCharm |
|---------|-----------|----------|
| **Cost** | Free, open source | Free Community edition; paid Professional |
| **Weight** | Lightweight (~100 MB) | Heavier (~700 MB+) |
| **Setup effort** | A few extensions needed | Python-ready out of the box |
| **Best for** | All languages, flexible | Python/Django/FastAPI projects |
| **Virtual env detection** | Manual (select interpreter) | Automatic (detects venv on open) |
| **Beginner friendliness** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Professional use** | Very common | Common in Python-heavy teams |

> **Recommendation:** Start with **VS Code** — it is the most widely used editor on real engineering teams and works for every language you'll ever encounter. PyCharm is an excellent alternative if you find VS Code confusing or want automatic Python setup.

---

### 1.2 Installing VS Code

**Download:** [code.visualstudio.com](https://code.visualstudio.com/)

<details markdown="1">
<summary>🪟 Windows</summary>

1. Go to [code.visualstudio.com](https://code.visualstudio.com/) and click **Download for Windows**.
2. Run the downloaded `.exe` installer.
3. On the "Select Additional Tasks" screen, tick **both** of these:
   - ✅ `Add "Open with Code" action to Windows Explorer file context menu`
   - ✅ `Add to PATH (requires shell restart)`
4. Finish the installer and reopen any terminal windows.
5. Verify: open a terminal and run `code --version`. You should see a version number.

> **Common issue:** If `code` is not recognised after install, restart your computer to apply the PATH change.

</details>

<details markdown="1">
<summary>🍎 Mac</summary>

1. Go to [code.visualstudio.com](https://code.visualstudio.com/) and click **Download for Mac**.
2. Open the downloaded `.zip` — it extracts `Visual Studio Code.app`.
3. Drag `Visual Studio Code.app` into your **Applications** folder.
4. Open VS Code, then press `Cmd+Shift+P`, type `shell command`, and select **"Shell Command: Install 'code' command in PATH"**.
5. Verify: open Terminal and run `code --version`.

> **Common issue:** "VS Code cannot be opened because it is from an unidentified developer." Go to **System Settings → Privacy & Security** and click **Open Anyway**.

</details>

<details markdown="1">
<summary>🐧 Linux (Ubuntu/Debian)</summary>

```bash
# Download and install the Microsoft GPG key and repository
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'

# Install VS Code
sudo apt update
sudo apt install code
```

Verify: run `code --version`.

</details>

#### Install Essential VS Code Extensions

After installing VS Code, add these extensions (they make Python development much easier):

1. Press `Ctrl+Shift+X` (Windows/Linux) or `Cmd+Shift+X` (Mac) to open the Extensions panel.
2. Search for and install each of these:

| Extension | Publisher | Why you need it |
|-----------|-----------|----------------|
| **Python** | Microsoft | Syntax highlighting, linting, debugger, interpreter selection |
| **Pylance** | Microsoft | Smarter code completion and type checking |
| **GitLens** | GitKraken | See who changed what and when, right in the editor |
| **Docker** | Microsoft | View and manage containers from the sidebar |

---

### 1.3 Installing PyCharm (Alternative)

**Download:** [jetbrains.com/pycharm/download](https://www.jetbrains.com/pycharm/download/)

Download the **Community** edition — it is free and covers everything in this handbook.

<details markdown="1">
<summary>🪟 Windows</summary>

1. Download the Community installer (`.exe`).
2. Run it and follow the wizard.
3. On the Installation Options screen, tick **"Add launchers to PATH"** so you can open PyCharm from a terminal.
4. Launch PyCharm from the Start menu after install.

</details>

<details markdown="1">
<summary>🍎 Mac</summary>

1. Download the Community `.dmg`.
2. Open the `.dmg` and drag **PyCharm CE** into **Applications**.
3. Open PyCharm from Applications — macOS may ask you to confirm opening an app from the internet; click **Open**.

</details>

<details markdown="1">
<summary>🐧 Linux</summary>

```bash
# Install using the JetBrains Toolbox (recommended) or snap
sudo snap install pycharm-community --classic
```

Or download the `.tar.gz` from the website, extract it, and run `bin/pycharm.sh`.

</details>

#### PyCharm First-Run Setup

1. On first launch, PyCharm asks about plugins — skip and use defaults.
2. Choose **New Project** to create your first project (covered in section 1.7).
3. PyCharm automatically detects Python installed on your machine — you'll see it pre-selected in the interpreter dropdown.

> **PyCharm tip:** Unlike VS Code, PyCharm creates a virtual environment automatically when you create a new project. It also shows you which interpreter is active in the bottom-right status bar at all times.

---

### 1.4 Installing Python

Python is the programming language this handbook is built around.

**Download:** [python.org/downloads](https://www.python.org/downloads/)

Always install **Python 3.11 or newer**. Python 2 is obsolete — do not install it.

<details markdown="1">
<summary>🪟 Windows</summary>

1. Go to [python.org/downloads](https://www.python.org/downloads/) and click the latest **Python 3.x.x** button.
2. Run the downloaded `.exe` installer.
3. ⚠️ **Critical step:** On the first screen, tick **"Add Python to PATH"** before clicking Install Now.

   ```
   ☑ Add Python 3.x to PATH    ← TICK THIS or python won't work in any terminal
   ```

4. Click **Install Now**.
5. Once done, open a new terminal window (important — old windows don't see the new PATH) and verify:

   ```
   python --version
   ```

   Expected output: `Python 3.11.x` (or newer)

**Common Windows issues:**

| Problem | What you see | Fix |
|---------|-------------|-----|
| Forgot to add to PATH | `'python' is not recognized as an internal or external command` | Reinstall Python and tick "Add to PATH", or add it manually via System Properties → Environment Variables |
| Python opens Microsoft Store | Microsoft Store opens instead of Python | Go to **Settings → Apps → Advanced app settings → App execution aliases** and turn off `python.exe` and `python3.exe` |
| Wrong version | `Python 2.7.x` | You have Python 2 installed — use `python3` command instead, or uninstall Python 2 |

</details>

<details markdown="1">
<summary>🍎 Mac</summary>

macOS comes with Python 2 pre-installed (old and unusable for our purposes). Install Python 3 separately.

**Option A — Official installer (recommended for beginners):**

1. Download from [python.org/downloads](https://www.python.org/downloads/) — the macOS installer `.pkg`.
2. Run the package installer and follow the steps.
3. After install, run the SSL certificate fixer that comes with it:

   ```bash
   /Applications/Python\ 3.11/Install\ Certificates.command
   ```

   (Replace `3.11` with your installed version number.)

4. Verify in Terminal:

   ```bash
   python3 --version
   ```

**Option B — Homebrew (for developers who already have Homebrew):**

```bash
brew install python@3.11
```

**Common Mac issues:**

| Problem | Fix |
|---------|-----|
| `python: command not found` | Use `python3` — macOS ships Python 2 as `python` |
| `SSL: CERTIFICATE_VERIFY_FAILED` | Run the Install Certificates script from the Python Applications folder |
| `zsh: command not found: python3` | Python not on PATH — add to `~/.zshrc`: `export PATH="/usr/local/bin:$PATH"` |

</details>

<details markdown="1">
<summary>🐧 Linux (Ubuntu/Debian)</summary>

```bash
# Update package list
sudo apt update

# Install Python 3.11
sudo apt install python3.11 python3.11-venv python3-pip -y

# Verify
python3.11 --version
python3 --version
pip3 --version
```

> **Tip:** On Ubuntu 22.04+, `python3` is Python 3.10+. Use `python3.11` explicitly if you need that specific version.

</details>

---

### 1.5 Installing Git

Git is the tool that tracks every change you make to your code. It lets you go back in time, work in teams, and share code on GitHub.

**Download:** [git-scm.com](https://git-scm.com/)

<details markdown="1">
<summary>🪟 Windows</summary>

1. Download the Git for Windows installer from [git-scm.com](https://git-scm.com/).
2. Run the installer — you can accept defaults on most screens.
3. On the screen **"Choosing the default editor used by Git"**, select **VS Code** from the dropdown if you installed it.
4. On **"Adjusting your PATH environment"**, keep the default: `Git from the command line and also from 3rd-party software`.
5. Finish the install.

After install, you now have **Git Bash** — a Unix-style terminal on Windows. You can use this instead of Command Prompt for all terminal commands in this handbook.

Verify (open Git Bash or PowerShell):

```bash
git --version
```

Expected: `git version 2.x.x`

</details>

<details markdown="1">
<summary>🍎 Mac</summary>

**Option A — Xcode Command Line Tools (simplest):**

```bash
xcode-select --install
```

A dialog will appear — click **Install**. This installs Git (and other dev tools) automatically.

**Option B — Homebrew:**

```bash
brew install git
```

Verify:

```bash
git --version
```

</details>

<details markdown="1">
<summary>🐧 Linux (Ubuntu/Debian)</summary>

```bash
sudo apt update
sudo apt install git -y

git --version
```

</details>

#### First-Time Git Configuration

After installing Git on any OS, tell it your name and email. Git uses this to label your commits.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

You only do this **once** per machine. It applies to all your Git projects.

---

### 1.6 Installing Docker Desktop

Docker lets you run software in isolated containers — same code, same environment, on any machine. You'll use it later for running databases and packaging your API.

**Download:** [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)

<details markdown="1">
<summary>🪟 Windows</summary>

1. Download Docker Desktop for Windows.
2. Run the installer. It may prompt you to enable **WSL 2** (Windows Subsystem for Linux) — click Enable/Install if asked. This is required.
3. Restart your computer if asked.
4. Launch Docker Desktop from the Start menu — it starts in the background (look for the whale icon in the system tray).

Verify (in any terminal):

```bash
docker --version
docker compose version
```

**Common Windows issues:**

| Problem | Fix |
|---------|-----|
| "WSL 2 installation is incomplete" | Open PowerShell as Admin and run: `wsl --update` |
| Docker Desktop won't start | Make sure virtualisation is enabled in BIOS (usually is on modern PCs) |
| `docker: command not found` | Docker Desktop is not running — start it from the Start menu |

</details>

<details markdown="1">
<summary>🍎 Mac</summary>

1. Download Docker Desktop for Mac — choose **Apple Silicon** (M1/M2/M3) or **Intel** based on your Mac.
2. Open the `.dmg` and drag Docker to Applications.
3. Open Docker from Applications — it starts in the background (whale icon in the menu bar).
4. Wait for the whale icon to become steady (not animated) — Docker is ready.

Verify:

```bash
docker --version
docker compose version
```

</details>

<details markdown="1">
<summary>🐧 Linux (Ubuntu)</summary>

```bash
# Install Docker Engine
sudo apt update
sudo apt install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Run Docker without sudo (log out and back in after this)
sudo usermod -aG docker $USER
```

Verify after logging out and back in:

```bash
docker --version
docker compose version
```

> **Note:** On Linux there is no Docker Desktop GUI — Docker runs as a background service. This is fine for this handbook.

</details>

---

### 1.7 Verify All Installations

Open a terminal (Git Bash on Windows, Terminal on Mac/Linux) and run each command. Every command should return a version number — not an error.

```bash
# Check Python
python --version         # Windows
python3 --version        # Mac / Linux

# Check pip (Python package manager)
pip --version            # Windows
pip3 --version           # Mac / Linux

# Check Git
git --version

# Check Docker
docker --version
docker compose version
```

Expected output (your exact versions may differ — that's fine):

```text
Python 3.11.9
pip 23.3.1 from ...
git version 2.44.0
Docker version 26.1.1
Docker Compose version v2.27.0
```

> ⚠️ If any command gives an error, stop here and fix it before continuing. Every other part of this handbook depends on these tools working.

---

### 1.8 Open a Project in Your IDE

Now that everything is installed, let's open a project folder and get comfortable in the IDE.

#### VS Code

1. Create a new folder on your computer called `hello-project` (anywhere you like — your Desktop or Documents is fine).
2. Open VS Code.
3. Go to **File → Open Folder** and select `hello-project`.
4. VS Code opens the folder. The left sidebar shows your project files (currently empty).

**Open the integrated terminal in VS Code:**

- Press `` Ctrl+` `` (Windows/Linux) or `` Cmd+` `` (Mac) — the backtick key, top-left of most keyboards.
- Or go to **Terminal → New Terminal** from the menu bar.

A terminal panel opens at the bottom, already pointing to your project folder. You can run all commands from here — you do not need a separate terminal window.

**Select the Python interpreter in VS Code:**

1. Press `Ctrl+Shift+P` (Windows/Linux) or `Cmd+Shift+P` (Mac) to open the Command Palette.
2. Type `Python: Select Interpreter` and press Enter.
3. Choose the Python version you installed (e.g., `Python 3.11.x`).

> **Common issue:** If no Python shows up, VS Code can't find it. Make sure Python is on your PATH (check by running `python --version` in the terminal). If it works in the terminal but not in VS Code, restart VS Code.

#### PyCharm

1. Open PyCharm.
2. Click **New Project**.
3. Set the **Location** to a folder called `hello-project`.
4. Under **Python Interpreter**, select **New environment using Virtualenv** — PyCharm will create a venv automatically.
5. Click **Create**.

**Open the terminal in PyCharm:**

- Press `Alt+F12` (Windows/Linux) or `Option+F12` (Mac).
- Or go to **View → Tool Windows → Terminal**.

**Check the active interpreter in PyCharm:**

Look at the bottom-right corner of the PyCharm window. It shows the active Python interpreter (e.g., `Python 3.11 (hello-project)`). Click it to change.

---

### 1.9 Create and Activate a Virtual Environment

A **virtual environment** (venv) is an isolated box of Python packages for your project. It prevents version conflicts between projects and keeps your global Python installation clean.

> Every project should have its own venv. This is non-negotiable on real teams.

**Skip this section if you are using PyCharm** — PyCharm already created the venv for you in step 1.8.

#### Step 1 — Create the virtual environment

Open the terminal in your IDE (or any terminal) and make sure you are in your project folder:

```bash
# Make sure you are in your project folder
cd hello-project     # skip if you are already there
```

Now create the venv:

| OS | Command |
|----|---------|
| 🪟 Windows | `python -m venv venv` |
| 🍎 Mac | `python3 -m venv venv` |
| 🐧 Linux | `python3 -m venv venv` |

This creates a folder called `venv/` inside your project. You only do this **once** per project.

#### Step 2 — Activate the virtual environment

You must activate the venv every time you open a new terminal window. When active, your terminal prompt shows `(venv)` at the start.

| OS | Command |
|----|---------|
| 🪟 Windows (PowerShell) | `venv\Scripts\Activate.ps1` |
| 🪟 Windows (Command Prompt) | `venv\Scripts\activate.bat` |
| 🪟 Windows (Git Bash) | `source venv/Scripts/activate` |
| 🍎 Mac | `source venv/bin/activate` |
| 🐧 Linux | `source venv/bin/activate` |

After activation, your prompt looks like this:

```text
(venv) user@machine:~/hello-project$     ← (venv) means it's active
```

#### Step 3 — Verify the venv is active

```bash
# Should show a path inside your project's venv folder
which python          # Mac / Linux
where python          # Windows
```

Expected output (Mac/Linux):

```text
/home/yourname/hello-project/venv/bin/python
```

Expected output (Windows):

```text
C:\Users\yourname\hello-project\venv\Scripts\python.exe
```

If it shows a system path instead (like `/usr/bin/python`), the venv is **not** activated.

#### Deactivate when done

```bash
deactivate
```

#### Common venv errors

| Error | Cause | Fix |
|-------|-------|-----|
| `No such file or directory: venv/bin/activate` | venv not created yet | Run the `python -m venv venv` command first |
| `Activate.ps1 cannot be loaded because running scripts is disabled` | Windows PowerShell security policy | Run: `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` and try again |
| `python: command not found` after activation | venv was created with a Python that is no longer on PATH | Delete the venv folder and recreate it: `rm -rf venv && python3 -m venv venv` |
| Packages installed but not found | venv is not activated | Activate it first — check for `(venv)` in your prompt |
| Wrong Python version in venv | System Python is old | Specify the version: `python3.11 -m venv venv` |

---

### 1.10 Your First Python Hello World Project

Now you will write and run your first Python program. No extra packages needed — just Python and your venv.

#### Create the file

In your IDE, create a new file called `hello.py` in your project folder:

**VS Code:** Click the **New File** icon in the Explorer sidebar (or `File → New File`), name it `hello.py`.

**PyCharm:** Right-click the project folder in the Project panel → **New → Python File** → name it `hello`.

#### Write the code

```python
# hello.py
# This is your first Python program.
# The print() function outputs text to the terminal.

print("Hello, world!")
```

That's it. One line of real Python code.

#### Run from the terminal

Make sure your venv is active (`(venv)` in the prompt), then:

```bash
python hello.py         # Windows
python3 hello.py        # Mac / Linux
```

Expected output:

```text
Hello, world!
```

> **If you see `Hello, world!` — your setup is working.** 🎉

#### Run from VS Code

1. Open `hello.py` in VS Code.
2. Click the **▶ Run** button in the top-right corner of the editor, OR press `F5`.
3. A terminal panel opens at the bottom and shows the output.

> **Common issue:** VS Code runs with the wrong Python. Check the bottom-left status bar — it should show your venv Python path (e.g., `Python 3.11.x ('venv': venv)`). If it shows the global Python, click the interpreter name and select your venv.

#### Run from PyCharm

1. Open `hello.py` in PyCharm.
2. Right-click in the editor → **Run 'hello'**, OR press `Shift+F10`.
3. The output panel at the bottom shows `Hello, world!`.

> **PyCharm note:** The Run button only appears once you have a Python file open. If you see a grey triangle (not green), make sure the interpreter at the bottom-right is set to your venv.

#### Common run errors

| Error | What it means | Fix |
|-------|--------------|-----|
| `ModuleNotFoundError: No module named 'hello'` | You ran `python hello` instead of `python hello.py` | Add `.py` extension: `python hello.py` |
| `SyntaxError: invalid syntax` | Typo in your code | Check the line number in the error — look for missing quotes or brackets |
| `NameError: name 'Print' is not defined` | Python is case-sensitive — `Print` ≠ `print` | Use lowercase: `print("Hello, world!")` |
| `IndentationError` | Unexpected spaces or tabs | Python cares about indentation — remove any accidental spaces before `print` |
| `No such file or directory: hello.py` | You are not in the right folder | Run `cd hello-project` first, then run the file |

---

### 1.11 Setup Checklist ✅

Go through each item below. Do not move to Part 2 until every box is ticked.

- [ ] **VS Code** (or PyCharm) is installed and opens without errors
- [ ] VS Code Python extension is installed (or PyCharm is detecting Python automatically)
- [ ] `python --version` (or `python3 --version`) shows `3.11.x` or newer in a terminal
- [ ] `pip --version` (or `pip3 --version`) shows a version number
- [ ] `git --version` shows `2.x.x` or newer
- [ ] `git config --global user.name` and `user.email` are set
- [ ] `docker --version` shows a version number
- [ ] `docker compose version` shows a version number
- [ ] Docker Desktop is running (whale icon visible, or `docker ps` returns without error)
- [ ] A project folder (`hello-project`) is open in your IDE
- [ ] A virtual environment (`venv/`) has been created inside the project folder
- [ ] The venv is activated — `(venv)` is visible in your terminal prompt
- [ ] `hello.py` contains `print("Hello, world!")`
- [ ] Running `python hello.py` in the terminal prints `Hello, world!`
- [ ] Running `hello.py` from the IDE (Run button or F5) also prints `Hello, world!`

> **All boxes ticked?** You have a fully working development environment. Everything in the rest of this handbook builds on this foundation.

> **Stuck on something?** Check the error tables in each section above. If you're still stuck, search the exact error message — you are very unlikely to be the first person to see it.

---

## 2. How to Approach Learning

The goal is not to memorise commands. The goal is to understand the system.

Every tool exists to solve a problem. Before using a tool, ask:

- What problem does this solve?
- How does it fit into the system?
- What would break without it?

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
| Dev environment | Install VS Code (or PyCharm), Python 3.11+, Git, Docker Desktop — verify all with version commands |
| IDE setup | Open project folder, use IDE terminal, select Python interpreter from venv |
| Virtual environment | Create once per project (`python -m venv venv`), activate every terminal session |
| Hello World | `python hello.py` prints `Hello, world!` — confirms your entire setup works |
| Learning approach | Understand systems, not just commands |
| Git model | Working Directory → Staging → Commit → Remote |
| Branching | Always branch, never commit directly to main |
| HTTP | Every web interaction is a request/response cycle |
| Terminal | Learn the basic navigation commands — you'll use them every day |

---

## Navigation

[← Handbook Home](index) | [Part 2: Backend →](part-2-backend-fastapi)
