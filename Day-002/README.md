## Mission

Today you'll answer one question:

> **"What exactly happens when I run `docker run nginx`?"**

By the end of today, you should be able to explain every major step from typing the command until the container starts.

**If you cannot explain the journey, today's mission is incomplete.**

---

## Real-World Scenario

You're the only DevOps Engineer.

A developer says:

> **"I typed `docker run nginx`, but I have no idea what actually happened."**

Your job is to explain every step.

Today's learning is designed to prepare you to do exactly that.

---

## Topics Learned

### 1. What Happens Internally When You Run a Docker Command?

Understanding:

* Docker Client
* Docker Daemon
* Docker Engine
* REST API
* Unix Socket
* Image lookup
* Container creation
* Namespaces
* cgroups
* Networking
* Volumes
* Main process (PID 1)

### 2. Where Do Docker Images Come From?

Understanding:

* Docker Registry
* Docker Hub
* Repository
* Tags
* Image Cache

### 3. What Is Inside a Docker Image?

Understanding:

* Image Layers
* Read-only Layers
* Writable Container Layer
* Copy-on-Write
* Layer Caching
* Image inspection

---

# Hands-on Lab

## Understanding the Docker Container Lifecycle

### Objective

By the end of this lab, you should understand the complete lifecycle of a Docker container and the difference between an image and a container.

### Lifecycle Covered

```text
Image
  ↓
Create
  ↓
Created
  ↓
Start
  ↓
Running
  ↓
Stop
  ↓
Exited
  ↓
Restart
  ↓
Running
  ↓
Stop
  ↓
Remove
  ↓
Container Deleted
```

The complete hands-on exercise is documented in:

**[`challenge_lab.md`](./challenge_lab.md)**

---

# AI Exercise

Ask ChatGPT:

> **"Draw the complete lifecycle of a Docker container."**

Then:

1. Compare the AI-generated diagram with your own drawing.
2. Identify which representation makes more sense to you.
3. Check whether any important lifecycle state or transition is missing.
4. Improve your own diagram if necessary.

The goal is not to copy the AI's answer. The goal is to **compare, reason, and improve your own understanding.**

---

# Day 2 Success Criteria

By the end of Day 2, I should be able to explain:

* What happens internally when I run `docker run nginx`.
* How the Docker Client communicates with the Docker Daemon.
* Where Docker gets an image when it is not available locally.
* What a Docker Registry and Repository are.
* What an image tag represents.
* Why Docker does not download the same image every time.
* What image layers are.
* Why image layers are read-only.
* What the writable container layer is.
* What Copy-on-Write means.
* The difference between an image and a container.
* The difference between `docker create`, `docker start`, `docker stop`, `docker restart`, and `docker rm`.
* Why a stopped container still appears in `docker ps -a`.
* Why removing a container does not automatically remove its image.

---

## Day 2 Mission Check

### Core Question

**Can I explain what happens from this command:**

```bash
docker run nginx
```

**until the Nginx container starts?**

If yes → **Day 2 mission complete.**

If not → revisit the relevant session and lab before moving to Day 3.
