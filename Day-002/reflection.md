# Day 2 – Reflection

## 1. What Did I Learn?

* Docker image vs container
* Container lifecycle
* `docker run`
* Writable container layer
* Copy-on-Write
* Why multiple containers can use the same image
* What happens when a container is created from an image

---

## 2. What Confused Me?

Copy-on-Write initially confused me.

I understood it after learning that containers share the read-only image layers and only store their changes in a separate writable layer.

---

## 3. What Mistake Did I Make?

I did not make a major mistake while practicing the lab.

However, it took me a long time to understand how layers work in a Dockerfile and how those layers are used by containers.

---

## 4. Which Concept Clicked?

The biggest concept that clicked was the relationship between an image and a container.

An image is the immutable template, while a container is a running instance that adds its own writable layer.

This also helped me understand how multiple containers can run from the same image without modifying the original image.

---

## 5. Can I Explain `docker run` From Memory?

Yes.

I can explain the high-level flow:

```text
docker run
    ↓
Docker receives the request
    ↓
Checks whether the required image is available locally
    ↓
Pulls the image from a registry if necessary
    ↓
Creates a container
    ↓
Adds a writable layer on top of the image layers
    ↓
Configures required resources such as networking
    ↓
Starts the container's main process
    ↓
Container starts running
```

---

# 6. AI Exercise – Docker Container Lifecycle

### Task

Ask ChatGPT:

> Draw the complete lifecycle of a Docker container.

Then compare the AI-generated diagram with my own understanding.

### AI-Generated Lifecycle

```text
Docker Image
     │
     │ docker create
     ▼
  CREATED
     │
     │ docker start
     ▼
  RUNNING
   /       \
  /         \
docker pause  docker stop
  │             │
  ▼             ▼
PAUSED       STOPPED
  │             │
docker unpause  │ docker start
  │             │
  └───────┬─────┘
          ▼
       RUNNING
          │
          │ docker rm
          ▼
       REMOVED
```

### Comparison

The AI diagram helped me visualize the different states a container can go through and the commands that cause transitions between those states.

It also made the difference between **stopping**, **pausing**, **starting**, **unpausing**, and **removing** a container easier to visualize.

### What I Learned From the Exercise

The lifecycle is not simply:

```text
Created → Running → Stopped → Removed
```

A container can also be paused and later unpaused while remaining the same container.

---

# 7. Notice Something Different?

### Day 1

**Day 1 answered:**

> Why does Docker exist?

### Day 2

**Day 2 answers:**

> How does Docker actually work internally?


