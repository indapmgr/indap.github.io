---
title: "Part 4 — Docker + Docker Compose"
layout: page
---

# Part 4 — Docker + Docker Compose

> **Key idea:** Docker eliminates "works on my machine." Every developer and every server runs the exact same environment.

---

## Navigation

[← Part 3: Frontend](part-3-frontend-nextjs.md) | [Part 5: PostgreSQL →](part-5-postgresql.md)

---

## 1. What Docker Does

Without Docker, software behaves differently on different machines because of different OS versions, tool versions, and configuration. Docker solves this by packaging everything a service needs into an **image**, which runs as a **container** anywhere Docker is installed.

### Image vs Container

| Concept | Analogy | Description |
|---------|---------|-------------|
| **Image** | Recipe | Blueprint for building a container. Read-only. |
| **Container** | Cooked meal | A running instance of an image. |

---

## 2. FastAPI Dockerfile

**`backend/Dockerfile`**

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Line-by-line explanation

| Line | What it does |
|------|-------------|
| `FROM python:3.11-slim` | Start from an official Python 3.11 image |
| `WORKDIR /app` | Set the working directory inside the container |
| `COPY requirements.txt .` | Copy only the requirements file first (better caching) |
| `RUN pip install ...` | Install Python dependencies |
| `COPY . .` | Copy all application code |
| `EXPOSE 8000` | Document that the container uses port 8000 |
| `CMD [...]` | The command to run when the container starts |

---

## 3. Build and Run Commands

Build the image:

```bash
docker build -t backend-app .
```

Run the container and map port 8000:

```bash
docker run -p 8000:8000 backend-app
```

Access the API at [http://localhost:8000](http://localhost:8000).

---

## 4. Debugging Docker Containers

| Command | Purpose |
|---------|---------|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers including stopped ones |
| `docker logs <container_id>` | View stdout/stderr output from a container |
| `docker stop <container_id>` | Stop a running container |
| `docker exec -it <container_id> bash` | Open a shell inside a running container |

---

## 5. Docker Networking

Inside Docker Compose, containers communicate using **service names**, not `localhost`.

> `localhost` inside a container refers to that container itself — not to your host machine or other containers.

| Where you are | How to reach another service |
|---------------|------------------------------|
| Your machine | `localhost:8000` |
| Inside the backend container | `db:5432` (use the service name) |
| Inside the frontend container | `backend:8000` (use the service name) |

---

## 6. Docker Compose

Docker Compose lets you define and run multiple containers together with one file.

**`docker-compose.yml`**

```yaml
version: "3.9"

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/appdb
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:8000
    depends_on:
      - backend

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: appdb
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

## 7. Docker Compose Commands

| Command | Purpose |
|---------|---------|
| `docker compose up --build` | Build images and start all services |
| `docker compose up -d` | Start services in the background (detached) |
| `docker compose down` | Stop and remove containers |
| `docker compose logs` | View logs from all services |
| `docker compose logs backend` | View logs from a specific service |
| `docker compose ps` | List running Compose services |

---

## 8. Environment Variables

Never hardcode secrets or configuration. Use environment variables.

In `docker-compose.yml`:

```yaml
environment:
  - DATABASE_URL=postgresql://postgres:password@db:5432/appdb
```

In Python code:

```python
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://postgres:password@localhost:5432/appdb")
```

---

## 9. Real Docker Failures and Fixes

| Failure | Cause | Fix |
|---------|-------|-----|
| `Connection refused` from backend to db | `depends_on` starts container but doesn't wait for DB to be ready | Add a health check or retry logic |
| `Port already in use` | Another process is on port 8000/3000 | Stop the other process or change the port |
| Container keeps restarting | Application crashes on start | Run `docker compose logs <service>` to see the error |
| Changes not reflected | Old image cached | Run `docker compose up --build` to force rebuild |
| `Module not found` | Dependency not in `requirements.txt` | Add it and rebuild |

---

## 10. Mini Project

1. Write a `Dockerfile` for your FastAPI backend.
2. Build the image: `docker build -t my-backend .`
3. Run it: `docker run -p 8000:8000 my-backend`
4. Verify it works at `http://localhost:8000/docs`

---

## Exercises

1. Add a `HEALTHCHECK` instruction to the Dockerfile.
2. Write a `docker-compose.yml` with backend and a postgres database.
3. Use `docker compose logs backend` to find an error you introduced intentionally.

---

## Part 4 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Image vs container | Image is the blueprint; container is the running instance |
| Dockerfile | Step-by-step instructions to build an image |
| Docker Compose | Orchestrates multiple containers together |
| Networking | Use service names (not `localhost`) between containers |
| Environment variables | Never hardcode config — use `os.getenv()` |

---

## Navigation

[← Part 3: Frontend](part-3-frontend-nextjs.md) | [Part 5: PostgreSQL →](part-5-postgresql.md)
