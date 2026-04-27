# Docker + CI/CD Pipeline Project (Flask App Deployment)

## Project Overview

This project demonstrates a **complete DevOps CI/CD pipeline** where a Flask application is:

* Tested automatically
* Built into a Docker image
* Pushed to Docker Hub
* Deployed automatically on an EC2 instance

---

## Project Goal

* Automate build, test, and deployment
* Containerize application using Docker
* Push images to a registry
* Deploy application automatically

---

## Architecture

Developer → GitHub → GitHub Actions → Docker Hub → EC2 → Running Container

---

## Tech Stack

* Python (Flask)
* Docker
* GitHub Actions
* AWS EC2

---

## Project Structure

```id="str1"
project/
│
├── app/
│   └── main.py
├── requirements.txt
├── Dockerfile
└── .github/
    └── workflows/
        └── docker.yml
```

---

## Implementation Steps

### 1️) Create Dockerfile

```dockerfile id="df1"
FROM python:3.10-slim

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

EXPOSE 5000

CMD ["python3", "app/main.py"]
```

---

### 2️) Test Locally

```bash id="cmd1"
docker build -t myapp .
docker run -p 5000:5000 myapp
```

 Open:

```id="url1"
http://localhost:5000
```

---

### 3️) Push to Docker Hub

```bash id="cmd2"
docker login
docker tag myapp your-dockerhub-username/myapp
docker push your-dockerhub-username/myapp
```

---

### 4️) GitHub Secrets

Add in repository:

* `DOCKER_USERNAME`
* `DOCKER_PASSWORD`
* `EC2_HOST`
* `EC2_USER`
* `EC2_SSH_KEY`

---

### 5️) GitHub Actions Workflow

```yaml id="wf1"
name: Docker CI/CD

on:
  push:
    branches: ["main"]

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Login to DockerHub
      run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

    - name: Build Image
      run: docker build -t ${{ secrets.DOCKER_USERNAME }}/myapp .

    - name: Push Image
      run: docker push ${{ secrets.DOCKER_USERNAME }}/myapp

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest

    steps:
    - name: Deploy to EC2
      uses: appleboy/ssh-action@v1.0.0
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USER }}
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          docker pull ${{ secrets.DOCKER_USERNAME }}/myapp
          docker stop myapp || true
          docker rm myapp || true
          docker run -d -p 5000:5000 --name myapp ${{ secrets.DOCKER_USERNAME }}/myapp
```

---

### 6️) Prepare EC2 (One-Time Setup)

```bash id="cmd3"
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu
```

 Re-login after this step

---

## CI/CD Flow

1. Push code to GitHub
2. GitHub Actions triggers pipeline
3. Docker image is built
4. Image pushed to Docker Hub
5. EC2 pulls latest image
6. Container runs automatically

---

## Key Learnings

* Containerization with Docker
* CI/CD pipeline automation
* Image registry usage
* Automated deployment to EC2

---

## Output: 

<img width="699" height="281" alt="Screenshot 2026-04-27 161111" src="https://github.com/user-attachments/assets/e690208e-1b0e-4d30-86d5-65158e88d5ed" />

## Author
Dhinakar M
