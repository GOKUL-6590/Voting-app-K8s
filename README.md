# Voting App - DevOps Project

A distributed voting application with Docker, Kubernetes, and Jenkins CI/CD.

## Quick Start

### Local Testing with Docker Compose
```bash
docker-compose up --build -d
# Vote: http://localhost:5000
# Results: http://localhost:5001
docker-compose down
```

### Build Docker Images
```bash
docker build -t denish952/voting-app-vote:v1 -f vote/Dockerfile vote/
docker build -t denish952/voting-app-worker:v1 -f worker/Dockerfile worker/
docker build -t denish952/voting-app-result:v1 -f result/Dockerfile result/

docker push denish952/voting-app-vote:v1
docker push denish952/voting-app-worker:v1
docker push denish952/voting-app-result:v1
```

### Deploy to Kubernetes
```bash
kubectl create namespace voting-app-dev
kubectl apply -f k8s-yaml/ -n voting-app-dev
kubectl get svc -n voting-app-dev
```

## Architecture

Vote Frontend → Redis Queue → Worker → PostgreSQL → Result Dashboard

## Technologies

- Docker & Docker Compose
- Kubernetes
- Jenkins CI/CD
- Python Flask
- PostgreSQL
- Redis
- AWS EC2
- GitHub

## Project Structure

- `vote/` - Voting frontend service
- `worker/` - Background worker service
- `result/` - Results dashboard service
- `k8s-yaml/` - Kubernetes manifests
- `docker-compose.yml` - Local development setup
- `Jenkinsfile` - CI/CD pipeline

## Author
Gokul M P - gokulmp504@gmail.com
