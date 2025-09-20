# Docker Basics Tutorial - Part 1

This guide covers Docker installation and basic commands for getting started with Docker containerization.

## Table of Contents
- [Installation](#installation)
- [Verification](#verification)
- [Basic Docker Commands](#basic-docker-commands)
- [Working with Images](#working-with-images)
- [Running Containers](#running-containers)
- [Container Management](#container-management)
- [Image Management](#image-management)
- [Cleanup Operations](#cleanup-operations)

## Installation

### Ubuntu/Debian
```bash
# Update package index
sudo apt update

# Install required packages
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up the stable repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Add user to docker group (optional, to run without sudo)
sudo usermod -aG docker $USER
```

### CentOS/RHEL/Fedora
```bash
# Install Docker
sudo dnf install docker

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group (optional)
sudo usermod -aG docker $USER
```

## Verification

### Check Docker Version
```bash
$ docker -v
Docker version 28.4.0, build d8eb465
```

## Basic Docker Commands

### 1. Pulling Images
Pull the hello-world image to test Docker installation:
```bash
$ docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
Digest: sha256:54e66cc1dd1fcb1c3c58bd8017914dbed8701e2d8c74d9262e26bd9cc1642d31
Status: Image is up to date for hello-world:latest
docker.io/library/hello-world:latest
```

### 2. Listing Images
View all downloaded images:
```bash
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
ubuntu        latest    6d79abd4c962   10 days ago   78.1MB
nginx         latest    41f689c20910   5 weeks ago   192MB
hello-world   latest    1b44b5a3e06a   6 weeks ago   10.1kB
```

## Running Containers

### 1. Running Hello World
Test Docker installation with the hello-world container:
```bash
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### 2. Running Ubuntu Container
Run Ubuntu container in different modes:

**Non-interactive mode:**
```bash
$ docker run ubuntu
```

**Interactive mode with terminal:**
```bash
$ docker run -it ubuntu
root@e319addfc166:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@e319addfc166:/# env
HOSTNAME=e319addfc166
PWD=/
HOME=/root
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=00:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.avif=01;35:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:*~=00;90:*#=00;90:*.bak=00;90:*.crdownload=00;90:*.dpkg-dist=00;90:*.dpkg-new=00;90:*.dpkg-old=00;90:*.dpkg-tmp=00;90:*.old=00;90:*.orig=00;90:*.part=00;90:*.rej=00;90:*.rpmnew=00;90:*.rpmorig=00;90:*.rpmsave=00;90:*.swp=00;90:*.tmp=00;90:*.ucf-dist=00;90:*.ucf-new=00;90:*.ucf-old=00;90:
TERM=xterm
SHLVL=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
root@e319addfc166:/# exit
exit
```

## Container Management

### 1. Listing Containers
**Show all containers (running and stopped):**
```bash
$ docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED              STATUS                          PORTS     NAMES
e319addfc166   ubuntu        "/bin/bash"              About a minute ago   Exited (0) 15 seconds ago                 mystifying_heisenberg
9d45c666b231   ubuntu        "/bin/bash"              About a minute ago   Exited (0) About a minute ago             serene_shamir
9e5c082a6148   hello-world   "/hello"                 3 minutes ago        Exited (0) 3 minutes ago                  silly_saha
c0e299b70782   nginx         "/docker-entrypoint.…"   16 hours ago         Exited (0) 16 hours ago                   recursing_almeida
f39125c4966a   ubuntu        "bash"                   16 hours ago         Exited (0) 16 hours ago                   trusting_robinson
6c48ce81c0c8   hello-world   "/hello"                 17 hours ago         Exited (0) 17 hours ago                   reverent_shannon
```

**Show only running containers:**
```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### 2. Removing Containers
Remove specific containers by ID:
```bash
$ docker rm e319addfc166
e319addfc166
$ docker rm 9d45c666b231
9d45c666b231
$ docker rm 9e5c082a6148
9e5c082a6148
$ docker rm c0e299b70782
c0e299b70782
$ docker rm f39125c4966a
f39125c4966a
$ docker rm 6c48ce81c0c8
6c48ce81c0c8
```

**Note:** Attempting to remove by name when using ID is expected to fail:
```bash
$ docker rm ubuntu
Error response from daemon: No such container: ubuntu
```

## Image Management

### 1. Removing Images
**Attempting to remove an image in use will show an error:**
```bash
$ docker rmi ubuntu
Error response from daemon: conflict: unable to remove repository reference "ubuntu" (must force) - container 9d45c666b231 is using its referenced image 6d79abd4c962
```

**After removing all containers using the image:**
```bash
$ docker rmi ubuntu
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:353675e2a41babd526e2b837d7ec780c2a05bca0164f7ea5dbbd433d21d166fc
Deleted: sha256:6d79abd4c96299aa91f5a4a46551042407568a3858b00ab460f4ba430984f62c
Deleted: sha256:f9f52dc133e2af9188960e5a5165cafaa51657ef740ff20219e45a561d78c591

$ docker rmi nginx
Untagged: nginx:latest
Untagged: nginx@sha256:d5f28ef21aabddd098f3dbc21fe5b7a7d7a184720bc07da0b6c9b9820e97f25e
Deleted: sha256:41f689c209100e6cadf3ce7fdd02035e90dbd1d586716bf8fc6ea55c365b2d81
Deleted: sha256:6d03e4aefb16ea9e0d73cacb9a9fcb8f7fb3a806c41606600cab179aa381550f
Deleted: sha256:0951af45805a90677b83c3e3ae0ff7c1d6114b796206381914d1976df19d5af7
Deleted: sha256:dfa09858601e1678c6300924eeb880cbe0070bd41b3cc1bd8245f39104f1c8d7
Deleted: sha256:0502b463a609459d9b7bbd8044fd58e77f4309818da8841029dbf638a3a1581e
Deleted: sha256:39ffa72f273801580dbcd954e125f78fd5bb430896c32ab303667c5bbf154f39
Deleted: sha256:e57750b7e7ad7cb2436b3e9d3aafa338e89ee68f50d7e63bc09af37166ebc68e
Deleted: sha256:36f5f951f60a9fa1d51878e76fc16ba7b752f4d464a21b758a8ac88f0992c488

$ docker rmi hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:54e66cc1dd1fcb1c3c58bd8017914dbed8701e2d8c74d9262e26bd9cc1642d31
Deleted: sha256:1b44b5a3e06a9aae883e7bf25e45c100be0bb81a0e01b32de604f3ac44711634
Deleted: sha256:53d204b3dc5ddbc129df4ce71996b8168711e211274c785de5e0d4eb68ec3851
```

## Cleanup Operations

After removing all images, your Docker environment will be clean:
```bash
$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE

$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker ps 
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

## Key Takeaways

1. **Docker Installation**: Always verify installation with `docker -v`
2. **Image Management**: Use `docker pull` to download images, `docker images` to list them
3. **Container Lifecycle**: Containers can be run, stopped, and removed independently of images
4. **Interactive Mode**: Use `-it` flags for interactive terminal access
5. **Cleanup**: Remove containers before removing their associated images
6. **Resource Management**: Regular cleanup prevents disk space issues

## Next Steps

- Learn about Dockerfile creation
- Explore Docker networking
- Study volume management
- Practice with multi-container applications

