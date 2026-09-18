# Day 002 - How Docker Works Internally

## 🎯 Mission

Understand what happens internally when a Docker command is executed.

By the end

You should be able to explain to yourself:

> What exactly happens when I run `docker run nginx`?

If you cannot explain the journey from the Docker command to the running container, today's mission is incomplete.

---

## 📚 Topics Covered

- Docker Client
- Docker Daemon
- Docker Engine
- REST API
- Unix Socket
- Docker Registry
- Docker Hub
- Repository
- Image Tags
- Image Cache
- Docker Image Layers
- Read-only Layers
- Writable Container Layer
- Copy-on-Write
- Layer Caching
- Container Lifecycle
- Container Main Process (PID 1)

---

## 🧪 Hands-on Labs

- Inspected existing Docker images
- Created a container without running it
- Started a container
- Stopped a container
- Restarted a container
- Removed a container
- Removed a Docker image
- Ran Nginx using an existing local image
- Practiced the complete Docker container lifecycle
- Investigated a "missing container" production scenario
- Completed the Break-It Challenge

---

## 🤖 AI Exercise

- Asked ChatGPT to draw the Docker container lifecycle
- Compared the AI-generated lifecycle with my own understanding
- Identified missing or unclear lifecycle states
- Improved my understanding based on the comparison

---

## 🎯 Outcome

After completing today's mission, I can explain how Docker Client and Docker Daemon communicate, where Docker images come from, how images are built from layers, how containers use those layers, and how a container moves through its lifecycle.

I can also explain the high-level journey of:

`docker run nginx`

from the command being executed to the container starting.