# Day 5 — Docker Image Optimization & Production Dockerfiles

## Mission

Learn to investigate Docker images and review Dockerfiles from a production perspective.

By the end of Day 5, I should be able to:

- Identify why a Docker image is large.
- Identify which layer is responsible for the size.
- Understand what created that layer.
- Identify unnecessary packages and files.
- Understand build-time vs runtime dependencies.
- Understand when multi-stage builds are useful.
- Review a Dockerfile and identify what should and should not be in a production image.
- Explain the reasoning behind each recommended change.

---
## Progressive Production Dockerfile Review Model

Dockerfile review will become more advanced as new Docker concepts are learned.

The goal is not to repeatedly review the same type of Dockerfile.

Instead, each new Docker concept will be added to the review process.

I still need to learn docker networks, docker volumes and docker security

After learning the major Docker concepts, will perform a complete production Dockerfile review. And keep adding more reviews to this 

The final review should require me to identify:

- Image optimization issues
- Build-cache issues
- Build/runtime dependency issues
- Multi-stage opportunities
- Networking concerns
- Storage concerns
- Security concerns
- Configuration concerns
- Production-readiness issues

The review should focus on reasoning and evidence, not memorizing rules.
## Topics Learned

### Session 1 — Image Size Optimization

- Why image size matters.
- Docker image layers and image size.
- Using `docker images` to check image size.
- Using `docker history` to investigate image layers.
- Investigation mindset: measure → inspect → identify → optimize → measure again.
- Base image selection.
- Why a smaller base image is not automatically better.
- Unnecessary packages.
- Unnecessary files.
- `.dockerignore`.
- Build artifacts.
- Build dependencies vs runtime dependencies.
- Multi-stage builds.
- Difference between Docker build cache and package-manager cache.
- Image size vs container writable-layer size.

### Session 2 — Production Dockerfile Review

- Reviewing a Dockerfile instead of blindly rewriting it.
- Identifying build-time components.
- Identifying runtime components.
- Identifying unnecessary software.
- Base image review.
- `COPY . .` review.
- `.dockerignore` vs multi-stage builds.
- Maven build environment vs Java runtime.
- `EXPOSE`.
- Environment variables and production configuration.
- `CMD` and application startup.
- Production-oriented multi-stage build thinking.

---

# Hands-On

## Session 1 — Image Size Investigation

Worked with an intentionally inefficient Ubuntu-based Python image.

### Investigation

- Built the large image.
- Encountered Ubuntu's `externally-managed-environment` error.
- Fixed the build using a Python virtual environment.
- Used `docker images` to inspect image size.
- Used `docker history` to investigate layers.
- Identified the large package-installation layer.
- Investigated which installed packages were actually required.
- Investigated the `COPY . .` layer.
- Compared different Python base images.
- Accidentally increased the image size by switching to `python:3.12`.
- Diagnosed the reason for the increase.
- Built a smaller version using `python:3.12-slim`.
- Used `pip --no-cache-dir`.
- Separated `requirements.txt` from application code to improve caching.

### Key Lesson

> Do not assume an optimization worked. Measure the image, inspect the layers, understand the cause, and compare again.

---

## Session 2 — Production Dockerfile Review

Reviewed a new Java production scenario:

```dockerfile
FROM eclipse-temurin:17-jdk

WORKDIR /app

RUN apt-get update && \
    apt-get install -y git curl vim wget

COPY . .

RUN ./mvnw clean package -DskipTests

EXPOSE 8080

ENV ENVIRONMENT=production
ENV DEBUG=true

CMD ["java", "-jar", "target/app.jar"]