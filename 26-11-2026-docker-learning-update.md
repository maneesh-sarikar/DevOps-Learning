# Docker Learning Update — 26 November 2026

**Learning Area:** Containerization  
**Technology:** Docker  
**Focus:** Image Creation, Container Communication, Persistent Storage and Multi-Container Orchestration

---

## 1. Learning Objective

Today's learning focused on understanding how individual Docker concepts work together to create a complete containerized application.

Instead of treating Dockerfile, networking, storage and Docker Compose as independent topics, I explored their relationship within an application deployment workflow.

The overall learning sequence:

```text
Application Source Code
        |
        v
    Dockerfile
        |
        v
    Docker Image
        |
        v
   Docker Container
        |
        +----------------+
        |                |
        v                v
    Networking       Persistent
                     Storage
        |                |
        +--------+-------+
                 |
                 v
           Docker Compose
                 |
                 v
       Multi-Container Application
```

---

## 2. Stage 1 — Converting Application Code into a Container Image

### Concept: Dockerfile and Image Building

**Problem being solved:**

An application developed on one machine may require specific runtime versions, libraries and dependencies to work correctly on another machine.

A Dockerfile addresses this problem by defining the environment and instructions required to package an application into a Docker image.

### Understanding the relationship

| Component | Responsibility |
|---|---|
| Application Code | Contains the actual application logic |
| Dockerfile | Defines the image-building instructions |
| Docker Image | Packages the application and its dependencies |
| Container | Runs an instance of the image |

### Important Dockerfile instructions

| Instruction | Technical purpose |
|---|---|
| `FROM` | Defines the base image |
| `WORKDIR` | Sets the working directory |
| `COPY` | Copies files into the image |
| `ADD` | Copies files with additional extraction capabilities |
| `RUN` | Executes commands during image building |
| `CMD` | Defines the default runtime command |
| `ENTRYPOINT` | Defines the main executable |
| `EXPOSE` | Documents the application's listening port |
| `ENV` | Defines runtime environment variables |
| `ARG` | Defines build-time variables |
| `USER` | Defines the user for subsequent instructions and runtime |

### Example Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV PORT=5000

EXPOSE 5000

CMD ["python", "app.py"]
```

### Important distinctions

**RUN vs CMD**

- RUN executes during image building.
- CMD defines the default command when the container starts.

**CMD vs ENTRYPOINT**

- CMD provides default commands or arguments.
- ENTRYPOINT defines the primary executable.

**ARG vs ENV**

- ARG is intended for build-time configuration.
- ENV defines environment variables available in the resulting image and container.

### Commands

```bash
# Build an image
docker build -t python-app .

# List available images
docker images

# Inspect an image
docker inspect python-app

# Run a container
docker run -d -p 5000:5000 --name python-container python-app

# Check running containers
docker ps

# View application logs
docker logs python-container

# Stop a container
docker stop python-container

# Remove a container
docker rm python-container
```

### Connection to the next stage

Once an application is packaged into an image and executed as a container, the next requirement is communication.

A real-world application rarely operates as a single independent container. Frontend, backend and database components need to communicate.

This introduces **Docker Networking**.

---

## 3. Stage 2 — Establishing Communication Between Containers

### Concept: Docker Networking

**Problem being solved:**

Running multiple containers independently does not automatically provide a complete application communication architecture.

Docker networking provides communication between containers, the host machine and external networks.

### Understanding network drivers

| Network | Purpose |
|---|---|
| Bridge | Connects containers through a virtual network |
| Host | Shares the host's network namespace |
| None | Disables normal external network connectivity |

### Why custom bridge networking matters

A user-defined bridge network provides:

- Network isolation.
- Container-to-container communication.
- Automatic DNS resolution.
- Communication through container or service names.

### Commands

```bash
# List existing networks
docker network ls

# Inspect the default bridge
docker network inspect bridge

# Create a custom network
docker network create app-network

# Run the first container
docker run -d \
  --name app1 \
  --network app-network \
  nginx

# Run another container
docker run -d \
  --name app2 \
  --network app-network \
  nginx

# Inspect the custom network
docker network inspect app-network
```

### Additional network commands

```bash
# Connect a container to a network
docker network connect app-network app1

# Disconnect a container
docker network disconnect app-network app1

# Remove a network
docker network rm app-network

# Remove unused networks
docker network prune
```

### Port Mapping

Port mapping makes an application running inside a container accessible through a port on the host machine.

Syntax:

```bash
docker run -p HOST_PORT:CONTAINER_PORT IMAGE
```

Example:

```bash
docker run -d -p 8080:80 nginx
```

Access:

```text
http://localhost:8080
```

**Important distinction:**

`EXPOSE` documents a container port, whereas `-p` publishes the port to the host.

### Container-to-container communication

Containers connected to the same user-defined bridge network can communicate using their names.

Example:

```bash
docker run -d \
  --name database \
  --network app-network \
  postgres:16
```

A connected application can use:

```text
database:5432
```

Here, `database` acts as the hostname and `5432` is the PostgreSQL container port.

### Connection to the next stage

Networking solves communication, but communication alone does not guarantee application reliability.

For example, a database container may communicate perfectly with a backend container, but what happens when the database container is removed?

If the database records exist only in the container's writable filesystem layer, they can be lost when that container is removed.

This introduces **Docker Storage and Persistent Volumes**.

---

## 4. Stage 3 — Managing Data Beyond Container Lifecycles

### Concept: Docker Storage

**Problem being solved:**

Containers are replaceable runtime instances. Application data should not depend entirely on the lifecycle of an individual container.

Docker provides different storage mechanisms to handle this requirement.

### Understanding the container filesystem

Docker images contain read-only layers.

When a container starts, Docker adds a writable layer above those image layers.

```text
       Running Container
              |
              v
       Writable Layer
              |
              v
        Image Layers
              |
              v
          Base Image
```

Changes made only in the writable layer are associated with that container's lifecycle.

### Three important storage mechanisms

| Storage type | Purpose | Persistence |
|---|---|---|
| Volume | Docker-managed persistent data | Survives container removal |
| Bind Mount | Connects a host directory to a container | Survives container removal |
| tmpfs | Stores temporary data in memory | Temporary |

### Docker Volume commands

```bash
# Create a volume
docker volume create app-data

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect app-data

# Remove a volume
docker volume rm app-data

# Remove unused volumes
docker volume prune
```

### Mounting a volume

```bash
docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=example \
  -v app-data:/var/lib/postgresql/data \
  postgres:16
```

The volume is mounted at the PostgreSQL data directory.

### Bind Mount example

```bash
docker run -d \
  --name nginx-bind \
  -v "$PWD/html:/usr/share/nginx/html" \
  nginx
```

This connects a host directory with the container's web directory.

### tmpfs example

```bash
docker run -d \
  --name temporary-container \
  --tmpfs /app/temp \
  nginx
```

The mounted data is temporary and stored in memory.

### Important operational distinction

- Use volumes for application data that must survive container replacement.
- Use bind mounts when the host and container need access to the same files.
- Use tmpfs for temporary in-memory data.

**Connection to the next stage**

We now have three essential building blocks:

1. Images package application environments.
2. Networks enable communication between application components.
3. Volumes preserve data independently of container lifecycles.

However, configuring each container, network and volume manually becomes difficult as application complexity increases.

This leads to **Docker Compose**.

---

## 5. Stage 4 — Bringing Everything Together with Docker Compose

### Concept: Multi-Container Orchestration

**Problem being solved:**

Consider a three-tier application:

- Frontend — React
- Backend — Python/FastAPI
- Database — PostgreSQL

Without Compose, each component needs individual container configuration and lifecycle management.

Docker Compose allows these components to be declared in a single YAML configuration file.

### Understanding the Compose architecture

```text
             compose.yaml
                  |
        +---------+---------+
        |         |         |
        v         v         v
     Frontend  Backend   Database
        |         |         |
        +---------+---------+
                  |
            Docker Network
                  |
             Data Volume
```

### YAML fundamentals

YAML is a human-readable data serialization format used for configuration.

Important rules:

- Indentation defines structure.
- Spaces should be used instead of tabs.
- Key-value pairs define configuration.
- Hyphens represent list items.

### Example Compose configuration

Create a file named `compose.yaml`.

```yaml
services:

  frontend:
    image: nginx:alpine
    ports:
      - "8080:80"

  backend:
    image: python:3.12-slim
    environment:
      APP_ENV: development

  database:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

This illustrates how services and persistent storage can be defined together.

### Important Compose components

| Component | Responsibility |
|---|---|
| services | Defines application components |
| image | Specifies the container image |
| build | Defines how an image is built |
| ports | Publishes container ports |
| environment | Defines environment variables |
| env_file | Loads environment variables from a file |
| volumes | Configures persistent storage |
| networks | Controls service communication |
| depends_on | Defines service startup dependencies |

### Docker Compose commands

```bash
# Validate Compose configuration
docker compose config

# Start all services
docker compose up

# Start in detached mode
docker compose up -d

# Build and start services
docker compose up --build -d

# List services and their status
docker compose ps

# View logs
docker compose logs

# Follow logs continuously
docker compose logs -f

# View logs of a specific service
docker compose logs backend

# Execute a command inside a running service
docker compose exec backend sh

# Stop services
docker compose stop

# Restart services
docker compose restart

# Stop and remove containers and networks
docker compose down

# Rebuild service images
docker compose build
```

**Important:** `docker compose down -v` also removes associated named volumes and can permanently delete application data.

### How the previous concepts connect to Compose

| Previously learned concept | How Compose uses it |
|---|---|
| Dockerfile | Builds custom service images |
| Docker Image | Provides the runtime environment |
| Networking | Enables communication between services |
| Port Mapping | Exposes selected services to the host |
| Volumes | Maintains persistent application data |
| Environment Variables | Configures services |
| Containers | Execute the application components |

Compose therefore acts as a unified configuration and lifecycle management layer for the concepts studied earlier.

---

## 6. Practical Command Reference

The following commands form the operational sequence for working with a containerized application.

### Image management

```bash
docker build -t app:v1 .
docker images
docker inspect app:v1
docker rmi app:v1
```

### Container management

```bash
docker run -d --name app app:v1
docker ps
docker ps -a
docker logs app
docker exec -it app sh
docker stop app
docker rm app
```

### Network management

```bash
docker network ls
docker network create app-network
docker network inspect app-network
docker network connect app-network app
docker network rm app-network
```

### Storage management

```bash
docker volume create app-data
docker volume ls
docker volume inspect app-data
docker volume rm app-data
```

### Compose management

```bash
docker compose config
docker compose up -d
docker compose ps
docker compose logs -f
docker compose exec backend sh
docker compose down
```

---

## 7. Key Technical Takeaways

Today's learning established the following relationships:

1. A Dockerfile provides the instructions required to create an image.
2. An image packages the application and its runtime dependencies.
3. A container executes an instance of that image.
4. Networking enables communication between independent containers.
5. Port mapping makes selected container services accessible externally.
6. Volumes separate persistent application data from container lifecycles.
7. Bind mounts connect host filesystems with containers.
8. Docker Compose combines services, networking and storage into a manageable application configuration.

### Overall Understanding

Containerization is not just about running applications inside containers.

It involves designing how applications are packaged, executed, connected, configured and provided with persistent storage.

Understanding these relationships is essential before moving towards containerized application deployment and cloud infrastructure.

---

## 8. Learning Sources and References

The following official documentation was used as the technical reference for today's learning.

| Topic | Official documentation |
|---|---|
| Dockerfile | https://docs.docker.com/build/concepts/dockerfile/ |
| Dockerfile Instructions | https://docs.docker.com/reference/dockerfile/ |
| Build Context | https://docs.docker.com/build/concepts/context/ |
| Docker Networking | https://docs.docker.com/engine/network/ |
| Bridge Networks | https://docs.docker.com/engine/network/drivers/bridge/ |
| Docker Storage | https://docs.docker.com/engine/storage/ |
| Docker Volumes | https://docs.docker.com/engine/storage/volumes/ |
| Bind Mounts | https://docs.docker.com/engine/storage/bind-mounts/ |
| tmpfs Mounts | https://docs.docker.com/engine/storage/tmpfs/ |
| Docker Compose | https://docs.docker.com/compose/ |
| Compose Application Model | https://docs.docker.com/compose/intro/compose-application-model |
| Compose CLI Reference | https://docs.docker.com/reference/cli/docker/compose/ |

**Primary learning reference:** Official Docker Documentation.

---

## 9. Next Learning Direction

The next step is to move from theoretical understanding towards practical implementation.

Planned progression:

```text
Dockerfile
    |
    v
Build Custom Images
    |
    v
Run Individual Containers
    |
    v
Configure Container Networking
    |
    v
Implement Persistent Storage
    |
    v
Create Compose Configuration
    |
    v
Deploy a Three-Tier Application
    |
    v
Integrate with CI/CD
    |
    v
Cloud Deployment
```

**Learning Status:** Core concepts studied; practical implementation and validation remain the next focus.
