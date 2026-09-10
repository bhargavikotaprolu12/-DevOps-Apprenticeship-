
# AI Exercise – Docker Container Lifecycle

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