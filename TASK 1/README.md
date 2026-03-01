# Task 1: Dockerfile Optimization

## Scenario

Below is a Dockerfile for a simple Node.js application. However, it was written very poorly. It results in a massive image size, has security vulnerabilities, and doesn't take advantage of Docker's caching mechanisms.

### The Bad Dockerfile

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

---

## Goal

Rewrite this Dockerfile to be production-ready by addressing the following:

1. Switch to a more appropriate, lightweight base image.
2. Optimize layer caching so that code changes don't force a re-installation of dependencies every time.
3. Ensure the container runs as a non-root user for better security.
4. Create a `.dockerignore` file to prevent unnecessary files from being copied into the image.

---

## Optimized Dockerfile

```dockerfile
# Use official lightweight Node.js LTS alpine image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy dependency files FIRST to leverage Docker layer caching
# Dependencies are only re-installed when package.json or package-lock.json change
COPY package.json package-lock.json ./

# Install only production dependencies
RUN npm ci --only=production

# Copy the rest of the application source code
COPY . .

# Create a non-root user and group for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Switch to the non-root user
USER appuser

# Expose the port the app listens on
EXPOSE 3000

# Start the application
CMD ["node", "server.js"]
```


## .dockerignore File

```
node_modules/
npm-debug.log
yarn-debug.log
yarn-error.log
.git/
.gitignore
.env
.env.*
Dockerfile
.dockerignore
dist/
build/
tmp/
.cache/
.DS_Store
Thumbs.db
.vscode/
.idea/
*.swp
*.swo
__tests__/
*.test.js
*.spec.js
coverage/
```
