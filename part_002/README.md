# Docker Advanced Commands & Container Management - Part 2

This guide covers advanced Docker commands, container lifecycle management, and building custom images with practical examples.

## Table of Contents
- [Docker Command Overview](#docker-command-overview)
- [Key Concept Differences](#key-concept-differences)
- [Container Lifecycle Management](#container-lifecycle-management)
- [Working with Custom Images](#working-with-custom-images)
- [Port Mapping & Networking](#port-mapping--networking)
- [Environment Variables](#environment-variables)
- [Docker Registry Operations](#docker-registry-operations)
- [Advanced Operations](#advanced-operations)
- [Practical Examples](#practical-examples)

## Docker Command Overview

### Complete Docker Help Output
```bash
$ docker
Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Common Commands:
  run         Create and run a new container from an image
  exec        Execute a command in a running container
  ps          List containers
  build       Build an image from a Dockerfile
  bake        Build from a file
  pull        Download an image from a registry
  push        Upload an image to a registry
  images      List images
  login       Authenticate to a registry
  logout      Log out from a registry
  search      Search Docker Hub for images
  version     Show the Docker version information
  info        Display system-wide information

Management Commands:
  builder     Manage builds
  buildx*     Docker Buildx
  compose*    Docker Compose
  container   Manage containers
  context     Manage contexts
  image       Manage images
  manifest    Manage Docker image manifests and manifest lists
  model*      Docker Model Runner (EXPERIMENTAL)
  network     Manage networks
  plugin      Manage plugins
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Swarm Commands:
  swarm       Manage Swarm

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  wait        Block until one or more containers stop, then print their exit codes
```

## Key Concept Differences

### 🔄 `docker run` vs `docker start`

**`docker run`** - Creates and starts a NEW container
- Always creates a fresh container from an image
- Can specify all configuration options (ports, volumes, environment variables)
- Can run in interactive (`-it`) or detached (`-d`) mode

**`docker start`** - Starts an EXISTING stopped container
- Uses a container that was previously created
- Maintains all original configuration from when it was created
- Cannot change configuration options

**Example:**
```bash
# First time - creates and runs new container
$ docker run -it ubuntu
root@ee8cf1fa45a3:/# exit

# Container is now stopped but exists
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                      PORTS     NAMES
ee8cf1fa45a3   ubuntu    "/bin/bash"   5 minutes ago   Exited (0) 15 seconds ago             awesome_blackwell

# Start the existing container (no new container created)
$ docker start awesome_blackwell
awesome_blackwell

# Now it's running again
$ docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS         PORTS     NAMES
ee8cf1fa45a3   ubuntu    "/bin/bash"   10 minutes ago   Up 6 seconds             awesome_blackwell
```

### 🎯 `-it` vs `-d` Flags

**`-it` (Interactive Terminal)**
- `-i` = Interactive (keep STDIN open)
- `-t` = TTY (allocate a pseudo-terminal)
- Used when you want to interact with the container directly
- Attaches your terminal to the container's terminal
- Perfect for debugging, development, or manual operations

**`-d` (Detached Mode)**
- Runs container in the background
- Container runs independently of your terminal session
- Perfect for servers, web applications, databases
- You get the container ID back immediately

**Examples:**
```bash
# Interactive mode - you get a shell inside the container
$ docker run -it ubuntu
root@container:/# 

# Detached mode - runs in background
$ docker run -d --name mynginx -p 8080:80 nginx
c07b0b73715f2a2b463c44342b0acf9c5fdbdf1b13d8253d82f8b4b5d36d1f0c

# Check running containers
$ docker ps
CONTAINER ID   IMAGE   COMMAND                  CREATED       STATUS       PORTS                  NAMES
c07b0b73715f   nginx   "/docker-entrypoint.…"   1 minute ago  Up 1 minute  0.0.0.0:8080->80/tcp   mynginx
```

### 🆔 Container ID vs Container Name

**Container ID**
- Unique hexadecimal identifier (64 characters, usually shortened to 12)
- Automatically generated by Docker
- Can use full ID or shortened version for commands

**Container Name**
- Human-readable identifier
- Can be assigned with `--name` flag
- If not specified, Docker generates a random name
- Must be unique across all containers

**Examples:**
```bash
# Using container ID
$ docker stop c07b0b73715f2a2b463c44342b0acf9c5fdbdf1b13d8253d82f8b4b5d36d1f0c
$ docker stop c07b0b73715f  # Shortened version

# Using container name (much easier!)
$ docker stop mynginx

# Error example - trying to stop by image name (won't work)
$ docker stop first-docker-server
Error response from daemon: No such container: first-docker-server
```

## Container Lifecycle Management

### 1. Creating and Running Containers

**Interactive Ubuntu Container:**
```bash
$ docker run -it ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
953cdd413371: Pull complete 
Digest: sha256:353675e2a41babd526e2b837d7ec780c2a05bca0164f7ea5dbbd433d21d166fc
Status: Downloaded newer image for ubuntu:latest
root@ee8cf1fa45a3:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
```

**Detached NGINX Container with Port Mapping:**
```bash
$ docker run -d --name mynginx -p 8080:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
[... pulling layers ...]
Status: Downloaded newer image for nginx:latest
c07b0b73715f2a2b463c44342b0acf9c5fdbdf1b13d8253d82f8b4b5d36d1f0c
```

### 2. Starting and Stopping Containers

**Start an existing stopped container:**
```bash
$ docker start awesome_blackwell
awesome_blackwell

$ docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS         PORTS     NAMES
ee8cf1fa45a3   ubuntu    "/bin/bash"   10 minutes ago   Up 6 seconds             awesome_blackwell
```

**Stop a running container:**
```bash
$ docker stop awesome_blackwell
awesome_blackwell

$ docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                        PORTS     NAMES
ee8cf1fa45a3   ubuntu    "/bin/bash"   11 minutes ago   Exited (137) 11 seconds ago             awesome_blackwell
```

### 3. Container Status Understanding

**Exit Codes:**
- `Exited (0)` = Normal, successful termination
- `Exited (137)` = Container was killed (usually by `docker stop`)
- `Exited (1)` = General application error

## Working with Custom Images

### 1. Building Custom Images

**Common Typo Warning:**
```bash
# WRONG - This is a common typo
$ docker built -t first-docker-server .
unknown shorthand flag: 't' in -t

# CORRECT
$ docker build -t first-docker-server .
```

**Successful Build Output:**
```bash
$ docker build -t first-docker-server .
[+] Building 165.9s (15/15) FINISHED                             docker:default
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 356B                                       0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest          17.7s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [ 1/10] FROM docker.io/library/ubuntu:latest@sha256:353675e2a41babd5  47.3s
 => [internal] load build context                                          0.0s
 => => transferring context: 30.32kB                                       0.0s
 => [ 2/10] RUN apt-get update                                            21.1s
 => [ 3/10] RUN apt-get install -y curl                                   16.5s 
 => [ 4/10] RUN curl -sL https://deb.nodesource.com/setup_18.x | bash -   27.5s 
 => [ 5/10] RUN apt-get upgrade -y                                         7.9s 
 => [ 6/10] RUN apt-get install -y nodejs                                 25.1s 
 => [ 7/10] COPY package.json package.json                                 0.0s 
 => [ 8/10] COPY package-lock.json package-lock.json                       0.0s 
 => [ 9/10] COPY main.js main.js                                           0.0s 
 => [10/10] RUN npm install                                                1.7s
 => exporting to image                                                     0.7s
 => => exporting layers                                                    0.7s
 => => writing image sha256:7789bd168fce2563e06bae8046cd0ef622269d44f4e51  0.0s
 => => naming to docker.io/library/first-docker-server                     0.0s
```

### 2. Running Custom Images

**Check built images:**
```bash
$ docker images
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
first-docker-server   latest    7789bd168fce   2 minutes ago   345MB
```

**Run in interactive mode:**
```bash
$ docker run -it first-docker-server
Server is running on port http://localhost:3000
```

**Run in detached mode with port mapping:**
```bash
$ docker run -p 3000:3000 -d first-docker-server
8ec4c2a421d09bf6ad9cce8daa2a745bdcdd31824b6a4f1df91d903da917fa0b

$ docker ps
CONTAINER ID   IMAGE                 COMMAND          CREATED         STATUS         PORTS                                         NAMES
8ec4c2a421d0   first-docker-server   "node main.js"   7 seconds ago   Up 6 seconds   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   focused_lamport
```

## Port Mapping & Networking

### Understanding Port Mapping

**Format:** `-p HOST_PORT:CONTAINER_PORT`

**Examples:**
```bash
# Map host port 8080 to container port 80
$ docker run -d --name mynginx -p 8080:80 nginx

# Map host port 3000 to container port 3000
$ docker run -p 3000:3000 -d first-docker-server

# The PORTS column shows the mapping
$ docker ps
CONTAINER ID   IMAGE                 COMMAND          CREATED         STATUS         PORTS                                         NAMES
8ec4c2a421d0   first-docker-server   "node main.js"   7 seconds ago   Up 6 seconds   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   focused_lamport
```

**Port Mapping Breakdown:**
- `0.0.0.0:3000` = All interfaces on host port 3000
- `->3000/tcp` = Maps to container port 3000 using TCP protocol
- `[::]:3000` = IPv6 equivalent

## Environment Variables

### Using Environment Variables in Containers

**Setting environment variables with `-e` flag:**
```bash
# Set PORT environment variable to 4000
$ docker run -it -e PORT=4000 -p 4000:4000 first-docker-server bash
Server is running on port http://localhost:4000
```

**Environment Variable Precedence:**
1. `-e` flag values (highest priority)
2. Values defined in Dockerfile
3. Default values in application code

**Inside the Node.js application:**
```javascript
const port = process.env.PORT || 3000;  // Uses PORT env var or defaults to 3000
```

## Executing Commands in Running Containers

### Using `docker exec`

**Access running container shell:**
```bash
$ docker exec -it focused_lamport bash
root@8ec4c2a421d0:/# ls
bin   etc   lib64    mnt           package-lock.json  root  srv  usr
boot  home  main.js  node_modules  package.json       run   sys  var
dev   lib   media    opt           proc               sbin  tmp

root@8ec4c2a421d0:/# cat main.js
import  express from 'express';
const app = express();

const port = process.env.PORT || 3000;
app.get('/', (req, res) => {
    res.status(200).json({ message: 'Hello, World!' });
});

app.listen(port , () => {
    console.log(`Server is running on port http://localhost:${port}`);
});

root@8ec4c2a421d0:/# exit
exit
```

**Key Points:**
- Container continues running after you exit the exec session
- Perfect for debugging, inspecting files, or running additional commands
- Always use with running containers (not stopped ones)

## Docker Registry Operations

### 1. Checking Docker Hub Username
```bash
$ docker info | grep Username
 Username: kushkumarkashyap728065
```

### 2. Tagging Images for Registry
```bash
# Tag local image for Docker Hub
$ docker tag node-server-container:latest kushkumarkashyap728065/node-server:latest
```

### 3. Pushing to Docker Hub
```bash
$ docker push kushkumarkashyap728065/node-server:latest
The push refers to repository [docker.io/kushkumarkashyap728065/node-server]
f4cc287df37a: Pushed 
f14705520828: Pushed 
af5f2649e16d: Pushed 
ce185b69be5b: Pushed 
d0f6b39dfecc: Pushed 
54e7647757a6: Pushed 
5a713e031c43: Pushed 
b577fcd0ae1b: Pushed 
d37ca62195a7: Pushed 
f9f52dc133e2: Pushed 
latest: digest: sha256:b1ba06fa2ef1b9f9799332c60493d436e6b94315a47c7530025464cf2ec1089f size: 2418
```

### 4. Pulling from Docker Hub
```bash
$ docker pull kushkumarkashyap728065/node-server:latest
latest: Pulling from kushkumarkashyap728065/node-server
Digest: sha256:b1ba06fa2ef1b9f9799332c60493d436e6b94315a47c7530025464cf2ec1089f
Status: Image is up to date for kushkumarkashyap728065/node-server:latest
docker.io/kushkumarkashyap728065/node-server:latest
```

## Advanced Operations

### 1. Bulk Container Operations

**Remove all containers:**
```bash
$ docker rm $(docker ps -aq)
c07b0b73715f
ee8cf1fa45a3
```

**Remove all images:**
```bash
$ docker rmi $(docker images -q)
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:353675e2a41babd526e2b837d7ec780c2a05bca0164f7ea5dbbd433d21d166fc
Deleted: sha256:6d79abd4c96299aa91f5a4a46551042407568a3858b00ab460f4ba430984f62c
[... more deletion output ...]
```

**Understanding the Commands:**
- `$(docker ps -aq)` = Gets all container IDs (running and stopped)
- `$(docker images -q)` = Gets all image IDs
- `-a` = All containers (including stopped)
- `-q` = Quiet mode (only IDs)

### 2. Common Error Scenarios

**Wrong Container Reference:**
```bash
# Trying to use container ID as image name
$ docker run -it -e PORT=4000 -p 4000:4000 8ec4c2a421d0 bash
Unable to find image '8ec4c2a421d0:latest' locally
docker: Error response from daemon: pull access denied for 8ec4c2a421d0, repository does not exist

# Trying to use container name as image name
$ docker run -it -e PORT=4000 -p 4000:4000 focused_lamport bash
Unable to find image 'focused_lamport:latest' locally
docker: Error response from daemon: pull access denied for focused_lamport, repository does not exist

# CORRECT - Use image name
$ docker run -it -e PORT=4000 -p 4000:4000 first-docker-server bash
Server is running on port http://localhost:4000
```

## Practical Examples Summary

### Node.js Application Deployment Workflow

1. **Build the image:**
   ```bash
   docker build -t first-docker-server .
   ```

2. **Run for testing (interactive):**
   ```bash
   docker run -it first-docker-server
   ```

3. **Run for production (detached with port mapping):**
   ```bash
   docker run -p 3000:3000 -d first-docker-server
   ```

4. **Debug running container:**
   ```bash
   docker exec -it container_name bash
   ```

5. **Tag and push to registry:**
   ```bash
   docker tag first-docker-server:latest username/app-name:latest
   docker push username/app-name:latest
   ```

## Key Takeaways

1. **Command Differences:**
   - `docker run` = Create + Start (new container)
   - `docker start` = Start existing container
   - `docker exec` = Execute command in running container

2. **Flag Meanings:**
   - `-it` = Interactive terminal (for development/debugging)
   - `-d` = Detached mode (for production services)
   - `-p` = Port mapping (host:container)
   - `-e` = Environment variables
   - `--name` = Assign container name

3. **Container References:**
   - Use **Image Name** when creating containers (`docker run`)
   - Use **Container ID** or **Container Name** when managing existing containers

4. **Best Practices:**
   - Always use meaningful names for containers (`--name`)
   - Use detached mode (`-d`) for services
   - Map ports explicitly when needed (`-p`)
   - Tag images properly before pushing to registries

## Next Steps

- Learn about Dockerfile best practices
- Explore Docker Compose for multi-container applications
- Study Docker volumes for data persistence
- Practice with different base images and optimization techniques 
