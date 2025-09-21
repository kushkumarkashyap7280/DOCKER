# Docker Networking and Volumes - Part 4

This guide covers Docker networking modes, custom networks, and data persistence with volumes, demonstrating container-to-container communication and file sharing between host and containers.

## Table of Contents
- [Docker Networking Fundamentals](#docker-networking-fundamentals)
- [Network Modes Deep Dive](#network-modes-deep-dive)
- [Custom Networks and Container Communication](#custom-networks-and-container-communication)
- [Docker Volumes and Data Persistence](#docker-volumes-and-data-persistence)
- [Hands-on Network Examples](#hands-on-network-examples)
- [Volume Mount Demonstrations](#volume-mount-demonstrations)
- [Container Networking Best Practices](#container-networking-best-practices)
- [Troubleshooting Network and Volume Issues](#troubleshooting-network-and-volume-issues)
- [Production Considerations](#production-considerations)
- [Common Commands Reference](#common-commands-reference)

## Docker Networking Fundamentals

### What is Docker Networking?

Docker networking enables containers to communicate with:
- **Other containers** on the same host
- **External networks** and the internet
- **Host system** resources
- **Services** running on different hosts

### Core Networking Concepts

**1. Network Drivers:**
- `bridge`: Default network driver for standalone containers
- `host`: Removes network isolation between container and host
- `none`: Disables all networking
- `overlay`: Enables swarm services to communicate
- `macvlan`: Assigns MAC addresses to containers

**2. Network Isolation:**
- Each network provides isolated communication channels
- Containers in different networks cannot communicate by default
- Custom networks enable service discovery by container name

**3. Port Publishing:**
- Maps container ports to host ports
- Enables external access to containerized services
- Uses iptables rules for traffic routing

## Network Modes Deep Dive

### 1. Bridge Network (Default)

**Create and test default bridge networking:**
```bash
$ docker run -it --name mycontainer busybox
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
80bfbb8a41a2: Pull complete 
Digest: sha256:d82f458899c9696cb26a7c02d5568f81c8c8223f8661bb2a7988b269c8b9051e
Status: Downloaded newer image for busybox:latest
/ # ping google.com
PING google.com (142.250.182.174): 56 data bytes
64 bytes from 142.250.182.174: seq=0 ttl=116 time=200.100 ms
64 bytes from 142.250.182.174: seq=1 ttl=116 time=33.792 ms
64 bytes from 142.250.182.174: seq=2 ttl=116 time=47.250 ms
# ... more ping results
^C
--- google.com ping statistics ---
71 packets transmitted, 67 packets received, 5% packet loss
round-trip min/avg/max = 22.511/123.267/518.917 ms
/ # exit
```

**Bridge network characteristics:**
- **Internet Access**: ✅ Containers can reach external networks
- **Isolation**: Containers get private IP addresses
- **NAT**: Network Address Translation handles outbound traffic
- **Default Gateway**: Docker creates a bridge interface (docker0)

### 2. Bridge Network Inspection

**Inspect the default bridge network:**
```bash
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "21e9a77d6a669fd332a3083c2f774bb209bc0fcbfb37f66bddd1fdd9e9644b82",
        "Created": "2025-09-21T14:38:49.635816551+05:30",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

**Key network configuration details:**
- **Subnet**: `172.17.0.0/16` provides 65,534 possible IP addresses
- **Gateway**: `172.17.0.1` is the Docker bridge gateway
- **Driver**: `bridge` creates isolated network segments
- **MTU**: `1500` bytes maximum transmission unit
- **ICC (Inter-Container Communication)**: Enabled by default

**Running container network inspection:**
```bash
$ docker run -it busybox
/ # # Container running in background
```

**Inspect bridge with active container:**
```bash
$ docker network inspect bridge
# ... network details ...
        "Containers": {
            "3a3f55e1150c0594271966a0a74a4a4896cf0791610f6c0c53df97215d1870d2": {
                "Name": "bold_thompson",
                "EndpointID": "30c215921a0351fbac248c8e291747e5930899a91845a8313fd38a89c7c327ef",
                "MacAddress": "82:1e:e3:84:9e:80",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
# ...
```

**Active container analysis:**
- **Container IP**: `172.17.0.2/16` (first available IP after gateway)
- **MAC Address**: `82:1e:e3:84:9e:80` (unique hardware identifier)
- **Endpoint ID**: Unique network connection identifier
- **Network Membership**: Container actively connected to bridge network

### 3. Host Network Mode

**Run container with host networking:**
```bash
$ docker run -it --network=host busybox
/ # ping google.com
PING google.com (142.250.182.174): 56 data bytes
64 bytes from 142.250.182.174: seq=0 ttl=117 time=204.741 ms
64 bytes from 142.250.182.174: seq=1 ttl=117 time=125.555 ms
64 bytes from 142.250.182.174: seq=2 ttl=117 time=49.940 ms
^C
--- google.com ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 49.940/126.745/204.741 ms
```

**Host network characteristics:**
- **No Network Isolation**: Container shares host's network stack
- **Direct Host Access**: Container uses host's IP address
- **Port Conflicts**: Container ports directly bind to host ports
- **Performance**: Minimal network overhead
- **Security Risk**: Container has full network access

### 4. None Network Mode

**Run container with no networking:**
```bash
$ docker run -it --network=none busybox
/ # ping google.com
ping: bad address 'google.com'
```

**None network characteristics:**
- **Complete Isolation**: No network interfaces except loopback
- **No Internet Access**: Cannot resolve DNS or reach external networks
- **Security**: Maximum network isolation
- **Use Cases**: Batch jobs, data processing, security-sensitive workloads

## Custom Networks and Container Communication

### Creating Custom Networks

**Demonstration requires creating a custom network first:**
```bash
# Note: The tutorial shows containers running on 'mynetwork'
# This network must be created before running the containers
docker network create mynetwork
```

### Multi-Terminal Container Communication Demo

**Terminal 1 - Ubuntu Container (net1):**
```bash
$ docker run -it --network=mynetwork --name net1 ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
953cdd413371: Already exists 
Digest: sha256:353675e2a41babd526e2b837d7ec780c2a05bca0164f7ea5dbbd433d21d166fc
Status: Downloaded newer image for ubuntu:latest
root@17c12dc6b0b6:/# 
```

**Terminal 2 - Busybox Container (net2):**
```bash
$ docker run -it --network=mynetwork --name net2 busybox
/ # ping net1
PING net1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.121 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.109 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.107 ms
64 bytes from 172.18.0.2: seq=3 ttl=64 time=0.138 ms
64 bytes from 172.18.0.2: seq=4 ttl=64 time=0.160 ms
64 bytes from 172.18.0.2: seq=5 ttl=64 time=0.106 ms
^C
--- net1 ping statistics ---
6 packets transmitted, 6 packets received, 0% packet loss
round-trip min/avg/max = 0.106/0.123/0.160 ms
```

**Communication analysis:**
- **Service Discovery**: Container `net2` can ping `net1` by name
- **Low Latency**: ~0.1ms response time (local network)
- **100% Success Rate**: All packets transmitted successfully
- **Custom Network Benefits**: Name-based communication without IP addresses

**Terminal 3 - Network Inspection:**
```bash
$ docker network inspect mynetwork
[
    {
        "Name": "mynetwork",
        "Id": "3f5f251d53f2c31cad1efbc1f6d7d23d1c76648b8bf86788fecc83c9bcf46a54",
        "Created": "2025-09-21T15:08:39.7123987+05:30",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "17c12dc6b0b639b1cdd522aff60ec05b41fdec84c6debb2d661d7937b613657a": {
                "Name": "net1",
                "EndpointID": "a7a317d85225881e63c495abf67ef7090b40b348e5c3be4179c5bc4c5dc7cf84",
                "MacAddress": "3e:a8:e2:17:08:cf",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "2ca8a6ded173399f2ac22964076d693d60b2c370d103bda927e71122f8111beb": {
                "Name": "net2",
                "EndpointID": "8a32870dc8d03e8c464d153138498bfda37518e7dd5ddc1daafeeb316f894b24",
                "MacAddress": "5a:48:9d:54:f1:be",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

**Custom network analysis:**
- **Different Subnet**: `172.18.0.0/16` (vs default `172.17.0.0/16`)
- **Automatic IP Assignment**: 
  - `net1`: `172.18.0.2/16`
  - `net2`: `172.18.0.3/16`
- **Built-in DNS**: Containers resolve each other by name
- **Network Isolation**: Only containers on `mynetwork` can communicate

## Docker Volumes and Data Persistence

### Volume Concepts

**Docker Storage Types:**
- **Volumes**: Docker-managed storage, preferred method
- **Bind Mounts**: Direct host filesystem mapping
- **tmpfs Mounts**: Temporary filesystem in memory

### Bind Mount Demonstration

**Prepare host directory:**
```bash
$ cd ~/Desktop/DOCKER/part_004
$ mkdir test-folder
$ ls
README.md  test-folder
$ pwd
/home/kushkumarkashyap7280/Desktop/DOCKER/part_004
```

**Create container with bind mount:**
```bash
$ docker run -it -v /home/kushkumarkashyap7280/Desktop/DOCKER/part_004:/tmp/fun ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
953cdd413371: Already exists 
Digest: sha256:353675e2a41babd526e2b837d7ec780c2a05bca0164f7ea5dbbd433d21d166fc
Status: Downloaded newer image for ubuntu:latest
root@c78d7b826fee:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@c78d7b826fee:/# cd tmp
root@c78d7b826fee:/tmp# ls
fun
root@c78d7b826fee:/tmp# mkdir hello-docker
root@c78d7b826fee:/tmp# ls
fun  hello-docker
root@c78d7b826fee:/tmp# cd fun
root@c78d7b826fee:/tmp/fun# mkdir just-to-persist 
root@c78d7b826fee:/tmp/fun# ls
README.md  just-to-persist  test-folder
root@c78d7b826fee:/tmp/fun# exit
```

**Volume mount analysis:**
- **Host Path**: `/home/kushkumarkashyap7280/Desktop/DOCKER/part_004`
- **Container Path**: `/tmp/fun`
- **Bidirectional Sync**: Changes reflect on both host and container
- **Data Persistence**: Files survive container deletion

**Container lifecycle management:**
```bash
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                      PORTS     NAMES
c78d7b826fee   ubuntu    "/bin/bash"   6 minutes ago   Exited (0) 18 seconds ago             exciting_matsumoto
$ docker rm c78d7b826fee
c78d7b826fee
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

**Image cleanup and re-pull:**
```bash
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    6d79abd4c962   11 days ago   78.1MB
$ docker rmi ubuntu
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:353675e2a41babd526e2b837d7ec780c2a05bca0164f7ea5dbbd433d21d166fc
Deleted: sha256:6d79abd4c96299aa91f5a4a46551042407568a3858b00ab460f4ba430984f62c
$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

**Data persistence verification:**
```bash
$ docker run -it -v /home/kushkumarkashyap7280/Desktop/DOCKER/part_004:/tmp/fun ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
953cdd413371: Already exists 
Digest: sha256:353675e2a41babd526e2b837d7ec780c2a05bca0164f7ea5dbbd433d21d166fc
Status: Downloaded newer image for ubuntu:latest
root@e32e0c755ace:/# cd tmp/fun
root@e32e0c755ace:/tmp/fun# ls
README.md  just-to-persist  test-folder
root@e32e0c755ace:/tmp/fun# exit
```

**Data persistence verification:**
- **Container Deleted**: Original container `c78d7b826fee` removed
- **Image Deleted**: Ubuntu image completely removed
- **Data Survived**: `just-to-persist` folder still exists
- **New Container**: Fresh container `e32e0c755ace` sees persistent data

## Hands-on Network Examples

### 1. Network Mode Comparison

| Network Mode | Internet Access | Container-to-Container | Performance | Security | Use Case |
|--------------|----------------|----------------------|-------------|----------|----------|
| **bridge** (default) | ✅ Yes | ✅ Same network only | Good | Medium | Standard apps |
| **host** | ✅ Yes | ❌ Host networking only | Excellent | Low | High-performance |
| **none** | ❌ No | ❌ No | N/A | High | Batch processing |
| **custom** | ✅ Yes | ✅ Name-based discovery | Good | High | Microservices |

### 2. Service Discovery Demo

**Create application network:**
```bash
docker network create app-network
```

**Run database container:**
```bash
docker run -d --name database --network app-network postgres:13
```

**Run application container:**
```bash
docker run -d --name webapp --network app-network -p 8080:80 nginx
```

**Test connectivity:**
```bash
docker exec webapp ping database
# DNS resolution works: webapp can reach 'database' by name
```

### 3. Multi-Container Application

**Example: WordPress with MySQL**
```bash
# Create network
docker network create wordpress-net

# Run MySQL
docker run -d --name mysql \
  --network wordpress-net \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=wordpress \
  mysql:5.7

# Run WordPress
docker run -d --name wordpress \
  --network wordpress-net \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=mysql \
  -e WORDPRESS_DB_NAME=wordpress \
  wordpress
```

## Volume Mount Demonstrations

### 1. Bind Mount vs Named Volume

**Bind Mount (demonstrated above):**
```bash
# Direct host path mapping
docker run -v /host/path:/container/path image
```

**Named Volume:**
```bash
# Docker-managed storage
docker volume create my-data
docker run -v my-data:/container/path image
```

### 2. Volume Types Comparison

| Type | Management | Performance | Portability | Backup | Security |
|------|------------|-------------|-------------|---------|-----------|
| **Bind Mount** | Manual | Good | Low | Manual | Host dependent |
| **Named Volume** | Docker | Excellent | High | Docker | Isolated |
| **tmpfs** | Temporary | Excellent | None | None | Memory only |

### 3. Data Sharing Between Containers

**Shared volume example:**
```bash
# Create shared volume
docker volume create shared-data

# Container 1 writes data
docker run -it -v shared-data:/data ubuntu bash
# Inside container: echo "Hello from container 1" > /data/message.txt

# Container 2 reads data
docker run -it -v shared-data:/data ubuntu bash
# Inside container: cat /data/message.txt
# Output: Hello from container 1
```

## Container Networking Best Practices

### 1. Network Security

**Principle of Least Privilege:**
```bash
# Create isolated networks for different application tiers
docker network create frontend-net
docker network create backend-net
docker network create database-net

# Web servers only on frontend
docker run --network frontend-net web-server

# Application servers on frontend + backend
docker run --network frontend-net --network backend-net app-server

# Database only on backend
docker run --network backend-net database
```

### 2. Port Management

**Explicit port mapping:**
```bash
# Good: Explicit port mapping
docker run -p 127.0.0.1:8080:80 nginx

# Avoid: Binding to all interfaces unnecessarily
docker run -p 8080:80 nginx
```

### 3. Network Naming Conventions

**Descriptive network names:**
```bash
# Good: Environment and purpose clear
docker network create prod-frontend-net
docker network create staging-api-net
docker network create dev-database-net

# Avoid: Generic or unclear names
docker network create network1
docker network create mynet
```

### 4. Service Discovery

**Use container names for internal communication:**
```bash
# Application configuration
DATABASE_HOST=postgres-db
REDIS_HOST=redis-cache
API_HOST=api-server

# Container deployment
docker run --name postgres-db --network app-net postgres
docker run --name redis-cache --network app-net redis
docker run --name api-server --network app-net api-image
```

## Volume Best Practices

### 1. Data Persistence Strategy

**Production volume management:**
```bash
# Named volumes for databases
docker volume create postgres-data
docker run -v postgres-data:/var/lib/postgresql/data postgres

# Bind mounts for configuration
docker run -v /host/config:/app/config app-image

# tmpfs for temporary data
docker run --tmpfs /tmp app-image
```

### 2. Backup and Recovery

**Volume backup strategy:**
```bash
# Backup volume data
docker run --rm -v postgres-data:/data -v $(pwd):/backup ubuntu \
  tar czf /backup/postgres-backup.tar.gz -C /data .

# Restore volume data
docker run --rm -v postgres-data:/data -v $(pwd):/backup ubuntu \
  tar xzf /backup/postgres-backup.tar.gz -C /data
```

### 3. Security Considerations

**Read-only mounts:**
```bash
# Configuration files as read-only
docker run -v /host/config:/app/config:ro app-image

# Logs with specific permissions
docker run -v /host/logs:/app/logs:Z app-image  # SELinux label
```

## Troubleshooting Network and Volume Issues

### Common Network Problems

**1. Container cannot reach internet:**
```bash
# Check DNS resolution
docker exec container nslookup google.com

# Check routing
docker exec container ip route

# Check firewall rules
sudo iptables -L DOCKER-USER
```

**2. Containers cannot communicate:**
```bash
# Verify same network
docker network inspect network-name

# Check container names
docker ps --format "table {{.Names}}\t{{.Networks}}"

# Test connectivity
docker exec container1 ping container2
```

**3. Port binding failures:**
```bash
# Check port usage
sudo netstat -tlnp | grep :8080

# Verify container ports
docker port container-name

# Check iptables rules
sudo iptables -L DOCKER
```

### Common Volume Problems

**1. Permission issues:**
```bash
# Check mount point permissions
docker exec container ls -la /mount/point

# Fix ownership (if needed)
docker exec container chown -R user:group /mount/point
```

**2. Mount failures:**
```bash
# Verify host path exists
ls -la /host/path

# Check mount syntax
docker inspect container | jq '.[0].Mounts'

# Test with simple mount
docker run -it -v /tmp:/tmp ubuntu bash
```

## Production Considerations

### 1. Network Performance

**Optimization strategies:**
- Use `--network host` for high-performance applications
- Minimize network hops between containers
- Consider container placement on multi-host setups
- Monitor network metrics with tools like `docker stats`

### 2. Security Hardening

**Network security:**
```bash
# Disable inter-container communication on default bridge
docker daemon --icc=false

# Use custom networks with encryption
docker network create --opt encrypted overlay-net

# Implement network policies
docker network create --internal internal-net
```

### 3. Monitoring and Logging

**Network monitoring:**
```bash
# Container network statistics
docker stats --format "table {{.Container}}\t{{.NetIO}}"

# Network traffic analysis
docker exec container netstat -i

# Log network events
journalctl -u docker.service | grep network
```

## Common Commands Reference

### Network Commands

**Network management:**
```bash
# List networks
docker network ls

# Create custom network
docker network create [OPTIONS] NETWORK

# Remove network
docker network rm NETWORK

# Connect container to network
docker network connect NETWORK CONTAINER

# Disconnect container from network
docker network disconnect NETWORK CONTAINER

# Inspect network details
docker network inspect NETWORK

# Cleanup unused networks
docker network prune
```

**Container networking:**
```bash
# Run with specific network
docker run --network NETWORK image

# Run with port mapping
docker run -p HOST_PORT:CONTAINER_PORT image

# Run with host networking
docker run --network host image

# Run without networking
docker run --network none image

# Run with multiple networks
docker run --network net1 image
docker network connect net2 container
```

### Volume Commands

**Volume management:**
```bash
# List volumes
docker volume ls

# Create volume
docker volume create [OPTIONS] VOLUME

# Remove volume
docker volume rm VOLUME

# Inspect volume
docker volume inspect VOLUME

# Cleanup unused volumes
docker volume prune
```

**Container volumes:**
```bash
# Named volume mount
docker run -v VOLUME:/path image

# Bind mount
docker run -v /host/path:/container/path image

# Read-only mount
docker run -v /host/path:/container/path:ro image

# Temporary filesystem
docker run --tmpfs /path image

# Volume with specific options
docker run -v volume:/path:Z image  # SELinux label
```

### Inspection and Debugging

**Container inspection:**
```bash
# View container networks
docker inspect container | jq '.[0].NetworkSettings'

# View container mounts
docker inspect container | jq '.[0].Mounts'

# Container resource usage
docker stats container

# Container processes
docker exec container ps aux

# Container logs
docker logs container
```

**Network debugging:**
```bash
# Test connectivity
docker exec container ping target

# Check open ports
docker exec container netstat -tlnp

# View routing table
docker exec container ip route

# DNS resolution test
docker exec container nslookup hostname
```

## Key Takeaways

### Networking Insights

1. **Default Bridge Limitations**: Containers on default bridge cannot communicate by name
2. **Custom Networks Benefits**: Enable name-based service discovery and better isolation
3. **Host Networking Trade-offs**: Maximum performance but reduced security and port conflicts
4. **Network Isolation**: Use separate networks for different application tiers

### Volume Management

1. **Data Persistence**: Volumes outlive containers and provide persistent storage
2. **Performance**: Named volumes generally perform better than bind mounts
3. **Portability**: Named volumes are more portable across different environments
4. **Security**: Use read-only mounts and proper permission management

### Best Practices Summary

1. **Use custom networks** for multi-container applications
2. **Implement network segmentation** for security
3. **Choose appropriate volume types** based on use case
4. **Monitor network and storage performance** in production
5. **Plan backup and recovery strategies** for persistent data

## Next Steps

- Learn about Docker Swarm mode for multi-host networking
- Explore advanced volume drivers and storage backends
- Study container orchestration with Kubernetes networking
- Implement monitoring and observability for containerized applications 



