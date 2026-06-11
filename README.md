# GitOps with ArgoCD on Kubernetes

## Project Overview

This project demonstrates a complete GitOps workflow using ArgoCD and Kubernetes.

The application is containerized using Docker, stored in Docker Hub, deployed to Kubernetes, and managed through ArgoCD. All deployment changes are performed through Git, making Git the single source of truth.

---

## Technologies Used

* Kubernetes
* Minikube
* ArgoCD
* Docker
* Docker Hub
* GitHub
* Git
* Flask

---

## What is GitOps?

GitOps is a deployment methodology where Git repositories serve as the single source of truth for infrastructure and application deployments.

Instead of manually running:

```bash
kubectl apply -f deployment.yaml
```

changes are made in Git and automatically synchronized to the cluster by ArgoCD.

---

## Architecture

```text
Developer
    ↓
GitHub Repository
    ↓
ArgoCD
    ↓
Kubernetes Cluster
    ↓
Deployment
    ↓
Pods
    ↓
Service
    ↓
Users
```

---

## Project Structure

```text
gitops-project/
│
├── deployment.yaml
├── service.yaml
└── README.md
```

---

# Day 1 – Install ArgoCD

## Create Namespace

```bash
kubectl create namespace argocd
```

## Install ArgoCD

```bash
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Verify Installation

```bash
kubectl get pods -n argocd
```

## Access ArgoCD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

```text
https://localhost:8080
```

## Get Initial Password

```bash
kubectl get secret argocd-initial-admin-secret \
-n argocd \
-o jsonpath="{.data.password}" | base64 --decode
```

---

# Day 2 – Create GitOps Repository

Created a GitHub repository containing:

```text
deployment.yaml
service.yaml
```

Repository becomes the source of truth for Kubernetes deployments.

---

# Day 3 – Connect Git Repository

Created an ArgoCD Application.

Configuration:

```text
Application Name : flask-app
Project          : default
Repository URL   : GitHub Repository
Revision         : HEAD
Path             : .
Cluster          : https://kubernetes.default.svc
Namespace        : default
```

After synchronization:

```text
Git Repository
      ↓
ArgoCD
      ↓
Kubernetes
```

---

# Day 4 – Auto Sync

Enabled:

```text
Auto Sync
Self Heal
Prune
```

## Auto Sync

Automatically deploys Git changes.

## Self Heal

Detects configuration drift and restores the cluster to match Git.

Example:

```bash
kubectl scale deployment flask-deployment --replicas=10
```

ArgoCD automatically restores the desired replica count.

## Prune

Removes resources deleted from Git.

---

# Day 5 – Deployment Updates

Updated application version.

## Build Image

```bash
docker build -t flask-app:v2 .
```

## Tag Image

```bash
docker tag flask-app:v2 <dockerhub-user>/flask-app:v2
```

## Push Image

```bash
docker push <dockerhub-user>/flask-app:v2
```

## Update Manifest

```yaml
image: <dockerhub-user>/flask-app:v2
```

## Commit Changes

```bash
git add .
git commit -m "Deploy v2"
git push
```

ArgoCD automatically synchronized the deployment.

---

# Day 6 – GitOps Rollback

Traditional rollback:

```bash
kubectl rollout undo deployment/flask-deployment
```

GitOps rollback:

```bash
git revert <commit-id>
git push
```

ArgoCD detects the Git change and restores the previous deployment version.

## Key Learning

In GitOps:

```text
Git = Source of Truth
```

Manual cluster changes are eventually overwritten by ArgoCD.

---

# Day 7 – End-to-End Testing

Validated the complete deployment workflow.

```text
Code Change
     ↓
Docker Build
     ↓
Docker Push
     ↓
Update deployment.yaml
     ↓
Git Commit
     ↓
Git Push
     ↓
ArgoCD Sync
     ↓
Kubernetes Deployment
     ↓
Application Updated
```

---

# Common Commands

## ArgoCD

```bash
kubectl get pods -n argocd

kubectl get svc -n argocd

kubectl port-forward svc/argocd-server \
-n argocd 8080:443
```

## Kubernetes

```bash
kubectl get deployments

kubectl get pods

kubectl get svc

kubectl describe deployment flask-deployment

kubectl rollout history deployment/flask-deployment
```

## Git

```bash
git log --oneline

git revert <commit-id>

git push
```

---

# Troubleshooting

## ImagePullBackOff

Cause:

```text
Image tag updated in Git before image was pushed to Docker Hub.
```

Resolution:

```bash
docker push <dockerhub-user>/flask-app:<tag>
```

---

## OutOfSync State

Cause:

```text
Git state differs from cluster state.
```

Resolution:

```text
Wait for Auto Sync
or
Click Refresh in ArgoCD
```

---

## Rollback Not Working

Cause:

```bash
kubectl rollout undo
```

used directly in cluster.

Resolution:

```bash
git revert <commit-id>
git push
```

Use Git-based rollback in GitOps environments.

---

# Skills Demonstrated

* GitOps
* ArgoCD Installation
* Kubernetes Deployments
* Docker Image Management
* Auto Sync
* Self Healing
* Git-Based Rollbacks
* Continuous Deployment
* Kubernetes Troubleshooting
* Production Deployment Workflows

---

# Learning Outcomes

Through this project I learned:

* GitOps principles
* ArgoCD architecture
* Kubernetes application synchronization
* Automated deployments
* Drift detection and self-healing
* Git-based rollbacks
* Continuous deployment practices
* End-to-end Kubernetes delivery workflows

```
```
