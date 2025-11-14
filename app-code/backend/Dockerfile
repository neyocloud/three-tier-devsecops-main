FROM python:3.12-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        supervisor && \
    rm -rf /var/lib/apt/lists/*

COPY ./requirements /home/devops-backend/requirements
RUN mkdir -p /home/devops-backend/media
WORKDIR /home/devops-backend


RUN pip install --upgrade pip
RUN pip install -r ./requirements/requirements.txt



EXPOSE 8000

RUN mkdir -p /var/logs/devops

COPY ./scripts /home/docker/scripts


RUN sed -i 's/\r$//' /home/docker/scripts/boot.sh  && \
        chmod +x /home/docker/scripts/boot.sh



COPY ./ /home/devops-backend
ENTRYPOINT /home/docker/scripts/boot.sh
