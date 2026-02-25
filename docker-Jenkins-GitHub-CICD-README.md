# Fullstack Docker CI/CD with Jenkins (Spring Boot + Streamlit)

Jenkins builds BOTH backend + frontend Docker images and runs them so you can open the Streamlit UI in the browser.

Assumptions:
- Jenkins already installed on an EC2 / VM (Ubuntu)
- Repository name: **Java-springboot-project-docker-jenkins**

---

## Architecture Overview

GitHub → Jenkins → Docker Build → Multi-Container → Docker Network → Live UI

Backend: Spring Boot (port 8084)  
Frontend: Streamlit (port 8501)

Frontend communicates with backend using Docker network hostname.

---

## 🖥️ EC2 Server Initial Setup (One-Time Setup)

Before running Jenkins or GitHub Actions deployment,
install required tools on EC2.

SSH into EC2:

    ssh ec2-user@<EC2-PUBLIC-IP>

Update system:

    sudo yum update -y

Install Git:

    sudo yum install git -y

Install Docker:

    sudo yum install docker -y

Start Docker:

    sudo systemctl start docker

Enable Docker on boot:

    sudo systemctl enable docker

Give ec2-user Docker permission:

    sudo usermod -aG docker ec2-user

Logout and login again:

    exit
    ssh ec2-user@<EC2-PUBLIC-IP>

Verify:

    git --version
    docker --version
## craete RDS DATABSE 
    change RDS endpoint and password in backend dockerfile
    inside SG port should be enable custom TCP :3306 

## intall git and clone repo
    yum install git
    git clone https://github.com/sasipreethamchandaka/Docker-Jenkins-Java-springboot-project.git

## STEP 0 — Fix Frontend (VERY IMPORTANT)

Open:

    frontend/Dockerfile

Change:

    ENV API_URL=http://172.31.27.229:8082

Replace with:

    ENV API_URL=http://backend:8084
    
Fix backend (VERY IMPORTANT)
open:
      
    backend/Dockerfile
    
Change:

    ENV SPRING_DATASOURCE_URL="jdbc:mysql://database-1.cgfce626af0y.us-east-1.rds.amazonaws.com:3306/datastore?createDatabaseIfNotExist=true"
    ENV SPRING_DATASOURCE_USERNAME="admin"
    ENV SPRING_DATASOURCE_PASSWORD="admin123"
    ENV SPRING_JPA_HIBERNATE_DDL_AUTO="update"
    ENV SERVER_PORT=8084
    Commit and push to GitHub.

  ## Commit & Push
      git add .
      git commit -m "Fixed repo"
      git push
    

Reason: Containers must communicate via container name inside Docker network, not EC2 IP.

---

## STEP 1 — Install Docker on Jenkins Server

SSH into Jenkins machine:

    sudo yum update
    sudo yum install docker.io -y

Give Jenkins permission:

    sudo usermod -aG docker jenkins
    sudo systemctl restart docker
    sudo systemctl restart jenkins

Verify:

    sudo su - jenkins
    docker ps

If no permission error → success.

---

## STEP 2 — Create Jenkins Pipeline Job

1. Open Jenkins UI
2. New Item
3. Name: fullstack-docker
4. Select: Pipeline
5. Click OK

---

## STEP 3 — Configure Git Repo

Pipeline → Definition → Pipeline script from SCM

SCM:

    Git

Repository URL:

    https://github.com/sasipreethamchandaka/Docker-Jenkins-Java-springboot-project.git

Branch:

    */main

Script Path:

    Jenkinsfile

Save (do not build yet)

---

## STEP 4 — Create Jenkinsfile in Project Root

Create file:

    Jenkinsfile

Paste:

    pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "springboot-backend"
        FRONTEND_IMAGE = "streamlit-frontend"
        NETWORK = "app-network"
        BACKEND_CONTAINER = "backend"
        FRONTEND_CONTAINER = "frontend"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                url: 'https://github.com/sasipreethamchandaka/Docker-Jenkins-Java-springboot-project.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    sh 'docker build -t $BACKEND_IMAGE .'
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    sh 'docker build -t $FRONTEND_IMAGE .'
                }
            }
        }

        stage('Create Network') {
            steps {
                sh 'docker network inspect $NETWORK >/dev/null 2>&1 || docker network create $NETWORK'
            }
        }

        stage('Run Backend Container') {
            steps {
                sh '''
                docker stop $BACKEND_CONTAINER || true
                docker rm $BACKEND_CONTAINER || true

                docker run -d \
                --name $BACKEND_CONTAINER \
                --network $NETWORK \
                -p 8084:8084 \
                $BACKEND_IMAGE
                '''
            }
        }

        stage('Run Frontend Container') {
            steps {
                sh '''
                docker stop $FRONTEND_CONTAINER || true
                docker rm $FRONTEND_CONTAINER || true

                docker run -d \
                --name $FRONTEND_CONTAINER \
                --network $NETWORK \
                -p 8501:8501 \
                $FRONTEND_IMAGE
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed!"
        }
    }
    }

Commit and push.

---

## STEP 5 — Run Pipeline

Go to Jenkins:

    fullstack-docker → Build Now

First build may take longer because Maven dependencies download.

---

## STEP 6 — Verify Containers

SSH into server:

    docker ps

Expected:

    backend     0.0.0.0:8084->8084
    frontend    0.0.0.0:8501->8501

---

## STEP 7 — Open Application

Frontend UI:

    http://<EC2-PUBLIC-IP>:8501

Backend API:

    http://<EC2-PUBLIC-IP>:8084

---

## STEP 8 — If Page Not Loading (AWS Security Group)

Add inbound rules:

| Type        | Port |
|------------|----|
| Custom TCP | 8084 |
| Custom TCP | 8501 |

---

## What You Achieved

You built a real DevOps flow:

GitHub → Jenkins → Docker Build → Multi-Container → Network → Live UI

This represents CI/CD for microservices without Kubernetes.


# 🚀 CI/CD Deployment to EC2 using GitHub Actions + Docker

This project demonstrates a **complete DevOps deployment pipeline**:

GitHub Push → GitHub Actions → SSH → EC2 → Docker Build → Run Containers → Live Application

The workflow automatically builds and deploys both:

- Spring Boot Backend
- Streamlit Frontend

---

## 🏗️ Architecture Overview

    Developer Push
          ↓
    GitHub Repository
          ↓
    GitHub Actions Workflow
          ↓ (SSH)
        AWS EC2
          ↓
    Docker Network
      ├── Backend (Spring Boot :8084)
      └── Frontend (Streamlit :8501)

Open in browser:

    http://<EC2-PUBLIC-IP>:8501

---

## 🔐 Required GitHub Secrets

Go to:

Repository → Settings → Secrets and variables → Actions

Add the following secrets:

| Secret Name | Description |
|-----------|------|
| EC2_HOST | Public IP of EC2 |
| EC2_USER | ec2-user |
| EC2_SSH_KEY | Private SSH key from EC2 |
| APP_PATH | /home/ec2-user/Java-springboot-project-docker-jenkins |
| AWS_ACCESS_KEY_ID | (Optional AWS access) |
| AWS_SECRET_ACCESS_KEY | (Optional AWS access) |

---

## 🐳 Docker Deployment Behavior

On every push to `main` branch:

1. Connect to EC2 via SSH
2. Pull latest code
3. Build backend Docker image
4. Build frontend Docker image
5. Create Docker network
6. Remove old containers
7. Run new containers
8. Show running containers

Zero downtime redeploy 🚀

---

## 📄 GitHub Actions Workflow

Create file:

    .github/workflows/deploy.yml

Then paste:

    name: Deploy to EC2 via SSH

    on:
      push:
        branches: [ "main" ]

    jobs:
      deploy:
        runs-on: ubuntu-latest

        steps:

        - name: Deploy Application
          uses: appleboy/ssh-action@v1.0.3
          with:
            host: ${{ secrets.EC2_HOST }}
            username: ${{ secrets.EC2_USER }}
            key: ${{ secrets.EC2_SSH_KEY }}
            script: |
              echo "🚀 Starting deployment"
            
              cd ${{ secrets.APP_PATH }}
            
              echo "📂 Current files:"
              ls -la
            
              echo "📥 Pull latest code"
              git fetch --all
              git reset --hard origin/main
            
              echo "🐳 Build backend"
              cd backend
              ls
              sudo docker build -t springboot-backend .
              cd ..
            
              echo "🐳 Build frontend"
              cd frontend
              ls
              sudo docker build -t streamlit-frontend .
              cd ..
            
              echo "🌐 Create network"
              sudo docker network create app-network || true
            
              echo "🛑 Stop old containers"
              sudo docker rm -f backend || true
              sudo docker rm -f frontend || true
            
              echo "▶️ Start backend"
              sudo docker run -d \
                --name backend \
                --network app-network \
                -p 8084:8084 \
                springboot-backend
            
              echo "▶️ Start frontend"
              sudo docker run -d \
                --name frontend \
                --network app-network \
                -p 8501:8501 \
                streamlit-frontend
            
              echo "📦 Running containers:"
              sudo docker ps
            
              echo "✅ Deployment complete"

---

## 🧪 Verify Deployment

SSH into EC2:

    docker ps

You should see:

    backend    0.0.0.0:8084->8084
    frontend   0.0.0.0:8501->8501

Open browser:

    http://<EC2-PUBLIC-IP>:8501

---

## 🎯 What This Project Demonstrates

- GitHub Actions CI/CD
- Secure SSH Deployment
- Docker Multi-Container Setup
- Backend + Frontend Networking
- Automatic Redeploy on Push
- Real Production-style Pipeline

---

## 📌 DevOps Concepts Used

- Infrastructure as Code
- Continuous Deployment
- Container Networking
- Immutable Deployments
- Least Privilege Access
- Remote Server Automation

---

## 🏁 Final Result

Every time you push code:

    git push origin main

Your live application automatically updates.

No manual SSH.
No manual docker commands.
Fully automated deployment 🎉
