# Docker Compose - Multi-Container Applications - Part 3

This guide covers Docker Compose for orchestrating multi-container applications, demonstrating how to run PostgreSQL and Redis services together.

## Table of Contents
- [What is Docker Compose?](#what-is-docker-compose)
- [Docker Compose vs Docker CLI](#docker-compose-vs-docker-cli)
- [Understanding docker-compose.yml](#understanding-docker-composeyml)
- [Running Multi-Container Applications](#running-multi-container-applications)
- [Container Management with Compose](#container-management-with-compose)
- [Networking in Docker Compose](#networking-in-docker-compose)
- [Environment Variables](#environment-variables)
- [Service Logs and Monitoring](#service-logs-and-monitoring)
- [Best Practices](#best-practices)
- [Common Commands](#common-commands)

## What is Docker Compose?

**Docker Compose** is a tool for defining and running multi-container Docker applications. Instead of managing individual containers with multiple `docker run` commands, you define your entire application stack in a single YAML file.

### Benefits of Docker Compose
- **Declarative Configuration**: Define your entire stack in one file
- **Service Orchestration**: Manage multiple containers as a single application
- **Dependency Management**: Control startup order and service dependencies
- **Environment Isolation**: Each compose project gets its own network
- **Easy Scaling**: Scale services up or down with simple commands
- **Development Consistency**: Same environment across development, testing, and production

## Docker Compose vs Docker CLI

### Without Docker Compose (Manual Approach)
```bash
# Create a custom network
docker network create myapp-network

# Run PostgreSQL
docker run -d \
  --name postgres-db \
  --network myapp-network \
  -p 5432:5432 \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=mydb \
  postgres:latest

# Run Redis
docker run -d \
  --name redis-cache \
  --network myapp-network \
  -p 6379:6379 \
  redis:latest
```

### With Docker Compose (Declarative Approach)
```bash
# Single command to start entire stack
docker compose up
```

**Advantages of Compose:**
- ✅ Less typing and fewer commands
- ✅ Configuration stored in version control
- ✅ Automatic network creation
- ✅ Easy to reproduce environments
- ✅ Simple service discovery

## Understanding docker-compose.yml

### Our Complete Configuration
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:latest
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb

  redis:
    image: redis:latest
    ports:
      - "6379:6379"
```

### Section-by-Section Breakdown

#### 1. Version Declaration
```yaml
version: '3.8'
```
**Note**: Modern Docker Compose versions show this warning:
```
WARN[0000] the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
```
- **Purpose**: Originally specified Docker Compose file format version
- **Current Status**: No longer required in recent versions
- **Best Practice**: Can be omitted in new projects

#### 2. Services Definition
```yaml
services:
```
- **Purpose**: Defines all containers (services) in your application
- **Structure**: Each service is a named container configuration
- **Naming**: Service names become hostnames within the network

#### 3. PostgreSQL Service
```yaml
postgres:
  image: postgres:latest
  ports:
    - "5432:5432"
  environment:
    POSTGRES_USER: user
    POSTGRES_PASSWORD: password
    POSTGRES_DB: mydb
```

**Configuration Breakdown:**
- **`image: postgres:latest`**: Uses official PostgreSQL Docker image
- **`ports: - "5432:5432"`**: Maps host port 5432 to container port 5432
- **`environment:`**: Sets required PostgreSQL environment variables
  - `POSTGRES_USER`: Database username
  - `POSTGRES_PASSWORD`: Database password
  - `POSTGRES_DB`: Initial database name

#### 4. Redis Service
```yaml
redis:
  image: redis:latest
  ports:
    - "6379:6379"
```

**Configuration Breakdown:**
- **`image: redis:latest`**: Uses official Redis Docker image
- **`ports: - "6379:6379"`**: Maps host port 6379 to container port 6379
- **No environment variables**: Redis works with defaults

## Running Multi-Container Applications

### Starting the Application Stack

**Navigate to the project directory:**
```bash
$ cd part_003
$ cd node_app/
```

**Start all services:**
```bash
$ docker compose up
WARN[0000] /home/kushkumarkashyap7280/Desktop/DOCKER/part_003/node_app/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] Running 23/23
 ✔ postgres Pulled                                                                                      82.7s 
 ✔ redis Pulled                                                                                         36.6s 
[+] Running 3/3
 ✔ Network node_app_default       Created                                                                0.1s 
 ✔ Container node_app-postgres-1  Created                                                                0.4s 
 ✔ Container node_app-redis-1     Created                                                                0.4s 
Attaching to postgres-1, redis-1
```

### Understanding the Startup Process

#### 1. Image Pulling
```
✔ postgres Pulled    82.7s 
✔ redis Pulled       36.6s
```
- **Purpose**: Downloads required images if not locally available
- **Time Varies**: Depends on image size and internet speed
- **One-Time Process**: Subsequent runs use cached images

#### 2. Resource Creation
```
✔ Network node_app_default       Created    0.1s 
✔ Container node_app-postgres-1  Created    0.4s 
✔ Container node_app-redis-1     Created    0.4s
```
- **Network**: Creates isolated network for service communication
- **Containers**: Creates containers based on service definitions
- **Naming Convention**: `{project-name}-{service-name}-{replica-number}`

#### 3. Container Attachment
```
Attaching to postgres-1, redis-1
```
- **Log Streaming**: Shows logs from all services in real-time
- **Interleaved Output**: Logs from different services are mixed
- **Service Identification**: Each line prefixed with service name

## Service Logs and Monitoring

### Redis Startup Logs
```bash
redis-1  | Starting Redis Server
redis-1  | 1:C 21 Sep 2025 05:40:26.220 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-1  | 1:C 21 Sep 2025 05:40:26.220 * Redis version=8.2.1, bits=64, commit=00000000, modified=1, pid=1, just started
redis-1  | 1:M 21 Sep 2025 05:40:26.221 * Running mode=standalone, port=6379.
redis-1  | 1:M 21 Sep 2025 05:40:26.228 * Server initialized
redis-1  | 1:M 21 Sep 2025 05:40:26.228 * Ready to accept connections tcp
```

**Key Information:**
- **Version**: Redis 8.2.1 is running
- **Mode**: Standalone (not clustered)
- **Port**: Listening on 6379
- **Status**: Ready to accept connections

### PostgreSQL Startup Logs
```bash
postgres-1  | The files belonging to this database system will be owned by user "postgres".
postgres-1  | The database cluster will be initialized with locale "en_US.utf8".
postgres-1  | The default database encoding has accordingly been set to "UTF8".
postgres-1  | creating configuration files ... ok
postgres-1  | running bootstrap script ... ok
postgres-1  | PostgreSQL init process complete; ready for start up.
postgres-1  | 2025-09-21 05:40:27.573 UTC [1] LOG:  starting PostgreSQL 17.6
postgres-1  | 2025-09-21 05:40:27.574 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
postgres-1  | 2025-09-21 05:40:27.597 UTC [1] LOG:  database system is ready to accept connections
```

**Key Information:**
- **Version**: PostgreSQL 17.6 is running
- **Initialization**: Database cluster created successfully
- **Encoding**: UTF8 with en_US.utf8 locale
- **Network**: Listening on all interfaces (0.0.0.0) port 5432
- **Status**: Ready to accept connections

### Checking Running Services

**View running containers:**
```bash
$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                                         NAMES
5257eca214ce   postgres:latest   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp   node_app-postgres-1
7465c2814582   redis:latest      "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp   node_app-redis-1
```

**Container Analysis:**
- **PostgreSQL Container**: 
  - Name: `node_app-postgres-1`
  - Port mapping: `0.0.0.0:5432->5432/tcp`
  - Status: Running for 2 minutes
- **Redis Container**:
  - Name: `node_app-redis-1`
  - Port mapping: `0.0.0.0:6379->6379/tcp`
  - Status: Running for 2 minutes

## Container Management with Compose

### Stopping Services

**Graceful shutdown (when compose is running in foreground):**
```
^C
Container node_app-postgres-1  Stopped
postgres-1 exited with code 0
Container node_app-redis-1  Stopped
redis-1 exited with code 0
```

**Manual shutdown:**
```bash
docker compose down
```

### Service Status Verification

**After shutdown:**
```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
- **Empty output**: All containers have been stopped and removed
- **Clean state**: Ready for next `docker compose up`

## Networking in Docker Compose

### Automatic Network Creation

**Network naming convention:**
```
✔ Network node_app_default       Created
```
- **Pattern**: `{project-directory-name}_default`
- **Isolation**: Services in different projects can't communicate by default
- **DNS Resolution**: Services can reach each other by service name

### Service Discovery Example

Within the compose network, services can communicate using service names:
```bash
# From any container in the stack:
# Connect to PostgreSQL: hostname = "postgres", port = 5432
# Connect to Redis: hostname = "redis", port = 6379
```

## Environment Variables

### Database Configuration

**PostgreSQL environment variables explained:**
```yaml
environment:
  POSTGRES_USER: user        # Creates a user named "user"
  POSTGRES_PASSWORD: password # Sets password for the user
  POSTGRES_DB: mydb          # Creates initial database "mydb"
```

### Security Considerations

**Current configuration is for development only:**
- ⚠️ **Weak passwords**: Using simple passwords
- ⚠️ **No secrets management**: Credentials in plain text
- ⚠️ **Default ports**: Using standard ports

**Production improvements:**
```yaml
environment:
  POSTGRES_USER: ${DB_USER}
  POSTGRES_PASSWORD: ${DB_PASSWORD}
  POSTGRES_DB: ${DB_NAME}
```

## Best Practices

### 1. Environment Variables

**Use .env file for sensitive data:**
```bash
# .env file
DB_USER=myuser
DB_PASSWORD=securepassword123
DB_NAME=production_db
REDIS_PASSWORD=redispassword123
```

**Updated docker-compose.yml:**
```yaml
services:
  postgres:
    image: postgres:latest
    ports:
      - "${DB_PORT:-5432}:5432"
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:latest
    ports:
      - "${REDIS_PORT:-6379}:6379"
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### 2. Data Persistence

**Add volumes for data persistence:**
```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data  # PostgreSQL data
  - redis_data:/data                        # Redis data
```

### 3. Health Checks

**Add health checks:**
```yaml
postgres:
  # ... other config
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
    interval: 30s
    timeout: 10s
    retries: 3
```

### 4. Resource Limits

**Add resource constraints:**
```yaml
postgres:
  # ... other config
  deploy:
    resources:
      limits:
        memory: 512M
        cpus: '0.5'
```

## Common Commands

### Essential Docker Compose Commands

**Start services in background:**
```bash
docker compose up -d
```

**Stop and remove containers:**
```bash
docker compose down
```

**View service logs:**
```bash
docker compose logs
docker compose logs postgres  # Specific service
docker compose logs -f        # Follow logs
```

**Scale services:**
```bash
docker compose up --scale redis=3
```

**Execute commands in running containers:**
```bash
docker compose exec postgres psql -U user -d mydb
docker compose exec redis redis-cli
```

**Build custom images (if using build context):**
```bash
docker compose build
docker compose up --build
```

**View running services:**
```bash
docker compose ps
```

**Stop specific service:**
```bash
docker compose stop postgres
```

**Remove volumes (⚠️ deletes data):**
```bash
docker compose down -v
```

## Troubleshooting

### Common Issues

**Port conflicts:**
```bash
# If ports are already in use
docker compose up
# Error: port is already allocated
```
**Solution**: Change ports in docker-compose.yml or stop conflicting services

**Permission issues:**
```bash
# PostgreSQL data directory permissions
chown -R 999:999 /path/to/postgres/data
```

**Network issues:**
```bash
# Clean up networks
docker network prune
```

**Image issues:**
```bash
# Force pull latest images
docker compose pull
docker compose up --force-recreate
```

## Key Takeaways

1. **Docker Compose Benefits**:
   - Simplifies multi-container application management
   - Provides consistent development environments
   - Enables declarative infrastructure as code

2. **Service Communication**:
   - Services automatically get DNS names matching service names
   - Isolated networks prevent conflicts between projects
   - Port mapping allows external access

3. **Configuration Management**:
   - YAML format is human-readable and version-controllable
   - Environment variables enable different configurations
   - Volumes provide data persistence

4. **Development Workflow**:
   - `docker compose up` starts entire application stack
   - Real-time log aggregation from all services
   - Easy teardown and recreation for testing

## Next Steps

- Learn about Docker volumes for data persistence
- Explore Docker Compose override files for different environments
- Study service dependencies and startup ordering
- Practice with more complex multi-service applications