# Day 5 — Session 1: Image Size Optimization

## Mission

By the end of this session, I should be able to look at a large Docker image and answer:

- Why is this image large?
- Which layer is responsible?
- What created that layer?
- Do I actually need that content?

---

## Why Does Image Size Matter?

Suppose:

- Image A → 200 MB
- Image B → 2 GB

Both run the same application.

A larger image may require more:

- Storage
- Time to pull
- Time to transfer
- Time to deploy

Deployment flow:

    Developer
        ↓
    Build image
        ↓
    Push image to registry
        ↓
    Server pulls image
        ↓
    Container starts

If the image is unnecessarily large, more data has to move through this process.

So image size can affect deployment efficiency.

---

# Where Does Image Size Come From?

A Docker image consists of layers.

    Image
    │
    ├── Base image
    ├── Package installation
    ├── Application dependencies
    ├── Application files
    └── Other changes

Some layers can be large.

Instead of simply saying:

> "My image is large."

I should ask:

> **Which layer is making it large?**

This is the investigation mindset.

---

# Investigating Image Size

## 1. Check Overall Image Size

Command:

    docker images

Example:

    REPOSITORY    TAG       IMAGE ID    SIZE
    myapp         latest    abc123      1.2GB

This shows the overall image size.

It does not tell me which layer is responsible.

---

## 2. Inspect Image Layers

Command:

    docker history <image-name>

Example:

    docker history myapp

Example output:

    IMAGE    CREATED BY                         SIZE
    ...      CMD [...]                          0B
    ...      COPY . .                           800MB
    ...      RUN pip install ...                250MB
    ...      WORKDIR /app                       0B
    ...      FROM python:3.12                   ...

Now I can investigate which Dockerfile instruction created a large layer.

For example:

    COPY . . → 800 MB

Instead of immediately trying to optimize it, ask:

> Why did copying the application create such a large layer?

The project might contain:

    .git/
    node_modules/
    logs/
    videos/
    backups/
    temporary files/

This leads to `.dockerignore`.

---

# Investigation Mindset

When an image is 1.5 GB, don't immediately change the Dockerfile.

Follow this process:

    Why is the image large?
            ↓
    Which layer is large?
            ↓
    What created that layer?
            ↓
    What files/packages created the size?
            ↓
    Do I actually need them?

This is better than blindly applying optimization tricks.

---

# Base Image Selection

The base image itself contributes to the final image size.

For example:

    FROM ubuntu

versus:

    FROM python:3.12

The important question is not:

> "Which image is smallest?"

Instead:

> **Which base image provides what my application needs without unnecessary components?**

If an application needs Python, starting with a suitable Python image may make more sense than starting with Ubuntu and manually installing Python.

---

## Don't Assume "Alpine Is Always Better"

A common statement is:

> "Use Alpine because it's smaller."

The important principle is:

> **Choose an appropriate minimal base image that supports the application's runtime and requirements.**

A smaller image can still cause problems if:

- Required libraries are unavailable
- Application dependencies behave differently
- Compatibility changes
- Debugging becomes harder
- The image does not properly support the application

The decision should be:

    Application requirements
            ↓
    Runtime requirements
            ↓
    Appropriate base image

---

# Unnecessary Packages

Consider:

    FROM ubuntu

    RUN apt update

    RUN apt install -y python3 curl wget vim git

Ask:

> Does the production application actually need all of these?

If the application only needs Python, then installing:

- curl
- wget
- vim
- git

may be unnecessary.

Unnecessary packages can contribute to:

- Image size
- Maintenance
- Unnecessary software in the runtime environment

The key question is:

> **Does the application need this package at runtime?**

---

# Unnecessary Files

Suppose the project contains:

    project/
    ├── Dockerfile
    ├── app.py
    ├── requirements.txt
    ├── .git/
    ├── logs/
    ├── videos/
    ├── backups/
    ├── screenshots/
    └── temporary-files/

If the Dockerfile contains:

    COPY . .

all of these files may be included in the build context and potentially copied into the image.

This is why `.dockerignore` matters.

For example:

    .git/
    logs/
    videos/
    backups/
    screenshots/
    *.log

can exclude files that are not required by the image.

---

# Build Artifacts

A build artifact is something produced during the build process.

A build process may create:

- Temporary files
- Compiler files
- Build directories
- Test output
- Development dependencies

These are not necessarily required in the production image.

This is where multi-stage builds become useful.

---

# Multi-Stage Builds and Image Size

Without multi-stage builds:

    Final Image
    ├── JDK
    ├── Compiler
    ├── Source code
    ├── Build tools
    └── Application

With multi-stage builds:

    Builder
    ├── JDK
    ├── Compiler
    ├── Source code
    └── Build tools
            ↓
        Artifact
            ↓
    Runtime
    ├── JRE
    └── Application

The final runtime image does not need the entire builder environment.

Therefore, multi-stage builds can reduce unnecessary content in the final image.

---

# Runtime Dependencies vs Build Dependencies

## Build Dependency

Something required to **create** the application.

Example:

    Java compiler

## Runtime Dependency

Something required to **run** the application.

Example:

    Java runtime

If something is only required during the build, ask:

> **Does it really need to be in the final runtime image?**

This question often leads to multi-stage builds.

---

# Complete Mental Model

    DOCKER IMAGE
          │
          ↓
    Why is it large?
          │
          ↓
    docker history
          │
          ↓
    Which layer is large?
          │
          ↓
    What created the layer?
          │
          ↓
    ┌─────┴──────────┐
    ↓                ↓
    Unnecessary      Necessary
    content          content
    ↓                ↓
    Remove/optimize  Keep
    │
    ├── .dockerignore
    ├── Better base image
    └── Multi-stage build

---

# Key Takeaway

**Image optimization starts with investigation, not assumptions.**

The process is:

    Measure
      ↓
    Inspect layers
      ↓
    Identify the cause
      ↓
    Decide what is actually required
      ↓
    Optimize
      ↓
    Build again
      ↓
    Measure again