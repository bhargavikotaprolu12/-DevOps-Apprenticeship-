# Day 4 — Docker Image Optimization & Multi-Stage Builds
# Session 1 — Images, Layers, Writable Layers & Build Cache

## What Is a Docker Image Made Of?

A Docker image is **not just one giant file**.

It is built from **multiple layers**.

For example, consider this Dockerfile:

    FROM python:3.12

    WORKDIR /app

    COPY requirements.txt .

    RUN pip install -r requirements.txt

    COPY app.py .

    CMD ["python", "app.py"]

Conceptually, Docker builds an image from layers and configuration:

    Docker Image
    │
    ├── Layer → Base image
    ├── Layer → Application directory/filesystem change
    ├── Layer → requirements.txt
    ├── Layer → Installed dependencies
    ├── Layer → app.py
    └── Configuration → CMD

> **Important:** Do not assume that every Dockerfile instruction creates a layer. That is an oversimplification.

For now, remember:

**Filesystem-changing build steps can produce image layers, while instructions such as `CMD` primarily contribute image configuration.**

---

## Why Does Docker Use Layers?

Imagine Docker did everything as one giant block.

If you changed one small part of your application, Docker would have less opportunity to reuse previously completed work.

With layers:

    Layer 1 ─────── unchanged
    Layer 2 ─────── unchanged
    Layer 3 ─────── unchanged
    Layer 4 ─────── changed

Docker can potentially **reuse the unchanged parts**.

This gives us two major benefits:

### 1. Reuse

The same layer can potentially be reused by multiple images and builds.

### 2. Caching

Docker can reuse previously built results when the relevant build step is still valid.

---

# Layer vs. Dockerfile Instruction

Do **not** say:

> "Every Dockerfile instruction creates a layer."

That is an oversimplification and can be wrong.

Instead, say:

> **"Docker builds images from layers. Filesystem-changing build instructions generally contribute filesystem layers, while instructions such as `CMD` primarily define image configuration."**

This is a more technically accurate way to explain Docker image layers in an interview.

---

# Image Layers vs. Container Writable Layer

A Docker image is generally made up of **read-only layers**.

When Docker creates a container from that image, the container gets a **writable layer** on top.

Think of it like this:

    Container
    ┌─────────────────┐
    │ Writable Layer  │
    ├─────────────────┤
    │ Image Layer     │
    ├─────────────────┤
    │ Image Layer     │
    ├─────────────────┤
    │ Image Layer     │
    └─────────────────┘

The image layers remain unchanged.

The container gets its own writable layer for changes made while the container is running.

---

# Why Is the Image Read-Only?

Think of an image as a **template**.

For example, three containers can be created from the same image.

We do not want Container 1 modifying the underlying image and affecting Container 2.

Instead:

    Image
    ┌───────────────┐
    │ Read-only     │
    │ layers        │
    └───────┬───────┘
            │
       ┌────┼────┐
       ↓    ↓    ↓
    Container 1  Container 2  Container 3
    Writable     Writable     Writable
    Layer        Layer        Layer

Each container gets its own writable layer while sharing the underlying image layers.

---

# Copy-on-Write

Suppose the image contains:

    /app/config.txt

The container starts from that image.

Initially, the container can use the file from the image's read-only layer.

If the application modifies that file, Docker's storage system handles the change through the container's writable layer.

## Before Modification

    Image Layer
    └── config.txt
            ↑
       Container reads

## After Modification

    Image Layer
    └── config.txt        ← original remains

    Container Writable Layer
    └── config.txt        ← modified version

The original image layer is **not modified**.

This behavior is commonly described as **copy-on-write (CoW)**.

It is useful because multiple containers can share the same underlying image layers.

The containers do not need completely separate copies of all the image data.

This:

- Saves storage
- Allows image layers to be shared
- Makes container creation more efficient

---

# Important Distinction: Image Layer vs. Container Layer

This is an **interview favorite**.

| Image Layers | Container Writable Layer |
|---|---|
| Part of the image | Belongs to a particular container |
| Read-only | Writable |
| Created as part of image construction | Stores changes made to the container's filesystem |
| Can be shared/reused | Specific to that container |
| Remain as part of the image | Removed when the container is removed, unless data is stored separately, such as in a volume |

---

# Connecting Layers to Build Cache

Consider this Dockerfile:

    FROM python:3.12

    WORKDIR /app

    COPY requirements.txt .

    RUN pip install -r requirements.txt

    COPY app.py .

    CMD ["python", "app.py"]

During a build, Docker can store and reuse the results of eligible build steps.

Conceptually:

    FROM python:3.12
            ↓
    WORKDIR /app
            ↓
    COPY requirements.txt .
            ↓
    RUN pip install -r requirements.txt
            ↓
    COPY app.py .
            ↓
    CMD ["python", "app.py"]

Docker can use previously built results when the relevant build steps are still valid.

---

# What Happens When `app.py` Changes?

Suppose the first build has already completed.

Now you change:

    app.py

and rebuild the image.

Conceptually:

    FROM python:3.12              → reused
    WORKDIR /app                  → reused
    COPY requirements.txt .       → reused
    RUN pip install -r ...        → reused
    COPY app.py .                 → changed
    CMD ["python", "app.py"]      → processed

The earlier valid build results can be reused, while the changed step needs to be rebuilt.

---

# What If `requirements.txt` Changes?

Suppose you modify:

    requirements.txt

Now this step changes:

    COPY requirements.txt .
            ↓
         changed

That can invalidate the cache for the following dependent build step:

    RUN pip install -r requirements.txt
            ↓
       may need to run again

And subsequent steps may also need to be rebuilt.

The general idea is:

    Earlier change
          ↓
    Cache invalidation
          ↓
    Later dependent steps
          ↓
    May need rebuilding

---

# Why Does Instruction Ordering Matter?

Docker's build cache means that the order of instructions in a Dockerfile can affect how much work Docker needs to repeat after a change.

For example:

    COPY requirements.txt .

    RUN pip install -r requirements.txt

    COPY app.py .

If only `app.py` changes, the dependency installation step can potentially remain cached.

But if `requirements.txt` changes, the dependency installation step may need to run again, and later dependent steps may also need to be rebuilt.

Therefore:

> **Instruction ordering matters because a change in an earlier build step can invalidate the cache for later dependent steps.**


# Session 2 - Docker Build Cache & Dockerfile Optimization

## Mission

We know that Docker builds an image from top to bottom. When a build step changes or its cache can no longer be reused, Docker may need to rebuild that step and the dependent steps that follow it.

By the end of this session, you should be able to answer:

> **Why does Docker sometimes rebuild expensive steps, and how can I arrange my Dockerfile so that Docker reuses the cache as much as possible?**

---

# Docker Build Cache

Docker stores the results of eligible build steps in its **build cache**.

Now imagine you change only:

    app.py

and build the image again.

Docker can reuse the previous results for the steps that are still valid instead of doing the expensive work again.

This is called **build cache**.

The main idea is:

> **Instead of repeating work that has already been completed, Docker can reuse valid cached results.**

---

# Why Is Cache Important?

Imagine this step:

    RUN pip install -r requirements.txt

takes **2 minutes**.

Your application code changes every few minutes.

If Docker had to reinstall all dependencies every time you changed `app.py`, development would become slow.

The goal is to structure the Dockerfile so that expensive and relatively stable steps can remain cached even when frequently changing application files are modified.

---

# Bad Dockerfile Structure

Consider:

    FROM python:3.12

    WORKDIR /app

    COPY . .

    RUN pip install -r requirements.txt

    CMD ["python", "app.py"]

The problem is:

    COPY . .

copies everything from the build context into the image.

That includes files such as:

    requirements.txt
    app.py
    other application files

If `app.py` changes:

    COPY . .
        ↓
    COPY step changes
        ↓
    Following RUN step may lose its cache
        ↓
    RUN pip install -r requirements.txt
        ↓
    Dependencies may need to be installed again

So a small application-code change can potentially cause an expensive dependency-installation step to run again.

---

# Better Dockerfile Structure

Instead of copying everything before installing dependencies, copy the dependency file first:

    FROM python:3.12

    WORKDIR /app

    COPY requirements.txt .

    RUN pip install -r requirements.txt

    COPY . .

    CMD ["python", "app.py"]

Now the dependency installation is separated from the frequently changing application code.

If only `app.py` changes:

    COPY requirements.txt .     → CACHE

    RUN pip install -r requirements.txt
                                → CACHE

    COPY . .                    → REBUILD

    CMD ["python", "app.py"]    → processed as needed

The expensive dependency installation can therefore be reused.

---

# Why Does the Better Structure Work?

The important difference is the order of the instructions.

## Bad Structure

    COPY . .
        ↓
    RUN pip install -r requirements.txt

A change to `app.py` changes the `COPY . .` step.

That can cause the following dependency-installation step to lose its cache.

## Better Structure

    COPY requirements.txt .
        ↓
    RUN pip install -r requirements.txt
        ↓
    COPY . .

Now:

- `requirements.txt` changes less frequently.
- Dependency installation happens before application source code is copied.
- Changes to `app.py` do not change the `COPY requirements.txt .` step.
- The dependency-installation step can potentially remain cached.

---

# Docker Builds from Top to Bottom

Docker evaluates the Dockerfile **from top to bottom**.

For example:

    Step 1
      ↓
    Step 2
      ↓
    Step 3
      ↓
    Step 4
      ↓
    Step 5

A change in an earlier step can affect the cacheability of later steps.

Therefore:

> **A change in an earlier step can affect the steps that come after it.**

This is why **Dockerfile instruction order matters**.

---

# Stable vs. Frequently Changing Files

When designing a Dockerfile, think about which files change frequently and which files are relatively stable.

## Usually More Stable

Examples:

- `requirements.txt`
- `package.json`
- `pom.xml`

## Usually Change More Frequently

Examples:

- `app.py`
- Source code
- HTML
- CSS
- Application files

The general optimization idea is:

> **Place stable files and expensive operations earlier, and frequently changing files later.**

---

# Cache Hit vs. Cache Miss

These two terms are important for interviews.

## Cache Hit

A **cache hit** occurs when Docker finds a previously built result that it can reuse.

Conceptually:

    Dockerfile step
          ↓
    Previous result available
          ↓
       CACHE HIT
          ↓
        Reuse it

---

## Cache Miss

A **cache miss** occurs when Docker cannot reuse the previous result for a build step.

Conceptually:

    Dockerfile step
          ↓
    Something changed / cache unavailable
          ↓
       CACHE MISS
          ↓
       Build again

When a step needs to be rebuilt, subsequent dependent steps may also need to be rebuilt.

---

# One Mental Model

Think about Docker's build process like this:

    Dockerfile
        ↓
    Build top → bottom
        ↓
    Check cache at each step
        ↓
    Cache available?
       / \
     YES  NO
      ↓    ↓
    Reuse  Build
           ↓
      Following steps
      may also rebuild

The key idea is:

> **An earlier cache miss can affect later steps.**

---

# Dockerfile Optimization Strategy

A simple mental model is:

    STABLE
       ↓
    EXPENSIVE
       ↓
    FREQUENTLY CHANGING

For example:

    COPY requirements.txt .
       ↓
    RUN pip install -r requirements.txt
       ↓
    COPY . .

This structure gives Docker a better opportunity to reuse the expensive dependency-installation step when only application source code changes.


# Session 3 - Build Context, `.dockerignore`, and `COPY` vs `ADD`

## Mission

Understand:

- What Docker build context is
- Why a large build context can slow down builds
- How `.dockerignore` controls what is included in the build context
- The difference between `.gitignore` and `.dockerignore`
- The difference between `COPY` and `ADD`
- How to investigate an unnecessarily slow Docker build

---

# Why Can a Large Build Context Be a Problem?

Imagine your project contains:

    project/
    ├── Dockerfile
    ├── app.py
    ├── requirements.txt
    ├── .git/
    ├── node_modules/
    ├── logs/
    ├── videos/
    ├── backups/
    └── 5 GB temporary files

Then you run:

    docker build -t myapp .

The final `.` tells Docker to use the current directory as the **build context**.

In this example, you are effectively telling Docker:

> **"Use this entire directory as my build context."**

That can be unnecessary if most of those files are not required for the build.

---

# What Is Build Context?

The **build context** is the set of files and directories available to Docker during the build.

For example:

    docker build -t myapp .

Here:

- `docker build` → starts the image build
- `-t myapp` → gives the image a name/tag
- `.` → specifies the current directory as the build context

Conceptually:

    Project Directory
          │
          ↓
    Build Context
          │
          ↓
    Docker Build

Docker can use files from the build context for instructions such as:

    COPY
    ADD

---

# Problems With a Large Build Context

A large build context can cause several problems:

- More data needs to be processed or transferred to the builder.
- Builds can become slower.
- Unnecessary files may be available to the build.
- There is more opportunity to accidentally copy unwanted files into the image.

For example, if your project contains:

    videos/
    backups/
    .git/
    node_modules/
    logs/
    5 GB temporary files

but your application only needs:

    Dockerfile
    app.py
    requirements.txt

sending everything as the build context is wasteful.

This is where `.dockerignore` becomes useful.

---

# What Is `.dockerignore`?

`.dockerignore` tells Docker:

> **"Do not include these files or directories in the build context sent to the builder."**

Create a file named:

    .dockerignore

Example:

    .git
    node_modules
    logs
    *.log
    README.md

Now consider this project:

    project/
    ├── Dockerfile
    ├── app.py
    ├── requirements.txt
    ├── .dockerignore
    ├── .git/             ← ignored
    ├── node_modules/     ← ignored
    ├── logs/             ← ignored
    ├── app.log           ← ignored
    └── README.md         ← ignored

The ignored files are excluded from the build context.

---

# Why Use `.dockerignore`?

`.dockerignore` helps:

- Reduce the amount of data sent to the builder.
- Make builds more efficient.
- Prevent unnecessary files from being available to the build.
- Reduce the chance of accidentally copying unwanted files into the image.

For example, you usually do not need:

    .git/
    node_modules/
    logs/
    *.log
    __pycache__/
    .env

inside your Docker build context.

---

# `.dockerignore` vs `.gitignore`

Do not confuse these two files.

They solve different problems.

## `.gitignore`

Controls what Git ignores.

    .gitignore
         ↓
        Git

For example, Git can be configured to ignore files that should not be committed to the repository.

---

## `.dockerignore`

Controls what Docker excludes from the build context.

    .dockerignore
         ↓
    Docker build

For example:

    .dockerignore
    ├── .git
    ├── node_modules/
    ├── *.log
    └── .env

These files are excluded from the Docker build context.

---

## Can You Have Both?

Yes.

You may have:

    .gitignore

without:

    .dockerignore

and you may also have:

    .dockerignore

without:

    .gitignore

They are independent files used by different tools.

---

# Important `.dockerignore` Patterns

You should recognize common patterns such as:

    .git

Ignore the Git directory.

---

    *.log

Ignore files ending in `.log`.

For example:

    app.log
    error.log
    server.log

---

    node_modules/

Ignore the `node_modules` directory.

---

    __pycache__/

Ignore Python cache directories.

---

    .env

Ignore the `.env` file.

---

# Example `.dockerignore`

A common real-world `.dockerignore` might contain:

    .git
    .gitignore
    .dockerignore
    node_modules/
    __pycache__/
    *.log
    .env

The exact contents should depend on the project and what the build actually needs.

---

# Important: `.dockerignore` Is Not Secret Management

`.dockerignore` can help prevent files from entering the build context, but it is **not a replacement for proper secret management**.

For example:

    .env

can be excluded from the build context using `.dockerignore`.

However, you should not treat `.dockerignore` as your primary secret-management mechanism.

> **Secrets should not be baked into Docker images.**

Use appropriate secret-management mechanisms instead of storing sensitive credentials inside the image.

---

# `COPY` vs. `ADD`

This is a common Docker interview question.

Both `COPY` and `ADD` can place files into a Docker image.

---

# `COPY`

Example:

    COPY app.py /app/

`COPY` is primarily used for straightforward file and directory copying.

It has a simple and explicit purpose:

> **Copy files or directories from the build context into the image.**

---

# `ADD`

Example:

    ADD app.py /app/

ADD provides additional behaviour beyond COPY, including local archive extraction.
For ordinary file and directory copying, prefer COPY because it is more explicit.


Because `ADD` has additional behavior, it can be less explicit when you only need simple file copying.

---

# Which One Should You Prefer?

For ordinary file and directory copying:

> **Prefer `COPY` because its purpose is simple and explicit.**

Use `ADD` when you specifically need one of its additional behaviors.

### Simple Rule

    Normal file/directory copy
            ↓
          COPY

    Need ADD-specific behavior
            ↓
           ADD

Do not use `ADD` just because it is shorter or because it can do everything `COPY` does.

---

# Production Scenario

A developer gives you this project:

    payment-service/
    ├── Dockerfile
    ├── app.py
    ├── requirements.txt
    ├── .git/
    ├── logs/
    ├── tests/
    ├── screenshots/
    ├── node_modules/
    ├── .env
    ├── README.md
    └── backup/

The Docker build is slow.

The Dockerfile contains:

    FROM python:3.12

    WORKDIR /app

    COPY . .

    RUN pip install -r requirements.txt

    CMD ["python", "app.py"]

You need to investigate the build.

---

# Investigation Checklist

Your first questions should be:

### 1. What is the build context?

Check what directory is being passed to:

    docker build

For example:

    docker build -t payment-service .

The `.` means the current directory is being used as the build context.

---

### 2. What unnecessary files are entering the context?

Look for files and directories such as:

    .git/
    logs/
    screenshots/
    node_modules/
    .env
    backup/
    temporary files

Determine whether they are actually needed during the build.

---

### 3. Should we create `.dockerignore`?

If unnecessary files are entering the build context, create a `.dockerignore` file.

For example:

    .git
    node_modules/
    logs/
    screenshots/
    backup/
    *.log
    .env

Only ignore files that the build does not actually require.

---

### 4. Can `requirements.txt` be copied separately?

Instead of:

    COPY . .

consider:

    COPY requirements.txt .

    RUN pip install -r requirements.txt

    COPY . .

This separates the relatively stable dependency file from frequently changing application files and can improve cache reuse.

---

### 5. Is `COPY` appropriate here?

If you are simply copying files and directories from the build context into the image, `COPY` is usually the clearer choice.

For example:

    COPY app.py /app/

or:

    COPY . .

---

### 6. Are There Files That Should Never Be Included?

Look for sensitive or unnecessary files such as:

    .env
    credentials
    private keys
    secrets
    .git/
    logs/
    backups/
    temporary files

Sensitive information should not be baked into the image.

---

# Session 3 Mental Model

    docker build
          ↓
    Build Context
          ↓
    .dockerignore
          ↓
    Remove unnecessary files
          ↓
    Dockerfile
          ↓
    COPY / ADD
          ↓
    Build Image

The key ideas to remember:

> **Build context = what Docker can access during the build.**

> **`.dockerignore` = what Docker excludes from the build context.**

> **`.gitignore` = what Git ignores.**

> **`COPY` = simple, explicit copying.**

> **`ADD` = copying plus additional behavior.**

> **Keep the build context small and avoid putting unnecessary or sensitive files into the image.**


# Day 4 — Session 4: Multi-Stage Builds

## Mission

> **How do we build an application using all the tools we need, but keep those build tools out of the final production image?**

---

# The Problem

Imagine you have a Java application.

## To Build the Application, You Need

- JDK
- `javac`
- Build tools
- Source code

## After Compilation, the Application Only Needs

- Java runtime
- `Main.class`

So why should the production image contain:

- JDK
- Compiler
- Source code
- Build tools

if the application does not need them at runtime?

That is the problem **multi-stage builds** solve.

---

# 1. Single-Stage Build

Consider:

    FROM eclipse-temurin:17-jdk-alpine

    WORKDIR /app

    COPY src/Main.java .

    RUN javac Main.java

    CMD ["java", "Main"]

This works.

The Docker image contains:

    JDK
      +
    Main.java
      +
    Main.class

But the application only needs the Java runtime to execute:

    java Main

It does not need `javac` anymore.

So we are carrying unnecessary build tools into the final image.

---

# 2. What Is a Multi-Stage Build?

A multi-stage Dockerfile contains **multiple `FROM` instructions**.

For example:

    FROM eclipse-temurin:17-jdk-alpine AS builder

    WORKDIR /app

    COPY src/Main.java .

    RUN javac Main.java

    FROM eclipse-temurin:17-jre-alpine

    WORKDIR /app

    COPY --from=builder /app/Main.class .

    CMD ["java", "Main"]

Notice:

    FROM eclipse-temurin:17-jdk-alpine AS builder

and then another:

    FROM eclipse-temurin:17-jre-alpine

These are two separate build stages.

---

# 3. Understand the Two Stages

## Stage 1 — Builder

    FROM eclipse-temurin:17-jdk-alpine AS builder

    WORKDIR /app

    COPY src/Main.java .

    RUN javac Main.java

### Its Job

The builder stage is responsible for **building the application**.

It contains:

- JDK
- `javac`
- Source code
- Build tools

It produces:

    Main.class

---

## Stage 2 — Runtime

    FROM eclipse-temurin:17-jre-alpine

    WORKDIR /app

    COPY --from=builder /app/Main.class .

    CMD ["java", "Main"]

### Its Job

The runtime stage is responsible for **running the application**.

It receives only what is required to run the application:

    Main.class

The key command is:

    COPY --from=builder /app/Main.class .

Do not think of `COPY --from=builder` as a normal `COPY` from the local build context.

It copies a file from another build stage.

---

# 4. Mental Model

    BUILD STAGE
    ┌─────────────────┐
    │ JDK             │
    │ Source code     │
    │ javac           │
    │                 │
    │ compile         │
    └────────┬────────┘
             │
             │ Main.class
             ↓
    RUNTIME STAGE
    ┌─────────────────┐
    │ JRE             │
    │ Main.class      │
    │                 │
    │ run application │
    └─────────────────┘

The builder creates the product.

The runtime stage only receives what is needed to run the product.

---

# 5. Why Is Multi-Stage Build Useful?

## Without Multi-Stage

The final image may contain:

    Final Image
    ├── JDK
    ├── Compiler
    ├── Source code
    ├── Build files
    └── Application

There can be unnecessary build-time components in the runtime image.

---

## With Multi-Stage

The final image can contain:

    Final Image
    ├── JRE
    └── Application

This can result in:

- Smaller final images
- Fewer unnecessary tools
- Fewer unnecessary files
- A cleaner runtime environment
- Reduced attack surface

> **Important:** Do not memorize "multi-stage = small image" as the entire concept.

The deeper idea is:

> **Separate the environment required to build the application from the environment required to run it.**

---

# 6. Build the Application

Create this project structure:

    day4-session4/
    ├── Dockerfile
    └── src/
        └── Main.java

---

## `Main.java`

    public class Main {

        public static void main(String[] args) {

            System.out.println("Hello from Multi-Stage Build");

        }

    }

---

# 7. Build a Single-Stage Image

Use:

    FROM eclipse-temurin:17-jdk-alpine

    WORKDIR /app

    COPY src/Main.java .

    RUN javac Main.java

    CMD ["java", "Main"]

Build the image:

    docker build -t java-single-stage .

Run the container:

    docker run --rm java-single-stage

Expected output:

    Hello from Multi-Stage Build

---

# 8. Inspect the Single-Stage Image

Check the image:

    docker images java-single-stage

Inspect the image layers:

    docker history java-single-stage

The goal is to understand what exists in the image and where it came from.

---

# 9. Convert It to a Multi-Stage Build

Use:

    FROM eclipse-temurin:17-jdk-alpine AS builder

    WORKDIR /app

    COPY src/Main.java .

    RUN javac Main.java

    FROM eclipse-temurin:17-jre-alpine

    WORKDIR /app

    COPY --from=builder /app/Main.class .

    CMD ["java", "Main"]

---

# 10. Build the Multi-Stage Image

Build:

    docker build -t java-multi-stage .

Run:

    docker run --rm java-multi-stage

Expected output:

    Hello from Multi-Stage Build

The application still works.

But the final image is different.

---

# 11. Compare the Images

List the images:

    docker images

Compare:

    java-single-stage
    java-multi-stage

![](/screenshots/day-4_screenshots/dockerimages.png)

You can also inspect their layers:

    docker history java-single-stage

![](/screenshots/day-4_screenshots/inspect_single_stage.png)

    docker history java-multi-stage

![](/screenshots/day-4_screenshots/inspect_multi_stage.png)

---

# 12. Important Question

> **"If the builder stage contains the JDK, doesn't the final image also contain it?"**

**No.**

This is the key benefit of multi-stage builds.

The final stage starts from:

    FROM eclipse-temurin:17-jre-alpine

Notice that this is a **JRE** image.

Now we are creating the runtime environment.

---

# 13. Understand `FROM` in Multi-Stage Builds

The first stage starts with:

    FROM eclipse-temurin:17-jdk-alpine AS builder

This provides the JDK and development tools required to compile the application.

The second stage starts with:

    FROM eclipse-temurin:17-jre-alpine

This creates a **new build stage** using the JRE image.

It is saying:

> **"For this stage, start with the JRE image."**

It is **not** saying:

> "Take everything from the builder stage."

The builder stage already produced:

    Main.class

We do not need to compile it again.

We only need to run:

    java Main

Therefore, we use the JRE in the runtime stage.

---

# 14. JDK vs. JRE in This Example

## Builder

    JDK
    ├── Compiler
    ├── Development tools
    └── Source code

        ↓

    Main.class

## Runtime

    JRE
    └── Main.class

The JDK is used to **build** the application.

The JRE is used to **run** the application in this example.

---

# 15. `AS builder`

Consider:

    FROM eclipse-temurin:17-jdk-alpine AS builder

`AS builder` gives the stage a name:

    builder

We can then reference that stage later:

    COPY --from=builder ...

The name is simply the **stage identifier**.

---

# 16. `COPY --from=builder`

Suppose the builder contains:

    /app/
    ├── Main.java
    ├── Main.class
    ├── build.log
    └── temporary-files/

When you write:

    COPY --from=builder /app/Main.class .

only `Main.class` is copied into the runtime stage.

It does **not** copy:

    Main.java
    build.log
    temporary-files/

This is another important benefit:

> **You explicitly choose what moves from the build stage into the final image.**

---

# 17. Multi-Stage Builds Can Have More Than Two Stages

Multi-stage builds do not have to contain only two stages.

You could have:

    FROM ... AS builder

    # Build

    FROM ... AS tester

    # Test

    FROM ... AS runtime

    # Run

Each stage can have a specific purpose.

For example:

    Builder
       ↓
    Build application
       ↓
    Tester
       ↓
    Run tests
       ↓
    Runtime
       ↓
    Run application

---

# 18. Commands

## Build Normally

    docker build -t <image-name> .

## Build Without Cache

    docker build --no-cache -t <image-name> .

## Run Container

    docker run --rm <image-name>

## List Files Inside Container

    docker run --rm <image-name> ls -l /app

## Find Files Recursively

    docker run --rm <image-name> find /app -type f

## Inspect Image History

    docker history <image-name>

## Inspect Image Metadata

    docker image inspect <image-name>

---

# 19. Key Concepts to Remember

### Multi-Stage Build

A Dockerfile with multiple `FROM` instructions where different stages have different purposes.

### Builder Stage

Contains the tools and files required to build the application.

Example:

    JDK
    javac
    Source code

### Runtime Stage

Contains only what is required to run the application.

Example:

    JRE
    Main.class

### `AS builder`

Gives a build stage a name so it can be referenced later.

### `COPY --from=builder`

Copies selected files from the builder stage into the current stage.

### New `FROM`

Every new `FROM` starts a new build stage.

It does not automatically inherit everything from the previous stage.

---

# Final Mental Model

    ┌─────────────────────────────┐
    │       BUILDER STAGE         │
    │                             │
    │ JDK                         │
    │ javac                       │
    │ Source code                 │
    │ Build tools                 │
    │                             │
    │        Compile              │
    │           ↓                 │
    │       Main.class            │
    └──────────────┬──────────────┘
                   │
                   │ COPY --from=builder
                   │
                   ↓
    ┌─────────────────────────────┐
    │       RUNTIME STAGE         │
    │                             │
    │ JRE                         │
    │ Main.class                  │
    │                             │
    │       java Main             │
    └─────────────────────────────┘

> **Core idea: Build with everything you need. Run with only what you need.**
