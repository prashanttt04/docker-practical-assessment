# 🐳 Docker Practical Assessment – Three Task Problems

This repository contains three task-based Docker problems focusing on:

- Dockerfile optimization
- Multi-container orchestration using Docker Compose
- Troubleshooting container failures

---

# 📌 Task 1: Dockerfile Optimization

## Scenario

Below is a poorly written Dockerfile for a simple Node.js application.

It:
- Produces a massive image size
- Contains security risks
- Does not use Docker layer caching properly

---

## The Bad Dockerfile

```Dockerfile
FROM ubuntu:latest

RUN apt-get update
RUN apt-get install -y nodejs npm
RUN apt-get install -y git

COPY . /app
WORKDIR /app

RUN npm install

CMD ["node", "server.js"]
```

---

## Your Goal

Rewrite this Dockerfile to be production-ready.

You must:

- Switch to a lightweight and appropriate base image
- Optimize Docker layer caching so dependency installation is not repeated unnecessarily
- Ensure the container runs as a non-root user
- Create a `.dockerignore` file
- List files to exclude (e.g., node_modules, .git, etc.)

---

# 📌 Task 2: Multi-Container Application (Docker Compose)

## Scenario

Set up a local development environment for a three-tier web application using Docker Compose.

You do NOT need to write application code.  
Only create the `docker-compose.yml` file.

---

## The Stack

- Frontend: Nginx web server
- Backend: Node.js API (`my-backend-api:latest`)
- Database: PostgreSQL (`postgres:15-alpine`)

---

## Your Goal

Write a `docker-compose.yml` file that satisfies the following:

### Ports
- Only the Frontend should be accessible from the host machine on port 8080
- Backend and Database must NOT expose their ports

### Networking
- Frontend must communicate with Backend
- Backend must communicate with Database
- Use internal Docker networking

### Volumes
- PostgreSQL must use a named volume
- Data must persist even if containers are destroyed

### Environment Variables
PostgreSQL requires:
- POSTGRES_USER
- POSTGRES_PASSWORD
- POSTGRES_DB

Pass them securely using:
- env_file  
OR  
- environment block

---

# 📌 Task 3: Troubleshooting a "Broken" Container

## Scenario

A developer created the following Dockerfile, but:

- The application crashes immediately
- OR it cannot be accessed in the browser

---

## The Broken Setup

```Dockerfile
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

---

## Your Goal

Identify the issues and explain how to fix them.

---

### 1️⃣ Networking Issue

The developer ran:

```
docker run -p 3000:3000 my-python-app
```

- Why can’t they access the application in the browser?
- How should the Dockerfile or docker run command be corrected?

---

### 2️⃣ Crash Issue

The container exits immediately after starting.

- What Docker CLI command would you use to inspect why it crashed?

---

### 3️⃣ The Fix

The application requires an environment variable:

```
API_KEY
```

- How would you provide this variable to the container?
- Show the exact `docker run` command required for it to start successfully.

---

# 🎯 Objective of This Assessment

These tasks evaluate understanding of:

- Docker image optimization
- Layer caching
- Security best practices
- Container networking
- Port mapping
- Named volumes
- Environment variable management
- Debugging and troubleshooting containers

---

End of Assessment
