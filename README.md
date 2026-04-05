# Kubernetes Automated Deployment Demo

This project demonstrates how to develop and automate deployment of a containerized application using Kubernetes and GitHub Actions.

## Project structure

- `app/`: Flask application and Dockerfile
- `k8s/`: Kubernetes manifests (namespace, deployment, service, HPA)
- `.github/workflows/ci-cd.yaml`: CI/CD pipeline to build, push, and deploy

## Prerequisites

- Docker
- Kubernetes cluster (Minikube, kind, AKS, EKS, GKE, etc.)
- kubectl configured for the cluster
- GitHub repository
- Docker Hub account

## Local build and run

```bash
docker build -t demo-app:local ./app
docker run -p 5000:5000 demo-app:local
```


Test endpoints:

- `http://localhost:5000/`
- `http://localhost:5000/healthz`

## Manual Kubernetes deployment

1. Replace `IMAGE_PLACEHOLDER` in `k8s/deployment.yaml` with your image, for example:
   - `docker.io/<dockerhub-username>/demo-app:latest`
2. Apply manifests:

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/hpa.yaml
```

3. Verify rollout:

```bash
kubectl get all -n demo-app
kubectl get hpa -n demo-app
```

## GitHub Actions CI/CD automation

The pipeline in `.github/workflows/ci-cd.yaml` triggers on pushes to `main`.

### Required GitHub secrets

Add these repository secrets:

- `DOCKERHUB_USERNAME`: Docker Hub username
- `DOCKERHUB_TOKEN`: Docker Hub access token
- `KUBE_CONFIG_DATA`: Base64 encoded kubeconfig

Generate `KUBE_CONFIG_DATA` from your local kubeconfig:

```bash
cat ~/.kube/config | base64
```

Use one-line base64 output if your shell splits lines.

### What the workflow does

1. Checks out code
2. Logs in to Docker Hub
3. Builds and pushes image with tags:
   - `<username>/demo-app:<commit-sha>`
   - `<username>/demo-app:latest`
4. Connects to Kubernetes cluster using kubeconfig secret
5. Applies Kubernetes manifests
6. Waits for deployment rollout success

## Notes

- `Service` is configured as `LoadBalancer`. For local clusters, you may switch to `NodePort`.
- HPA requires metrics server installed in your cluster.
