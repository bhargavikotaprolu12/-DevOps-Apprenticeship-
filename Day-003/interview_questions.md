# Day 3 — Interview Questions & Answers

## 1. What is a Dockerfile?
A Dockerfile is a text file containing instructions that Docker uses to build a Docker image.

## 2. Why do we need a Dockerfile?
A Dockerfile provides a repeatable way to build an image.
Without a Dockerfile, we would have to manually install runtimes, dependencies, copy application files, and configure the environment repeatedly.

## What is the difference between a Dockerfile and a Docker image?
Dockerfile is the set of instructions used to build an image.
Docker image is the result produced from those instructions.

## What happens when you run docker build?
At a high level:
```
Dockerfile + Build Context
          ↓
      docker build
          ↓
 Docker processes instructions
          ↓
    Checks build cache
          ↓
 Executes required build steps
          ↓
      Docker Image
```
Docker uses the Dockerfile and files available in the build context to create the image.

## 5. Explain docker build -t my-app .
docker build -t my-app .
  • docker build → builds a Docker image.
  • -t my-app → gives the image a name/tag.
. → uses the current directory as the build context.

## 6. What is the purpose of FROM?
FROM defines the base image from which the image build starts.

Example:
```
FROM python:3.12
```

It provides the starting filesystem and software environment needed for the application.

## 7. Is a Docker base image the same as a virtual machine?
No.
A Docker image is not a virtual machine.
Containers share the host OS kernel, whereas a VM normally contains its own guest OS.

## 8. What is the purpose of WORKDIR?
WORKDIR sets the working directory for subsequent Dockerfile instructions and the container's default working directory.
Example:
WORKDIR /app
If /app does not exist, Docker creates it.


## 9. Why use WORKDIR instead of RUN cd/app?
RUN cd /app only changes the directory for that particular shell command.
WORKDIR /app establishes the working directory for subsequent instructions and the container.
Therefore, WORKDIR is the appropriate instruction for defining the working directory.

## 10. What is CMD?
CMD defines the default command that runs when a container starts.

example: CMD ["python", "app.py"]

when we run: docker run my-app

Docker uses the default CMD

## 11. What is ENTRYPOINT?

ENTRYPOINT defines the main executable or entrypoint of the container.

Example:

ENTRYPOINT ["kubectl"]

It is useful when the image is designed around a particular executable.

## 12. What happens when you run docker build -t my-app .?

At a high level:
```
Dockerfile + Build Context
          ↓
      docker build
          ↓
 Docker processes instructions
          ↓
    Checks build cache
          ↓
 Executes required build steps
          ↓
      Docker Image
```

The result is a Docker image.

## 13. What is Docker build cache?

Docker build cache allows Docker to reuse previously completed build steps when the relevant inputs have not changed.

This can make subsequent builds faster.

## 14. What happens when a container's main process exits?

The container stops when its main process exits.

For example:

CMD ["python", "hello.py"]

If hello.py prints a message and finishes, the main process exits and the container stops.

This does not necessarily mean the application crashed.

## 15. Explain the complete flow from application source code to a running container.

The complete flow is:

```
Application Source Code
          +
      Dockerfile
          ↓
     Build Context
          ↓
      docker build
          ↓
     Docker Image
          ↓
      docker run
          ↓
       Container
          ↓
   CMD / ENTRYPOINT
          ↓
     Main Process
          ↓
      Application
```
In an interview, I would explain:

I first create a Dockerfile that defines how the image should be built. The application files and Dockerfile are supplied as the build context. When I run docker build, Docker processes the Dockerfile, uses the build context, checks the build cache, and executes the required build steps to create an image. When I run docker run, Docker creates a container from that image. At startup, CMD and/or ENTRYPOINT determine the process that starts. The container continues running while its main process is running. When that process exits, the container stops.















