# CRUD Task – MEAN App Deployment with Docker & Nginx

## 📌 Overview

This project demonstrates the containerized deployment of a full-stack MEAN (MongoDB, Express, Angular, Node.js) CRUD application using:

🐳 Docker & Docker Compose

🌐 Nginx Reverse Proxy

☁️ Ubuntu VM (Cloud Deployment)

🔄 CI/CD Pipeline (GitHub Actions)

The goal is to automate build, containerization, and deployment of a production-like environment.

---

## 🏗 Architecture Overview

```
User (Browser)
      ↓
Nginx (Port 80)
      ├── /        → Angular Frontend
      └── /api     → Node.js Backend
                         ↓
                     MongoDB
```

**Services**

- mongo → MongoDB database container

- backend → Node.js / Express API

- frontend → Angular application

- nginx → Reverse proxy & static file server

---

## 📁 Project Structure

```
├── frontend/
│   ├── Dockerfile
│   └── nginx.conf
├── backend/
│   └── Dockerfile
├── docker-compose.yml
└── .github/workflows/
    ├── frontend-deploy.yml
    └── backend-deploy.yml
```

---

## ☁️ Cloud Environment Setup

- OS: Ubuntu 22.04

- Instance Type: t2.micro (1 GB RAM)

- Open Ports: 22 (SSH), 80 (HTTP)

---

## 📸 Cloud Infrastructure (EC2 Instance)

![EC2 Instance](screenshots/Nginx%20setup%20and%20infrastructure%20details/aws-1.png)

![security-group](screenshots/Nginx%20setup%20and%20infrastructure%20details/aws-2.png)

## Ngnix

![ngnix](screenshots/Nginx%20setup%20and%20infrastructure%20details/nginx.png)

---

## 🚀 Step-by-Step Deployment Guide

### 1️⃣ Connect to VM

```
ssh -i "crud-dd-key.pem" ubuntu@<EC2_PUBLIC_IP>
```

### 2️⃣ Install Docker & Docker Compose

```
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

Install Docker Compose:

```
sudo apt install docker-compose -y
```

Verify installation:

```
docker --version
docker-compose --version
```

### 3️⃣ Clone the Repository

```
git clone https://github.com/vrpvivek22/crud-dd-task-mean-app.git
cd crud-dd-task-mean-app
```

### 4️⃣ Build & Run Application

```
docker-compose up -d --build
```

Check running containers:

```
docker ps
```

## Docker Image

![docker-image-push](screenshots/Docker%20image%20build%20and%20push%20process/docker-image-push.png)

---

## 🌐 Access the Application

- Frontend → `http://<EC2_PUBLIC_IP>/`

- Backend API → `http://<EC2_PUBLIC_IP>/api/tutorials`

- MongoDB → `mongodb://mongo:27017/tutorialsdb` (internal Docker network)

---

## 🐳 Docker Images

Docker images are built and pushed to Docker Hub:

- `<dockerhub-username>/frontend:latest`

- `<dockerhub-username>/backend:latest`

## Docker Hub

![dockerhub](screenshots/Docker%20image%20build%20and%20push%20process/docker-hub.png)

Manual build example:

```
docker build -t <dockerhub-username>/frontend ./frontend
docker build -t <dockerhub-username>/backend ./backend
docker push <dockerhub-username>/frontend
docker push <dockerhub-username>/backend
```

## 🌐 Nginx Reverse Proxy Configuration

`frontend/nginx.conf`

```
server {
    listen 80;

    index index.html;
    root /usr/share/nginx/html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**What It Does**

- Serves Angular static files

- Proxies /api requests to backend container

- Exposes entire app via Port 80

---

## 🔄 CI/CD Pipeline (GitHub Actions)

CI/CD is implemented using **two separate workflows** for better modularity:

### 1️⃣ Frontend Workflow

`frontend-deploy.yml`

- Trigger: Push changes to frontend directory

- Builds Angular Docker image

- Pushes image to Docker Hub

- SSH into VM

- Pulls latest frontend image

- Restarts frontend container

### 2️⃣ Backend Workflow

`backend-deploy.yml`

- Trigger: Push changes to backend directory

- Builds Node.js Docker image

- Pushes image to Docker Hub

- SSH into VM

- Pulls latest backend image

- Restarts backend container

## CI-CD Execution

![CI-CD Execution-1](screenshots/CI_CD%20configuration%20and%20execution/cicd-execution-1.png)
![CI-CD Execution-2](screenshots/CI_CD%20configuration%20and%20execution/cicd-execution-2.png)
![CI-CD Pipelines](screenshots/CI_CD%20configuration%20and%20execution/pipelines.png)

---

## 🧪 Testing

Open in Browser

```
http://<EC2_PUBLIC_IP>/
```

Test API via Curl

```
curl http://<EC2_PUBLIC_IP>/api/tutorials
```

## Application-UI

![Application-UI-1](screenshots/Application%20deployment%20and%20working%20UI/Application-UI-1.png)
![Application-UI-2](screenshots/Application%20deployment%20and%20working%20UI/Application-UI-2.png)

---

## 🐞 Troubleshooting Notes

- 403 Forbidden → Fixed by copying Angular build files into frontend/dist and mounting correctly in Nginx.

- Frontend static only → Caused by hardcoded localhost:8080 in Angular service. Solution: use `environment.baseUrl = '/api/tutorials'` and rebuild.

- 500 Internal Server Error → Usually indicates backend issues:

- Check backend logs: docker logs nodejs-app-container

- Common causes: MongoDB not reachable, missing environment variables, or invalid API routes.

- Fix: Ensure `MONGO_URL=mongodb://mongo:27017/tutorialsdb` is set and backend service is healthy.

- Mongo connection issues → Ensure mongo service is running and accessible via Docker network.
