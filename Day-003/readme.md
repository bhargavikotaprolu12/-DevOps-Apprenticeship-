# Day 3


## Mission

Today you'll answer one question:

"What happens when I build my own Docker Image?"

By the end of today, you should be able to explain how application source code becomes a Docker Image and how that image is used to create a container.

If you cannot explain the journey, today's mission is incomplete.

## Real-World Scenario

A developer gives you an application and says:

"I don't want every developer or server to manually install the runtime and dependencies. I want this application packaged so it can run consistently across environments."

Your job is to package the application into a Docker Image.

Today's learning is designed to prepare you to do exactly that.

## Topics Learned

### 1. What Is a Dockerfile?

Understanding:
	• Why Dockerfiles exist
	• Declarative configuration
	• Dockerfile instructions
	• Docker build process
	• Dockerfile vs Docker Image

### 2. How Does Docker Build an Image?

Understanding:
	• docker build
	• Build Context
	• Dockerfile
	• Build instructions
	• Build output
	• Image creation
	• Build cache

### 3. Dockerfile Instructions

Understanding:
	• FROM
	• WORKDIR
	• COPY
	• RUN
	• CMD
	• ENTRYPOINT

### 4. CMD vs ENTRYPOINT

Understanding:
	• Default command
	• Main executable
	• Command overriding
	• How CMD and ENTRYPOINT behave together

### 5. Build Context

Understanding:
	• What the build context is
	• Why Docker needs a build context
	• What the "." means in docker build .
	• Files available during the build
	• Why unnecessary files should not be included

## Build Flow
```
Application Source Code
        ↓
Dockerfile
        ↓
docker build
        ↓
Docker Image
        ↓
docker run
        ↓
Container
```

## Hands-on Lab

### Building My First Docker Image

Objective

By the end of this lab, I should be able to create a Dockerfile, build my own Docker Image, and run a container from that image.

### Build Cache
Objective

Observed Docker reuse previously completed build steps.

Then changed application code and rebuilt the image to observe which steps were reused and which needed to run again.

The complete hands-on exercise is documented in:
hands_on.md

## Break-It Challenge

Intentionally introduced Dockerfile problems such as:
	• Incorrect COPY path
	• Missing file
	• Incorrect WORKDIR
	• Incorrect CMD
	• Incorrect ENTRYPOINT

Investigated the error messages and fixed the Dockerfile.

## Production Simulation

A developer says:

"My Docker Image builds successfully, but when I run the container, it exits immediately."

My task is to investigate why the container exits and determine whether the problem is related to the container's main process, CMD, or ENTRYPOINT.


## Day 3 Success Criteria

By the end of Day 3, I should be able to explain:

	• Why Dockerfiles exist.
	• What a Dockerfile defines.
	• What happens when docker build is executed.
	• What the Docker build context is.
	• What the "." means in docker build .
	• What FROM does.
	• What WORKDIR does.
	• What COPY does.
	• What RUN does.
	• What CMD does.
	• What ENTRYPOINT does.
	• The difference between CMD and ENTRYPOINT.
	• The difference between docker build and docker run.
	• How source code becomes a Docker Image.
	• How a Docker Image becomes a Container.
	• Why Dockerfile instruction order matters.
	• How to diagnose a basic Dockerfile build failure.
	• Why a container can build successfully but exit immediately.


