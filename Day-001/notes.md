# Session:1 - The Problem Before Docker
| Question | Answer |
|---|---|
| **What is an application?** | A program that performs a task for users. |
| **What does it need to run?** | Code, runtime, dependencies, OS, CPU, memory, and configuration. |
| **What is a runtime?** |A runtime is software that provides the environment required to execute application code. (python - python interpeter, java - JVM). |
| **What are dependencies?** | External libraries or software your application relies on. |
| **Why does software fail on another computer?** | Differences in runtime versions, dependencies, operating systems, configurations, or environment. |
| **What is "Works on my machine"?** | The application runs in the developer's environment but fails elsewhere because the environments differ. |

---
## Why can't I simply zip my project and send it?
A ZIP file only contains the application files, such as the source code, configuration files, and project assets. It does not include the runtime, installed dependencies, operating system environment, system libraries, or environment configuration required to run the application. As a result, the application may work on the developer's machine but fail on another system due to differences in software versions or missing dependencies. Docker solves this by packaging the application together with its runtime, dependencies, libraries, and configuration into a Docker image, allowing it to run consistently across different environments.
On thing to remember: docker does not package the CPU or RAM. The container uses the host machine's CPU, memory, kernel, and hardware. Docker packages the software environment, not the hardware.

# Session 2: Virtual Machines

## Physical Server

A physical server is a real computer designed to continuously provide resources and services to applications or other computers.

It contains:
- CPU
- RAM
- Storage
- Motherboard
- Network interfaces

---

## Hypervisor

A hypervisor is software that creates and manages Virtual Machines on a physical server.

It allocates physical resources such as:
- CPU
- Memory
- Storage
- Network

This allows multiple VMs to run independently on the same physical hardware.

---

## Virtual Machine (VM)

A Virtual Machine is a software-based computer that behaves like a physical computer.

A VM has:
- Virtual CPU
- Virtual memory
- Virtual storage
- Virtual network
- Guest OS
- Applications

---

## Guest OS

The Guest OS is the operating system running inside a Virtual Machine.

Example:

```
Physical Server
      ↓
  Hypervisor
      ↓
      VM
      ↓
 Ubuntu (Guest OS)
      ↓
 Application
````
## Resource Usage

Resource usage means how much of the available system resources are being consumed.

Common resources:

* CPU
* Memory
* Storage
* Network

Each VM consumes resources from the physical server.

---

## Boot Time

Boot time is the time required for a computer or VM to start its operating system and become ready to use.

VMs generally have longer startup times because the Guest OS must boot.

---

## Advantages of VMs

* Run multiple operating systems on one physical server.
* Improve physical hardware utilization.
* Provide isolation between environments.
* Useful for development and testing.
* Make cloning and backup easier.
* Useful for disaster recovery.

---

## Disadvantages of VMs

* Each VM requires its own Guest OS.
* Higher CPU, memory, and storage usage.
* Slower startup compared with containers.
* More OS maintenance and patching.
* More virtualization overhead.
* Fewer workloads can fit on the same hardware compared with containers.

---

## VM Architecture

```text
Physical Server
      ↓
  Hypervisor
      ↓
 ┌────┼────┐
 ↓    ↓    ↓
VM1  VM2  VM3
 ↓    ↓    ↓
OS   OS   OS
 ↓    ↓    ↓
App  App  App
```

---

## Key Mental Model

```text
Physical Hardware
       ↓
   Hypervisor
       ↓
 Virtual Machines
       ↓
    Guest OS
       ↓
  Applications
```
![virtualization](/screenshots/day-1_screenshots/virtualization.png)

# Session 3: Containers

### Containerization

Containerization is the practice of packaging an application together with its dependencies and running it in an isolated container environment.

```text
Application
     +
Dependencies
     ↓
  Container
  ```

### Why Containers Are Lightweight

Containers share the host operating system's kernel instead of requiring a separate Guest OS for each application.

Because containers share the host kernel, they generally:

- Use less memory.
- Require less storage.
- Start much faster.
- Allow more containers to run on the same hardware.

### Kernel Sharing

A container does not contain its own kernel.

Containers use the host operating system's kernel instead of having their own kernel. The container provides the application and its user-space files, while the host kernel handles things such as:

- System calls
- Process scheduling
- Networking
- Resource management



## Isolation

Although containers share the same host kernel, they are isolated from each other.

Container isolation can separate:
- Processes
- Filesystems
- Networking
- Resources
```
Host Kernel
     ↓
 ┌─────────────┐
 │             │
Container A  Container B
 │             │
App A         App B
```
## Dockerfile

A Dockerfile is a text file containing instructions that tell Docker how to build a Docker image.

Common instructions include:

- FROM
- RUN
- COPY
- CMD

Example:
```
FROM ubuntu
COPY app.py /app/
CMD ["python", "/app/app.py"]
```
### Docker Image

A Docker image is a read-only template used to create containers.

An image can contain:
- Application code
- Runtime
- Libraries
- Dependencies
- Required files
- Configuration
```
Docker Image
     ↓
Creates
     ↓
Container
```
## Docker Container

A Docker container is an isolated instance created from a Docker image.

The application's process runs inside the container.
```
Docker Image
     ↓
Docker Container
     ↓
Application Process
```
A container also has a writable container layer on top of the image's read-only layers.
![](/screenshots/day-1_screenshots/containerization.png)

# Session:4 - Basic Docker Hands On
Check commands.md file

# Session:5 - understand docker Architecture

Docker follows a Client-Server Architecture.

the main componnets are:
- Docker Client
- Docker 
## Docker Client
   The Docker Client is the command-line interface that you interact with. 
   
   The client does not create containers. Its only job is to accept your command and send it to docker daemon. 
   Examples:
   docker run nginx
   docker build -t myapp .
   docker pull ubuntu
   docker ps
   
   When you execute any Docker command: docker run nginx. It sends an API request to the Docker Daemon.
   
## Docker Daemon
Docker Daemon(dockerd) is the core Docker service. 
     
Docker Daemon is a background service responsible for creating and managing Docker objects such as images, containers, networks, and volumes.
     
## REST API
Communication interface between Docker Client and Docker Daemon. 

The Docker Client sends an HTTP request to the Docker REST API (for example, GET /containers/json) through the Unix socket.

## So How Does Docker Client Talk To Docker Daemon?
The client and daemon communicate using the **Docker REST API**
On Linux this communication happened through a Unix Socket located at: /var/run/docker.sock

## What is Unix socket?

A Unix socket is a special file provided by the Linux kernel that allows two processes running on the same machine to communicate efficiently.

The socket acts as the communication channel between the client and the daemon.

Docker creates:

`/var/run/docker.sock`

Verify:

```bash
ls -l /var/run/docker.sock
````

Output:

```text
srw-rw---- 1 root docker …
```

> **Notice:** `s` at the beginning means **Socket**, not a regular file.

This socket is the communication channel between:

```text
Docker Client
      ↕
Docker Daemon
```

For example, you type:

```bash
docker ps
```

### Step 1: Docker Client starts

```text
Terminal
   ↓
Docker Client
```

### Step 2: Client sends a REST API request

The Docker Client sends a REST API request through the Unix socket:

```text
/var/run/docker.sock
```

### Step 3: Client sends the request

The request is:

```text
"Show me all running containers"
```

### Step 4: Docker Daemon receives the request

The Docker Daemon receives the request and checks its container database/state.

### Step 5: Daemon sends the response back

The Docker Daemon sends the information about the running containers back:

```text
Container 1
Container 2
Container 3
```

### Step 6: Docker Client prints the output

The Docker Client receives the response and prints the output in the terminal:

```text
CONTAINER ID
...
```

### Complete Flow

```text
User types:
docker ps
    │
    ▼
Terminal
    │
    ▼
Docker Client
    │
    │ REST API request
    │ "Show me all running containers"
    ▼
/var/run/docker.sock
    │
    ▼
Docker Daemon
    │
    │ Checks container state/database
    ▼
Running Containers
    │
    │ Response
    ▼
/var/run/docker.sock
    │
    ▼
Docker Client
    │
    ▼
Terminal
    │
    ▼
CONTAINER ID
...
```

### Key Point

The Docker Client does not directly manage containers.

It communicates with the Docker Daemon through the Unix socket:

```text
/var/run/docker.sock
```

The **Docker Daemon** receives the request, performs the actual work, and sends the result back to the Docker Client

## Docker Engine
- Docker engine and docker daemon are not same thing. 
- Docker engine is a complete platform installed on your machine. 
- It includes docker client, docker daemon, REST API
- Together they form the docker engine.

## Docker Hub
- Default public image registry. 
- Stores Docker images. 
## Image Registry
- Repository for storing Docker images. 
- Examples: Docker Hub, Amazon ECR, Azure ACR, Harbor. 

## Docker Image

A **Docker Image** is a **read-only template** that contains everything needed to run an application.

### What does a Docker Image contain?

A Docker Image can contain:

- **Application Code**
- **Libraries**
- **Dependencies**
- **Runtime**
- **Configuration**

Think of a Docker Image as a **package that is ready to run**.

### Example

Suppose you have a **Python application**.

To run it, you need:

- Python
- Application Code
- Required Libraries
- OS Dependencies

A Docker Image **bundles all of these together** into a single package.

```text
Python Application
       │
       ├── Application Code
       ├── Python Runtime
       ├── Required Libraries
       ├── Dependencies
       └── Configuration
                │
                ▼
          Docker Image
````

### Purpose

A Docker Image is used to **create containers**.

```text
Docker Image
     │
     │ creates
     ▼
Container
```
## Docker Container
- A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. 
- A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.
### Container contains:
- Running Process
- Memory Allocation
- Network Interface
- Writable Layer

## Docker Volume
- A Docker volume is a place outside the container's writable layer where data can be stored persistently.
### Without volume:
```
             Container
        ┌─────────────────┐
        │ Writable Layer  │
        │                 │
        │ notes.txt       │
        │ database data   │
        └─────────────────┘
                 │
          Container deleted
                 │
                 ▼
             Data ❌ 
```
### With Volume:
```
             Container
        ┌─────────────────┐
        │ Writable Layer  │
        │                 │
        │ Application     │
        └────────┬────────┘
                 │
                 │ mounted
                 ▼
        ┌─────────────────┐
        │ Docker Volume   │
        │                 │
        │ database data   │
        │ notes.txt       │
        └─────────────────┘
```

## Docker Network
- Provides communication between containers, the host, and external systems. 
- Enables port mapping and container-to-container communication.
## What Happens When You Run `docker run nginx`

```text
You
 |
 ▼
Docker Client
 |
 | REST API
 ▼
Docker Daemon
 |
 |
 ├── Check local image
 |
 ├── If missing
 |      |
 |      ▼
 |   Docker Hub
 |      |
 |      ▼
 |   Download image
 |
 ├── Create Container
 |
 ├── Attach Network
 |
 ├── Attach Volumes (if specified)
 |
 └── Start nginx process
```
## Docker Architecture 

```text
+---------------------+
|        User         |
+---------------------+
          |
          | docker run nginx
          |
+---------v-----------+
|    Docker Client   |
+---------------------+
          |
          | REST API
          |
+---------v-----------+
|    Docker Daemon   |
+---------------------+
          |
          |
+---------+---------------------------+
|                 |                   |
|                 |                   |
+----v------+ +----v---------+ +------v------+
|   Image   | |   Volumes    | |   Networks  |
| Registry  | | Persistence  | | Communication|
| (Docker   | |    Data      | |              |
|   Hub)    | |              | |              |
+----+------+ +--------------+ +------+-------+
     |                               |
     |                               |
+----v------+                         |
|  Docker   |                         |
|  Image    |                         |
+----+------+                         |
     |                               |
     |                               |
+----v------+                         |
| Container |<------------------------+
+-----------+
```
# Session:6 - Docker In Real World

## Why Docker became famous?
Docker became popular because it solved the "it works on my machine" problem by packaging an application together with its runtime, dependencies, and configuration into a portable image. It provides consistent environments across development, testing, and production, is lightweight compared to virtual machines, starts quickly, simplifies deployment, and integrates well with CI/CD pipelines.

## Why Netflix, Google, Amazon adopted containers?
Companies like Netflix, Google, and Amazon adopted containers because they provide consistent environments, efficient resource utilization, fast deployments, easy scalability, support for microservices, seamless CI/CD integration, and quick rollbacks. Containers are lightweight compared to virtual machines, allowing organizations to run more applications on the same hardware while deploying software reliably across development, testing, and production.




