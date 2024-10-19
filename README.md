Here’s the complete content formatted in Markdown for a README file, including all headings, commands, and manifests. You can copy this entire block at once.

```markdown
# CI/CD Pipeline Infrastructure Setup

This document outlines the steps to set up a CI/CD pipeline using GitHub Actions, Docker, Kubernetes, Prometheus, Grafana, SonarQube, and Trivy.

## Prerequisites

Before you begin, ensure you have the following:

- A Kubernetes cluster (e.g., Minikube, GKE, EKS, AKS)
- `kubectl` installed and configured
- Docker installed
- GitHub account and repository
- Helm installed (for easier installation of services)

## 1. Setting Up GitHub Actions

Create a `.github/workflows` directory in your repository and add a YAML file for your workflow (e.g., `ci-cd-pipeline.yml`):

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Check out code
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Log in to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: your-docker-repo/your-image:latest

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s/deployment.yaml
          kubectl apply -f k8s/service.yaml
```

### Secrets
Add your Docker Hub credentials to your GitHub repository secrets.

## 2. Docker Setup

Create a `Dockerfile` in your repository root:

```Dockerfile
# Example Dockerfile
FROM node:14

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

CMD ["npm", "start"]
```

## 3. Kubernetes Deployment

Create a directory named `k8s` in your repository and add the following files:

### Deployment Configuration (`deployment.yaml`):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: your-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: your-app
  template:
    metadata:
      labels:
        app: your-app
    spec:
      containers:
        - name: your-app
          image: your-docker-repo/your-image:latest
          ports:
            - containerPort: 3000
```

### Service Configuration (`service.yaml`):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: your-app
spec:
  type: NodePort
  ports:
    - port: 3000
      nodePort: 30001
  selector:
    app: your-app
```

## 4. Setting Up Monitoring with Prometheus and Grafana

### Install Prometheus and Grafana using Helm:

```bash
# Add Helm repo for Prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace

# Add Helm repo for Grafana
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Grafana
helm install grafana grafana/grafana --namespace monitoring
```

### Access Grafana:

1. Get the Grafana admin password:
   ```bash
   kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
   ```

2. Port-forward Grafana:
   ```bash
   kubectl port-forward --namespace monitoring service/grafana 3000:80
   ```

3. Access Grafana at `http://localhost:3000` with username `admin` and the password obtained earlier.

## 5. Setting Up SonarQube

### Deploy SonarQube:

Create a `sonarqube.yaml` file in your `k8s` directory:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sonarqube
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sonarqube
  template:
    metadata:
      labels:
        app: sonarqube
    spec:
      containers:
        - name: sonarqube
          image: sonarqube
          ports:
            - containerPort: 9000
```

Create a service for SonarQube:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sonarqube
spec:
  type: NodePort
  ports:
    - port: 9000
      nodePort: 30002
  selector:
    app: sonarqube
```

### Deploy SonarQube:

```bash
kubectl apply -f sonarqube.yaml
kubectl apply -f sonarqube-service.yaml
```

### Access SonarQube:

Access SonarQube at `http://<node-ip>:30002`.

## 6. Integrating Trivy for Vulnerability Scanning

### Install Trivy:

You can run Trivy locally or as part of your CI/CD pipeline to scan your Docker images for vulnerabilities.

```bash
# Install Trivy
brew install aquasecurity/trivy/trivy  # For macOS
```

### Scan Docker Image:

Run Trivy to scan your Docker image:

```bash
trivy image your-docker-repo/your-image:latest
```

## Conclusion

You now have a complete CI/CD pipeline set up with GitHub Actions, Docker, Kubernetes, Prometheus, Grafana, SonarQube, and Trivy. Customize the configurations as needed for your specific applications and workflows.

Feel free to reach out if you have any questions or need further assistance!
```

You can now copy this entire block into your README file. If you need any further adjustments or additions, let me know!
