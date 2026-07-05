# Day 001 Notes

---

## 1. What is an Application?

An application is software program that performs tasks for the users like watching videos, onling shopping, browsing the web, listening to songs etc etc.

Examples:
- Chrome
- VS Code
- Spotify
- WhatsApp

---

## 2. What is a Runtime?

runtime is the stage where code is being executed. Runtime software is the software layer that provides the environment and services needed for a program to execute. e.g., Python interpreter, JVM, Node.js

---

## 3. What are Dependencies?

External libraries or software your application relies on.


---

## 4. Why does software work on one machine but fail on another?

Differences in runtime versions, dependencies, operating systems, configurations, or environment.


---

## 5. What is the "Works on My Machine" problem?

The application runs in the developer's environment but fails elsewhere because the environments differ.

---
## Why can't I simply zip my project and send it?
A ZIP file only contains the application files, such as the source code, configuration files, and project assets. It does not include the runtime, installed dependencies, operating system environment, system libraries, or environment configuration required to run the application. As a result, the application may work on the developer's machine but fail on another system due to differences in software versions or missing dependencies. Docker solves this by packaging the application together with its runtime, dependencies, libraries, and configuration into a Docker image, allowing it to run consistently across different environments.
On thing to remember: docker does not package the CPU or RAM. The container uses the host machine's CPU, memory, kernel, and hardware. Docker packages the software environment, not the hardware.

---

## 6. Virtual Machines

 a virtual machine is a software-based computer that runs like a real computer inside a physical machine. It has its own operating system, apps and virtual resources such as CPU, memory, storage and network access. 

Advantages: they let you run multiple OS on one physical machine. They improve hardware utilization so one server can do more work. They are useful for  testing and development because each VM is isolated. they make backup, cloning and disaster recovery easier.

Disadvantages: Each VM requires a complete guest operating system, which consumes significant CPU, memory, and storage. They have slower boot times, require more maintenance, and have higher virtualization overhead compared to containers. As a result, fewer VMs can run on the same hardware, making them less resource-efficient and more expensive for large-scale deployments.

---

## 7. Containerization

What is it? Containerization packages an application along with its dependencies into a container. Containers share the Host OS Kernel instead of having separate operating systems.


Why is it lightweight? Containers share the same kernel, making them lightweight and fast. That means they need less storage, less memory, and they start much faster than virtual machines.


---
## 8. Docker Architecture

Client → Daemon → Images → Containers
Docker is client-server architecture. Its major components are docker client, docker server and  docker registry. 
Docker client request via command on docker CLI sends REST API requests through unix  socket and dockerd returns the request. If it’s a read-only command it sends to output or if it asks for image or container creation dockerd checks the local if the image is exists if not it pulls from the docker registry and create a container, runs it.

---

## 9. Image vs Container



---

## 10. Why Docker?

