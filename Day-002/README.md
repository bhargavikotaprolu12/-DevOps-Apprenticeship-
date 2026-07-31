# Day 002 - Docker Architecture & Image Lifecycle


## 🎯 Mission

Understand what happens internally when `docker run nginx` is executed.

##  Real-World Scenario

A developer asks:
"What actually happens after I type `docker run nginx`?"

Today's goal was to understand every step from the Docker Client receiving the command to the container starting.

---

## 🏭 Production Relevance

Understanding the Docker request flow helps troubleshoot common issues such as:

- Images not downloading
- Containers failing to start
- Docker Daemon connection errors
- Image caching issues
- Slow container startup

## 📚 Topics Covered

- Docker Client
- Docker Daemon
- Docker Engine
- Docker Hub
- Docker Registry
- Image Layers
- Container Lifecycle
- docker create
- docker start
- docker run
- docker stop
- docker rm

---

## 🧪 Hands-on Labs

- Explored Docker images
- Pulled images from Docker Hub
- Created containers
- Started containers
- Stopped containers
- Removed containers
- Compared `docker create` vs `docker run`
- Inspected Docker images
- Viewed Docker image history

---

## ✅ Key Takeaways

- Docker Client communicates with Docker Daemon.
- Images are read-only templates.
- Containers are running instances of images.
- `docker run` performs multiple operations.
- Images are downloaded only once unless updated.