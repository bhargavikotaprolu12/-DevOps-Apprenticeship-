Why is FROM always first?

`FROM` is always first because it **defines the base image** that every other instruction builds on, and Docker’s build process requires a base before it can do anything else. 
### Why `FROM` must be first

- The `FROM` instruction **initializes a new build stage** and sets the **base image** for all subsequent instructions.
- Docker images are built as a stack of **layers** on top of a base filesystem. Without a base, there’s nothing to layer on. 
- The Dockerfile spec explicitly says:  
  > “A Dockerfile **must begin with a `FROM` instruction** (after optional parser directives, comments, and global `ARG`).” 

So the order is:

1. Optional: `# syntax=...`, comments, global `ARG`.  
2. **First real instruction:** `FROM <image>` → choose base (e.g., `ubuntu:24.04`, `node:20`, `scratch`).  
3. Then: `RUN`, `COPY`, `ENV`, `WORKDIR`, `EXPOSE`, `CMD`, etc., all executed **in that base**. 

Even for “from scratch” images, you still write:

```dockerfile
FROM scratch
```

as the first instruction. 
Interview‑ready one‑liner:

> `FROM` is always first because it sets the base image and initializes the build stage; Docker needs that foundation before it can apply any other instructions as layers on top.