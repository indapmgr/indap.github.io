# 🐳 Part 4 — Docker + Docker Compose

> **Key idea:** Docker eliminates "works on my machine." Every developer and every server runs the exact same environment.

---

## Navigation

[← Part 3: Virtualenv & Testing](part-3-frontend-nextjs) | [Part 5: PostgreSQL →](part-5-postgresql)

---

## 1. What Docker Does

Without Docker, software behaves differently on different machines because of different OS versions, tool versions, and configuration. Docker solves this by packaging everything a service needs into an **image**, which runs as a **container** anywhere Docker is installed.

### Image vs Container

| Concept | Analogy | Description |
|---------|---------|-------------|
| **Image** | Recipe | Blueprint for building a container. Read-only. Built once. |
| **Container** | Cooked meal | A running instance of an image. Can be started and stopped. |

---

## 2. FastAPI Dockerfile

A Dockerfile is a script of instructions for building a Docker image.

**`backend/Dockerfile`**

```dockerfile
# Use the official Python 3.11 slim image as the base
# "slim" means it's stripped down — smaller file size, faster to download
FROM python:3.11-slim

# Set the working directory inside the container
# All subsequent commands run from /app
WORKDIR /app

# Copy requirements.txt first — before the rest of the code
# Docker caches layers; if requirements.txt hasn't changed, pip install is skipped
COPY requirements.txt .

# Install Python dependencies
# --no-cache-dir reduces image size by not caching the pip download files
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code into the container
COPY . .

# Tell Docker this container will listen on port 8000 (documentation only)
EXPOSE 8000

# The command to run when the container starts
# --host 0.0.0.0 means "listen on all network interfaces" (required inside Docker)
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Line-by-line explanation

| Instruction | What it does |
|-------------|-------------|
| `FROM python:3.11-slim` | Start from an official Python 3.11 image |
| `WORKDIR /app` | Set the working directory inside the container |
| `COPY requirements.txt .` | Copy only requirements first (enables layer caching) |
| `RUN pip install ...` | Install Python dependencies |
| `COPY . .` | Copy all application code |
| `EXPOSE 8000` | Documents that the container uses port 8000 |
| `CMD [...]` | The command to run when the container starts |

---

## 3. Build and Run Commands

The following commands build your image and run a container from it.

Build the image (run from the folder containing your Dockerfile):

```bash
docker build -t backend-app .
```

Run the container and map port 8000 on your machine to port 8000 inside the container:

```bash
docker run -p 8000:8000 backend-app
```

Access the API at [http://localhost:8000/docs](http://localhost:8000/docs).

> **Tip:** `-t backend-app` gives the image a name (tag). Without it, you'd have to refer to it by its auto-generated ID.

---

## 4. Debugging Docker Containers

| Command | Purpose |
|---------|---------|
| `docker ps` | List running containers (id, name, ports) |
| `docker ps -a` | List all containers including stopped ones |
| `docker logs <container_id>` | View stdout/stderr output from a container |
| `docker stop <container_id>` | Gracefully stop a running container |
| `docker exec -it <container_id> bash` | Open an interactive shell inside the container |

> **Tip:** You only need the first few characters of a container ID. `docker logs abc1` works if that's enough to be unique.

---

## 5. Docker Networking

Inside Docker Compose, containers communicate using **service names** as hostnames, not `localhost`.

> **Why?** `localhost` inside a container refers to *that container itself* — not your host machine, not other containers.

| Where you are | How to reach another service |
|---------------|------------------------------|
| Your host machine | `localhost:8000` |
| Inside the backend container | `db:5432` (use the service name `db`) |

This is a very common source of confusion. Remember: use service names inside Docker Compose.

---

## 6. Docker Compose

Docker Compose lets you define and run multiple containers together using a single configuration file. Instead of running multiple `docker run` commands, you run one command.

**`docker-compose.yml`**

```yaml
version: "3.9"   # Docker Compose file format version

services:

  # ── Backend service ──────────────────────────────────────────────────────
  backend:
    build: ./backend           # Build from the backend/ folder's Dockerfile
    ports:
      - "8000:8000"            # Map host port 8000 → container port 8000
    environment:
      - DB_HOST=db             # Use the 'db' service name as the database host
      - DB_NAME=appdb
      - DB_USER=postgres
      - DB_PASSWORD=password
    depends_on:
      - db                     # Start the db container before this one

  # ── Database service ─────────────────────────────────────────────────────
  db:
    image: postgres:15         # Use the official PostgreSQL 15 image (no build needed)
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: appdb       # Create this database on first start
    volumes:
      - postgres_data:/var/lib/postgresql/data   # Persist data between container restarts

# Named volumes are managed by Docker; data survives container restarts
volumes:
  postgres_data:
```

---

## 7. Docker Compose Commands

| Command | Purpose |
|---------|---------|
| `docker compose up --build` | Build images (if changed) and start all services |
| `docker compose up -d` | Start services in the background (detached mode) |
| `docker compose down` | Stop and remove containers (data in volumes is kept) |
| `docker compose logs` | View logs from all services |
| `docker compose logs backend` | View logs from a specific service only |
| `docker compose ps` | List running Compose services |

---

## 8. Environment Variables

Never hardcode secrets or configuration. Use environment variables.

In `docker-compose.yml`, pass variables to a container:

```yaml
environment:
  - DB_HOST=db
  - DB_NAME=appdb
  - DB_USER=postgres
  - DB_PASSWORD=password
```

In Python code, read them with `os.getenv()`:

```python
import os   # Standard library module for interacting with the OS

# os.getenv("KEY", "default") returns the env var, or the default if not set
DB_HOST = os.getenv("DB_HOST", "localhost")
DB_NAME = os.getenv("DB_NAME", "appdb")
DB_USER = os.getenv("DB_USER", "postgres")
DB_PASS = os.getenv("DB_PASSWORD", "password")
```

> **Why not hardcode?** If you push a hardcoded password to GitHub, it becomes public. Environment variables keep secrets out of source code.

---

## 9. Real Docker Failures and Fixes

| Failure | Cause | Fix |
|---------|-------|-----|
| `Connection refused` from backend to db | `depends_on` starts the container but doesn't wait for PostgreSQL to be ready | Add a retry loop in your app, or use a health-check based `depends_on` |
| `Port already in use` | Another process is using port 8000 | Stop the other process or change the port mapping |
| Container keeps restarting | Application crashes on start | Run `docker compose logs <service>` to read the error |
| Changes not reflected | Old image is cached | Run `docker compose up --build` to force rebuild |
| `Module not found` | Dependency not in `requirements.txt` | Add it to `requirements.txt` and rebuild |
| `database "appdb" does not exist` | Volume from old run still exists | Run `docker compose down -v` to delete the volume, then restart |

---

## 10. Mini Project

1. Write a `Dockerfile` for your FastAPI backend.
2. Build the image: `docker build -t my-backend .`
3. Run it: `docker run -p 8000:8000 my-backend`
4. Verify it works at `http://localhost:8000/docs`
5. Write a `docker-compose.yml` with your backend and a PostgreSQL database.
6. Start both with: `docker compose up --build`

---

## Exercises

1. Add a `HEALTHCHECK` instruction to the Dockerfile.
2. Use `docker compose logs backend` to find an error you introduced intentionally.
3. Change the database password in `docker-compose.yml` and verify the backend can still connect.

---

## Part 4 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Image vs container | Image is the blueprint; container is the running instance |
| Dockerfile | Step-by-step instructions to build a Docker image |
| Docker Compose | Orchestrates multiple containers with one config file |
| Networking | Use service names (not `localhost`) between containers |
| Environment variables | Never hardcode config — use `os.getenv()` |

---

## Navigation

[← Part 3: Virtualenv & Testing](part-3-frontend-nextjs) | [Part 5: PostgreSQL →](part-5-postgresql)
