# Day 4 — Session 4: Multi-Stage Builds — Break-It Challenge

## What We Are Testing

The purpose of this challenge is to understand:

- What the **builder stage** contains
- What the **runtime stage** contains
- What `COPY --from=builder` actually does
- Why changing the copied file can cause confusing results
- How Docker cache can make debugging misleading
- Why `--no-cache` is useful when testing Dockerfile changes

---

# 1. Our Multi-Stage Dockerfile

The original Dockerfile is:

    FROM eclipse-temurin:17-jdk-alpine AS builder

    WORKDIR /app

    COPY src/Main.java .

    RUN javac Main.java

    FROM eclipse-temurin:17-jre-alpine

    WORKDIR /app

    COPY --from=builder /app/Main.class .

    CMD ["java", "Main"]


# 2. Intentionally Modify the Multi-Stage Dockerfile

Change:

    COPY --from=builder /app/Main.class .

to:

    COPY --from=builder /app/Main.java .

The modified Dockerfile now contains:

    FROM eclipse-temurin:17-jdk-alpine AS builder

    WORKDIR /app

    COPY src/Main.java .

    RUN javac Main.java

    FROM eclipse-temurin:17-jre-alpine

    WORKDIR /app

    COPY --from=builder /app/Main.java .

    CMD ["java", "Main"]

Build the image:

    docker build -t java-multi-stage-test .

---

# 3. Initial Observation

The expectation is that:

    java Main

should fail because the runtime image no longer receives:

    Main.class

Instead, it receives:

    Main.java

However, if the application still appears to work, **do not immediately assume**:

> "Docker doesn't need `Main.class`."

Instead, investigate what is actually inside the image.

---

# 4. Inspect What Is Actually Inside the Container

Run:

    docker run --rm java-multi-stage-test ls -l /app

This asks the container to:

> Show the files inside `/app`.

You should inspect whether you actually have:

    Main.java

or:

    Main.class

or both.

---

# 5. Inspect All Files

To get a clearer view of what files actually exist inside `/app`, run:

    docker run --rm java-multi-stage-test find /app -type f

This shows all files under `/app`.

![](/screenshots/day-4_screenshots/inspect_files.png)

This is an important DevOps habit:

> **Don't assume what is inside the image. Inspect it.**

---

# 6. Could It Be Cache?

Docker uses build cache.

Suppose you previously built the image using:

    COPY --from=builder /app/Main.class .

Docker may have cached parts of that build.

Then you change the Dockerfile to:

    COPY --from=builder /app/Main.java .

When debugging, you want to be certain that Docker is rebuilding according to the current Dockerfile rather than reusing cached results that could confuse your observation.

---

# 7. Use `--no-cache`

To force Docker to rebuild without using the existing build cache, run:

    docker build --no-cache -t java-multi-stage-test .

![](/screenshots/day-4_screenshots/--no--cache.png)

Now you are testing the current Dockerfile with the existing build cache disabled.

---

# 8. Rebuild and Test

After rebuilding:

    docker run --rm java-multi-stage-test

![](/screenshots/day-4_screenshots/rebuilding.png)


---

# 9. Completely Clean Test

If you want a clean experiment, use:

    docker build --no-cache -t java-multi-stage-test .

Then inspect the runtime image:

    docker run --rm java-multi-stage-test ls -l /app

Then inspect all files:

    docker run --rm java-multi-stage-test find /app -type f

Finally, run the application:

    docker run --rm java-multi-stage-test

---

# 10. Debugging Flow

Use this sequence when testing:

    Change Dockerfile
          ↓
    Build image
          ↓
    Inspect /app
          ↓
    Check which files exist
          ↓
    Run the application
          ↓
    If results are confusing
          ↓
    Rebuild with --no-cache
          ↓
    Inspect again
          ↓
    Run again

---

# Key Lessons

### 1. Inspect Instead of Assuming

Use:

    docker run --rm java-multi-stage-test ls -l /app

and:

    docker run --rm java-multi-stage-test find /app -type f

to verify what is actually inside the container.

### 2. Understand `COPY --from`

`COPY --from=builder` copies files from the specified build stage into the current stage.

For example:

    COPY --from=builder /app/Main.class .

means:

> Copy `/app/Main.class` from the `builder` stage into the current runtime stage.

### 3. Understand the Runtime Requirement

For:

    CMD ["java", "Main"]

the runtime needs the compiled Java class:

    Main.class

The Java source file:

    Main.java

is not itself the compiled runtime artifact.

### 4. Know How to Disable the Build Cache

Use:

    docker build --no-cache -t java-multi-stage-test .

when you need to perform a build without reusing the existing build cache.

### 5. Debug Systematically

A good DevOps debugging habit is:

> **Observe → Inspect → Reproduce → Change → Test again**

Do not jump to conclusions based only on what you expected Docker to do.
