# Hands-On Notes, Break It Intentionally
## Build, Run and Inspect a Docker Image

---

# Create the Dockerfile

## Project Structure

hands_on/
├── Dockerfile
├── app/
│   └── hello.py
└── notes.md
    
The hello.py file contains: print("Hello from my Docker container!")

## write Docker File Yourself
Requirements:

Use Python 3.12.
Work inside /app.
Copy hello.py into the image.
Make sure the container starts the Python application.

 FROM python:3.12

WORKDIR /app

COPY app/hello.py .

CMD ["python", "hello.py"]

## Build and Run
Build: docker build -t day3-session3 .
![](/screenshots/day-3_screenshots/handson_imagebuild.png)

verify: docker images
![](/screenshots/day-3_screenshots/handson_dockerimages.png)

Run Container : docker run --name day3-session3-container day3-session3
Output: Hello from my Docker container!