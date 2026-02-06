# Docker Cheatsheet

## Quick Reference Guide for Platform Engineers

### Docker Basics

#### Images
```bash
# List images
docker images                              # List all local images
docker images -a                           # Include intermediate images
docker images --filter "dangling=true"     # List unused images

# Pull images
docker pull ubuntu:22.04                   # Pull specific tag
docker pull nginx:latest                   # Pull latest version
docker pull myregistry.com/myimage:v1      # Pull from private registry

# Build images
docker build -t myapp:v1 .                 # Build with tag
docker build -t myapp:v1 -f Dockerfile.prod . # Use specific Dockerfile
docker build --no-cache -t myapp:v1 .      # Build without cache
docker build --build-arg VERSION=1.0 .     # Pass build arguments

# Image management
docker tag myapp:v1 myapp:latest           # Tag image
docker rmi image_id                        # Remove image
docker rmi $(docker images -q)             # Remove all images
docker image prune                         # Remove unused images
docker image prune -a                      # Remove all unused images
docker image inspect nginx                 # Detailed image info

# Save and load images
docker save -o myapp.tar myapp:v1          # Export image to tar
docker load -i myapp.tar                   # Import image from tar
docker export container_id > backup.tar    # Export container filesystem
docker import backup.tar myapp:v1          # Import filesystem as image

# Image history and layers
docker history myapp:v1                    # Show image layers
docker history --no-trunc myapp:v1         # Full layer details
```

#### Containers
```bash
# Run containers
docker run nginx                           # Run container (foreground)
docker run -d nginx                        # Run in detached mode (background)
docker run -it ubuntu bash                 # Interactive with terminal
docker run --name mycontainer nginx        # Run with custom name
docker run -p 8080:80 nginx               # Port mapping (host:container)
docker run -p 80-90:80-90 nginx           # Map port range
docker run -P nginx                        # Publish all exposed ports randomly
docker run --rm nginx                      # Auto-remove when stopped
docker run -v /host/path:/container/path nginx  # Volume mount
docker run -e "ENV_VAR=value" nginx       # Set environment variable
docker run --env-file .env nginx          # Load env vars from file
docker run --restart unless-stopped nginx  # Auto-restart policy
docker run --memory="512m" nginx          # Memory limit
docker run --cpus="1.5" nginx             # CPU limit

# Container lifecycle
docker start container_id                  # Start stopped container
docker stop container_id                   # Graceful stop (10s timeout)
docker stop -t 30 container_id            # Stop with 30s timeout
docker restart container_id                # Restart container
docker kill container_id                   # Force stop immediately
docker pause container_id                  # Pause container
docker unpause container_id                # Unpause container
docker rm container_id                     # Remove stopped container
docker rm -f container_id                  # Force remove running container
docker container prune                     # Remove all stopped containers

# List containers
docker ps                                  # List running containers
docker ps -a                              # List all containers
docker ps -q                              # List container IDs only
docker ps --filter "status=exited"        # Filter by status
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}" # Custom format

# Container inspection
docker inspect container_id                # Full container details (JSON)
docker logs container_id                   # View container logs
docker logs -f container_id               # Follow log output
docker logs --tail 100 container_id       # Last 100 lines
docker logs --since 30m container_id      # Logs from last 30 minutes
docker stats                              # Live resource usage stats
docker stats --no-stream                  # One-time stats snapshot
docker top container_id                   # Running processes in container
docker port container_id                  # Port mappings
docker diff container_id                  # Changes to filesystem

# Execute commands in running container
docker exec container_id ls /app          # Run command
docker exec -it container_id bash         # Interactive shell
docker exec -u root container_id whoami   # Run as specific user
docker attach container_id                 # Attach to running container

# Copy files
docker cp file.txt container_id:/path     # Copy to container
docker cp container_id:/path/file.txt .   # Copy from container
```

#### Volumes
```bash
# Create and manage volumes
docker volume create myvolume              # Create named volume
docker volume ls                          # List volumes
docker volume inspect myvolume            # Volume details
docker volume rm myvolume                 # Remove volume
docker volume prune                       # Remove unused volumes

# Use volumes
docker run -v myvolume:/data nginx        # Mount named volume
docker run -v /host/path:/data nginx      # Bind mount
docker run -v /data nginx                 # Anonymous volume
docker run --mount type=volume,source=myvolume,target=/data nginx  # Mount syntax
docker run --mount type=bind,source=/host,target=/data,readonly nginx # Read-only bind

# Volume backup and restore
docker run --rm -v myvolume:/data -v $(pwd):/backup ubuntu tar czf /backup/backup.tar.gz /data
docker run --rm -v myvolume:/data -v $(pwd):/backup ubuntu tar xzf /backup/backup.tar.gz -C /
```

#### Networks
```bash
# List networks
docker network ls                          # List all networks
docker network inspect bridge             # Network details

# Create networks
docker network create mynetwork           # Create bridge network
docker network create --driver bridge mynetwork
docker network create --subnet=172.18.0.0/16 mynetwork  # Custom subnet
docker network create --driver overlay mynetwork        # Overlay for swarm

# Connect containers to networks
docker network connect mynetwork container_id   # Connect to network
docker network disconnect mynetwork container_id # Disconnect from network
docker run --network mynetwork nginx      # Run container on network
docker run --network host nginx           # Use host networking
docker run --network none nginx           # No networking

# Network management
docker network rm mynetwork               # Remove network
docker network prune                      # Remove unused networks
```

### Dockerfile Best Practices

```dockerfile
# Use specific base image versions
FROM node:18.17-alpine3.18
# NOT: FROM node:latest

# Use multi-stage builds
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]

# Minimize layers - combine RUN commands
RUN apt-get update && apt-get install -y \
    curl \
    git \
    vim \
    && rm -rf /var/lib/apt/lists/*

# Order matters - place frequently changing instructions last
COPY package*.json ./                      # Dependencies change less often
RUN npm install
COPY . .                                   # Source code changes frequently

# Use .dockerignore
# Create .dockerignore file with:
# node_modules
# .git
# *.md
# .env

# Don't run as root
RUN addgroup -g 1001 appgroup && \
    adduser -D -u 1001 -G appgroup appuser
USER appuser

# Use COPY instead of ADD (unless you need tar extraction)
COPY app.js /app/
# NOT: ADD app.js /app/

# Specify WORKDIR
WORKDIR /app

# Use exec form for CMD and ENTRYPOINT
CMD ["python", "app.py"]
# NOT: CMD python app.py

# Combine LABEL metadata
LABEL maintainer="team@example.com" \
      version="1.0" \
      description="My application"

# Use ARG for build-time variables
ARG VERSION=1.0
ARG BUILD_DATE
RUN echo "Building version ${VERSION}"

# Use ENV for runtime variables
ENV NODE_ENV=production \
    PORT=3000

# Expose ports (documentation only)
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

# Use specific ENTRYPOINT and CMD
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["start-server"]
```

### Multi-Stage Builds

```dockerfile
# Example 1: Node.js application
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -g 1001 nodejs && \
    adduser -D -u 1001 -G nodejs nextjs
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER nextjs
EXPOSE 3000
CMD ["node", "dist/index.js"]

# Example 2: Go application
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

FROM alpine:3.18
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]

# Example 3: Python application with build dependencies
FROM python:3.11-slim AS builder
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /app/wheels -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /app/wheels /wheels
RUN pip install --no-cache /wheels/*
COPY . .
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser
CMD ["python", "app.py"]
```

### Docker Compose Essentials

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Web application
  web:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - VERSION=1.0
    image: myapp:latest
    container_name: web-app
    restart: unless-stopped
    ports:
      - "8080:80"
      - "8443:443"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    env_file:
      - .env
    volumes:
      - ./app:/app
      - uploads:/app/uploads
      - /etc/localtime:/etc/localtime:ro
    networks:
      - frontend
      - backend
    depends_on:
      - db
      - redis
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M

  # Database
  db:
    image: postgres:15-alpine
    container_name: postgres-db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
      - POSTGRES_DB=mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - backend
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Cache
  redis:
    image: redis:7-alpine
    container_name: redis-cache
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - backend
    ports:
      - "6379:6379"

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
    networks:
      - frontend
    depends_on:
      - web

volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local
  uploads:
    driver: local

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

#### Docker Compose Commands
```bash
# Basic operations
docker-compose up                          # Start all services
docker-compose up -d                       # Start in detached mode
docker-compose up --build                  # Rebuild images before start
docker-compose up --force-recreate         # Recreate containers
docker-compose down                        # Stop and remove containers
docker-compose down -v                     # Also remove volumes
docker-compose down --rmi all             # Also remove images

# Service management
docker-compose start                       # Start existing containers
docker-compose stop                        # Stop running containers
docker-compose restart                     # Restart containers
docker-compose pause                       # Pause services
docker-compose unpause                     # Unpause services

# Specific services
docker-compose up -d web db               # Start specific services
docker-compose stop web                   # Stop specific service
docker-compose restart nginx              # Restart specific service

# Logs
docker-compose logs                        # View logs from all services
docker-compose logs -f                     # Follow log output
docker-compose logs -f web                 # Follow logs for specific service
docker-compose logs --tail=100 web        # Last 100 lines

# Execute commands
docker-compose exec web bash              # Execute in running service
docker-compose exec -T web ls             # No TTY allocation
docker-compose run web python manage.py migrate  # Run one-off command
docker-compose run --rm web npm test      # Run and auto-remove

# Status and info
docker-compose ps                          # List containers
docker-compose ps -a                       # Include stopped containers
docker-compose top                         # Display running processes
docker-compose images                      # List images
docker-compose config                      # Validate and view config
docker-compose config --services          # List services
docker-compose port web 80                # Show public port for service port

# Scaling (for non-swarm)
docker-compose up -d --scale web=3        # Scale web service to 3 instances

# Build
docker-compose build                       # Build all services
docker-compose build --no-cache web       # Build without cache
docker-compose build --pull               # Pull newer versions of base images
```

### Image Optimization Tips

```dockerfile
# 1. Use minimal base images
FROM alpine:3.18              # ~5 MB
FROM busybox:1.36            # ~2 MB
FROM scratch                  # 0 MB (for static binaries)
FROM gcr.io/distroless/base  # Minimal Google image

# 2. Multi-stage builds (reduce final image size)
FROM node:18 AS builder
# ... build steps
FROM node:18-alpine
COPY --from=builder /app/dist ./dist

# 3. Combine RUN commands to reduce layers
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 4. Remove unnecessary files
RUN npm install && \
    npm cache clean --force && \
    rm -rf /tmp/*

# 5. Use .dockerignore
# Add to .dockerignore:
**/.git
**/node_modules
**/npm-debug.log
**/.env
**/README.md
**/docker-compose*.yml

# 6. Don't install recommended packages (Debian/Ubuntu)
RUN apt-get install -y --no-install-recommends package

# 7. Order instructions by change frequency
COPY package.json package-lock.json ./  # Changes rarely
RUN npm ci --only=production
COPY . .                                # Changes frequently

# 8. Use specific package versions
RUN apt-get install -y nginx=1.18.0-1

# 9. Leverage build cache
COPY package*.json ./
RUN npm install
COPY . .              # Source changes won't invalidate npm install cache

# 10. Analyze image size
docker history myapp:v1                 # See layer sizes
docker image inspect myapp:v1           # Image metadata
```

### Security Best Practices

```dockerfile
# 1. Never run as root
RUN groupadd -r appgroup && useradd -r -g appgroup appuser
USER appuser

# 2. Use trusted base images
FROM node:18-alpine
# Verify image signature
docker trust inspect --pretty node:18-alpine

# 3. Scan images for vulnerabilities
docker scan myapp:v1
# Use Trivy, Snyk, or other scanners
trivy image myapp:v1

# 4. Don't store secrets in images
# WRONG:
ENV DB_PASSWORD=mysecret
# RIGHT:
# Pass at runtime via environment variables or secrets management

# 5. Use read-only filesystem where possible
docker run --read-only --tmpfs /tmp myapp:v1

# 6. Drop unnecessary capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp:v1

# 7. Limit resources
docker run --memory="512m" --cpus="1.0" --pids-limit=100 myapp:v1

# 8. Use security scanning in CI/CD
# Example GitHub Actions
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:${{ github.sha }}'
    format: 'sarif'

# 9. Update base images regularly
FROM node:18.17-alpine3.18  # Pin version
# Schedule regular rebuilds to get security patches

# 10. Use Docker Content Trust
export DOCKER_CONTENT_TRUST=1
docker pull nginx:latest    # Verifies signature
```

#### Security Commands
```bash
# Scan for vulnerabilities
docker scan nginx:latest
docker scan --severity=high myapp:v1

# Run with security options
docker run --security-opt=no-new-privileges myapp:v1
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp:v1
docker run --read-only --tmpfs /tmp myapp:v1
docker run --pids-limit 100 myapp:v1

# Use secrets (Docker Swarm)
docker secret create my_secret ./secret.txt
docker service create --secret my_secret nginx

# AppArmor profile
docker run --security-opt apparmor=docker-default nginx

# SELinux labels
docker run --security-opt label=level:s0:c100,c200 nginx

# Check for Docker daemon issues
docker system info | grep -i security
docker info --format '{{.SecurityOptions}}'
```

### Common Troubleshooting

```bash
# Container won't start
docker logs container_id                   # Check logs
docker inspect container_id | grep -i error
docker events --filter 'container=mycontainer'

# Port already in use
docker ps -a | grep 8080                  # Find container using port
lsof -i :8080                            # Find process using port
docker run -p 8081:80 nginx              # Use different port

# Cannot connect to containers
docker inspect container_id | grep IPAddress  # Get container IP
docker network inspect bridge            # Check network config
docker exec container_id ping other_container # Test connectivity
docker run --network=host myapp          # Use host networking (testing)

# Out of disk space
docker system df                          # Show disk usage
docker system prune                       # Remove unused data
docker system prune -a                    # Remove all unused data
docker volume prune                       # Remove unused volumes
docker image prune -a --filter "until=24h" # Remove old images

# Container performance issues
docker stats                              # Check resource usage
docker top container_id                   # Check processes
docker exec container_id top              # Interactive top in container
docker update --memory="1g" container_id  # Update resource limits

# Image build fails
docker build --no-cache -t myapp .        # Build without cache
docker build --progress=plain .           # Show all build output
docker builder prune                      # Clear build cache

# Network issues
docker network ls                         # List networks
docker network inspect bridge             # Inspect network
docker exec container_id ping 8.8.8.8    # Test internet connectivity
docker exec container_id cat /etc/resolv.conf  # Check DNS config
docker run --dns 8.8.8.8 myapp           # Override DNS

# Permission issues with volumes
docker run -v /host:/container:rw myapp  # Ensure read-write
docker exec -u root container_id chown -R appuser /data  # Fix ownership
ls -la /host/path                        # Check host permissions

# Container exits immediately
docker run -it myapp bash                 # Run interactively to debug
docker logs container_id                  # Check logs
docker inspect container_id | grep -i cmd # Check command
docker run --entrypoint=/bin/sh myapp    # Override entrypoint

# Clean everything (DANGER!)
docker stop $(docker ps -aq)              # Stop all containers
docker rm $(docker ps -aq)                # Remove all containers
docker rmi $(docker images -q)            # Remove all images
docker volume rm $(docker volume ls -q)   # Remove all volumes
docker network prune                      # Remove unused networks
docker system prune -a --volumes         # Nuclear option
```

### Common Scenarios and Solutions

#### Scenario 1: Database Container with Persistent Data
```bash
# Create named volume for persistence
docker volume create postgres-data

# Run PostgreSQL with volume
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_DB=mydb \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15-alpine

# Backup database
docker exec postgres pg_dump -U myuser mydb > backup.sql

# Restore database
docker exec -i postgres psql -U myuser mydb < backup.sql

# Access database
docker exec -it postgres psql -U myuser -d mydb
```

#### Scenario 2: Development Environment with Live Reload
```bash
# Mount source code as volume for live reload
docker run -d \
  --name dev-server \
  -v $(pwd):/app \
  -v /app/node_modules \
  -p 3000:3000 \
  -e NODE_ENV=development \
  node:18-alpine \
  sh -c "cd /app && npm install && npm run dev"

# With docker-compose
services:
  web:
    build: .
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    command: npm run dev
```

#### Scenario 3: Multi-Container Application
```yaml
# docker-compose.yml for full stack app
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
    environment:
      - REACT_APP_API_URL=http://localhost:8000

  backend:
    build: ./backend
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379

  db:
    image: postgres:15-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=secret

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:
```

#### Scenario 4: Running Tests in Container
```bash
# Run tests in isolated container
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  node:18-alpine \
  npm test

# Run with docker-compose
docker-compose run --rm web npm test

# Run specific test file
docker-compose run --rm web npm test -- users.test.js

# Interactive test mode
docker-compose run --rm web npm test -- --watch
```

#### Scenario 5: CI/CD Integration
```yaml
# .github/workflows/docker.yml
name: Docker Build and Push

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: myapp:${{ github.sha }},myapp:latest
          cache-from: type=registry,ref=myapp:latest
          cache-to: type=inline
      
      - name: Run Trivy scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
```

#### Scenario 6: Debugging Running Container
```bash
# Get shell access
docker exec -it container_id /bin/bash
docker exec -it container_id /bin/sh  # For alpine

# Check logs with timestamp
docker logs -t container_id

# Follow logs from multiple containers
docker-compose logs -f web db

# Inspect container details
docker inspect container_id | jq '.[0].State'
docker inspect container_id | jq '.[0].NetworkSettings.IPAddress'

# Check environment variables
docker exec container_id env

# Check processes
docker top container_id
docker exec container_id ps aux

# Monitor resource usage
docker stats container_id --no-stream

# Check mounted volumes
docker inspect -f '{{ .Mounts }}' container_id
```

#### Scenario 7: Private Registry
```bash
# Login to private registry
docker login myregistry.com
docker login myregistry.com -u username -p password

# Tag for private registry
docker tag myapp:v1 myregistry.com/myapp:v1

# Push to private registry
docker push myregistry.com/myapp:v1

# Pull from private registry
docker pull myregistry.com/myapp:v1

# Run local registry
docker run -d -p 5000:5000 --name registry registry:2

# Push to local registry
docker tag myapp:v1 localhost:5000/myapp:v1
docker push localhost:5000/myapp:v1
```

### System Management

```bash
# System information
docker version                            # Docker version info
docker info                              # System-wide information
docker system df                          # Disk usage
docker system df -v                       # Verbose disk usage
docker system events                      # Real-time events
docker system events --since 1h          # Events from last hour

# Cleanup commands
docker system prune                       # Remove unused data
docker system prune -a                    # Remove all unused data
docker system prune --volumes            # Include volumes
docker container prune                    # Remove stopped containers
docker image prune                        # Remove dangling images
docker image prune -a                     # Remove all unused images
docker volume prune                       # Remove unused volumes
docker network prune                      # Remove unused networks
docker builder prune                      # Remove build cache

# Resource monitoring
docker stats                              # All containers
docker stats container1 container2        # Specific containers
docker stats --no-stream                  # One-time snapshot
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Update container resources
docker update --memory="1g" container_id
docker update --cpus="2" container_id
docker update --restart=always container_id
```

### Docker Registry Operations

```bash
# Docker Hub
docker login                              # Login to Docker Hub
docker logout                             # Logout
docker search nginx                       # Search for images
docker pull nginx:alpine                  # Pull image
docker push username/myapp:v1            # Push image

# Private registry operations
docker login registry.example.com
docker tag myapp:v1 registry.example.com/myapp:v1
docker push registry.example.com/myapp:v1

# Inspect remote image without pulling
docker manifest inspect nginx:alpine

# Export/Import images (for air-gapped environments)
docker save -o images.tar image1:tag image2:tag
docker load -i images.tar
```

### Advanced Docker Commands

```bash
# Build with BuildKit (faster builds)
DOCKER_BUILDKIT=1 docker build -t myapp .
docker buildx build --platform linux/amd64,linux/arm64 -t myapp .

# Container commit (create image from container)
docker commit container_id myapp:v2
docker commit -m "Added nginx" container_id myapp:v2

# Import/Export
docker export container_id > backup.tar
cat backup.tar | docker import - myapp:imported

# Copy between containers
docker cp container1:/data/file.txt container2:/data/

# Wait for container to finish
docker wait container_id

# Rename container
docker rename old_name new_name

# Pause all processes in container
docker pause container_id
docker unpause container_id

# Create container without starting
docker create --name mycontainer nginx
docker start mycontainer

# Attach to running container
docker attach container_id  # Ctrl+P, Ctrl+Q to detach without stopping
```

## Pro Tips for Platform Engineers

1. **Use BuildKit for Faster Builds**
   ```bash
   export DOCKER_BUILDKIT=1
   # Or add to /etc/docker/daemon.json:
   { "features": { "buildkit": true } }
   ```

2. **Layer Caching Strategy**
   - Put least frequently changing instructions first
   - Copy dependency files before source code
   - Use `.dockerignore` aggressively

3. **Health Checks are Critical**
   ```dockerfile
   HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
     CMD curl -f http://localhost/ || exit 1
   ```

4. **Use docker-compose for Local Development**
   - One command to start entire stack
   - Consistent environment across team
   - Easy to share and version control

5. **Standardize Base Images**
   - Create organization base images
   - Include common tools and configurations
   - Regular security updates

6. **Resource Limits are Essential**
   ```bash
   docker run -m 512m --cpus="1.5" --pids-limit=100 myapp
   ```

7. **Use Named Volumes Over Bind Mounts**
   - Better performance (especially on Mac/Windows)
   - Docker manages lifecycle
   - Easier backups

8. **Network Isolation**
   ```bash
   # Create isolated network
   docker network create --internal backend
   # External-facing services on different network
   ```

9. **Automated Image Scanning**
   - Integrate Trivy, Snyk, or Clair in CI/CD
   - Fail builds on high/critical vulnerabilities
   - Regular rescans of deployed images

10. **Use Multi-Stage Builds Always**
    - Smaller final images
    - Separate build and runtime dependencies
    - Better security (fewer attack vectors)

11. **Container Logging Best Practices**
    ```bash
    # Configure logging driver
    docker run --log-driver=json-file --log-opt max-size=10m --log-opt max-file=3 myapp
    
    # Use centralized logging
    docker run --log-driver=syslog --log-opt syslog-address=tcp://logserver:514 myapp
    ```

12. **Image Versioning Strategy**
    - Always tag with version number
    - Use semantic versioning
    - Tag with git commit SHA in CI/CD
    - Avoid relying on `latest` tag

13. **Docker Contexts for Multiple Environments**
    ```bash
    docker context create remote-server --docker "host=ssh://user@remote"
    docker context use remote-server
    docker ps  # Now runs on remote server
    ```

14. **Useful Aliases**
    ```bash
    alias dps='docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'
    alias dlog='docker logs -f'
    alias dex='docker exec -it'
    alias dclean='docker system prune -af --volumes'
    ```

15. **Docker Secrets for Sensitive Data**
    ```yaml
    # In docker-compose.yml
    secrets:
      db_password:
        file: ./secrets/db_password.txt
    services:
      app:
        secrets:
          - db_password
    ```

16. **Use .dockerignore**
    ```
    # .dockerignore
    .git
    .gitignore
    node_modules
    npm-debug.log
    Dockerfile
    .dockerignore
    docker-compose*.yml
    README.md
    .env
    .vscode
    .idea
    ```

17. **Debugging Production Images**
    ```bash
    # Add debugging tools temporarily
    docker run --rm -it --entrypoint /bin/sh myapp:prod
    
    # Or use debug image variant
    docker run --rm -it myapp:debug
    ```

18. **Docker Events for Monitoring**
    ```bash
    # Monitor container events
    docker events --filter 'type=container' --format '{{.Status}}: {{.Actor.Attributes.name}}'
    
    # Monitor specific container
    docker events --filter 'container=myapp' --since '1h'
    ```

19. **Performance Optimization**
    ```bash
    # Use tmpfs for temporary data
    docker run --tmpfs /tmp:rw,noexec,nosuid,size=100m myapp
    
    # Adjust storage driver (in /etc/docker/daemon.json)
    { "storage-driver": "overlay2" }
    ```

20. **Container Restart Policies**
    ```bash
    # Production: always restart
    docker run --restart=always myapp
    
    # Development: restart unless stopped manually
    docker run --restart=unless-stopped myapp
    
    # Restart on failure only (max 3 attempts)
    docker run --restart=on-failure:3 myapp
    ```

## Quick Troubleshooting Checklist

- [ ] Check logs: `docker logs -f container_id`
- [ ] Verify container is running: `docker ps -a`
- [ ] Check resource usage: `docker stats`
- [ ] Inspect container config: `docker inspect container_id`
- [ ] Test network connectivity: `docker exec container_id ping other_service`
- [ ] Verify volume mounts: `docker inspect -f '{{ .Mounts }}' container_id`
- [ ] Check environment variables: `docker exec container_id env`
- [ ] Review health check: `docker inspect --format='{{json .State.Health}}' container_id`
- [ ] Examine processes: `docker top container_id`
- [ ] Check disk space: `docker system df`

## Useful Resources

- [Official Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Docker Security](https://docs.docker.com/engine/security/)
- [Play with Docker](https://labs.play-with-docker.com/) - Free online Docker playground

---

**Remember**: Always test in development before deploying to production, use version control for Dockerfiles and compose files, and implement proper monitoring and logging for production containers.
