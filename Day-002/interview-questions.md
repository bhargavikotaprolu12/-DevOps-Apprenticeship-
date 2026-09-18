# Session 6 – Docker Interview Questions (Day 2)

This file contains common Docker interview questions and concise answers, written without depending on Google during the interview.


---

## 1. What happens internally when Docker receives `docker run`?

**Answer:**  
When I execute `docker run nginx`, the Docker Client sends a REST API request to the Docker Daemon through the Docker socket. The daemon first checks whether the required image exists in the local image cache. If it is not available, the daemon downloads it from a registry such as Docker Hub and stores it locally.

Next, Docker creates a container by adding a writable layer on top of the image's read‑only layers. It then:

- Creates Linux namespaces for isolation  
- Configures cgroups to control resource usage  
- Sets up networking  
- Mounts any specified volumes  

Finally, Docker starts the container by executing the `ENTRYPOINT` (if defined) and then the `CMD`. The application started by these instructions becomes the container's main process (PID 1). The daemon returns the container ID to the Docker Client, which displays it to the user.

---

## 2. What is the difference between `docker run` and `docker create`?

**Answer:**  
The core difference is:

- `docker run` = **create + start**
- `docker create` = **only create (do not start)**

### What `docker run` does

- Creates a new container from the given image  
- Starts it immediately so the main process begins running  
- If the image is missing locally, it can pull it from a registry

### What `docker create` does

- Creates a new container from the given image without starting it  
- Prepares the writable container layer and metadata, then prints the container ID  
- You must later use `docker start <container>` to actually run it

---

## 3. What is the difference between `docker create` and `docker start`?

**Answer:**  
`docker create` makes a new container from an image but does **not** start it, while `docker start` starts an existing container that was already created.

- **`docker create`**: creates the container’s writable layer and metadata, then leaves it stopped.  
- **`docker start`**: starts that already‑created stopped container; it does not create a new one.

---

## 4. Why are Docker images read‑only?

**Answer:**  
Docker images are read‑only because they are built from immutable layers. Once an image is created, Docker never changes those layers; instead, any modifications happen in a separate writable layer when a container runs.

This immutability:

- Gives us consistent, repeatable environments  
- Lets us safely reuse the same image across many containers  
- Allows Docker to cache and share layers efficiently

---

## 5. What are Docker layers?

**Answer:**  
Docker layers are the building blocks of a Docker image. Each important instruction in a Dockerfile (like `FROM`, `RUN`, `COPY`, `ADD`) creates one read‑only layer that stores the filesystem changes from that step.

These layers are:

- Stacked on top of each other to form the final image  
- Reused across multiple images and containers, which saves space and speeds up builds

---

## 6. Why are layers useful?

**Answer:**  
Layers are useful because Docker can cache and reuse them. This:

- Makes builds faster  
- Reduces image size  
- Lets different images share common base layers

So if only your app code changes, Docker reuses the OS and dependency layers instead of rebuilding everything.

---

## 7. Can multiple containers use one image?

**Answer:**  
Yes. A single Docker image can be used to create any number of containers. For example, one `nginx` image can start many `nginx` containers, each with its own ports, volumes, and configuration, but all sharing the same underlying image.

---

## 8. Where are images downloaded from?
Docker images are downloaded from a **Docker registry**, which is a service that stores and distributes images. Examples include:

- Docker Hub  
- Amazon ECR  
- GitHub Container Registry  
- Private registries

If you don’t specify a registry in the image name, Docker uses **Docker Hub** as the default and pulls from there.

We push images to a registry with `docker push` and pull them with `docker pull`. In most teams, production images live in a private registry like ECR or GCR.

---

## 9. What is Docker Hub?
Docker Hub is Docker's **default public image registry**.

- When you run `docker pull nginx`, Docker actually pulls the image from Docker Hub because it's the default registry.  
- Docker Hub contains millions of images like `nginx`, `ubuntu`, `mysql`, `python`, `redis`, `node`, and many more.

---

## 10. Can Docker run without Internet?

**Answer:**  
Docker does not require internet to **run containers**. Once images are downloaded or built and stored locally, you can use `docker run` to start containers completely offline.

Internet is only needed to:

- Pull images from remote registries (like Docker Hub, ECR)  
- Fetch dependencies during image build