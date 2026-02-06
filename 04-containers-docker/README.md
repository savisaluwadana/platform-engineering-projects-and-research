# Containers and Docker

## Theory

### Core Concepts

#### Container Fundamentals: Understanding Containerization
Containers are lightweight, standalone packages that include everything needed to run an application:

- **Namespaces**: Linux kernel feature that isolates system resources for processes. Different namespace types provide isolation for:
  - **PID**: Process IDs, giving each container its own process tree
  - **Network**: Network interfaces, IP addresses, routing tables
  - **Mount**: Filesystem mount points
  - **UTS**: Hostname and domain name
  - **IPC**: Inter-process communication resources
  - **User**: User and group IDs
- **Control Groups (cgroups)**: Limit and monitor resource usage (CPU, memory, I/O, network) for containerized processes, preventing resource starvation.
- **Union File Systems**: Overlay filesystems (like OverlayFS, AUFS) that layer file system changes, enabling efficient image storage and fast container creation.
- **Container vs VM**: Containers share the host OS kernel (lightweight, fast startup) while VMs include a full OS (heavier, stronger isolation). Containers are ideal for microservices; VMs for complete isolation.

Understanding these fundamentals helps optimize container performance and troubleshoot runtime issues.

#### Docker Architecture: The Container Platform
Docker provides a complete platform for building, shipping, and running containers:

- **Docker Engine**: Core component consisting of:
  - **dockerd (Docker Daemon)**: Background service managing containers, images, networks, and volumes
  - **containerd**: Industry-standard container runtime handling container lifecycle
  - **runc**: Low-level container runtime implementing OCI specification
- **Docker CLI**: Command-line interface for interacting with Docker daemon. Commands follow a consistent pattern: `docker <object> <command>`.
- **Docker Desktop**: Desktop application for Mac and Windows providing Docker Engine, CLI tools, and GUI management.
- **Registry**: Storage and distribution system for Docker images:
  - **Docker Hub**: Public registry with millions of images
  - **Private Registries**: Self-hosted or cloud-based registries for proprietary images
  - **Image Layers**: Images stored as layers with deduplication for efficiency

Docker's architecture enables consistent application packaging across development, testing, and production.

#### Docker Images: Portable Application Packages
Images are immutable templates containing application code, runtime, libraries, and dependencies:

- **Dockerfile**: Text file with instructions for building images. Each instruction creates a layer.
  - **FROM**: Specifies base image
  - **RUN**: Executes commands during build
  - **COPY/ADD**: Adds files from host to image
  - **WORKDIR**: Sets working directory
  - **ENV**: Sets environment variables
  - **EXPOSE**: Documents ports the container listens on
  - **CMD/ENTRYPOINT**: Specifies default command to run
- **Image Layers**: Each Dockerfile instruction creates a read-only layer. Layers are cached and reused across images for efficiency.
- **Layer Caching**: Docker caches layers during builds. Order instructions from least to most frequently changing for optimal cache utilization.
- **Multi-stage Builds**: Use multiple FROM statements to create lean production images while keeping build dependencies separate. Essential for compiled languages.
- **Image Tagging**: Version images with tags (e.g., `nginx:1.21-alpine`). Use semantic versioning and avoid relying solely on `latest`.
- **Image Inspection**: Use `docker inspect` and `docker history` to examine image contents and layers.

Optimizing images reduces storage costs, speeds up deployments, and improves security by minimizing attack surface.

#### Container Lifecycle: From Creation to Deletion
Containers have a defined lifecycle managed through Docker commands:

- **Creation**: `docker create` or `docker run` creates a container from an image
- **Starting**: Container process begins execution
- **Running**: Container is executing its primary process
- **Pausing**: Temporarily freeze container process using cgroups freezer
- **Stopping**: Gracefully stop container (SIGTERM, then SIGKILL after timeout)
- **Restarting**: Stop and start container
- **Removing**: Delete stopped container with `docker rm`
- **Container State**: Check with `docker ps` (running) and `docker ps -a` (all)
- **Logs**: Access container stdout/stderr with `docker logs`
- **Exec**: Run additional commands in running containers with `docker exec`
- **Resource Limits**: Set memory, CPU, and I/O constraints at runtime

Understanding the lifecycle helps manage containers effectively and troubleshoot issues.

#### Docker Networking: Connecting Containers
Docker provides multiple networking modes for different use cases:

- **Bridge Network**: Default network creating virtual Ethernet bridge. Containers on same bridge can communicate; NAT enables external access.
- **Host Network**: Container shares host's network stack, eliminating network isolation for maximum performance.
- **Overlay Network**: Multi-host networking for Docker Swarm and Kubernetes, enabling containers on different hosts to communicate securely.
- **Macvlan Network**: Assigns MAC addresses to containers, making them appear as physical devices on the network.
- **None Network**: Disables networking for complete isolation.
- **Custom Networks**: Create user-defined networks with `docker network create` for better isolation and automatic DNS resolution.
- **Container Communication**: Containers on same network communicate using container names as DNS hostnames.
- **Port Mapping**: Publish container ports to host with `-p` flag (e.g., `-p 8080:80` maps container port 80 to host port 8080).
- **Network Inspection**: Use `docker network inspect` to troubleshoot connectivity.

Proper networking configuration is crucial for microservices architectures and security.

#### Docker Storage: Persisting Data
Containers are ephemeral; storage options persist data beyond container lifecycle:

- **Volumes**: Docker-managed storage on the host, stored in `/var/lib/docker/volumes/`. Best for production use.
  - Managed by Docker (backup, migration easier)
  - Can be shared between containers
  - Work on both Linux and Windows
  - Can use volume drivers for remote storage
- **Bind Mounts**: Mount host directories into containers. Good for development when you need to share code.
  - Direct access to host filesystem
  - File changes reflected immediately
  - Security implications if container is compromised
- **tmpfs Mounts**: Store data in host memory, never written to disk. Good for sensitive temporary data.
- **Volume Drivers**: Enable storing volumes on remote hosts or cloud storage (NFS, AWS EBS, etc.).
- **Storage Best Practices**:
  - Use volumes for persistent data in production
  - Use bind mounts for development
  - Never store important data only in container layer
  - Regularly backup volumes

Storage strategy impacts performance, data persistence, and disaster recovery capabilities.

#### Docker Compose: Multi-Container Applications
Docker Compose simplifies managing multi-container applications with declarative YAML configuration:

- **docker-compose.yml**: Define services, networks, and volumes in a single file
- **Services**: Each service runs one image and can have multiple container instances
- **Dependencies**: Use `depends_on` to define startup order
- **Environment Variables**: Configure containers with environment variables, .env files, or variable substitution
- **Networking**: Compose creates a default network for services to communicate
- **Volumes**: Define named volumes for data persistence
- **Scaling**: Run multiple instances of a service with `--scale`
- **Commands**: 
  - `docker-compose up`: Start all services
  - `docker-compose down`: Stop and remove containers, networks
  - `docker-compose ps`: List containers
  - `docker-compose logs`: View logs from all services
  - `docker-compose exec`: Execute commands in running service

Compose is essential for local development and testing of microservices.

#### Container Security: Protecting Your Deployments
Security must be built into container workflows:

- **Image Scanning**: Detect vulnerabilities in images using tools like Trivy, Clair, or Snyk. Scan during CI/CD pipeline.
- **Minimal Base Images**: Use Alpine or distroless images to reduce attack surface. Fewer packages mean fewer vulnerabilities.
- **Secrets Management**: Never hardcode secrets in images. Use Docker secrets, environment variables from secure sources, or tools like HashiCorp Vault.
- **User Privileges**: Run containers as non-root users. Add `USER` instruction in Dockerfile.
- **Read-only Filesystems**: Mount root filesystem as read-only with `--read-only` flag.
- **Security Scanning**: Implement automated scanning in CI/CD pipelines to catch vulnerabilities early.
- **Resource Limits**: Set CPU and memory limits to prevent DoS attacks.
- **Network Policies**: Limit container network access to only required services.
- **Security Contexts**: Use AppArmor or SELinux for additional mandatory access controls.
- **Image Signing**: Use Docker Content Trust (DCT) to verify image publisher and integrity.
- **Runtime Security**: Monitor container behavior with tools like Falco for anomaly detection.

Security practices reduce risk of breaches and ensure compliance with regulations.

### Key Skills
1. **Build efficient Docker images**: Write optimized Dockerfiles using multi-stage builds, layer caching, and minimal base images. Reduce image size and build time while maintaining functionality.
2. **Manage container lifecycle**: Understand container states, gracefully stop containers, handle restarts, and troubleshoot container issues using logs and inspection tools.
3. **Configure networking and storage**: Set up appropriate network modes for container communication, implement port mapping for external access, and choose correct storage options for data persistence.
4. **Optimize image size and build time**: Order Dockerfile instructions strategically, use .dockerignore, leverage build cache, and employ multi-stage builds to separate build and runtime dependencies.
5. **Implement security best practices**: Scan images for vulnerabilities, run containers as non-root, manage secrets properly, use minimal base images, and implement runtime security monitoring.

## Projects

### Beginner Level

#### Project 1: Multi-Service Application with Docker Compose
**Objective**: Containerize a full-stack application
- Frontend (React/Vue/Angular)
- Backend API (Node.js/Python/Java)
- Database (PostgreSQL/MySQL/MongoDB)
- Redis cache
- Nginx reverse proxy

**Learning Outcomes**: Docker Compose, multi-container apps, networking

#### Project 2: Docker Image Optimization Challenge
**Objective**: Optimize Docker images for size and performance
- Start with a bloated image
- Apply multi-stage builds
- Use Alpine Linux base images
- Minimize layers
- Document size reduction achieved

**Learning Outcomes**: Image optimization, build efficiency, best practices

#### Project 3: Development Environment in Containers
**Objective**: Create reproducible development environments
- Language-specific containers (Python, Node.js, Java, Go)
- Hot-reload for code changes
- Database with seed data
- Development tools (linters, formatters)
- VS Code devcontainer configuration

**Learning Outcomes**: Development workflow, volume mounting, tool integration

### Intermediate Level

#### Project 4: Container Registry and Image Scanner
**Objective**: Build a private container registry with security scanning
- Set up private Docker registry
- Implement image vulnerability scanning (Trivy/Clair)
- Image signing and verification
- Automated scanning pipeline
- Web UI for image management

**Learning Outcomes**: Registry management, security scanning, image signing

#### Project 5: Container Monitoring Stack
**Objective**: Create a comprehensive container monitoring solution
- Metrics collection (cAdvisor, Prometheus)
- Log aggregation (Fluentd, Elasticsearch)
- Visualization (Grafana)
- Alerting rules
- Performance dashboards

**Learning Outcomes**: Monitoring, metrics, logging, observability

#### Project 6: Docker Build Automation System
**Objective**: Automate Docker image builds and deployment
- Watch repositories for changes
- Automated builds with BuildKit
- Multi-architecture builds (amd64, arm64)
- Image tagging strategy
- Push to multiple registries
- Build notifications

**Learning Outcomes**: CI/CD integration, automation, multi-arch builds

### Advanced Level

#### Project 7: Container Orchestration Mini-Platform
**Objective**: Build a simple container orchestration system
- Container scheduling across multiple hosts
- Health checking and auto-restart
- Service discovery
- Load balancing
- Rolling updates
- Resource management

**Learning Outcomes**: Orchestration concepts, distributed systems, scheduling

#### Project 8: Serverless Container Platform
**Objective**: Create a serverless platform using containers
- Function-as-a-Service (FaaS) implementation
- Auto-scaling based on load
- Cold start optimization
- Event-driven triggers
- Multi-language runtime support
- API gateway integration

**Learning Outcomes**: Serverless architecture, auto-scaling, event-driven systems

#### Project 9: Container Security Hardening Framework
**Objective**: Implement comprehensive container security
- Runtime security monitoring (Falco)
- Network policies and segmentation
- Secrets management (Vault integration)
- Compliance scanning (CIS benchmarks)
- Rootless containers
- Image supply chain security (SBOM, SLSA)

**Learning Outcomes**: Container security, compliance, threat detection

## Resources

### Documentation
- [Docker Documentation](https://docs.docker.com/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)

### Books
- "Docker Deep Dive" by Nigel Poulton
- "Docker in Action" by Jeff Nickoloff
- "The Docker Book" by James Turnbull

### Tools
- Docker Desktop
- Docker Compose
- Dive (image layer analysis)
- Trivy (vulnerability scanner)
- Hadolint (Dockerfile linter)

### Certifications
- Docker Certified Associate (DCA)
- Kubernetes certifications (overlap with container knowledge)

### Practice
- [Play with Docker](https://labs.play-with-docker.com/)
- [Docker Samples](https://github.com/dockersamples)
