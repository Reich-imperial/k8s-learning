# Kubernetes Learning — Helix Automates

Hands-on Kubernetes learning series, built alongside the Fleet Platform project.
Documented publicly on LinkedIn as part of the Helix Automates build-in-public series.

## What this covers
- Pod, Deployment, Service, ConfigMap — the four core Kubernetes objects
- Self-healing reconciliation loop demonstrated live
- NodePort Service access via Minikube
- Runtime config injection via ConfigMap

## Environment
- Minikube on Docker driver (WSL2 Ubuntu 24)
- Kubernetes v1.35.1
- kubectl configured in WSL

## Structure
basics/
├── pod.yaml          # Raw Pod manifest (learning exercise)
├── deployment.yaml   # Deployment with 3 replicas
├── service.yaml      # NodePort Service on port 30080
└── configmap.yaml    # Runtime environment config

## Next
Deploying a simplified Fleet Platform stack to Kubernetes —
backend, PostgreSQL, Redis, with Secrets and PersistentVolumes.

## Author
Samson Olanipekun — github.com/Reich-imperial
LinkedIn: linkedin.com/in/samson-olanipekun-devops
