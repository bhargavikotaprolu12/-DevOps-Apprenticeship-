# Production-Style Scenario

A developer gives you this Dockerfile:

```Dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

RUN python app.py
````

The developer says:

> "The image builds successfully, but I want my application to run whenever I start the container."

---

## Identify the problem and explain:

* What `RUN python app.py` actually does
* When it happens
* What should normally be used to start the application
* How to change the Dockerfile


RUN executes a command during the image build process. In this example, RUN python app.py runs the application while the image is being built. If the purpose is to start the application when a container starts, we should use CMD instead.


## Correct Dockerfile

```Dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

---

## What Changed?

### Before

```Dockerfile
RUN python app.py
```

The application runs during **image build**.

### After

```Dockerfile
CMD ["python", "app.py"]
```

The application starts when the **container starts**.

---

## Final Mental Model

```text
Dockerfile
    ↓
docker build
    ↓
RUN commands execute
    ↓
Docker Image
    ↓
docker run
    ↓
Container starts
    ↓
CMD executes
    ↓
python app.py
    ↓
Application runs
```

---

## Key Lesson

> **RUN is for executing commands during image build. CMD is normally used to define the default command that starts the application when a container starts.**
