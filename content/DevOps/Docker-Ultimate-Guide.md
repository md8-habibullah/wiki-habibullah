---
title: The Ultimate Docker Handbook
date:
description: A deep dive into containerization, image layering, multi-stage builds, and security hardening on Linux.
tags:
  - devops
  - docker
  - containers
  - security
  - fedora
  - LLM
---

> [!abstract] Overview
> **Docker** is not just a tool; it is the industry standard for packaging software. This guide moves beyond `docker run` and explores the architecture, networking, storage patterns, and security practices required for production-grade engineering.

## 1. The Architecture (Under the Hood)
At its core, Docker is a wrapper around Linux Kernel primitives. It does not "virtualize" hardware like a VM; it "isolates" processes.

### Kernel Namespaces & Cgroups
When you start a container, Docker leverages two key Linux features:
1.  **Namespaces:** Provide isolation. The process thinks it has its own PID tree, Network stack, and Mount points.
2.  **Control Groups (cgroups):** Provide resource limiting. Prevents a container from eating 100% of your CPU or RAM.



---

## 2. Installation on Fedora (`JUICE`)

Since we are running **Fedora**, we use the official repository for the latest version.

```bash
# 1. Remove old versions (if any)
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

# 2. Add the repo
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo [https://download.docker.com/linux/fedora/docker-ce.repo](https://download.docker.com/linux/fedora/docker-ce.repo)

# 3. Install
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 4. Start & Enable
sudo systemctl start docker
sudo systemctl enable docker

# 5. Post-Install (Run without sudo)
sudo usermod -aG docker $USER
newgrp docker
````

> [!danger] Security Note
> 
> Adding your user to the docker group is equivalent to giving yourself root privileges. Ensure you trust the code you run. For higher security, look into Rootless Docker.

---

## 3. The Image Hierarchy (Layers)

Docker images are read-only templates built from a series of layers.

- **Union File System (UnionFS):** Docker stacks layers on top of each other.
    
- **Copy-on-Write (CoW):** When a container modifies a file, it creates a copy of that file in the top "writable" layer. The underlying image remains untouched.
    

### The `Dockerfile` Anatomy

A well-optimized Dockerfile is an art form.

Dockerfile

```
# 1. Base Image (Use Alpine or Slim for size)
FROM node:20-alpine

# 2. Set Working Directory
WORKDIR /app

# 3. Install Dependencies (Layer Caching Strategy)
# Copy package.json BEFORE source code.
# Docker will cache this layer if package.json hasn't changed.
COPY package*.json ./
RUN npm install

# 4. Copy Source Code
COPY . .

# 5. Documentation (Expose Port)
EXPOSE 3000

# 6. Runtime Command
CMD ["npm", "start"]
```

---

## 4. Advanced: Multi-Stage Builds

This is how you shrink a **1GB** image down to **50MB**. You build in one environment and ship in another.

**Scenario:** A Next.js Application.

Dockerfile

```
# --- Stage 1: The Builder ---
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# --- Stage 2: The Runner (Production) ---
FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV production

# Don't run as root!
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy ONLY the build artifacts from Stage 1
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
CMD ["node", "server.js"]
```

> [!tip] Why this matters
> 
> By discarding the node_modules used for building and keeping only the production files, we reduce attack surface and deployment time.

---

## 5. Storage Patterns (Persistence)

Containers are ephemeral (temporary). If you delete a container, the data inside is gone.

|**Type**|**Syntax**|**Use Case**|
|---|---|---|
|**Volume**|`-v my_vol:/data`|**Preferred.** Managed by Docker. stored in `/var/lib/docker/volumes`. Best for DBs.|
|**Bind Mount**|`-v $(pwd):/app`|**Dev Mode.** Maps a folder on your host to the container. Great for live reloading code.|
|**Tmpfs**|`--tmpfs /app`|**Security.** Stored in RAM only. Good for secrets/keys.|

---

## 6. Networking

Docker creates a `bridge` network by default.

- **Bridge:** Default. Containers talk via IP addresses.
    
- **Host:** Removes isolation. Container shares `localhost` with the Fedora host. (Fastest, but dangerous).
    
- **Overlay:** Used in Swarm/Kubernetes for multi-host communication.
    

DNS Magic:

In a custom network (docker network create my-net), containers can resolve each other by name.

ping postgres-db works inside the backend container automatically.

---

## 7. Orchestration: Docker Compose

Managing 5 containers with CLI flags is madness. We use `compose.yaml`.

YAML

```
version: '3.8'

services:
  # The Backend
  api:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=db
      - DB_PASSWORD=secret
    depends_on:
      - db
    networks:
      - app-net

  # The Database
  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - app-net

volumes:
  db_data:

networks:
  app-net:
```

---

## 8. Security Hardening (The Hacker Mindset) 🛡️

1. **Never Run as Root:** Always define a `USER` in your Dockerfile (like the Multi-stage example above).
    
2. **Read-Only Filesystem:** Use `--read-only` to prevent attackers from writing malicious scripts.
    
3. **Capabilities:** Drop Linux capabilities you don't need.
    
    Bash
    
    ```
    docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE my-app
    ```
    
4. **Image Scanning:** Use tools like **Trivy** or **Grype** to scan images for CVEs before deployment.
    
    Bash
    
    ```
    trivy image my-app:latest
    ```
    

---

## 9. The Survival Cheatsheet

**Housekeeping (Clean up disk space):**

Bash

```
docker system prune -a   # Delete all stopped containers, unused networks, and dangling images
docker volume prune      # WARNING: Deletes data volumes
```

**Debugging:**

Bash

```
docker logs -f <container_id>     # Follow logs
docker exec -it <container_id> sh # Shell into container
docker stats                      # Live CPU/RAM usage
```

**One-Liners:**

Bash

```
# Stop all running containers
docker stop $(docker ps -q)

# Kill everything (Nuclear option)
docker rm -f $(docker ps -aq)
```

---

## Linked Notes

- [[Linux-Kernel]] - Understanding Namespaces.
    
- [[Kubernetes-Basics]] - The next step after Docker.
    
- [[CI-CD-Pipelines]] - Automating image builds.