# Docker Interview Questions and Answers

A comprehensive guide covering Docker concepts, commands, and best practices for interview preparation.

---

## Table of Contents

1. [Docker Basics](#docker-basics)
2. [Docker Commands](#docker-commands)
3. [Docker Networking](#docker-networking)
4. [Docker Volumes](#docker-volumes)
5. [Docker Compose](#docker-compose)
6. [Dockerfile Best Practices](#dockerfile-best-practices)
7. [Multi-Stage Builds](#multi-stage-builds)
8. [Docker Security](#docker-security)
9. [Troubleshooting](#troubleshooting)
10. [Real-World Scenarios](#real-world-scenarios)

---

## Docker Basics

### Q1: What is Docker and how does it differ from virtual machines?

**Answer:**
Docker is a containerization platform that packages applications and their dependencies into containers.

**Key Differences:**

| Feature | Docker Containers | Virtual Machines |
|---------|------------------|------------------|
| OS | Shares host OS kernel | Requires full OS for each VM |
| Size | Lightweight (MBs) | Heavy (GBs) |
| Startup Time | Seconds | Minutes |
| Performance | Near-native | Slower due to hypervisor overhead |
| Isolation | Process-level | Complete isolation |
| Resource Usage | Efficient | More resource-intensive |

**Example:**
```bash
# Docker container starts in seconds
docker run -d nginx

# A VM would require full OS boot process
```

---

### Q2: What is a Docker container?

**Answer:**
A Docker container is a lightweight, standalone, executable package that includes everything needed to run a piece of software: code, runtime, system tools, libraries, and settings.

**Characteristics:**
- Isolated from other containers and the host system
- Portable across different environments
- Created from Docker images
- Ephemeral by default (stateless)

**Example:**
```bash
# Create and run a container
docker run -d --name my-nginx -p 8080:80 nginx

# Container runs in isolation with its own filesystem
docker exec my-nginx ls /etc/nginx
```

---

### Q3: What is a Docker image?

**Answer:**
A Docker image is a read-only template containing instructions for creating a Docker container. It includes the application code, runtime, libraries, environment variables, and configuration files.

**Key Points:**
- Images are built in layers
- Each layer represents an instruction in the Dockerfile
- Layers are cached for efficiency
- Images are stored in registries (Docker Hub, private registries)

**Example:**
```bash
# Pull an image from Docker Hub
docker pull ubuntu:22.04

# List local images
docker images

# Inspect image layers
docker history ubuntu:22.04
```

---

### Q4: What is a Dockerfile?

**Answer:**
A Dockerfile is a text file containing a series of instructions to build a Docker image. It defines the base image, application code, dependencies, and commands to run.

**Basic Structure:**
```dockerfile
# Base image
FROM node:18-alpine

# Working directory
WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Start command
CMD ["node", "app.js"]
```

**Common Instructions:**
- `FROM`: Specifies base image
- `WORKDIR`: Sets working directory
- `COPY`: Copies files from host to image
- `RUN`: Executes commands during build
- `CMD`: Default command when container starts
- `EXPOSE`: Documents which ports to expose
- `ENV`: Sets environment variables

---

### Q5: What is the difference between CMD and ENTRYPOINT?

**Answer:**

**CMD:**
- Provides default arguments that can be overridden
- Can be replaced entirely by command-line arguments
- Multiple CMD instructions: only the last one takes effect

**ENTRYPOINT:**
- Defines the main command that always runs
- Command-line arguments are appended to ENTRYPOINT
- Makes container behave like an executable

**Examples:**

```dockerfile
# CMD example
FROM ubuntu
CMD ["echo", "Hello World"]

# Run: docker run myimage
# Output: Hello World
# Run: docker run myimage echo "Goodbye"
# Output: Goodbye (CMD is overridden)
```

```dockerfile
# ENTRYPOINT example
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello World"]

# Run: docker run myimage
# Output: Hello World
# Run: docker run myimage "Goodbye"
# Output: Goodbye (CMD is overridden, ENTRYPOINT remains)
```

**Best Practice:**
```dockerfile
# Use both together
FROM alpine
ENTRYPOINT ["python", "app.py"]
CMD ["--help"]

# docker run myimage -> python app.py --help
# docker run myimage --production -> python app.py --production
```

---

### Q6: What are Docker layers?

**Answer:**
Docker images are built using a layered architecture. Each instruction in a Dockerfile creates a new layer. Layers are cached and reused to optimize build time and storage.

**How Layers Work:**
- Each layer is read-only
- Layers are stacked on top of each other
- Container adds a writable layer on top
- Unchanged layers are reused from cache

**Example:**
```dockerfile
FROM ubuntu:22.04           # Layer 1
RUN apt-get update          # Layer 2
RUN apt-get install -y curl # Layer 3
COPY app.js /app/           # Layer 4
CMD ["node", "/app/app.js"] # Layer 5
```

**View Layers:**
```bash
docker history my-image
```

**Benefits:**
- Faster builds (cached layers)
- Efficient storage (shared layers)
- Faster image transfer

---

### Q7: What is Docker Hub?

**Answer:**
Docker Hub is a cloud-based registry service for sharing and storing Docker images. It's the default registry for Docker.

**Features:**
- Public and private repositories
- Official images from vendors
- Automated builds
- Webhooks
- Team collaboration

**Usage:**
```bash
# Search for images
docker search nginx

# Pull an image
docker pull nginx:latest

# Tag an image
docker tag my-app:latest username/my-app:v1.0

# Push to Docker Hub
docker login
docker push username/my-app:v1.0
```

---

### Q8: What is the difference between COPY and ADD in Dockerfile?

**Answer:**

**COPY:**
- Simple, straightforward copying of files/directories
- Recommended for most use cases
- Only copies local files

**ADD:**
- Has additional features
- Can extract tar archives automatically
- Can download files from URLs
- Less transparent, can lead to unexpected behavior

**Examples:**
```dockerfile
# COPY - Preferred for simple file copying
COPY package.json /app/
COPY src/ /app/src/

# ADD - Use for specific cases
ADD https://example.com/file.tar.gz /tmp/  # Download from URL
ADD archive.tar.gz /app/                     # Auto-extract
```

**Best Practice:**
```dockerfile
# Use COPY unless you specifically need ADD's features
COPY . /app/
```

---

## Docker Commands

### Q9: Explain the most important Docker commands.

**Answer:**

**Container Management:**
```bash
# Run a container
docker run [OPTIONS] IMAGE [COMMAND]
docker run -d --name web -p 8080:80 nginx

# List containers
docker ps              # Running containers
docker ps -a           # All containers
docker ps -a -q        # Only container IDs

# Start/Stop/Restart
docker start <container>
docker stop <container>
docker restart <container>

# Remove containers
docker rm <container>
docker rm -f <container>     # Force remove running container
docker container prune       # Remove all stopped containers
```

**Image Management:**
```bash
# Build an image
docker build -t myapp:v1 .
docker build -t myapp:v1 -f Dockerfile.prod .

# List images
docker images
docker images -a           # Include intermediate images

# Pull/Push images
docker pull ubuntu:22.04
docker push username/myapp:v1

# Remove images
docker rmi <image>
docker image prune -a      # Remove unused images

# Tag images
docker tag myapp:v1 myapp:latest
```

**Inspection and Logs:**
```bash
# View logs
docker logs <container>
docker logs -f <container>       # Follow logs
docker logs --tail 100 <container>

# Inspect container/image
docker inspect <container>
docker inspect <image>

# View processes
docker top <container>

# View stats
docker stats
docker stats <container>
```

**Execution and Interaction:**
```bash
# Execute command in running container
docker exec <container> <command>
docker exec -it web bash        # Interactive shell

# Attach to running container
docker attach <container>

# Copy files
docker cp <container>:/path/file ./local/path
docker cp ./local/file <container>:/path/
```

**System Management:**
```bash
# View Docker info
docker info
docker version

# Clean up resources
docker system prune              # Remove unused data
docker system prune -a           # Remove all unused images
docker system df                 # Show disk usage

# Network management
docker network ls
docker network create mynetwork
docker network inspect mynetwork
docker network rm mynetwork

# Volume management
docker volume ls
docker volume create myvolume
docker volume inspect myvolume
docker volume rm myvolume
```

---

### Q10: What is the difference between `docker run` and `docker exec`?

**Answer:**

**docker run:**
- Creates and starts a new container from an image
- Used to launch containers
- Container starts with specified command

**docker exec:**
- Executes a command in an already running container
- Does not create a new container
- Used for debugging and administration

**Examples:**
```bash
# docker run - Start new container
docker run -d --name web nginx
docker run -it ubuntu bash

# docker exec - Run command in existing container
docker exec web ls /etc/nginx
docker exec -it web bash

# Practical use case
docker run -d --name db postgres          # Start database
docker exec -it db psql -U postgres       # Access database
```

---

### Q11: How do you view logs of a Docker container?

**Answer:**

```bash
# View all logs
docker logs <container-name>

# Follow logs in real-time
docker logs -f <container-name>

# View last N lines
docker logs --tail 50 <container-name>

# View logs with timestamps
docker logs -t <container-name>

# View logs since specific time
docker logs --since 2024-01-01 <container-name>
docker logs --since 30m <container-name>    # Last 30 minutes

# Combine options
docker logs -f --tail 100 --since 10m <container-name>
```

**Example:**
```bash
# Start nginx container
docker run -d --name web nginx

# View logs
docker logs web

# Follow logs and see new entries
docker logs -f web

# In another terminal, generate logs
curl http://localhost
```

---

### Q12: How do you remove all stopped containers?

**Answer:**

**Method 1: Using prune (Recommended)**
```bash
docker container prune

# With force flag (no confirmation)
docker container prune -f
```

**Method 2: Using rm with filters**
```bash
docker rm $(docker ps -a -q -f status=exited)

# For PowerShell
docker ps -a -q -f status=exited | ForEach-Object { docker rm $_ }
```

**Method 3: Remove all containers (including running)**
```bash
# Stop all containers first
docker stop $(docker ps -a -q)

# Remove all containers
docker rm $(docker ps -a -q)

# Force remove (including running)
docker rm -f $(docker ps -a -q)
```

---

## Docker Networking

### Q13: What are the different types of Docker networks?

**Answer:**

Docker provides several network drivers:

**1. Bridge (Default):**
- Default network for containers
- Containers can communicate with each other
- Port mapping required for external access

```bash
# Create bridge network
docker network create my-bridge

# Run containers on bridge network
docker run -d --name web --network my-bridge nginx
docker run -d --name app --network my-bridge node:18
```

**2. Host:**
- Container shares host's network stack
- No network isolation
- Best performance (no NAT)

```bash
docker run -d --network host nginx
# nginx accessible directly on host's port 80
```

**3. None:**
- No networking
- Complete network isolation

```bash
docker run -d --network none nginx
# Container has no network access
```

**4. Overlay:**
- Multi-host networking
- Used in Docker Swarm
- Containers across different hosts can communicate

```bash
docker network create -d overlay my-overlay
```

**5. Macvlan:**
- Assigns MAC address to container
- Container appears as physical device
- Direct connection to physical network

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 my-macvlan
```

---

### Q14: How do containers communicate with each other?

**Answer:**

**Method 1: Using Container Names (Same Network)**
```bash
# Create custom network
docker network create app-network

# Run database
docker run -d --name db --network app-network postgres

# Run application (can connect to 'db' hostname)
docker run -d --name app --network app-network \
  -e DATABASE_HOST=db my-app
```

**Method 2: Using Links (Legacy)**
```bash
docker run -d --name db postgres
docker run -d --name app --link db:database my-app
# app can access db via 'database' hostname
```

**Method 3: Using Container IP (Not Recommended)**
```bash
# Get container IP
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db

# Use IP in application (brittle, IPs change)
```

**Best Practice Example:**
```bash
# Create network
docker network create my-app-network

# Start services
docker run -d --name redis --network my-app-network redis
docker run -d --name db --network my-app-network postgres
docker run -d --name api --network my-app-network \
  -e REDIS_HOST=redis \
  -e DB_HOST=db \
  my-api

# Containers communicate using service names
```

---

### Q15: How do you expose a container port to the host?

**Answer:**

**Using -p flag:**
```bash
# Format: -p HOST_PORT:CONTAINER_PORT

# Map port 8080 on host to port 80 in container
docker run -d -p 8080:80 nginx

# Map specific IP
docker run -d -p 127.0.0.1:8080:80 nginx

# Map random host port
docker run -d -p 80 nginx

# Multiple port mappings
docker run -d -p 8080:80 -p 8443:443 nginx
```

**Using -P flag:**
```bash
# Map all EXPOSE'd ports to random host ports
docker run -d -P nginx

# Check mapped ports
docker port <container>
```

**Example:**
```dockerfile
# Dockerfile
FROM nginx
EXPOSE 80 443
```

```bash
# Build and run
docker build -t my-nginx .
docker run -d -P my-nginx

# Check which ports were assigned
docker ps
# Shows: 0.0.0.0:32768->80/tcp, 0.0.0.0:32769->443/tcp
```

**Inspect Port Mappings:**
```bash
# View port mappings
docker port <container>

# Inspect network settings
docker inspect -f '{{.NetworkSettings.Ports}}' <container>
```

---

### Q16: What is the difference between bridge and host networking?

**Answer:**

**Bridge Network:**
- **Isolation**: Separate network namespace
- **Port Mapping**: Required for external access
- **Performance**: Slight overhead due to NAT
- **Security**: Better isolation
- **Use Case**: Default, most applications

```bash
# Bridge example
docker run -d -p 8080:80 --name web nginx
# Access via: http://localhost:8080
```

**Host Network:**
- **Isolation**: No network isolation
- **Port Mapping**: Not needed/not possible
- **Performance**: Better (no NAT overhead)
- **Security**: Less isolated
- **Use Case**: High-performance apps, network monitoring

```bash
# Host example
docker run -d --network host --name web nginx
# Access via: http://localhost:80 (uses host's port directly)
```

**Comparison:**

| Feature | Bridge | Host |
|---------|--------|------|
| Network Namespace | Separate | Shared |
| Port Conflicts | Isolated | Can conflict |
| Performance | Good | Best |
| Port Mapping | Required | N/A |
| Isolation | High | None |

**Example Scenario:**
```bash
# Bridge - Can run multiple web servers
docker run -d -p 8081:80 nginx
docker run -d -p 8082:80 nginx
# Both work fine

# Host - Port conflicts
docker run -d --network host nginx
docker run -d --network host nginx
# Second fails - port 80 already in use
```

---

### Q17: How do you create a custom Docker network?

**Answer:**

**Create Network:**
```bash
# Basic bridge network
docker network create my-network

# With specific driver
docker network create --driver bridge my-bridge-network

# With subnet and gateway
docker network create \
  --driver bridge \
  --subnet 172.18.0.0/16 \
  --gateway 172.18.0.1 \
  my-custom-network

# With IP range
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --ip-range 172.20.240.0/20 \
  my-network
```

**Use Network:**
```bash
# Connect container at creation
docker run -d --name web --network my-network nginx

# Connect running container
docker network connect my-network <container>

# Disconnect from network
docker network disconnect my-network <container>
```

**Inspect Network:**
```bash
# View network details
docker network inspect my-network

# List all networks
docker network ls

# Remove network
docker network rm my-network

# Remove unused networks
docker network prune
```

**Practical Example:**
```bash
# Create network for microservices
docker network create microservices

# Deploy services
docker run -d --name frontend --network microservices -p 80:80 frontend-app
docker run -d --name backend --network microservices backend-app
docker run -d --name database --network microservices postgres

# Frontend can access backend via hostname 'backend'
# Backend can access database via hostname 'database'
```

---

## Docker Volumes

### Q18: What are Docker volumes and why are they important?

**Answer:**

Docker volumes are the preferred mechanism for persisting data generated by and used by Docker containers.

**Why Volumes are Important:**
- **Data Persistence**: Container data is lost when container is removed
- **Data Sharing**: Multiple containers can share the same volume
- **Performance**: Better I/O performance than bind mounts
- **Backup/Migration**: Easier to backup and migrate
- **Decoupling**: Separate data from container lifecycle

**Types of Mounts:**

1. **Volumes** (Recommended)
2. **Bind Mounts**
3. **tmpfs Mounts**

**Example:**
```bash
# Without volume - data is lost
docker run -d --name db1 postgres
docker exec db1 psql -U postgres -c "CREATE TABLE users(id INT);"
docker rm -f db1
docker run -d --name db2 postgres
# Table 'users' is gone!

# With volume - data persists
docker run -d --name db1 -v pgdata:/var/lib/postgresql/data postgres
docker exec db1 psql -U postgres -c "CREATE TABLE users(id INT);"
docker rm -f db1
docker run -d --name db2 -v pgdata:/var/lib/postgresql/data postgres
# Table 'users' still exists!
```

---

### Q19: What is the difference between volumes and bind mounts?

**Answer:**

**Volumes:**
- **Managed by Docker**: Stored in Docker's storage directory
- **Location**: `/var/lib/docker/volumes/` on Linux
- **Creation**: Created and managed by Docker
- **Best For**: Production, data that doesn't need direct access
- **Performance**: Better performance on Docker Desktop

**Bind Mounts:**
- **Managed by User**: Any location on host filesystem
- **Location**: Specified by absolute path
- **Creation**: Path must exist on host
- **Best For**: Development, configuration files, sharing code
- **Performance**: Direct filesystem access

**Examples:**

```bash
# Volume (Recommended for production)
docker run -d -v myvolume:/data nginx
# Docker manages the volume location

# Bind Mount (Good for development)
docker run -d -v /host/path:/container/path nginx
docker run -d -v "$(pwd)":/app node:18
# Direct access to host filesystem

# Windows example
docker run -d -v C:\Users\data:/data nginx

# tmpfs (Memory only, not persisted)
docker run -d --tmpfs /app/cache nginx
```

**Comparison:**

| Feature | Volume | Bind Mount |
|---------|--------|------------|
| Docker Managed | Yes | No |
| Portable | Yes | No (host-dependent) |
| Backup | Easy | Manual |
| Pre-populate | Yes | No |
| Direct Access | Limited | Full |
| Use Case | Databases, production | Development, config |

**Practical Example:**
```bash
# Development - Use bind mount for live code reload
docker run -d -v "$(pwd)/src":/app/src -p 3000:3000 my-app

# Production - Use volume for database
docker run -d -v db-data:/var/lib/mysql mysql:8

# Configuration - Use bind mount
docker run -d -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro nginx
```

---

### Q20: How do you create and manage Docker volumes?

**Answer:**

**Create Volume:**
```bash
# Create named volume
docker volume create my-volume

# Create with driver options
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.1,rw \
  --opt device=:/path/to/dir \
  my-nfs-volume
```

**List Volumes:**
```bash
# List all volumes
docker volume ls

# Filter volumes
docker volume ls -f dangling=true
```

**Inspect Volume:**
```bash
# View volume details
docker volume inspect my-volume

# Find mountpoint
docker volume inspect -f '{{.Mountpoint}}' my-volume
```

**Use Volume:**
```bash
# Named volume
docker run -d -v my-volume:/data nginx

# Anonymous volume (Docker generates name)
docker run -d -v /data nginx

# Read-only volume
docker run -d -v my-volume:/data:ro nginx
```

**Remove Volume:**
```bash
# Remove specific volume
docker volume rm my-volume

# Remove all unused volumes
docker volume prune

# Force remove
docker volume rm -f my-volume
```

**Backup Volume:**
```bash
# Backup volume to tar file
docker run --rm -v my-volume:/data -v $(pwd):/backup \
  ubuntu tar czf /backup/backup.tar.gz -C /data .

# Restore volume from backup
docker run --rm -v my-volume:/data -v $(pwd):/backup \
  ubuntu tar xzf /backup/backup.tar.gz -C /data
```

**Share Volume Between Containers:**
```bash
# Create volume
docker volume create shared-data

# Container 1 writes data
docker run -d --name writer -v shared-data:/data \
  ubuntu bash -c "echo 'Hello' > /data/message.txt"

# Container 2 reads data
docker run --rm -v shared-data:/data \
  ubuntu cat /data/message.txt
# Output: Hello
```

---

### Q21: How do you persist database data in Docker?

**Answer:**

**PostgreSQL Example:**
```bash
# Create volume for PostgreSQL data
docker volume create postgres-data

# Run PostgreSQL with volume
docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=secret \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15

# Data persists even after container removal
docker rm -f postgres-db
docker run -d \
  --name postgres-db-new \
  -e POSTGRES_PASSWORD=secret \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15
# All data still available
```

**MySQL Example:**
```bash
# Create volume
docker volume create mysql-data

# Run MySQL
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=myapp \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8
```

**MongoDB Example:**
```bash
# Create volume
docker volume create mongo-data

# Run MongoDB
docker run -d \
  --name mongodb \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  -v mongo-data:/data/db \
  -p 27017:27017 \
  mongo:7
```

**Redis Example:**
```bash
# Create volume
docker volume create redis-data

# Run Redis with persistence
docker run -d \
  --name redis \
  -v redis-data:/data \
  -p 6379:6379 \
  redis:7 redis-server --appendonly yes
```

**Backup Strategy:**
```bash
# Backup database
docker exec postgres-db pg_dump -U postgres mydb > backup.sql

# Or backup entire volume
docker run --rm \
  -v postgres-data:/data \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/db-backup-$(date +%Y%m%d).tar.gz -C /data .
```

---

## Docker Compose

### Q22: What is Docker Compose and when should you use it?

**Answer:**

Docker Compose is a tool for defining and running multi-container Docker applications using a YAML file.

**When to Use:**
- Applications with multiple services (web, database, cache)
- Development environments
- Testing environments
- Single-host deployments
- When you need to define service dependencies

**Benefits:**
- Single file configuration
- Easy to version control
- Simple commands to manage all services
- Automatic network creation
- Service dependencies and ordering

**Basic Example:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    depends_on:
      - api

  api:
    build: ./api
    environment:
      - DATABASE_URL=postgres://db:5432/myapp
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

**Commands:**
```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs -f

# Scale services
docker-compose up -d --scale api=3
```

---

### Q23: Explain the structure of a docker-compose.yml file.

**Answer:**

**Basic Structure:**
```yaml
version: '3.8'  # Compose file version

services:       # Define application services
  service1:
    # Service configuration
  service2:
    # Service configuration

networks:       # Custom networks (optional)
  network1:
    # Network configuration

volumes:        # Named volumes (optional)
  volume1:
    # Volume configuration

configs:        # Configuration files (optional)
  config1:
    # Config definition

secrets:        # Sensitive data (optional)
  secret1:
    # Secret definition
```

**Complete Example:**
```yaml
version: '3.8'

services:
  # Web Application
  web:
    build:
      context: ./web
      dockerfile: Dockerfile
      args:
        - NODE_ENV=production
    image: myapp/web:latest
    container_name: web-app
    ports:
      - "80:3000"
      - "443:3443"
    environment:
      - API_URL=http://api:8000
      - REDIS_URL=redis://cache:6379
    env_file:
      - .env
    depends_on:
      - api
      - cache
    networks:
      - frontend
      - backend
    volumes:
      - ./web/src:/app/src
      - web-logs:/var/log
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M

  # API Service
  api:
    build: ./api
    image: myapp/api:latest
    environment:
      - DATABASE_URL=postgres://postgres:secret@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend
    volumes:
      - ./api:/app
      - api-logs:/var/log
    restart: unless-stopped

  # Database
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Cache
  cache:
    image: redis:7-alpine
    networks:
      - backend
    volumes:
      - cache-data:/data
    restart: unless-stopped

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access

volumes:
  db-data:
    driver: local
  cache-data:
    driver: local
  web-logs:
  api-logs:
```

**Key Options Explained:**

```yaml
# Build Configuration
build:
  context: ./app       # Build context directory
  dockerfile: Dockerfile.prod
  args:               # Build arguments
    - VERSION=1.0
  target: production  # Multi-stage build target

# Image and Container
image: myimage:tag    # Image to use/create
container_name: myapp # Custom container name

# Networking
ports:
  - "8080:80"         # HOST:CONTAINER
expose:
  - "8000"            # Expose to other services only
networks:
  - mynetwork

# Storage
volumes:
  - ./app:/app        # Bind mount
  - data:/var/data    # Named volume
  - /app/node_modules # Anonymous volume

# Environment
environment:
  - KEY=value
env_file:
  - .env

# Dependencies
depends_on:
  - db
  - cache

# Restart Policy
restart: always           # always, unless-stopped, on-failure, no

# Resource Limits
deploy:
  resources:
    limits:
      cpus: '0.5'
      memory: 512M
    reservations:
      cpus: '0.25'
      memory: 256M
```

---

### Q24: How do you scale services using Docker Compose?

**Answer:**

**Method 1: Using --scale flag**
```bash
# Scale a service to 3 instances
docker-compose up -d --scale web=3

# Scale multiple services
docker-compose up -d --scale web=3 --scale api=5

# Scale down
docker-compose up -d --scale web=1
```

**Method 2: Using deploy.replicas (Swarm mode)**
```yaml
version: '3.8'

services:
  web:
    image: nginx
    deploy:
      replicas: 3
```

**Important Considerations:**

```yaml
# DON'T do this when scaling (port conflicts)
services:
  web:
    image: nginx
    ports:
      - "80:80"  # Can't bind same port multiple times
```

```yaml
# DO this when scaling (dynamic ports or load balancer)
services:
  web:
    image: nginx
    expose:
      - "80"

  load-balancer:
    image: nginx
    ports:
      - "80:80"
    depends_on:
      - web
```

**Complete Scaling Example:**
```yaml
version: '3.8'

services:
  # Nginx load balancer
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - web
    networks:
      - frontend

  # Scalable web service
  web:
    build: ./app
    expose:
      - "3000"
    environment:
      - NODE_ENV=production
    networks:
      - frontend
      - backend
    depends_on:
      - db

  # Database (don't scale)
  db:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  db-data:
```

**Nginx Load Balancer Configuration:**
```nginx
# nginx.conf
upstream web_servers {
    server web:3000;
}

server {
    listen 80;

    location / {
        proxy_pass http://web_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Scale the application:**
```bash
# Start with 1 instance
docker-compose up -d

# Scale to 5 instances
docker-compose up -d --scale web=5

# Check running containers
docker-compose ps

# View logs from all instances
docker-compose logs -f web
```

---

### Q25: How do you handle environment variables in Docker Compose?

**Answer:**

**Method 1: Inline in docker-compose.yml**
```yaml
services:
  web:
    image: myapp
    environment:
      - NODE_ENV=production
      - API_KEY=abc123
      - DATABASE_URL=postgres://db:5432/myapp
```

**Method 2: Using env_file**
```yaml
# docker-compose.yml
services:
  web:
    image: myapp
    env_file:
      - .env
      - .env.production
```

```bash
# .env file
NODE_ENV=production
API_KEY=abc123
DATABASE_URL=postgres://db:5432/myapp
LOG_LEVEL=info
```

**Method 3: Substitution from Host Environment**
```yaml
# docker-compose.yml
services:
  web:
    image: myapp
    environment:
      - API_KEY=${API_KEY}
      - DATABASE_URL=${DATABASE_URL:-postgres://localhost:5432/default}
```

```bash
# Set on host then run
export API_KEY=secret123
docker-compose up -d
```

**Method 4: Using .env file for Compose Variables**
```yaml
# docker-compose.yml
services:
  web:
    image: myapp:${VERSION}
    ports:
      - "${WEB_PORT}:3000"
```

```bash
# .env (used by Compose itself)
VERSION=1.0.0
WEB_PORT=8080
```

**Complete Example:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: ./web
    image: myapp/web:${VERSION:-latest}
    ports:
      - "${WEB_PORT:-3000}:3000"
    environment:
      - NODE_ENV=${NODE_ENV:-development}
      - API_URL=http://api:8000
    env_file:
      - .env.common
      - .env.${NODE_ENV:-development}
    depends_on:
      - api

  api:
    build: ./api
    environment:
      - DATABASE_URL=postgres://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      - JWT_SECRET=${JWT_SECRET}
      - REDIS_URL=redis://cache:6379
    env_file:
      - .env.api

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
```

```bash
# .env (Compose variables)
VERSION=1.0.0
WEB_PORT=3000
NODE_ENV=production
DB_NAME=myapp
DB_USER=appuser
DB_PASSWORD=secret123

# .env.common (common variables for containers)
LOG_LEVEL=info
TZ=UTC

# .env.production (production-specific)
DEBUG=false
CACHE_TTL=3600

# .env.api (API-specific)
JWT_SECRET=super-secret-key
API_RATE_LIMIT=100
```

**Best Practices:**
```bash
# .env.example (commit this to git)
VERSION=latest
WEB_PORT=3000
NODE_ENV=development
DB_NAME=myapp
DB_USER=postgres
DB_PASSWORD=changeme
JWT_SECRET=change-this-secret

# .gitignore
.env
.env.local
.env.production
.env.*.local
```

---

### Q26: What are the common Docker Compose commands?

**Answer:**

**Starting and Stopping:**
```bash
# Start services
docker-compose up                    # Foreground
docker-compose up -d                 # Background (detached)
docker-compose up --build            # Rebuild images before starting
docker-compose up --force-recreate   # Recreate containers

# Stop services
docker-compose stop                  # Stop containers (don't remove)
docker-compose down                  # Stop and remove containers, networks
docker-compose down -v               # Also remove volumes
docker-compose down --rmi all        # Also remove images

# Restart services
docker-compose restart               # Restart all services
docker-compose restart web           # Restart specific service
```

**Building:**
```bash
# Build or rebuild services
docker-compose build                 # Build all services
docker-compose build web             # Build specific service
docker-compose build --no-cache      # Build without cache
docker-compose build --pull          # Always pull newer base images
```

**Viewing and Monitoring:**
```bash
# List containers
docker-compose ps                    # All services
docker-compose ps web                # Specific service

# View logs
docker-compose logs                  # All services
docker-compose logs -f               # Follow logs
docker-compose logs -f web           # Follow specific service
docker-compose logs --tail=100 web   # Last 100 lines

# View processes
docker-compose top                   # All services
docker-compose top web               # Specific service

# View events
docker-compose events                # Stream events
docker-compose events --json         # JSON format
```

**Executing Commands:**
```bash
# Execute command in running container
docker-compose exec web bash         # Interactive shell
docker-compose exec db psql -U postgres
docker-compose exec -T web npm test  # Non-interactive

# Run one-off command (creates new container)
docker-compose run web npm install
docker-compose run --rm web bash     # Remove after execution
```

**Service Management:**
```bash
# Start specific services
docker-compose up -d web db

# Stop specific services
docker-compose stop api

# Pause services
docker-compose pause web
docker-compose unpause web

# Remove stopped containers
docker-compose rm
docker-compose rm -f                 # Force removal
```

**Configuration:**
```bash
# Validate and view composed configuration
docker-compose config                # Validate and show config
docker-compose config --services     # List services
docker-compose config --volumes      # List volumes

# Use specific compose file
docker-compose -f docker-compose.prod.yml up

# Use multiple compose files (override)
docker-compose -f docker-compose.yml -f docker-compose.override.yml up
```

**Scaling:**
```bash
# Scale services
docker-compose up -d --scale web=3
docker-compose up -d --scale api=5 --scale worker=2
```

**Images and Volumes:**
```bash
# Pull images
docker-compose pull                  # Pull all images
docker-compose pull web              # Pull specific image

# Push images
docker-compose push                  # Push all images

# List images
docker-compose images

# Create volumes
docker-compose up --no-start         # Create but don't start
```

**Useful Combinations:**
```bash
# Full rebuild and restart
docker-compose down && docker-compose up -d --build

# Clean restart
docker-compose down -v && docker-compose up -d

# View all logs from the beginning
docker-compose logs --tail=all

# Execute command in service
docker-compose exec web sh -c "npm run migrate && npm start"
```

---

## Dockerfile Best Practices

### Q27: What are the best practices for writing Dockerfiles?

**Answer:**

**1. Use Official Base Images:**
```dockerfile
# Good - Official image
FROM node:18-alpine

# Avoid - Unknown sources
FROM random/node-image
```

**2. Use Specific Tags (not 'latest'):**
```dockerfile
# Good - Specific version
FROM node:18.17.0-alpine

# Avoid - Unstable
FROM node:latest
```

**3. Minimize Layers:**
```dockerfile
# Bad - Multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git

# Good - Single layer
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*
```

**4. Order Instructions by Change Frequency:**
```dockerfile
# Good - Rarely changed first
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "app.js"]

# Bad - Frequently changed first
FROM node:18-alpine
COPY . .              # This invalidates cache often
RUN npm install
CMD ["node", "app.js"]
```

**5. Use .dockerignore:**
```dockerignore
# .dockerignore
node_modules
npm-debug.log
.git
.env
.DS_Store
*.md
dist
coverage
```

**6. Run as Non-Root User:**
```dockerfile
FROM node:18-alpine

# Create user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app
COPY --chown=nodejs:nodejs . .
RUN npm install

# Switch to user
USER nodejs

CMD ["node", "app.js"]
```

**7. Use COPY instead of ADD:**
```dockerfile
# Good
COPY package.json /app/

# Avoid (unless you need ADD's features)
ADD package.json /app/
```

**8. Combine and Clean Up:**
```dockerfile
# Good
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# Bad - Leaves cache in layer
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*
```

**9. Use Multi-Stage Builds:**
```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm install --production
CMD ["node", "dist/main.js"]
```

**10. Set Environment Variables:**
```dockerfile
ENV NODE_ENV=production \
    PORT=3000 \
    LOG_LEVEL=info
```

**Complete Example:**
```dockerfile
# Use specific version of Alpine-based image
FROM node:18.17.0-alpine AS builder

# Install build dependencies
RUN apk add --no-cache python3 make g++

# Set working directory
WORKDIR /app

# Copy dependency files first (better caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy application code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM node:18.17.0-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set working directory
WORKDIR /app

# Copy built files from builder
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package*.json ./

# Set environment
ENV NODE_ENV=production \
    PORT=3000

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start application
CMD ["node", "dist/main.js"]
```

---

### Q28: How do you reduce Docker image size?

**Answer:**

**1. Use Alpine-based Images:**
```dockerfile
# Large image (~900MB)
FROM node:18

# Small image (~170MB)
FROM node:18-alpine
```

**2. Multi-Stage Builds:**
```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage - much smaller
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm install --production
CMD ["node", "dist/main.js"]
```

**3. Remove Unnecessary Files:**
```dockerfile
# Install and clean up
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# For Alpine
RUN apk add --no-cache curl
```

**4. Use .dockerignore:**
```dockerignore
# Prevent copying unnecessary files
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.env.*
dist
build
coverage
.vscode
.idea
*.log
```

**5. Minimize Layers:**
```dockerfile
# Bad - 3 layers
RUN npm install
RUN npm run build
RUN npm prune --production

# Good - 1 layer
RUN npm install && \
    npm run build && \
    npm prune --production
```

**6. Use npm ci instead of npm install:**
```dockerfile
# Faster and produces reproducible builds
RUN npm ci --only=production
```

**7. Remove Development Dependencies:**
```dockerfile
# Install all dependencies for build
RUN npm install

# Build application
RUN npm run build

# Remove dev dependencies
RUN npm prune --production
```

**Complete Example - Python:**
```dockerfile
# Build stage
FROM python:3.11 AS builder

WORKDIR /app

# Install dependencies to a virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Production stage
FROM python:3.11-slim

# Copy only the virtual environment
COPY --from=builder /opt/venv /opt/venv

WORKDIR /app

# Set path to use virtual environment
ENV PATH="/opt/venv/bin:$PATH"

COPY . .

CMD ["python", "app.py"]
```

**Size Comparison Example:**
```dockerfile
# Before optimization - ~1.2GB
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "app.js"]

# After optimization - ~180MB
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app .
USER node
CMD ["node", "app.js"]
```

**Measure Image Size:**
```bash
# Check image size
docker images myapp

# Check layer sizes
docker history myapp

# Analyze image
docker image inspect myapp
```

---

### Q29: What is the difference between RUN, CMD, and ENTRYPOINT?

**Answer:**

**RUN:**
- Executes commands during image build
- Creates new layer
- Used to install packages, build application

```dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install -y python3
RUN pip3 install flask
```

**CMD:**
- Provides default command/arguments for container
- Can be overridden by docker run arguments
- Only last CMD is effective

```dockerfile
FROM ubuntu
CMD ["echo", "Hello World"]

# docker run myimage
# Output: Hello World

# docker run myimage echo "Goodbye"
# Output: Goodbye (CMD overridden)
```

**ENTRYPOINT:**
- Configures container to run as executable
- Arguments from docker run are appended
- Harder to override (requires --entrypoint flag)

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]

# docker run myimage "Hello"
# Output: Hello

# docker run myimage "Hello" "World"
# Output: Hello World
```

**Comparison Table:**

| Feature | RUN | CMD | ENTRYPOINT |
|---------|-----|-----|------------|
| When Executed | Build time | Run time | Run time |
| Purpose | Build image | Default command | Main executable |
| Can Override | No | Yes (easily) | Yes (--entrypoint) |
| Multiple Allowed | Yes (all execute) | Yes (last wins) | Yes (last wins) |

**Best Practice - Use Together:**
```dockerfile
FROM alpine

# ENTRYPOINT = main command
ENTRYPOINT ["python", "app.py"]

# CMD = default arguments
CMD ["--help"]

# docker run myimage
# Executes: python app.py --help

# docker run myimage --production
# Executes: python app.py --production
```

**Real-World Examples:**

```dockerfile
# Example 1: Web Server
FROM nginx
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]

# docker run myimage
# Runs: nginx -g "daemon off;"

# docker run myimage -g "daemon off;" -c /custom/nginx.conf
# Runs: nginx -g "daemon off;" -c /custom/nginx.conf
```

```dockerfile
# Example 2: Database
FROM postgres:15
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["postgres"]

# docker run myimage
# Runs: docker-entrypoint.sh postgres

# docker run myimage postgres -c max_connections=200
# Runs: docker-entrypoint.sh postgres -c max_connections=200
```

```dockerfile
# Example 3: Application
FROM node:18-alpine

WORKDIR /app
COPY . .
RUN npm install

# Make container act like the executable
ENTRYPOINT ["node", "app.js"]

# Default arguments
CMD ["--port", "3000"]

# docker run myimage
# Runs: node app.js --port 3000

# docker run myimage --port 8080 --debug
# Runs: node app.js --port 8080 --debug
```

**Shell vs Exec Form:**

```dockerfile
# Exec form (preferred) - Does not invoke shell
CMD ["echo", "$HOME"]      # Prints literally: $HOME
ENTRYPOINT ["echo", "hi"]  # Runs directly

# Shell form - Invokes shell (/bin/sh -c)
CMD echo $HOME             # Prints: /root
ENTRYPOINT echo "hi"       # Runs via shell
```

---

### Q30: How do you use build arguments (ARG) in Dockerfile?

**Answer:**

**Basic ARG Usage:**
```dockerfile
# Define build argument
FROM node:18-alpine

# Declare ARG
ARG NODE_ENV=production
ARG VERSION=1.0.0

# Use ARG
ENV NODE_ENV=${NODE_ENV}
LABEL version=${VERSION}

WORKDIR /app
COPY . .

# ARG in RUN commands
RUN echo "Building version ${VERSION} for ${NODE_ENV}"
RUN npm install
```

**Build with Arguments:**
```bash
# Use default values
docker build -t myapp .

# Override arguments
docker build --build-arg NODE_ENV=development --build-arg VERSION=2.0.0 -t myapp .
```

**ARG Scope:**
```dockerfile
# ARG before FROM is available only for FROM
ARG BASE_IMAGE=node:18-alpine
FROM ${BASE_IMAGE}

# ARG after FROM is available in that stage
ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}

# ARG needs to be redeclared in each stage
FROM node:18-alpine AS builder
ARG NODE_ENV=production  # Need to redeclare
RUN echo ${NODE_ENV}

FROM node:18-alpine
ARG NODE_ENV=production  # Need to redeclare again
RUN echo ${NODE_ENV}
```

**Complete Example:**
```dockerfile
# Build arguments for base image
ARG NODE_VERSION=18
ARG ALPINE_VERSION=3.18

FROM node:${NODE_VERSION}-alpine${ALPINE_VERSION} AS builder

# Build arguments for build process
ARG NODE_ENV=production
ARG API_URL=https://api.example.com
ARG BUILD_DATE
ARG VERSION

# Set environment variables from build args
ENV NODE_ENV=${NODE_ENV}

WORKDIR /app

# Use build args in labels
LABEL maintainer="dev@example.com" \
      version="${VERSION}" \
      build-date="${BUILD_DATE}" \
      description="Application built for ${NODE_ENV}"

COPY package*.json ./

# Use ARG in RUN
RUN if [ "$NODE_ENV" = "development" ]; then \
        npm install; \
    else \
        npm ci --only=production; \
    fi

COPY . .

# Pass ARG to application build
RUN API_URL=${API_URL} npm run build

# Production stage
FROM node:${NODE_VERSION}-alpine${ALPINE_VERSION}

ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

CMD ["node", "dist/main.js"]
```

**Build with All Arguments:**
```bash
docker build \
  --build-arg NODE_VERSION=18 \
  --build-arg ALPINE_VERSION=3.18 \
  --build-arg NODE_ENV=production \
  --build-arg API_URL=https://api.prod.example.com \
  --build-arg VERSION=1.2.3 \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  -t myapp:1.2.3 \
  .
```

**Practical Use Cases:**

```dockerfile
# Use case 1: Different build configurations
ARG BUILD_CONFIG=release

FROM golang:1.21-alpine AS builder
ARG BUILD_CONFIG
WORKDIR /app
COPY . .
RUN go build -tags ${BUILD_CONFIG} -o app

# Build for different configs
# docker build --build-arg BUILD_CONFIG=debug -t myapp:debug .
# docker build --build-arg BUILD_CONFIG=release -t myapp:release .
```

```dockerfile
# Use case 2: Private package registry
ARG NPM_TOKEN

FROM node:18-alpine
ARG NPM_TOKEN

# Use token to access private packages
RUN echo "//registry.npmjs.org/:_authToken=${NPM_TOKEN}" > ~/.npmrc
RUN npm install
RUN rm ~/.npmrc  # Remove token from image

# Build with token
# docker build --build-arg NPM_TOKEN=${NPM_TOKEN} -t myapp .
```

```dockerfile
# Use case 3: Conditional installations
ARG INSTALL_DEV_TOOLS=false

FROM ubuntu:22.04
ARG INSTALL_DEV_TOOLS

RUN apt-get update && \
    apt-get install -y python3 && \
    if [ "$INSTALL_DEV_TOOLS" = "true" ]; then \
        apt-get install -y python3-dev build-essential; \
    fi

# Development build
# docker build --build-arg INSTALL_DEV_TOOLS=true -t myapp:dev .

# Production build
# docker build -t myapp:prod .
```

**ARG vs ENV:**

```dockerfile
# ARG - Available only during build
ARG BUILD_VERSION=1.0.0
RUN echo ${BUILD_VERSION}  # Works

# ENV - Available during build AND runtime
ENV APP_VERSION=1.0.0
RUN echo ${APP_VERSION}    # Works

# In running container
# docker run myimage echo $BUILD_VERSION  # Empty (ARG not available)
# docker run myimage echo $APP_VERSION   # Prints: 1.0.0
```

---

## Multi-Stage Builds

### Q31: What are multi-stage builds and why are they important?

**Answer:**

Multi-stage builds allow you to use multiple FROM statements in a Dockerfile. Each FROM begins a new stage, and you can selectively copy artifacts from one stage to another.

**Benefits:**
- Smaller final images
- Separation of build and runtime dependencies
- Better security (no build tools in production)
- Cleaner Dockerfiles
- Different environments in one file

**Simple Example:**
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]

# Result: Only production files in final image
```

**Without Multi-Stage:**
```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
# Final image includes build tools, source files, etc. (~900MB)
CMD ["node", "dist/main.js"]
```

**With Multi-Stage:**
```dockerfile
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
# Final image only has runtime files (~180MB)
CMD ["node", "dist/main.js"]
```

---

### Q32: Provide examples of multi-stage builds for different languages.

**Answer:**

**Node.js Application:**
```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
RUN npm prune --production

# Production stage
FROM node:18-alpine
ENV NODE_ENV=production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

**Go Application:**
```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Production stage - Minimal image
FROM alpine:3.18
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]

# Or even smaller with scratch
FROM scratch
COPY --from=builder /app/main .
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
EXPOSE 8080
CMD ["./main"]
```

**Python Application:**
```dockerfile
# Build stage
FROM python:3.11 AS builder
WORKDIR /app

# Install dependencies in virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Production stage
FROM python:3.11-slim
WORKDIR /app

# Copy virtual environment from builder
COPY --from=builder /opt/venv /opt/venv

ENV PATH="/opt/venv/bin:$PATH"

COPY . .

EXPOSE 8000
CMD ["python", "app.py"]
```

**Java Spring Boot Application:**
```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Production stage
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**React Application:**
```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage - Serve with Nginx
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Rust Application:**
```dockerfile
# Build stage
FROM rust:1.74 AS builder
WORKDIR /app
COPY Cargo.toml Cargo.lock ./
# Cache dependencies
RUN mkdir src && echo "fn main() {}" > src/main.rs
RUN cargo build --release
RUN rm -rf src
# Build actual application
COPY src ./src
RUN cargo build --release

# Production stage
FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY --from=builder /app/target/release/myapp .
EXPOSE 8080
CMD ["./myapp"]
```

**Multi-Stage with Multiple Targets:**
```dockerfile
# Base stage
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./

# Development stage
FROM base AS development
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

# Build stage
FROM base AS builder
RUN npm ci
COPY . .
RUN npm run build
RUN npm prune --production

# Production stage
FROM node:18-alpine AS production
ENV NODE_ENV=production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]

# Test stage
FROM base AS test
RUN npm install
COPY . .
CMD ["npm", "test"]
```

**Build Specific Stages:**
```bash
# Build development image
docker build --target development -t myapp:dev .

# Build production image
docker build --target production -t myapp:prod .

# Run tests
docker build --target test -t myapp:test .
docker run myapp:test
```

---

### Q33: How do you copy files from a specific build stage?

**Answer:**

**Basic Syntax:**
```dockerfile
COPY --from=<stage-name> <source> <destination>
```

**Named Stages:**
```dockerfile
# Name the stage with AS
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm run build

# Copy from named stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

**Copy from Stage Number:**
```dockerfile
# First stage (0)
FROM node:18
WORKDIR /app
RUN npm install

# Second stage (1)
FROM node:18-alpine
# Copy from first stage using index
COPY --from=0 /app/node_modules ./node_modules
```

**Copy from External Image:**
```dockerfile
# Copy from any image, not just build stages
FROM alpine

# Copy nginx binary from nginx image
COPY --from=nginx:latest /usr/sbin/nginx /usr/sbin/nginx

# Copy from specific version
COPY --from=busybox:1.35 /bin/busybox /bin/busybox
```

**Multiple Copies from Different Stages:**
```dockerfile
# Builder stage
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Testing stage
FROM node:18 AS tester
WORKDIR /app
COPY . .
RUN npm install && npm test

# Production stage
FROM node:18-alpine
WORKDIR /app

# Copy build output from builder
COPY --from=builder /app/dist ./dist

# Copy test results from tester
COPY --from=tester /app/coverage ./coverage

# Copy dependencies from builder
COPY --from=builder /app/node_modules ./node_modules

CMD ["node", "dist/main.js"]
```

**Real-World Example:**
```dockerfile
# Dependencies stage
FROM node:18-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app

# Copy production dependencies from dependencies stage
COPY --from=dependencies /app/node_modules ./node_modules

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Copy only necessary files from builder
COPY --from=builder /app/package.json ./

USER node
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

**Copy Specific Files with Ownership:**
```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

FROM alpine:3.18
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

# Copy with specific ownership
COPY --from=builder --chown=appuser:appgroup /app/myapp /usr/local/bin/myapp

USER appuser
CMD ["myapp"]
```

**Copy from Multiple External Images:**
```dockerfile
FROM alpine:3.18

# Copy tools from different images
COPY --from=busybox:1.35 /bin/busybox /bin/busybox
COPY --from=curlimages/curl:latest /usr/bin/curl /usr/bin/curl
COPY --from=hashicorp/terraform:1.6 /bin/terraform /bin/terraform

# Now you have multiple tools in one image
```

---

## Docker Security

### Q34: What are Docker security best practices?

**Answer:**

**1. Use Official and Verified Images:**
```dockerfile
# Good - Official image
FROM node:18-alpine

# Risky - Unknown source
FROM random-user/node
```

**2. Don't Run as Root:**
```dockerfile
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app
COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

CMD ["node", "app.js"]
```

**3. Use Specific Image Tags:**
```dockerfile
# Good - Specific version
FROM node:18.17.0-alpine3.18

# Bad - Unpredictable
FROM node:latest
```

**4. Scan Images for Vulnerabilities:**
```bash
# Using Docker Scout
docker scout cves myapp:latest

# Using Trivy
trivy image myapp:latest

# Using Snyk
snyk container test myapp:latest
```

**5. Minimize Image Layers and Size:**
```dockerfile
# Smaller attack surface
FROM node:18-alpine AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM node:18-alpine
COPY --from=builder /app/dist ./dist
USER node
CMD ["node", "dist/main.js"]
```

**6. Don't Store Secrets in Images:**
```dockerfile
# Bad - Secret in image
ENV API_KEY=secret123

# Good - Use secrets at runtime
ENV API_KEY=${API_KEY}

# Or use Docker secrets
docker run -e API_KEY=$API_KEY myapp
```

**7. Use .dockerignore:**
```dockerignore
.env
.env.local
.git
.gitignore
secrets/
*.key
*.pem
node_modules
.npm
```

**8. Set Read-Only Root Filesystem:**
```bash
docker run --read-only --tmpfs /tmp myapp
```

**9. Limit Container Resources:**
```bash
# Limit memory and CPU
docker run -m 512m --cpus="1.0" myapp

# In docker-compose.yml
services:
  web:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

**10. Use Security Options:**
```bash
# Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

# Use AppArmor/SELinux
docker run --security-opt apparmor=docker-default myapp

# Disable privilege escalation
docker run --security-opt no-new-privileges myapp
```

**11. Sign and Verify Images:**
```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Push signed image
docker push myapp:latest

# Pull will verify signature
docker pull myapp:latest
```

**12. Regular Updates:**
```bash
# Regularly rebuild images with updated base images
docker build --pull --no-cache -t myapp:latest .
```

**Complete Secure Dockerfile:**
```dockerfile
# Use specific version
FROM node:18.17.0-alpine3.18 AS builder

# Install only what's needed
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

COPY . .
RUN npm run build

# Production stage
FROM node:18.17.0-alpine3.18

# Update packages
RUN apk update && apk upgrade && \
    apk add --no-cache dumb-init && \
    rm -rf /var/cache/apk/*

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 -G nodejs

WORKDIR /app

# Copy with ownership
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs package*.json ./

# Set environment
ENV NODE_ENV=production

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Use dumb-init for signal handling
ENTRYPOINT ["dumb-init", "--"]

# Start application
CMD ["node", "dist/main.js"]
```

**Runtime Security:**
```bash
# Run with security best practices
docker run -d \
  --name myapp \
  --read-only \
  --tmpfs /tmp \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --security-opt no-new-privileges \
  -m 512m \
  --cpus="0.5" \
  -e API_KEY=$API_KEY \
  -p 3000:3000 \
  myapp:latest
```

---

### Q35: How do you handle secrets in Docker?

**Answer:**

**1. Docker Secrets (Swarm Mode):**
```bash
# Create secret
echo "my-secret-password" | docker secret create db_password -

# Use in service
docker service create \
  --name myapp \
  --secret db_password \
  myimage

# Access in container at /run/secrets/db_password
```

```yaml
# docker-compose.yml (Swarm mode)
version: '3.8'

services:
  app:
    image: myapp
    secrets:
      - db_password
      - api_key

secrets:
  db_password:
    external: true
  api_key:
    file: ./api_key.txt
```

**2. Environment Variables (Less Secure):**
```bash
# Pass at runtime
docker run -e DB_PASSWORD=$DB_PASSWORD myapp

# From file
docker run --env-file .env myapp
```

```yaml
# docker-compose.yml
services:
  app:
    image: myapp
    environment:
      - DB_PASSWORD=${DB_PASSWORD}
    env_file:
      - .env
```

**3. Build-Time Secrets (BuildKit):**
```dockerfile
# Dockerfile
# syntax=docker/dockerfile:1

FROM alpine

# Mount secret during build
RUN --mount=type=secret,id=npmtoken \
    NPM_TOKEN=$(cat /run/secrets/npmtoken) && \
    echo "//registry.npmjs.org/:_authToken=${NPM_TOKEN}" > ~/.npmrc && \
    npm install && \
    rm ~/.npmrc
```

```bash
# Build with secret
docker build --secret id=npmtoken,src=.npmtoken -t myapp .
```

**4. External Secret Managers:**
```bash
# HashiCorp Vault
docker run \
  -e VAULT_ADDR=https://vault.example.com \
  -e VAULT_TOKEN=$VAULT_TOKEN \
  myapp

# AWS Secrets Manager
docker run \
  -e AWS_REGION=us-east-1 \
  -v ~/.aws:/root/.aws:ro \
  myapp
```

**5. Volume Mounts (Development):**
```bash
# Mount secrets file
docker run -v /path/to/secrets:/secrets:ro myapp
```

**Best Practices:**

```dockerfile
# DON'T - Secret in image layer
ENV API_KEY=secret123
RUN curl -H "Authorization: $API_KEY" ...

# DO - Use build secrets
RUN --mount=type=secret,id=api_key \
    API_KEY=$(cat /run/secrets/api_key) && \
    curl -H "Authorization: $API_KEY" ...
```

**Application Code Example:**
```javascript
// Node.js - Read Docker secret
const fs = require('fs');

function getSecret(secretName) {
  try {
    // Docker secrets are mounted at /run/secrets/
    return fs.readFileSync(`/run/secrets/${secretName}`, 'utf8').trim();
  } catch (err) {
    // Fallback to environment variable
    return process.env[secretName.toUpperCase()];
  }
}

const dbPassword = getSecret('db_password');
const apiKey = getSecret('api_key');
```

**Complete Example:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: ./web
    secrets:
      - db_password
      - api_key
    environment:
      - DB_HOST=db
      - DB_USER=postgres
    depends_on:
      - db

  db:
    image: postgres:15
    secrets:
      - db_password
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - db-data:/var/lib/postgresql/data

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    external: true

volumes:
  db-data:
```

**Create External Secret:**
```bash
# For Swarm mode
echo "my-api-key" | docker secret create api_key -

# For Compose (file-based)
echo "my-api-key" > ./secrets/api_key.txt
chmod 600 ./secrets/api_key.txt
```

---

### Q36: How do you run containers as non-root users?

**Answer:**

**Method 1: In Dockerfile:**
```dockerfile
FROM node:18-alpine

# Create user and group
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

WORKDIR /app

# Copy files with correct ownership
COPY --chown=appuser:appgroup . .

# Install dependencies as root (if needed)
RUN npm install

# Switch to non-root user
USER appuser

CMD ["node", "app.js"]
```

**Method 2: At Runtime:**
```bash
# Run as specific user
docker run --user 1001:1001 myapp

# Run as named user
docker run --user appuser myapp

# Run as current host user
docker run --user $(id -u):$(id -g) myapp
```

**Different Base Images:**

```dockerfile
# Alpine-based
FROM alpine:3.18
RUN addgroup -g 1001 -S mygroup && \
    adduser -S myuser -u 1001 -G mygroup
USER myuser

# Debian/Ubuntu-based
FROM ubuntu:22.04
RUN groupadd -g 1001 mygroup && \
    useradd -r -u 1001 -g mygroup myuser
USER myuser

# Using existing node user
FROM node:18-alpine
USER node
WORKDIR /home/node/app
```

**Handle Permissions:**
```dockerfile
FROM node:18-alpine

# Create app directory with correct permissions
WORKDIR /app

# Copy package files as root
COPY package*.json ./

# Install dependencies as root
RUN npm install && \
    chown -R node:node /app

# Switch to node user
USER node

# Copy app files (will be owned by node user)
COPY --chown=node:node . .

CMD ["node", "app.js"]
```

**Complete Example with Volume Permissions:**
```dockerfile
FROM node:18-alpine

# Use existing node user (UID 1000, GID 1000)
ENV HOME=/home/node

WORKDIR /app

# Install dependencies as root
COPY package*.json ./
RUN npm install && \
    chown -R node:node /app && \
    mkdir -p /app/logs && \
    chown -R node:node /app/logs

# Switch to non-root user
USER node

COPY --chown=node:node . .

EXPOSE 3000
CMD ["node", "app.js"]
```

**Docker Compose:**
```yaml
version: '3.8'

services:
  web:
    build: .
    user: "1001:1001"  # Override USER from Dockerfile
    volumes:
      - ./data:/app/data
    # Host volume needs matching permissions:
    # mkdir -p ./data && chown 1001:1001 ./data
```

**Handle Cases Requiring Root:**
```dockerfile
FROM node:18-alpine

# Install system packages as root
RUN apk add --no-cache curl

# Create user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

WORKDIR /app

# Copy and install as root
COPY package*.json ./
RUN npm install && \
    chown -R appuser:appgroup /app

# Switch to user for runtime
USER appuser

COPY --chown=appuser:appgroup . .

CMD ["node", "app.js"]
```

**Verify Running User:**
```bash
# Check user inside container
docker exec mycontainer whoami
docker exec mycontainer id

# Check in Dockerfile build
RUN whoami && id
```

**Python Example:**
```dockerfile
FROM python:3.11-slim

# Create non-root user
RUN useradd -m -u 1001 -s /bin/bash appuser

WORKDIR /app

# Install as root
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Change ownership
RUN chown -R appuser:appuser /app

# Switch to user
USER appuser

COPY --chown=appuser:appuser . .

CMD ["python", "app.py"]
```

**Security Scan:**
```bash
# Verify image runs as non-root
docker run myapp whoami
# Should not output 'root'

# Check with inspect
docker inspect myapp | grep -i user
```

---

## Troubleshooting

### Q37: How do you debug a failing container?

**Answer:**

**1. Check Container Status:**
```bash
# List all containers
docker ps -a

# Check last exit code
docker inspect <container> --format='{{.State.ExitCode}}'

# Common exit codes:
# 0 - Success
# 1 - Application error
# 137 - Out of memory (OOM) killed
# 139 - Segmentation fault
# 143 - Graceful termination (SIGTERM)
```

**2. View Container Logs:**
```bash
# View all logs
docker logs <container>

# Follow logs in real-time
docker logs -f <container>

# Last 100 lines
docker logs --tail 100 <container>

# Logs with timestamps
docker logs -t <container>

# Logs since specific time
docker logs --since 30m <container>
```

**3. Inspect Container:**
```bash
# Full inspection
docker inspect <container>

# Specific information
docker inspect --format='{{.State.Status}}' <container>
docker inspect --format='{{.NetworkSettings.IPAddress}}' <container>
docker inspect --format='{{range .Mounts}}{{.Source}}:{{.Destination}}{{"\n"}}{{end}}' <container>
```

**4. Execute Commands in Container:**
```bash
# Interactive shell
docker exec -it <container> sh
docker exec -it <container> bash

# Run specific commands
docker exec <container> ls -la /app
docker exec <container> cat /etc/hosts
docker exec <container> env

# Check processes
docker exec <container> ps aux

# Check disk space
docker exec <container> df -h
```

**5. Start Container in Debug Mode:**
```bash
# Override entrypoint
docker run -it --entrypoint sh myimage

# Override command
docker run -it myimage sh

# Run with different command
docker run -it myimage /bin/bash
```

**6. Check Resource Usage:**
```bash
# Real-time stats
docker stats <container>

# Check if OOM killed
docker inspect <container> | grep OOMKilled
```

**7. Check Network Connectivity:**
```bash
# From host to container
docker exec <container> ping google.com

# Check listening ports
docker exec <container> netstat -tuln

# Check if port is accessible
curl http://localhost:8080

# Inspect network
docker network inspect bridge
```

**8. Common Issues and Solutions:**

**Issue: Container Exits Immediately**
```bash
# Check logs
docker logs <container>

# Try running interactively
docker run -it myimage sh

# Check if main process exits
docker run myimage ps aux
```

**Issue: Cannot Connect to Container**
```bash
# Check port mapping
docker port <container>

# Check if service is listening
docker exec <container> netstat -tuln

# Check firewall
docker exec <container> curl localhost:8080

# Inspect network
docker network inspect <network>
```

**Issue: Permission Denied**
```bash
# Check file ownership
docker exec <container> ls -l /app

# Check running user
docker exec <container> whoami

# Check permissions
docker exec <container> stat /app/file.txt
```

**Issue: Out of Disk Space**
```bash
# Check container disk usage
docker exec <container> df -h

# Check Docker disk usage
docker system df

# Clean up
docker system prune -a
```

**9. Debugging Failed Builds:**
```bash
# Build with no cache
docker build --no-cache -t myimage .

# Build specific stage
docker build --target builder -t myimage:debug .

# Run intermediate stage
docker run -it myimage:debug sh

# Check build history
docker history myimage
```

**10. Complete Debugging Session:**
```bash
# 1. Check if container is running
docker ps -a | grep myapp

# 2. View logs
docker logs --tail 50 myapp

# 3. Check exit code
docker inspect myapp --format='{{.State.ExitCode}}'

# 4. Try starting with shell
docker run -it --entrypoint sh myimage

# 5. Inside container, debug
whoami
env
ls -la
cat /app/config.json
curl localhost:3000

# 6. Check from another container (network test)
docker run --rm --network container:myapp alpine wget -O- http://localhost:3000

# 7. Check resource limits
docker stats myapp

# 8. Inspect full configuration
docker inspect myapp > inspect.json
```

---

### Q38: How do you troubleshoot networking issues in Docker?

**Answer:**

**1. Check Container Network Configuration:**
```bash
# List networks
docker network ls

# Inspect network
docker network inspect bridge

# Check container's networks
docker inspect <container> --format='{{range $net,$v := .NetworkSettings.Networks}}{{$net}} {{end}}'

# Get container IP
docker inspect <container> --format='{{.NetworkSettings.IPAddress}}'
```

**2. Test Container-to-Container Communication:**
```bash
# From one container to another
docker exec container1 ping container2

# Check DNS resolution
docker exec container1 nslookup container2

# Test with curl
docker exec container1 curl http://container2:8080

# Check if containers are on same network
docker network inspect my-network
```

**3. Test Port Mapping:**
```bash
# Check port mappings
docker port <container>

# Test from host
curl http://localhost:8080

# Test from inside container
docker exec <container> curl http://localhost:8080

# Check if port is listening
docker exec <container> netstat -tuln | grep 8080
```

**4. Common Network Issues:**

**Issue: Cannot Connect to Container from Host**
```bash
# 1. Verify port mapping
docker ps
# Look for: 0.0.0.0:8080->80/tcp

# 2. Check if service is listening
docker exec <container> netstat -tuln

# 3. Test from inside container first
docker exec <container> curl localhost:80

# 4. Check firewall
# Windows
netstat -an | findstr 8080

# Linux/Mac
sudo iptables -L -n

# 5. Try different IP
curl http://127.0.0.1:8080
curl http://<docker-host-ip>:8080
```

**Issue: Containers Cannot Communicate**
```bash
# 1. Check if on same network
docker inspect container1 --format='{{.NetworkSettings.Networks}}'
docker inspect container2 --format='{{.NetworkSettings.Networks}}'

# 2. If not, connect to same network
docker network connect my-network container1
docker network connect my-network container2

# 3. Test connectivity
docker exec container1 ping container2

# 4. Check DNS
docker exec container1 nslookup container2

# 5. Use IP if DNS fails
docker exec container1 ping <container2-ip>
```

**Issue: DNS Not Working**
```bash
# 1. Check DNS config
docker exec <container> cat /etc/resolv.conf

# 2. Test DNS resolution
docker exec <container> nslookup google.com

# 3. Use custom DNS
docker run --dns 8.8.8.8 myimage

# 4. Add to hosts file
docker run --add-host=myhost:192.168.1.100 myimage
```

**5. Network Debugging Tools:**

```bash
# Run debug container on same network
docker run -it --network container:<container-name> nicolaka/netshoot

# Inside debug container:
# - curl, wget, ping available
# - nslookup, dig for DNS
# - netstat, ss for connections
# - tcpdump for packet capture

# Example usage
docker run -it --network container:myapp nicolaka/netshoot
# Then inside:
curl http://localhost:8080
netstat -tuln
nslookup api
```

**6. Inspect Network Traffic:**
```bash
# Capture packets
docker run --rm --net=host \
  -v /var/run/docker.sock:/var/run/docker.sock \
  nicolaka/netshoot tcpdump -i eth0 port 8080

# Monitor connections
docker exec <container> netstat -anp

# Check iptables rules
docker run --rm --privileged --net=host nicolaka/netshoot iptables -L -n -v
```

**7. Test External Connectivity:**
```bash
# Test internet access from container
docker exec <container> ping 8.8.8.8
docker exec <container> curl https://google.com

# Check DNS to external
docker exec <container> nslookup google.com

# Test with different network
docker run --rm alpine ping google.com
```

**8. Network Modes:**

```bash
# Bridge (default) - Test isolated network
docker run --network bridge myimage

# Host - Use host network stack
docker run --network host myimage

# None - No networking
docker run --network none myimage

# Container - Share network with another container
docker run --network container:other-container myimage
```

**9. Debug docker-compose Networking:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - frontend
    ports:
      - "80:80"

  api:
    image: myapi
    networks:
      - frontend
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
  backend:
```

```bash
# Check compose networks
docker-compose ps
docker network ls | grep projectname

# Test connectivity
docker-compose exec web ping api
docker-compose exec api ping db

# View network config
docker network inspect projectname_frontend
```

**10. Complete Troubleshooting Example:**

```bash
# Problem: Web container can't connect to API container

# Step 1: Verify containers are running
docker ps

# Step 2: Check networks
docker network inspect bridge
docker inspect web --format='{{.NetworkSettings.Networks}}'
docker inspect api --format='{{.NetworkSettings.Networks}}'

# Step 3: Get API container IP
API_IP=$(docker inspect api --format='{{.NetworkSettings.IPAddress}}')

# Step 4: Test ping
docker exec web ping $API_IP
docker exec web ping api  # Test DNS

# Step 5: Test port
docker exec api netstat -tuln | grep 8000
docker exec web curl http://$API_IP:8000
docker exec web curl http://api:8000

# Step 6: Check logs
docker logs web
docker logs api

# Step 7: If not on same network, create one
docker network create app-network
docker network connect app-network web
docker network connect app-network api

# Step 8: Test again
docker exec web curl http://api:8000
```

---

### Q39: How do you check Docker container resource usage?

**Answer:**

**1. Real-Time Stats:**
```bash
# All containers
docker stats

# Specific container
docker stats <container>

# Format output
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# No streaming (one-time)
docker stats --no-stream

# Example output:
# CONTAINER  CPU %   MEM USAGE / LIMIT    MEM %   NET I/O      BLOCK I/O
# web        0.50%   256MiB / 512MiB      50%     1.2GB / 2GB  50MB / 10MB
```

**2. Inspect Resource Limits:**
```bash
# Check memory limit
docker inspect <container> --format='{{.HostConfig.Memory}}'

# Check CPU limit
docker inspect <container> --format='{{.HostConfig.CpuShares}}'

# Check all resource settings
docker inspect <container> | grep -A 20 HostConfig
```

**3. Check Disk Usage:**
```bash
# Overall Docker disk usage
docker system df

# Detailed breakdown
docker system df -v

# Container specific
docker exec <container> df -h

# Check container layer size
docker ps -s
```

**4. Monitor Logs Size:**
```bash
# Find log file location
docker inspect <container> --format='{{.LogPath}}'

# Check log size
du -sh $(docker inspect <container> --format='{{.LogPath}}')

# Limit log size in docker-compose.yml
services:
  web:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

**5. Check Process Usage:**
```bash
# List processes
docker top <container>

# Detailed process info
docker exec <container> ps aux

# Process tree
docker exec <container> ps -ef --forest
```

**6. Memory Analysis:**
```bash
# Memory stats
docker stats --no-stream --format "table {{.Name}}\t{{.MemUsage}}\t{{.MemPerc}}"

# Check for OOM
docker inspect <container> --format='{{.State.OOMKilled}}'

# Inside container memory info
docker exec <container> free -h
docker exec <container> cat /proc/meminfo
```

**7. CPU Analysis:**
```bash
# CPU usage
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}"

# Inside container CPU info
docker exec <container> nproc
docker exec <container> cat /proc/cpuinfo
docker exec <container> top -bn1
```

**8. Network I/O:**
```bash
# Network stats
docker stats --no-stream --format "table {{.Name}}\t{{.NetIO}}"

# Detailed network stats
docker exec <container> netstat -i
docker exec <container> cat /proc/net/dev
```

**9. Set Resource Limits:**

```bash
# Memory limit
docker run -m 512m myimage

# Memory and swap
docker run -m 512m --memory-swap 1g myimage

# CPU limit
docker run --cpus="1.5" myimage
docker run --cpu-shares=512 myimage

# All together
docker run \
  -m 512m \
  --memory-swap 1g \
  --cpus="1.0" \
  --cpu-shares=1024 \
  myimage
```

**10. Docker Compose Resource Limits:**

```yaml
version: '3.8'

services:
  web:
    image: nginx
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

**11. Historical Monitoring:**

```bash
# Export stats to file
docker stats --no-stream >> stats.log

# Monitor with timestamp
while true; do
  echo "$(date): $(docker stats --no-stream --format '{{.Name}} {{.CPUPerc}} {{.MemPerc}}')"
  sleep 60
done >> monitoring.log
```

**12. Third-Party Monitoring:**

```yaml
# Prometheus + cAdvisor monitoring
version: '3.8'

services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    ports:
      - 8080:8080
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

  prometheus:
    image: prom/prometheus
    ports:
      - 9090:9090
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - 3000:3000
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
```

---

## Real-World Scenarios

### Q40: How would you containerize a MERN stack application?

**Answer:**

**Project Structure:**
```
mern-app/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
└── nginx/
    └── nginx.conf
```

**Backend Dockerfile:**
```dockerfile
# backend/Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

# Production stage
FROM node:18-alpine

ENV NODE_ENV=production

RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs . .

USER nodejs

EXPOSE 5000

CMD ["node", "server.js"]
```

**Frontend Dockerfile:**
```dockerfile
# frontend/Dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage - Serve with Nginx
FROM nginx:alpine

COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**Nginx Configuration:**
```nginx
# nginx/nginx.conf
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    # Frontend routes
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API proxy
    location /api {
        proxy_pass http://backend:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

**Docker Compose:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  # MongoDB Database
  mongodb:
    image: mongo:7
    container_name: mern-mongodb
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secret
      MONGO_INITDB_DATABASE: mernapp
    volumes:
      - mongodb-data:/data/db
      - ./mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js:ro
    networks:
      - mern-network
    restart: unless-stopped

  # Express Backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: mern-backend
    environment:
      - NODE_ENV=production
      - PORT=5000
      - MONGODB_URI=mongodb://admin:secret@mongodb:27017/mernapp?authSource=admin
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - mongodb
    networks:
      - mern-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # React Frontend + Nginx
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: mern-frontend
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - mern-network
    restart: unless-stopped

networks:
  mern-network:
    driver: bridge

volumes:
  mongodb-data:
    driver: local
```

**MongoDB Initialization:**
```javascript
// mongo-init.js
db = db.getSiblingDB('mernapp');

db.createUser({
  user: 'appuser',
  pwd: 'apppassword',
  roles: [
    {
      role: 'readWrite',
      db: 'mernapp'
    }
  ]
});

db.createCollection('users');
```

**Environment File:**
```bash
# .env
JWT_SECRET=your-super-secret-jwt-key-change-in-production
MONGODB_URI=mongodb://admin:secret@mongodb:27017/mernapp?authSource=admin
```

**Development Override:**
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    command: npm run dev
    ports:
      - "5000:5000"

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:5000
    command: npm start
```

**Commands:**
```bash
# Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker-compose up -d

# Build and start
docker-compose up -d --build

# View logs
docker-compose logs -f backend

# Stop
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

**Backend Health Endpoint:**
```javascript
// backend/server.js
const express = require('express');
const mongoose = require('mongoose');

const app = express();

// Health check
app.get('/health', (req, res) => {
  const health = {
    status: 'UP',
    timestamp: new Date(),
    mongodb: mongoose.connection.readyState === 1 ? 'Connected' : 'Disconnected'
  };
  res.json(health);
});

// ... rest of your app
```

---

### Q41: How would you set up a CI/CD pipeline with Docker?

**Answer:**

**GitHub Actions Example:**

```yaml
# .github/workflows/docker-ci-cd.yml
name: Docker CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Test job
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Build test image
        uses: docker/build-push-action@v4
        with:
          context: .
          target: test
          load: true
          tags: ${{ env.IMAGE_NAME }}:test

      - name: Run tests
        run: docker run --rm ${{ env.IMAGE_NAME }}:test npm test

  # Build and push job
  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix={{branch}}-

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:buildcache
          cache-to: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:buildcache,mode=max

      - name: Security scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  # Deploy job
  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Deploy to production
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PRODUCTION_HOST }}
          username: ${{ secrets.PRODUCTION_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/app
            docker-compose pull
            docker-compose up -d
            docker system prune -f
```

**GitLab CI/CD Example:**

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG

# Test stage
test:
  stage: test
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build --target test -t $IMAGE_TAG-test .
    - docker run --rm $IMAGE_TAG-test npm test
  only:
    - merge_requests
    - main

# Build stage
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build --pull -t $IMAGE_TAG .
    - docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest
    - docker push $IMAGE_TAG
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main

# Security scan
security_scan:
  stage: build
  image: aquasec/trivy:latest
  script:
    - trivy image --severity HIGH,CRITICAL $IMAGE_TAG
  only:
    - main

# Deploy to staging
deploy_staging:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
  script:
    - ssh -o StrictHostKeyChecking=no $STAGING_USER@$STAGING_HOST "
        cd /opt/app &&
        docker-compose pull &&
        docker-compose up -d &&
        docker system prune -f"
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main

# Deploy to production
deploy_production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
  script:
    - ssh -o StrictHostKeyChecking=no $PRODUCTION_USER@$PRODUCTION_HOST "
        cd /opt/app &&
        docker-compose pull &&
        docker-compose up -d --no-deps web &&
        docker system prune -f"
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

**Jenkins Pipeline:**

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        IMAGE_NAME = 'myorg/myapp'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image("${IMAGE_NAME}:${IMAGE_TAG}").inside {
                        sh 'npm test'
                    }
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy image ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Push') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", DOCKER_CREDENTIALS_ID) {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                sshagent(['ssh-credentials']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no user@staging-host '
                            cd /opt/app &&
                            docker-compose pull &&
                            docker-compose up -d &&
                            docker system prune -f
                        '
                    """
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sshagent(['ssh-credentials']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no user@production-host '
                            cd /opt/app &&
                            docker-compose pull &&
                            docker-compose up -d &&
                            docker system prune -f
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

**Multi-Stage Dockerfile for CI/CD:**

```dockerfile
# syntax=docker/dockerfile:1

# Test stage
FROM node:18-alpine AS test
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm test

# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
ENV NODE_ENV=production
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
WORKDIR /app
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs package*.json ./
USER nodejs
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"
CMD ["node", "dist/main.js"]
```

---

### Q42: How would you implement zero-downtime deployment with Docker?

**Answer:**

**Strategy 1: Blue-Green Deployment with Nginx:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app-blue
      - app-green
    networks:
      - app-network

  app-blue:
    image: myapp:${BLUE_VERSION}
    environment:
      - VERSION=blue
    networks:
      - app-network
    deploy:
      replicas: 3

  app-green:
    image: myapp:${GREEN_VERSION}
    environment:
      - VERSION=green
    networks:
      - app-network
    deploy:
      replicas: 3

networks:
  app-network:
```

```nginx
# nginx.conf
upstream app_servers {
    # Switch between blue and green
    server app-blue:3000;
    # server app-green:3000;
}

server {
    listen 80;

    location / {
        proxy_pass http://app_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /health {
        access_log off;
        proxy_pass http://app_servers/health;
    }
}
```

**Deployment Script:**
```bash
#!/bin/bash
# deploy.sh

CURRENT_COLOR=$(cat current_color.txt)
NEW_VERSION=$1

if [ "$CURRENT_COLOR" == "blue" ]; then
    NEW_COLOR="green"
    OLD_COLOR="blue"
else
    NEW_COLOR="blue"
    OLD_COLOR="green"
fi

echo "Deploying $NEW_VERSION to $NEW_COLOR environment..."

# Deploy to inactive environment
export ${NEW_COLOR^^}_VERSION=$NEW_VERSION
docker-compose up -d app-$NEW_COLOR

# Wait for health checks
echo "Waiting for health checks..."
for i in {1..30}; do
    if curl -f http://localhost/health > /dev/null 2>&1; then
        echo "Health check passed"
        break
    fi
    sleep 2
done

# Switch nginx to new environment
echo "Switching traffic to $NEW_COLOR..."
sed -i "s/server app-$OLD_COLOR:3000;/server app-$NEW_COLOR:3000;/g" nginx.conf
docker-compose exec nginx nginx -s reload

# Monitor for errors
sleep 10
echo "Checking for errors..."
ERROR_COUNT=$(docker-compose logs app-$NEW_COLOR | grep -c ERROR || true)

if [ $ERROR_COUNT -gt 0 ]; then
    echo "Errors detected! Rolling back..."
    sed -i "s/server app-$NEW_COLOR:3000;/server app-$OLD_COLOR:3000;/g" nginx.conf
    docker-compose exec nginx nginx -s reload
    exit 1
fi

# Success - remove old environment
echo "Deployment successful! Removing old environment..."
docker-compose stop app-$OLD_COLOR
docker-compose rm -f app-$OLD_COLOR
echo $NEW_COLOR > current_color.txt

echo "Deployment complete!"
```

**Strategy 2: Rolling Update with Docker Swarm:**

```yaml
# docker-stack.yml
version: '3.8'

services:
  web:
    image: myapp:${VERSION}
    deploy:
      replicas: 6
      update_config:
        parallelism: 2
        delay: 10s
        failure_action: rollback
        monitor: 60s
        max_failure_ratio: 0.3
        order: start-first
      rollback_config:
        parallelism: 2
        delay: 5s
        order: start-first
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 40s
    networks:
      - app-network
    ports:
      - "80:3000"

networks:
  app-network:
    driver: overlay
```

**Deploy Script:**
```bash
#!/bin/bash
# swarm-deploy.sh

NEW_VERSION=$1

echo "Deploying version $NEW_VERSION..."

# Update service
VERSION=$NEW_VERSION docker stack deploy -c docker-stack.yml myapp

# Monitor deployment
echo "Monitoring deployment..."
watch -n 2 'docker service ps myapp_web --filter "desired-state=running" --format "table {{.Name}}\t{{.Image}}\t{{.CurrentState}}"'

# Check if deployment succeeded
if docker service ps myapp_web | grep -q "Shutdown Failed"; then
    echo "Deployment failed! Rolling back..."
    docker service rollback myapp_web
    exit 1
fi

echo "Deployment successful!"
```

**Strategy 3: Using Kubernetes-like Deployment:**

```yaml
# docker-compose.yml with health checks
version: '3.8'

services:
  app:
    image: myapp:${VERSION}
    deploy:
      replicas: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s
    environment:
      - PORT=3000
    networks:
      - app-network

  loadbalancer:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx-lb.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    networks:
      - app-network

networks:
  app-network:
```

**Deployment with Health Checks:**
```bash
#!/bin/bash
# rolling-deploy.sh

NEW_VERSION=$1
REPLICAS=3

echo "Starting rolling deployment to version $NEW_VERSION..."

# Scale up with new version
export VERSION=$NEW_VERSION
docker-compose up -d --scale app=$((REPLICAS * 2)) --no-recreate

# Wait for new containers to be healthy
echo "Waiting for new containers to be healthy..."
for i in {1..60}; do
    HEALTHY=$(docker-compose ps app | grep "healthy" | wc -l)
    if [ $HEALTHY -ge $REPLICAS ]; then
        echo "New containers are healthy"
        break
    fi
    echo "Waiting... ($HEALTHY/$REPLICAS healthy)"
    sleep 5
done

# Remove old containers
echo "Removing old containers..."
OLD_CONTAINERS=$(docker-compose ps -q app | head -n $REPLICAS)
for container in $OLD_CONTAINERS; do
    docker stop $container
    docker rm $container
    sleep 5  # Gradual removal
done

# Scale back to normal
docker-compose up -d --scale app=$REPLICAS

echo "Deployment complete!"
```

**Strategy 4: HAProxy for Advanced Load Balancing:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  haproxy:
    image: haproxy:latest
    ports:
      - "80:80"
      - "8404:8404"  # Stats page
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
    networks:
      - app-network

  app:
    image: myapp:${VERSION}
    deploy:
      replicas: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
    networks:
      - app-network

networks:
  app-network:
```

```haproxy
# haproxy.cfg
global
    maxconn 256

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend http-in
    bind *:80
    default_backend servers

backend servers
    balance roundrobin
    option httpchk GET /health
    http-check expect status 200
    server-template app 1-10 app:3000 check inter 2s fall 3 rise 2

listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 5s
```

This comprehensive documentation covers Docker from basics to advanced topics with practical examples and real-world scenarios. Each section includes code examples, best practices, and detailed explanations to help with both learning and interview preparation.