# Homework 2 — CI/CD Pipeline

## Project Overview

This project demonstrates a CI/CD pipeline using GitHub, Jenkins, Docker, Amazon ECR, Amazon EC2, and Docker Swarm.

## Application

A simple web application served by Nginx in a Docker container.

## Run Locally

Build the Docker image:

```bash
docker build -t homework2-app:1.0 .
```

Run the container:

```bash
docker run -d --name homework2-web -p 8081:80 homework2-app:1.0
```

Open http://localhost:8081 in a browser.

## Technology Stack

- GitHub
- Jenkins
- Docker
- Amazon ECR
- Amazon EC2
- Docker Swarm

## Deployment Goal

Automate the build, image push, and deployment process when changes are pushed to GitHub.

## Status

Local Docker application tested successfully. AWS and Jenkins deployment configuration will be added during the next stages.


## Docker Swarm Rolling Update

The application runs as a Docker Swarm service with two replicas distributed across the manager and worker nodes.

The rolling update configuration uses:
- **Parallelism:** 1
- **Delay:** 10 seconds
- **Order:** start-first
- **Failure action:** pause

Jenkins builds the Docker image, pushes it to Amazon ECR, and deploys the new image to Docker Swarm.

The deployment was verified with two running replicas, and the application returned HTTP status 200.
