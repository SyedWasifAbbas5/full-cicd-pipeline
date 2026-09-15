# Full CI/CD Pipeline

A production-style CI/CD pipeline for a containerized Flask application using **GitHub Actions, Docker, GitHub Container Registry (GHCR) and Kubernetes**.

The project demonstrates automated application testing, Docker image building, container registry publishing and Kubernetes deployment validation.

---

## Architecture

```text
Developer
   │
   │ Git Push
   ▼
GitHub Repository
   │
   ▼
GitHub Actions
   │
   ├── Application Tests
   │
   ├── Docker Build
   │
   ├── Push Image → GitHub Container Registry
   │
   └── Deployment Validation
           │
           ▼
     Kubernetes Manifests
           │
           ▼
       Kind Cluster
           │
           ▼
     Running Application
```

---

## Technology Stack

* Python
* Flask
* Gunicorn
* Docker
* GitHub Actions
* GitHub Container Registry (GHCR)
* Kubernetes
* Kind
* kubectl
* Linux

---

## Project Structure

```text
full-cicd-pipeline/
│
├── .github/
│   └── workflows/
│       └── cicd.yml
│
├── app/
│   ├── app.py
│   └── requirements.txt
│
├── k8s/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   └── service.yaml
│
├── .dockerignore
├── .gitignore
└── Dockerfile
```

---

## Application

The Flask application provides multiple endpoints for health checks and application status.

### Endpoints

```text
GET /
GET /health
GET /ready
GET /api/v1/status
```

Example health response:

```json
{
  "status": "healthy",
  "timestamp": "2026-09-15T17:32:41.439533+00:00"
}
```

---

## Docker

The application is packaged using a lightweight Python 3.12 slim image.

The container includes:

* Gunicorn production server
* Non-root application user
* Healthcheck
* Environment-based configuration
* Resource-efficient base image

The application listens on:

```text
8080
```

### Build

```bash
docker build -t full-cicd-pipeline:1.0.0 .
```

### Run

```bash
docker run -d \
  --name full-cicd-test \
  -p 8080:8080 \
  full-cicd-pipeline:1.0.0
```

### Test

```bash
curl http://localhost:8080/health
```

---

## CI/CD Pipeline

The pipeline is implemented using GitHub Actions.

Workflow file:

```text
.github/workflows/cicd.yml
```

### Pipeline stages

#### 1. Test Application

The pipeline:

* Installs Python dependencies
* Checks Python syntax
* Executes an application health test

#### 2. Build and Push Docker Image

After successful tests, GitHub Actions:

* Builds the Docker image
* Authenticates with GitHub Container Registry
* Creates image metadata and tags
* Pushes the image to GHCR

Image:

```text
ghcr.io/syedwasifabbas5/full-cicd-pipeline:latest
```

#### 3. Kubernetes Deployment Validation

The pipeline verifies that:

* CI completed successfully
* The Docker image was published
* Kubernetes manifests are available for deployment

The local Kubernetes deployment is separately validated using Kind in GitHub Codespaces.

---

## Kubernetes

The application is deployed using:

* Namespace
* Deployment
* ClusterIP Service

### Deployment features

The Kubernetes Deployment uses:

* 2 application replicas
* RollingUpdate strategy
* `maxUnavailable: 0`
* `maxSurge: 1`
* CPU and memory requests
* CPU and memory limits
* Startup probe
* Readiness probe
* Liveness probe
* Non-root security context
* Dropped Linux capabilities
* Seccomp RuntimeDefault profile

---

## Kubernetes Deployment

A local Kind cluster was used to validate the Kubernetes deployment.

Create the cluster:

```bash
kind create cluster --name cicd-demo
```

Apply the manifests:

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Check the deployment:

```bash
kubectl -n cicd-demo get pods
```

Check rollout status:

```bash
kubectl -n cicd-demo rollout status deployment/cicd-demo
```

---

## Application Validation

The Kubernetes Service was tested locally using port forwarding:

```bash
kubectl -n cicd-demo port-forward svc/cicd-demo 8080:80
```

Health endpoint:

```bash
curl http://localhost:8080/health
```

Application status:

```bash
curl http://localhost:8080/api/v1/status
```

The application successfully returned healthy and running responses through Kubernetes.

---

## CI/CD Flow

Every push to the `main` branch triggers the pipeline:

```text
Git Push
   ↓
Application Tests
   ↓
Docker Build
   ↓
Push to GHCR
   ↓
Deployment Validation
```

This creates an automated quality gate before an image is considered ready for deployment.

---

## Production Practices Demonstrated

This project demonstrates several practices commonly used in production DevOps environments:

* Automated CI/CD
* Containerized application delivery
* Container registry integration
* Immutable Docker image builds
* Kubernetes rolling deployments
* Multiple application replicas
* Health monitoring through probes
* CPU and memory resource management
* Non-root containers
* Linux capability restrictions
* Git-based workflow
* Automated deployment validation

---

## Environment

The project was developed and tested using **GitHub Codespaces**.

Kubernetes deployment testing was performed on a local **Kind** cluster running inside the Codespace.

The GitHub Actions pipeline runs on GitHub-hosted runners.

This project does not claim a production cloud deployment.

---

## Future Improvements

Possible production extensions include:

* AWS EKS deployment
* Kubernetes Ingress
* Helm charts
* Automated rollback
* Environment-specific deployments
* Kubernetes secrets management
* Trivy container vulnerability scanning
* Prometheus and Grafana monitoring
* GitHub Actions deployment to a cloud Kubernetes cluster
* Canary or blue/green deployments

---

## Author

**Syed Wasif Abbas**
