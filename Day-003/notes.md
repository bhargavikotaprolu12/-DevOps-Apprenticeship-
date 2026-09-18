# Session 1 — What Is a Dockerfile?

## Why Does a Dockerfile Exist?

Manually setting up an application on every server is:

- Slow
- Error-prone
- Inconsistent
- Difficult to reproduce
- Difficult to maintain

For example, running a Python application on a new server may require:

```text
Install Python
      ↓
Install the correct Python version
      ↓
Install application dependencies
      ↓
Copy application code
      ↓
Configure the application
      ↓
Start the application
````

Doing this manually across many servers can result in different environments:

```text
Server 1 → Python 3.11
Server 2 → Python 3.12
Server 3 → Missing dependency
Server 4 → Different configuration
```

This creates the same problem we learned earlier:

> **"It works on my machine."**

A Dockerfile solves this by defining the image-building process as a repeatable set of instructions.

---

## What Is a Dockerfile?

A **Dockerfile** is a text file containing instructions that Docker uses to build a Docker image.

Think of it as a **recipe or blueprint for building an image**.

Example:

```Dockerfile
FROM python:3.11

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

These instructions describe:

* Which base image to start from
* Which working directory to use
* Which application files to copy
* Which commands to execute during the build
* Which default command should run when a container starts

---

## Why Dockerfiles Exist

The main purpose of a Dockerfile is **repeatability and reproducibility**.

### Without a Dockerfile

```text
Manual Setup
     ↓
Different people perform different steps
     ↓
Different environments
     ↓
Inconsistent results
```

### With a Dockerfile

```text
Dockerfile
     ↓
Same build instructions
     ↓
Docker Image
     ↓
Repeatable environment
```

The instructions are written once and can be used repeatedly to build the image.

### Key Benefits

* **Reproducibility** — the same build instructions can be used again.
* **Consistency** — environments can be built using the same instructions.
* **Automation** — Docker performs the build steps automatically.
* **Version control** — the Dockerfile can be stored in Git.
* **Maintainability** — environment changes can be tracked as code.

---

## Dockerfile as Declarative Configuration

A Dockerfile is generally considered **declarative configuration** because it describes how the image should be constructed rather than requiring someone to manually perform every setup step.

For example:

```Dockerfile
FROM python:3.11

RUN pip install flask
```

This describes the desired image construction:

```text
Start with Python 3.11
        ↓
Make Flask available
        ↓
Resulting Docker Image
```

The Docker build system performs the instructions.

Mental model:

```text
You
 ↓
Define image-building instructions
 ↓
Dockerfile
 ↓
Docker Build
 ↓
Docker Image
```

---

# Docker Image Building Process

Consider this project:

```text
simple-java-docker/

├── Dockerfile
└── src/
    └── Main.java
```

Dockerfile:

```Dockerfile
FROM eclipse-temurin:17-jdk-alpine

WORKDIR /app

COPY src/Main.java .

RUN javac Main.java

CMD ["java", "Main"]
```

Build the image:

```bash
docker build -t java-app .
```

The high-level process is:

```text
Dockerfile + Application Files
              ↓
         docker build
              ↓
     Docker processes instructions
              ↓
          Docker Image
```

---

## Step 1 — Docker Uses the Dockerfile

Docker uses the Dockerfile to determine how the image should be built.

The instructions are processed in order:

```text
FROM
  ↓
WORKDIR
  ↓
COPY
  ↓
RUN
  ↓
CMD
```

Each instruction contributes to the final image or its configuration.

---

## Step 2 — `FROM`

```Dockerfile
FROM eclipse-temurin:17-jdk-alpine
```

`FROM` specifies the **base image** for the build stage.

The base image provides a starting point for constructing your image.

```text
eclipse-temurin:17-jdk-alpine
              ↓
          Base Image
              ↓
         Your Image
```

For this example, the base image provides the Java environment needed to build and run the application.

> **Important:** A base image is not a virtual machine. Containers share the host kernel.

---

## Step 3 — `WORKDIR`

```Dockerfile
WORKDIR /app
```

`WORKDIR` sets the working directory for subsequent Dockerfile instructions and the container's default working directory.

In this example:

```text
/app
```

is the working directory.

So:

```Dockerfile
COPY src/Main.java .
```

copies the file into:

```text
/app/Main.java
```

---

## Step 4 — `COPY`

```Dockerfile
COPY src/Main.java .
```

Docker copies:

```text
src/Main.java
```

into the image.

Because the working directory is `/app`, the result is:

```text
/app/Main.java
```

Conceptually:

```text
Project
│
└── src/
    └── Main.java
          ↓
        COPY
          ↓
Docker Image
│
└── app/
    └── Main.java
```

---

## Step 5 — `RUN`

```Dockerfile
RUN javac Main.java
```

`RUN` executes a command **during the image build**.

Here:

```bash
javac Main.java
```

compiles the Java source code.

The result is:

```text
/app/

├── Main.java
└── Main.class
```

The important distinction is:

```text
RUN
 ↓
Build time
```

`RUN` does **not** wait until the container starts.

---

## Step 6 — `CMD`

```Dockerfile
CMD ["java", "Main"]
```

`CMD` specifies the **default command** to use when a container is started from the image.

It does **not** run during `docker build`.

Instead, Docker stores it as part of the image configuration.

```text
docker build
      ↓
Image created
      ↓
CMD stored as default startup configuration
      ↓
docker run
      ↓
java Main starts
```

Therefore:

```text
RUN
 ↓
Build time
```

while:

```text
CMD
 ↓
Container startup
```

---

# Docker Build Workflow

Suppose the project contains:

```text
my-app/

├── Dockerfile
├── app.py
└── requirements.txt
```

We run:

```bash
docker build -t my-app .
```

The complete workflow is:

```text
Application Source Code
          +
      Dockerfile
          ↓
     docker build
          ↓
     Docker Image
          ↓
      docker run
          ↓
      Container
          ↓
 Main Application Process
```

---

# Dockerfile vs Docker Image

## Dockerfile

A Dockerfile is:

> **A text file containing instructions used to build a Docker image.**

```text
Dockerfile
    ↓
Instructions
```

It is the **definition/instructions** for building the image.

---

## Docker Image

A Docker image is:

> **The built artifact produced from the Dockerfile and its build inputs.**

It contains the filesystem content and configuration needed to create containers.

```text
Dockerfile
    +
Build Inputs
    ↓
docker build
    ↓
Docker Image
```

---

# Important Distinction

```text
Dockerfile
     ↓
Instructions
```

```text
Docker Image
     ↓
Built Artifact
```

And:

```text
docker build
     ↓
Builds an image
```

```text
docker run
     ↓
Creates and starts a container from an image
```

---

# Final Mental Model

```text
Application Source Code
          +
      Dockerfile
          ↓
     docker build
          ↓
     Docker Image
          ↓
      docker run
          ↓
      Container
          ↓
 Main Application Process
```

The problem-solving journey is:

```text
Manual Installation
        ↓
Slow + Inconsistent + Difficult to Reproduce
        ↓
Dockerfile
        ↓
Repeatable Build Instructions
        ↓
docker build
        ↓
Docker Image
        ↓
docker run
        ↓
Container
        ↓
Application
```

## One-Sentence Summary

> **A Dockerfile defines repeatable instructions for building a Docker image; `docker build` turns those instructions into an image, and `docker run` uses that image to create and start a container.**