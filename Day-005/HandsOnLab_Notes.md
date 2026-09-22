# Day 5 — Hands-On Lab: Docker Image Size Optimization

## Objective

Investigate an intentionally inefficient Dockerfile, identify which layers make the image large, and optimize the image while keeping the application working.

---

## 1. Initial Dockerfile

Project structure:

    hands_on_lab/
    ├── Dockerfile
    ├── requirements.txt
    ├── app.py
    ├── README.md
    └── logs/
        └── app.log

Initial Dockerfile:

    FROM ubuntu

    RUN apt update

    RUN apt install -y python3 python3-pip curl wget

    WORKDIR /app

    COPY . .

    RUN pip3 install -r requirements.txt

    CMD ["python3", "app.py"]

---

## 2. Build Failure

The initial build failed at:

    RUN pip3 install -r requirements.txt

Error:

    externally-managed-environment

### Cause

Ubuntu's system Python is managed by the operating system, so `pip3` refused to install packages directly into the system Python environment.

### Investigation

The failure occurred specifically at:

    RUN pip3 install -r requirements.txt

### Lesson

The base image affects how application dependencies need to be installed.

---

## 3. Fixing the Build

Since the purpose of this lab was to investigate an intentionally large Ubuntu-based image, the `ubuntu` base image was kept.

A Python virtual environment was created instead.

Updated Dockerfile:

    FROM ubuntu

    RUN apt update && \
        apt install -y python3 python3-pip python3-venv curl wget

    WORKDIR /app

    COPY . .

    RUN python3 -m venv /opt/venv && \
        /opt/venv/bin/pip install -r requirements.txt

    CMD ["/opt/venv/bin/python", "app.py"]

### What Changed?

Previously:

    RUN pip3 install -r requirements.txt

This attempted to install `requests` into Ubuntu's system Python.

Now:

    RUN python3 -m venv /opt/venv

creates an isolated Python environment.

Then:

    /opt/venv/bin/pip install -r requirements.txt

installs `requests` inside that environment.

The application runs using:

    CMD ["/opt/venv/bin/python", "app.py"]

---

## 4. Build and Run

Build:

    docker build --no-cache -t day5-large-image .

Run:

    docker run --rm day5-large-image

The application ran successfully.

---

# 5. Investigating the Image

Check the image size:

    docker images day5-large-image

![](/screenshots/day-5_screenshots/check_image_size.png)
Inspect the image layers:

    docker history day5-large-image

![](/screenshots/day-5_screenshots/history_large_image.png)

The `SIZE` column was used to identify the large layers.

---

## Question 1 — Which Layer Is Large?

### Observation

The **136 MB layer was the largest layer created by my Dockerfile**.

There was also a `115 MB` layer in the history, but that belonged to the `ubuntu` base image.

---

## Question 2 — What Created the 136 MB Layer?

The layer was created by:

    RUN apt update && \
        apt install -y python3 python3-pip python3-venv curl wget

This installed Python, pip, virtual-environment support, and additional command-line tools.

---

## Question 3 — Do We Need All These Packages?

The application only needs the software required to build and run it.

### `python3`

The Python interpreter.

It allows the application to run:

    python3 app.py

**Needed:** Yes.

### `python3-pip`

Provides `pip`, Python's package installer.

The application has:

    requirements.txt

containing:

    requests

So pip is used to install the `requests` package.

**Needed for this build approach:** Yes.

### `python3-venv`

Provides support for creating a Python virtual environment:

    python3 -m venv /opt/venv

It was required because Ubuntu's system Python rejected the direct pip installation.

**Needed for this approach:** Yes.

### `curl`

A command-line tool for transferring data over network protocols.

The application does not use it.

**Needed:** No.

### `wget`

A command-line tool commonly used to download files.

The application does not use it.

**Needed:** No.

### Observation

The application does not need every package installed in the original Dockerfile.

This led to the next optimization step: identify and remove unnecessary components.

---

# 6. Investigating `COPY . .`

The original Dockerfile used:

    COPY . .

The resulting layer was only about **24.6 KB**.

### Observation

The application files themselves were very small compared with the image's larger base-image and package-installation layers.

The large image size was therefore not primarily caused by the application files.

### DevOps Investigation

When investigating a large `COPY . .` layer, ask:

- What exactly is being copied?
- Are logs being copied?
- Are `.git` files being copied?
- Are temporary files being copied?
- Are README files required?
- Can `.dockerignore` exclude unnecessary files?

---

# 7. First Optimization Attempt

A better Dockerfile structure was:

    FROM python:3.12

    WORKDIR /app

    COPY requirements.txt .

    RUN pip install -r requirements.txt

    COPY app.py .

    CMD ["python", "app.py"]

This improved the Dockerfile instruction ordering:

    requirements.txt
          ↓
    install dependencies
          ↓
    app.py

This allows Docker to reuse the dependency-installation step when only `app.py` changes.

---

## Unexpected Result

The first optimized image became much larger:

    day5-large-image       379 MB
    day5-optimized         1.62 GB

### Investigation

The Dockerfile had changed the base image from:

    FROM ubuntu

to:

    FROM python:3.12

The `python:3.12` image itself contained substantially larger base-image layers.

### Lesson

> **Dockerfile optimization is not only about instruction ordering. Base-image selection has a major impact on final image size.**

An optimization should always be measured rather than assumed to be successful.

---

# 8. Final Optimized Dockerfile

A smaller and more appropriate base image was selected:

    FROM python:3.12-slim

    WORKDIR /app

    COPY requirements.txt .

    RUN pip install --no-cache-dir -r requirements.txt

    COPY app.py .

    CMD ["python", "app.py"]

Build:

    docker build --no-cache -t day5-optimized .

Run:

    docker run --rm day5-optimized

Check the image:

    docker images day5-optimized

Inspect the layers:

    docker history day5-optimized

![](/screenshots/day-5_screenshots/final_optimised.png)

Result:

    day5-optimized:latest    195MB    47.7MB

---

# 9. Why Is This Version Smaller?

## 1. `python:3.12-slim`

`python:3.12-slim` provides the Python runtime with fewer additional components than the full `python:3.12` image.

The application does not require the additional components of the full image, so the smaller base image is appropriate for this application.

---

## 2. Removed `curl` and `wget`

The application does not use either tool.

Removing unnecessary software avoids adding components that provide no value to the application.

General principle:

> **Do not install software that the application does not require.**

---

## 3. No Separate Virtual Environment

The earlier Ubuntu-based image required a virtual environment because Ubuntu's system Python rejected the direct pip installation.

With the Python base image, the container already provides an isolated environment for the application.

For this simple container setup, an additional Python virtual environment is not necessary.

---

## 4. `pip --no-cache-dir`

The Dockerfile uses:

    RUN pip install --no-cache-dir -r requirements.txt

`--no-cache-dir` prevents pip from retaining its downloaded package cache in the image layer.

The application needs the installed package, not pip's download cache.

Important distinction:

    Docker build cache
        ≠
    pip package cache

---

## 5. Copy `requirements.txt` Separately

The Dockerfile uses:

    COPY requirements.txt .

    RUN pip install --no-cache-dir -r requirements.txt

    COPY app.py .

This separates the relatively stable dependency file from frequently changing application code.

If only `app.py` changes:

    COPY requirements.txt .
            ↓
          CACHE

    RUN pip install ...
            ↓
          CACHE

    COPY app.py .
            ↓
         REBUILD

This allows Docker to reuse the dependency-installation step when the dependencies have not changed.

---

## 6. Copy Only the Required Application File

Instead of:

    COPY . .

the optimized Dockerfile uses:

    COPY app.py .

This explicitly copies the application file required by the container.

It avoids blindly copying other files from the build context.

However, `COPY . .` is not automatically wrong. A properly configured `.dockerignore` can be used when an application requires multiple files.

---

# Key Lesson

Do not assume a Dockerfile is optimized just because it looks cleaner.

Use this investigation process:

    Large image
         ↓
    Check image size
         ↓
    docker history
         ↓
    Identify large layers
         ↓
    Find what created each layer
         ↓
    Determine what the application actually needs
         ↓
    Remove unnecessary components
         ↓
    Choose an appropriate base image
         ↓
    Optimize dependency installation
         ↓
    Improve Docker cache usage
         ↓
    Rebuild
         ↓
    Measure again
         ↓
    Verify the application still works

> **Measure first. Optimize based on evidence. Then measure again.**