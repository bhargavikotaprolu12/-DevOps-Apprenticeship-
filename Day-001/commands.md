# Session 4 — Basic Docker Hands-On

## Session Goal

The goal of this session is to move from understanding Docker concepts to actually using Docker commands and observing what Docker does internally.

For every command, think about these four questions:

1. **Why did I run this?**
2. **What happened internally?**
3. **What files changed?**
4. **Did Docker create an image or a container?**

> **Important:** Docker stores images, containers, metadata, and writable layers in its own managed storage. This is different from modifying your application's source files.

---

# 1. `docker version`

### Why did I run this?

To check the installed Docker version and verify that the Docker client and Docker server/daemon are available.

### What happened internally?

The Docker CLI requested version information from the Docker daemon.

Docker returned information about the client and server versions.

### What files changed?

No application files were changed.

This is a read-only operation.

### Did Docker create an image or container?

**No.**

---

# 2. `docker info`

### Why did I run this?

To view information about the Docker environment, such as:

- Number of containers
- Number of images
- Docker storage driver
- Docker server information
- Runtime information
- Docker configuration

### What happened internally?

The Docker CLI sent a request to the Docker daemon.

The daemon returned information about the Docker environment it manages.

### What files changed?

No application files were changed.

This is a read-only operation.

### Did Docker create an image or container?

**No.**

---

# 3. `docker images`

### Why did I run this?

To see the Docker images currently stored locally on my machine.

### What happened internally?

The Docker client requested the list of images from the Docker daemon.

The daemon queried its local image store and returned the available images.

### What files changed?

No application files were changed.

The command only displays information.

### Did Docker create an image or container?

No.

![](/screenshots/day-1_screenshots/command_dockerimages.png)
---

# 4. `docker ps`

### Why did I run this?

To see the currently **running containers** managed by Docker.

It can show information such as:

- Container ID
- Container name
- Image used
- Status
- Ports
- Uptime

### What happened internally?

The Docker client sent a request to the Docker daemon.

The daemon returned information about the containers that are currently running.

### What files changed?

No application files were changed.

This is a read-only operation.

### Did Docker create an image or container?

**No.**

---

# 5. `docker ps -a`

### Why did I run this?

To see **all containers** managed by Docker, including:

- Running containers
- Stopped containers
- Exited containers
- Created containers

The `-a` flag means **all**.

### What happened internally?

The Docker client sent a request to the Docker daemon.

The daemon returned metadata for all containers it manages.

### What files changed?

No application files were changed.

This is a read-only operation.

### Did Docker create an image or container?

**No.**

---

# 6. `docker pull hello-world`

### Why did I run this?

To download the `hello-world` image from Docker Hub so that I can use it to create a container.

`docker pull` **does not run a container**.

### What happened internally?

Docker:

1. Checks whether the requested image is already available locally.
2. If it is not available, contacts the configured container registry.
3. Downloads the required image layers.
4. Stores those layers in Docker's local image store.
5. Makes the image available for creating containers.

### What files changed?

My application/source files were not modified.

Docker may update its own internal image storage and metadata.

### Did Docker create an image or container?

Docker **downloaded an already-built image**.

It did not build a new image and did not create a container.

---

# 7. `docker run hello-world`

### Why did I run this?

To verify that Docker is working correctly by running the `hello-world` container.

### What happened internally?

Docker:

1. Checks whether the `hello-world` image exists locally.
2. If it is missing, Docker pulls it from the registry.
3. Creates a new container from the image.
4. Adds the container's writable layer.
5. Starts the container.
6. Runs the image's configured command.
7. The program prints its message.
8. The main process exits.
9. The container stops.

The important part is:

```text
Image
  ↓
Container
  ↓
Main Process
  ↓
Process exits
  ↓
Container stops
````

### What files changed?

My application/source files were not modified.

Docker may update its own image storage and create container metadata and writable-layer data.

### Did Docker create an image or container?

**Container.**

If the image was not already present, Docker also **pulled the existing image**.

---

# 8. `docker run ubuntu`

### Why did I run this?

To start a container using the Ubuntu image and observe what happens when the container's default process exits.

### What happened internally?

Docker:

1. Checks whether the Ubuntu image exists locally.
2. Pulls the image if it is not available.
3. Creates a new container from the image.
4. Creates the container's writable layer.
5. Sets up the container filesystem and isolation.
6. Starts the image's default command.
7. The command exits.
8. The container stops.

### What files changed?

My application/source files were not modified.

Docker may download image layers and create container metadata and writable-layer data in its own storage.

### Did Docker create an image or container?

**Container.**

If the Ubuntu image was missing locally, Docker also **pulled the existing image**.

---

# 9. `docker run -it ubuntu bash`

### Why did I run this?

To start an Ubuntu container and get an interactive Bash shell inside it.

The options mean:

* `-i` → keeps standard input open
* `-t` → allocates a terminal
* `ubuntu` → image to create the container from
* `bash` → command to run inside the container

### What happened internally?

Docker:

1. Checks whether the Ubuntu image exists locally.
2. Pulls it if necessary.
3. Creates a new container from the image.
4. Creates the container's writable layer.
5. Sets up the container filesystem and isolation.
6. Starts `bash` as the container's main process.
7. Connects the terminal to the Bash process.

This allows me to interact directly with the container.

### What files changed?

My application/source files were not modified.

Docker may download image layers and create container metadata and a writable layer in its own storage.

### Did Docker create an image or container?

**Container.**

---

# 10. `docker run nginx`

### Why did I run this?

To start an Nginx web server inside a Docker container.

### What happened internally?

Docker:

1. Checks whether the Nginx image exists locally.
2. Pulls it if necessary.
3. Creates a new container from the Nginx image.
4. Creates the container's writable layer.
5. Sets up the container's isolated environment.
6. Starts the Nginx process.
7. Nginx continues running as the main process.

Because the main process continues running, the container remains in the **running** state.

### What files changed?

My application/source files were not modified.

Docker may store image layers, container metadata, and writable-layer data in its own storage.

### Did Docker create an image or container?

**Container.**

If the image was missing locally, Docker also **pulled the existing Nginx image**.

---

# 11. `docker run httpd`

### Why did I run this?

To start an Apache HTTP Server container using the official `httpd` image.

This allows me to quickly run and test an Apache web server.

### What happened internally?

Docker:

1. Checks whether the `httpd` image exists locally.
2. Pulls the image from the registry if necessary.
3. Creates a new container from the image.
4. Sets up the container's filesystem and isolation.
5. Starts the Apache HTTP Server process.

Because the main Apache process continues running, the container normally remains running.

### What files changed?

My application/source files were not modified.

Docker may download image layers and create container metadata and writable-layer data in its own storage.

### Did Docker create an image or container?

**Container.**

If the image was missing locally, Docker also **pulled the existing image**.

---

# 12. `docker run alpine`

### Why did I run this?

To start a container using the lightweight Alpine Linux image.

### What happened internally?

Docker:

1. Checks the local image store for the Alpine image.
2. Pulls the image if it is missing.
3. Creates a new container from the image.
4. Creates the container's writable layer.
5. Sets up the container filesystem and isolation.
6. Starts the image's default command.

If the default command exits, the container also stops.

### What files changed?

My application/source files were not modified.

Docker may download image layers and create container metadata and writable-layer data in its own storage.

### Did Docker create an image or container?

**Container.**

If the Alpine image was missing locally, Docker also **pulled the existing image**.

---

# 13. `docker inspect hello-world`

### Why did I run this?

To view detailed metadata about an existing Docker object.

Depending on what object is specified, `docker inspect` can show information such as:

* Configuration
* Environment
* Network settings
* Mounts
* IDs
* Architecture
* Entrypoint
* CMD
* Other metadata

### What happened internally?

Docker looks up the object specified by the command and returns its stored metadata.

### What files changed?

No application files changed.

This is a read-only operation.

### Did Docker create an image or container?

**No.**

It only displays information about an existing Docker object.

> **Note:** `docker inspect hello-world` may inspect an image or container depending on what Docker object named `hello-world` exists.

---

# 14. `docker history nginx`

### Why did I run this?

To see how the Nginx image was built layer by layer.

It can show information about:

* Image layers
* Build instructions
* Commands associated with layers
* Layer sizes
* Image history

### What happened internally?

Docker reads the metadata stored for the Nginx image and displays its image history.

### What files changed?

No application files changed.

This command is read-only.

### Did Docker create an image or container?

**No.**

It only displays the build history of an existing image.

---

# 15. Commands and What They Do

| Command                      | Main Purpose                        | Creates Image? | Creates Container? |
| ---------------------------- | ----------------------------------- | -------------: | -----------------: |
| `docker version`             | Show Docker version information     |             No |                 No |
| `docker info`                | Show Docker environment information |             No |                 No |
| `docker images`              | List local images                   |             No |                 No |
| `docker ps`                  | List running containers             |             No |                 No |
| `docker ps -a`               | List all containers                 |             No |                 No |
| `docker pull hello-world`    | Download an existing image          |             No |                 No |
| `docker run hello-world`     | Create and run a container          |            No* |                Yes |
| `docker run ubuntu`          | Create and run Ubuntu container     |            No* |                Yes |
| `docker run -it ubuntu bash` | Create interactive Ubuntu container |            No* |                Yes |
| `docker run nginx`           | Create and run Nginx container      |            No* |                Yes |
| `docker run httpd`           | Create and run Apache container     |            No* |                Yes |
| `docker run alpine`          | Create and run Alpine container     |            No* |                Yes |
| `docker inspect`             | Display object metadata             |             No |                 No |
| `docker history nginx`       | Display image layer history         |             No |                 No |

`*` If the requested image is not already available locally, `docker run` may first **pull the existing image** from a registry. It does not build a new image.

---

# 16. Important Pattern Observed

The most important pattern from this hands-on session is:

```text
docker pull
     ↓
Download existing image
     ↓
Local Image Store
```

while:

```text
docker run
     ↓
Check for image
     ↓
Pull image if necessary
     ↓
Create container
     ↓
Start main process
     ↓
Container runs
```

And if the main process exits:

```text
Container
    ↓
Main Process exits
    ↓
Container becomes Exited
```

---

# 17. Key Takeaways

* `docker images` shows locally available images.
* `docker ps` shows running containers.
* `docker ps -a` shows all containers, including stopped/exited containers.
* `docker pull` downloads an existing image but does not create a container.
* `docker run` creates a container from an image and starts it.
* `docker run` may pull the image first if it is not available locally.
* A container's lifecycle is closely tied to its **main process**.
* If the main process exits, the container stops.
* `-it` is useful for interactive terminal sessions.
* `docker inspect` displays detailed Docker object metadata.
* `docker history` shows the layers/history of an image.
* Docker manages its own internal storage for images, containers, metadata, and writable layers.
* These commands do not normally modify your application's source files.

