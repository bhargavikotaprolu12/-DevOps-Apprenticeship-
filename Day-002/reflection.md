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


# 7. Notice Something Different?

### Day 1

**Day 1 answered:**

> Why does Docker exist?

### Day 2

**Day 2 answers:**

> How does Docker actually work internally?


