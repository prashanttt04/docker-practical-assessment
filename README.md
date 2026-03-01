# Docker Practical Assessment

**Instructions:** Please complete the following three tasks. These are designed to test your practical understanding of Docker optimization, orchestration with Docker Compose, and troubleshooting skills. Take your time, and document your steps and the commands you used to solve them.

---

## Table of Contents

- [Task 1: Dockerfile Optimization](#task-1-dockerfile-optimization)
- [Task 2: Multi-Container Application (Docker Compose)](#task-2-multi-container-application-docker-compose)
- [Task 3: Troubleshooting a "Broken" Container](#task-3-troubleshooting-a-broken-container)

---

## Task 1: Dockerfile Optimization

> 📄 Full details: [task1-dockerfile-optimization.md](./task1-dockerfile-optimization.md)

### Scenario

Below is a Dockerfile for a simple Node.js application. However, it was written very poorly. It results in a massive image size, has security vulnerabilities, and doesn't take advantage of Docker's caching mechanisms.

**The Bad Dockerfile:**

```dockerfile
FROM ubuntu:latest

RUN apt-get update
RUN apt-get install -y nodejs npm
RUN apt-get install -y git

COPY . /app
WORKDIR /app

RUN npm install

CMD ["node", "server.js"]
```

### Your Goal

- Rewrite this Dockerfile to be production-ready.
- Switch to a more appropriate, lightweight base image.
- Optimize the layer caching so that code changes don't force a re-installation of dependencies every time.
- Ensure the container runs as a non-root user for better security.
- Create a `.dockerignore` file and list what you would include in it to prevent unnecessary files (like `node_modules` or `.git`) from being copied into the image.

### Optimized Dockerfile

```dockerfile
# Use official lightweight Node.js LTS alpine image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy dependency files FIRST to leverage Docker layer caching
COPY package.json package-lock.json ./

# Install only production dependencies
RUN npm ci --only=production

# Copy the rest of the application source code
COPY . .

# Create a non-root user and group for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Switch to the non-root user
USER appuser

EXPOSE 3000

CMD ["node", "server.js"]
```

### .dockerignore File

```
node_modules/
npm-debug.log
.git/
.gitignore
.env
.env.*
Dockerfile
.dockerignore
dist/
build/
tmp/
.DS_Store
.vscode/
__tests__/
*.test.js
coverage/
```

### Key Changes Explained

| Issue | Old Dockerfile | Fixed Dockerfile |
|---|---|---|
| **Base Image** | `ubuntu:latest` (~70MB+, generic) | `node:18-alpine` (~50MB, purpose-built) |
| **Layer Caching** | `COPY . /app` before `npm install` | `package.json` copied first; deps cached unless changed |
| **Security** | Runs as `root` by default | Creates and uses a dedicated non-root `appuser` |
| **Unnecessary Tools** | Installs `git` (not needed at runtime) | No extra packages installed |

---

## Task 2: Multi-Container Application (Docker Compose)

> 📄 Full details: [task2-docker-compose.md](./task2-docker-compose.md)

### Scenario

You need to set up a local development environment for a three-tier web application using Docker Compose. You don't need to write the actual application code, just the `docker-compose.yml` file to orchestrate the infrastructure.

**The Stack:**

| Tier | Service | Image |
|---|---|---|
| Frontend | Nginx web server | `nginx:latest` |
| Backend | Node.js API | `my-backend-api:latest` |
| Database | PostgreSQL | `postgres:15-alpine` |

### Your Goal

Write the `docker-compose.yml` file that meets the following requirements:

- **Ports:** Only the Frontend (Nginx) should be accessible from the host machine on port 8080. The Backend and Database should NOT expose their ports to the host machine.
- **Networking:** Frontend must communicate with Backend, and Backend must communicate with Database using internal Docker networking.
- **Volumes:** PostgreSQL database data must be persisted using a named volume so data is not lost if the containers are destroyed.
- **Environment Variables:** PostgreSQL container requires `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`. Pass these securely via `env_file`.

### `.env` File

```env
POSTGRES_USER=admin
POSTGRES_PASSWORD=supersecretpassword
POSTGRES_DB=appdb
```

### `docker-compose.yml`

```yaml
version: "3.9"

services:

  frontend:
    image: nginx:latest
    ports:
      - "8080:80"
    networks:
      - frontend-network
    depends_on:
      - backend
    restart: unless-stopped

  backend:
    image: my-backend-api:latest
    # NO ports section — not accessible from host
    networks:
      - frontend-network
      - backend-network
    depends_on:
      - database
    environment:
      - NODE_ENV=production
    restart: unless-stopped

  database:
    image: postgres:15-alpine
    # NO ports section — not accessible from host
    networks:
      - backend-network
    env_file:
      - .env
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:

networks:
  frontend-network:
    driver: bridge
  backend-network:
    driver: bridge
```

### Architecture

```
HOST MACHINE
    │
    │  port 8080
    ▼
┌─────────────────┐   frontend-network   ┌─────────────────┐
│    FRONTEND     │ ──────────────────► │    BACKEND      │
│  nginx:latest   │                     │ my-backend-api  │
└─────────────────┘                     └────────┬────────┘
                                                 │ backend-network
                                        ┌────────▼────────┐
                                        │    DATABASE     │
                                        │ postgres:15-alp │
                                        └────────┬────────┘
                                                 │
                                        ┌────────▼────────┐
                                        │  postgres_data  │
                                        │  (named volume) │
                                        └─────────────────┘
```

---

## Task 3: Troubleshooting a "Broken" Container

> 📄 Full details: [task3-troubleshooting.md](./task3-troubleshooting.md)

### Scenario

You've been handed a Dockerfile by another developer. They complain that when they build and run it, the application crashes immediately, or they cannot access it in their browser.

**The Broken Setup:**

```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

EXPOSE 3000

# The app.py file expects an environment variable called 'API_KEY' to start.
# The app.py file is hardcoded to listen on port 8080 internally.
CMD ["python", "app.py"]
```

### Your Goal

Identify the issues and explain how you would fix them.

---

### Issue 1: Networking — Port Mismatch

**Problem:** The developer ran:

```bash
docker run -p 3000:3000 my-python-app
```

This maps host port 3000 → container port 3000. But `app.py` is **hardcoded to listen on port 8080**. Nothing is listening on container port 3000, so the connection is refused.

**Fix — Correct the `docker run` command:**

```bash
docker run -p 3000:8080 my-python-app
```

Also update the Dockerfile to document the correct port:

```dockerfile
EXPOSE 8080   # Was 3000, now matches actual app port
```

---

### Issue 2: Container Crash — Diagnosing with Docker CLI

**Command to see why the container crashed:**

```bash
docker logs <container_id_or_name>
```

If the container has already exited:

```bash
# Find the container ID
docker ps -a

# Read its logs
docker logs <container_id>
```

You would see an error such as:
```
KeyError: 'API_KEY'
```

---

### Issue 3: The Fix — Providing `API_KEY` at Runtime

**Exact `docker run` command to provide the missing `API_KEY`:**

```bash
docker run -e API_KEY="your_actual_api_key_here" -p 3000:8080 my-python-app
```

Or using an env file:

```bash
docker run --env-file .env -p 3000:8080 my-python-app
```

---

### Summary Table

| # | Issue | Root Cause | Fix |
|---|---|---|---|
| 1 | Cannot access app in browser | Port mismatch: app on `8080`, mapped to `3000:3000` | Use `docker run -p 3000:8080` |
| 2 | Container exits immediately | `API_KEY` env variable missing → app crashes | Run `docker logs <id>` to diagnose |
| 3 | `API_KEY` not set | Secret not provided at runtime | Use `-e API_KEY=your_key` or `--env-file .env` |

### Fully Fixed Dockerfile

```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

EXPOSE 8080   # Fixed: matches actual port app.py listens on

CMD ["python", "app.py"]
```

### Final Correct `docker run` Command

```bash
docker run -e API_KEY="your_actual_api_key_here" -p 3000:8080 my-python-app
```

Access the app at: **http://localhost:3000**
