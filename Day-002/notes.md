
# Docker Architecture - What Happens Internally When You Run a Docker Command?

## Mission

Understand Docker's architecture and the complete execution flow of a Docker command.

## Learn

- Docker Engine
- Docker Client
- Docker Daemon
- Docker REST API
- Unix Socket
- PID 1 (Main Process)

---

# Docker Engine

Docker Engine is the complete Docker platform installed on your machine.

It consists of:

- Docker Client (CLI)
- Docker Daemon (dockerd)
- Docker REST API

These components work together to build, run, and manage Docker containers.

---

# Example Command

```bash
docker run -d -p 80:80 nginx
```

---

# Step 1: Docker Client Receives the Command

When you execute a Docker command in the terminal, the Docker Client (CLI) receives it.

Examples of Docker CLI commands:

```bash
docker run
docker build
docker pull
docker ps
docker images
```

The Docker Client **does not create containers**.

Its job is to:

- Parse the command
- Send a request to the Docker Daemon
- Display the response received from the daemon

---

# Step 2: Docker Client Sends a REST API Request

The Docker Client parses the command and sends it as a Docker REST API request to the Docker Daemon.

On Linux, this communication usually happens through the Unix Domain Socket:

```text
/var/run/docker.sock
```

Communication Flow:

```text
Terminal
    │
    ▼
Docker Client
    │
    ▼
REST API Request
    │
    ▼
Unix Socket (/var/run/docker.sock)
    │
    ▼
Docker Daemon
```

### Why does Docker use a Unix Socket?

Both the Docker Client and Docker Daemon usually run on the same Linux machine.

Using a Unix Domain Socket is:

- Faster than TCP communication
- More secure (protected by Linux file permissions)
- Does not require network configuration

---

# Step 3: Docker Daemon Receives the Request

The Docker Daemon (`dockerd`) is the background service responsible for executing Docker operations.

When it receives the request, it starts processing it.

The daemon is responsible for:

- Pulling images
- Building images
- Creating containers
- Starting and stopping containers
- Managing networks
- Managing volumes

---

# Step 4: Check for the Image

The Docker Daemon checks whether the required image exists locally.

Example:

```bash
docker run nginx
```

Docker checks:

```text
Is the nginx image available locally?
```

### If the image exists

Docker uses the local image.

### If the image does not exist

Docker:

- Connects to Docker Hub (or another registry)
- Downloads the image
- Stores it in the local image cache

---

# Step 5: Create the Container

Once the image is available, Docker creates a new container.

Internally, Docker performs several tasks:

- Creates a writable container layer
- Creates Linux namespaces for isolation
- Creates cgroups to limit CPU and memory usage
- Configures container networking
- Mounts volumes (if specified)

---

# Step 6: Start the Main Process (PID 1)

A Docker container is simply a Linux process running in isolation.

The **first process** started inside the container is called **PID 1**.

Docker starts this process using the **ENTRYPOINT** and **CMD** instructions from the Dockerfile.

Examples:

| Container | Main Process (PID 1) |
|----------|----------------------|
| Nginx | nginx |
| Ubuntu | /bin/bash |
| Python App | python app.py |
| Java App | java Main |

Docker continuously monitors PID 1.

As long as PID 1 is running, the container remains in the **Running** state.

If PID 1 exits, Docker automatically stops the container.

### Example

```bash
docker run ubuntu echo "Hello"
```

Output:

```text
Hello
```

Internally:

```text
echo completed
      │
      ▼
PID 1 exited
      │
      ▼
Container stopped
```

---

# Step 7: Docker Daemon Sends the Response

After successfully creating the container, the Docker Daemon sends a response back to the Docker Client.

Example:

```text
4f5a7b8c9d...
```

This is the **Container ID**.

---

# Step 8: Docker Client Displays the Result

The Docker Client receives the response from the Docker Daemon and prints it to the terminal.

---

# Complete Flow

```text
User
   │
   ▼
docker run nginx
   │
   ▼
Docker Client (CLI)
   │
   ▼
REST API Request
   │
   ▼
Unix Socket (/var/run/docker.sock)   ← Linux
   │
   ▼
Docker Daemon (dockerd)
   │
   ▼
Check Local Image
   │
   ▼
Image Found?
   ├──────────── Yes ───────────► Create Container
   │
   └──────────── No
                  │
                  ▼
          Docker Hub / Registry
                  │
                  ▼
           Download Image
                  │
                  ▼
         Store Image Locally
                  │
                  ▼
          Create Container
                  │
                  ▼
     Create Linux Namespaces
                  │
                  ▼
          Create cgroups
                  │
                  ▼
       Configure Networking
                  │
                  ▼
      Mount Volumes (if any)
                  │
                  ▼
      Execute ENTRYPOINT + CMD
                  │
                  ▼
     Start Main Process (PID 1)
                  │
                  ▼
         Container Running
                  │
                  ▼
Response Sent to Docker Client
                  │
                  ▼
          Terminal Output
```

---

# Interview Questions

## What happens internally when you run a Docker command?

**Answer:**

When a user executes a Docker command, the Docker Client receives it and converts it into a REST API request. On Linux, this request is typically sent through the Unix socket `/var/run/docker.sock` to the Docker Daemon. The daemon checks whether the required image exists locally; if not, it downloads it from Docker Hub or another registry. It then creates the container by setting up the writable layer, Linux namespaces, cgroups, networking, and any requested volumes. Finally, it starts the container's main process (PID 1) using the ENTRYPOINT and CMD instructions, and sends the result back to the Docker Client, which displays the output to the user.

---

## What is PID 1 in Docker?

**Answer:**

PID 1 is the main process running inside a Docker container. It is the first process started when the container launches. Docker continuously monitors this process. As long as PID 1 is running, the container remains running. When PID 1 exits, Docker automatically stops the container.

---

# Key Takeaways

- Docker Engine consists of the Docker Client, Docker Daemon, and Docker REST API.
- The Docker Client never creates containers directly; it sends REST API requests to the Docker Daemon.
- On Linux, the Client and Daemon usually communicate through the Unix socket `/var/run/docker.sock`.
- The Docker Daemon is responsible for pulling images, creating containers, configuring namespaces, cgroups, networking, and volumes.
- Every container has one main process called **PID 1**.
- The container remains running only while **PID 1** is running.

# Where Do Docker Images Come From?

## Mission

Understand where Docker images are stored and how Docker retrieves them.

## Learn

- Docker Registry
- Docker Hub
- Repository
- Tags
- Image Cache

---

# Docker Registry

A **Docker Registry** is a service that stores and distributes Docker images.

Think of it as a **warehouse** that stores Docker images.

Examples:

- Docker Hub (Public)
- Amazon Elastic Container Registry (ECR)
- Azure Container Registry (ACR)
- Google Artifact Registry
- Harbor

---

# Docker Hub

Docker Hub is Docker's **default public image registry**.

When you run:

```bash
docker pull nginx
```

Docker automatically downloads the image from **Docker Hub**, unless another registry is specified.

Docker Hub contains millions of images such as:

- nginx
- ubuntu
- mysql
- python
- redis
- node

---

# Repository

A **repository** is a collection of different versions (tags) of the same Docker image.

Think of a repository as a **folder** that contains multiple versions of an application.

Example:

Repository:

```text
python
```

Available Tags:

```text
python:3.9
python:3.10
python:3.11
python:latest
```

Another example:

Repository:

```text
nginx
```

Available Tags:

```text
nginx:1.24
nginx:1.25
nginx:latest
```

---

# Tags

A **tag** identifies a specific version of a Docker image.

Syntax:

```text
image_name:tag
```

Examples:

```text
ubuntu:22.04
python:3.11
mysql:8.0
nginx:latest
```

If no tag is specified, Docker automatically uses:

```text
latest
```

Example:

```bash
docker pull nginx
```

is equivalent to:

```bash
docker pull nginx:latest
```

---

# Hands-on Commands

## Search Images

```bash
docker search nginx
```

**Purpose:**

Search Docker Hub for images related to **nginx**.

This helps identify:

- Official images
- Community images
- Available repositories

---

## List Local Images

```bash
docker images
```

Displays all Docker images currently stored on your local machine.

---

## Pull an Image

```bash
docker pull nginx
```

Downloads the **nginx** image from Docker Hub and stores it locally.

---

## Inspect an Image

```bash
docker image inspect nginx
```

Displays detailed metadata about the image in JSON format.

Information includes:

- Image ID
- Repository Tags
- Image Layers
- Creation Time
- Environment Variables
- Entrypoint
- Default Command (CMD)
- Operating System
- Architecture

---

# Observations

### Where did Docker download the image from?

Docker downloaded the image from **Docker Hub**, specifically the official **NGINX repository**.

---

### Where is the image stored?

The image is stored in Docker's **local image storage**.

On Linux, Docker typically stores images under:

```text
/var/lib/docker
```

---

### What is the default tag?

The default tag is:

```text
latest
```

---

# Reflection

## Why doesn't Docker download the nginx image every time?

Docker does not download the image every time because images are designed to be **reusable**.

Downloading the same image repeatedly would:

- Waste network bandwidth
- Increase deployment time
- Consume unnecessary resources

Instead, Docker stores downloaded images in a **local image cache** and reuses them to create new containers.

Docker downloads an image again only when:

- The image is not available locally
- A different tag/version is requested
- You explicitly pull a newer version

---

# Image Cache

Docker Image Cache is the local storage where Docker saves downloaded images and image layers.

Before downloading an image, Docker first checks the local image cache.

If the image already exists:

- Docker reuses the local image
- No download occurs

Benefits of Image Cache:

- Faster container creation
- Reduced deployment time
- Saves network bandwidth
- Optimizes storage by reusing image layers
- Allows applications to start quickly

---

# Interview Questions

## What is a Docker Registry?

A Docker Registry is a service used to store and distribute Docker images. Examples include Docker Hub, Amazon ECR, Azure Container Registry, Google Artifact Registry, and Harbor.

---

## What is Docker Hub?

Docker Hub is Docker's default public image registry where users can store, share, and download Docker images.

---

## What is a Repository?

A repository is a collection of different versions (tags) of the same Docker image.

---

## What is a Tag?

A tag identifies a specific version of a Docker image.

Example:

```text
python:3.11
```

Here:

- Repository: `python`
- Tag: `3.11`

---

## What is Image Cache?

Docker Image Cache is Docker's local storage for downloaded images and image layers. Docker checks this cache before downloading an image. If the image already exists locally, Docker reuses it instead of downloading it again.

---

# Key Takeaways

- A Docker Registry stores and distributes Docker images.
- Docker Hub is Docker's default public registry.
- A Repository contains multiple versions of the same image.
- A Tag identifies a specific version of an image.
- Docker stores downloaded images in a local image cache.
- Images are downloaded only when required and reused for future containers.



