# FROM ubuntu

# RUN apt update && \
#    apt install -y python3 python3-pip python3-venv curl wget

# WORKDIR /app

# COPY . .

# RUN python3 -m venv /opt/venv && \
#    /opt/venv/bin/pip install -r requirements.txt

#CMD ["/opt/venv/bin/python", "app.py"]

# optimised but issue with base image
#FROM python:3.12

#WORKDIR /app

#COPY requirements.txt .

#RUN pip install -r requirements.txt

#COPY app.py .

#CMD ["python", "app.py"]


# truly optimised
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]

