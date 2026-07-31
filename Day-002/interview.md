Interview Preparation
Without Google answer
# What happens internally when Docker receives docker run?
If image exists docker will continue, if not docker downloads the image from docker hub and stores in local image cache. 
Docker now create a new container from the image without modifying it creates a writable container layer.
Docker creates namespaces to isolate the container. Now the container have its own processes, hostname, filesystem and network even though it is sharing the host kernel.
Docker creates cgropus to control resources like CPU, memory, disk I/O, PIDs. Without cgroups one container could consume all system memory.
Then docker adds writable layer above the image.
    Container
    
    Writable Layer
    ────────────────
    
    Image Layer 4
    ────────────────
    
    Image Layer 3
    ────────────────
    
    Image Layer 2
    ────────────────
    
    Image Layer 1
    
Docker attaches the container to a network. It assigns IP Address, MAC Address, Routing, DNS.
Mount volumes if you want. If you used: docker run -v myvol:/app nginx
Docker mounts the volume.

The application starts and nginx becomes PID 1 inside the container. This process keeps the container alive.

                docker run nginx
                       │
                       ▼
              Docker Client (CLI)
                       │
         REST API (/var/run/docker.sock)
                       │
                       ▼
               Docker Daemon
                       │
        Check Local Image Cache
               │             │
         Image Exists?       No
               │             │
               │             ▼
               │       Download from
               │        Docker Hub
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
      Add Writable Layer
               │
               ▼
      Configure Networking
               │
               ▼
        Mount Volumes
               │
               ▼
      Execute ENTRYPOINT
               │
               ▼
         Execute CMD
               │
               ▼
      Start Main Process (PID 1)
               │
               ▼
      Return Container ID

When I execute docker run nginx, the Docker Client sends a REST API request to the Docker Daemon through the Docker socket. The daemon first checks whether the required image exists in the local image cache. If it is not available, the daemon downloads it from a registry such as Docker Hub and stores it locally. Next, Docker creates a container by adding a writable layer on top of the image's read-only layers. It then creates Linux namespaces for isolation, configures cgroups to control resource usage, sets up networking, mounts any specified volumes, and starts the container by executing the ENTRYPOINT followed by the CMD. The application started by these instructions becomes the container's main process (PID 1). Finally, the daemon returns the container ID to the Docker Client, which displays it to the user.

# Difference between docker run and docker create?
The core difference: docker run = create + start, while docker create = only create (do not start).
What docker run does
    • Creates a new container from the given image.
    • Starts it immediately so the main process begins running.
    • If the image is missing locally, it can pull it from a registry first.linuxhandbook+1
What docker create does
    • Creates a new container from the given image without starting it.
    • Prepares the writable container layer and metadata, then prints the container ID.
    • You must later use docker start <container> to actually run it.

# Difference between docker create and docker start?
docker create makes a new container from an image but does not start it, while docker start starts an existing container that was already created.docker+1
Difference
    • docker create: creates the container’s writable layer and metadata, then leaves it stopped.docker+1
    • docker start: starts that already-created stopped container; it does not create a new one.


# Why are Docker Images read-only?

# What are Docker Layers?
# Why are layers useful?
# Can multiple containers use one image?
# Where are images downloaded from?
# What is Docker Hub?
# Can Docker run without Internet?
