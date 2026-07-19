# Kubernetes Learning → Fleet Platform on EKS

![Status](https://img.shields.io/badge/status-EKS--deployment--complete-brightgreen)

Kubernetes learning series that progresses from raw manifests to a full Helm chart deployed on a real AWS EKS cluster. Built alongside the [Fleet Platform](https://github.com/Reich-imperial/fleet-platform) project as the Kubernetes track for the same application.

Three stages, in order:

1. **`basics/`** — core Kubernetes objects, learned on Minikube
2. **`fleet-k8s/`** — the Fleet Platform backend/DB/cache as raw Kubernetes manifests
3. **`fleet-platform/`** — the same application packaged as a Helm chart, with a dedicated values file for EKS

---

## 1. Fundamentals (`basics/`)

Minikube environment used to learn the four core Kubernetes objects before touching a real application.

**Environment:**
- Minikube on Docker driver (WSL2 Ubuntu 24)
- Kubernetes v1.35.1
- kubectl configured in WSL

**What's covered:**
- Pod, Deployment, Service, ConfigMap — the four core objects
- Self-healing reconciliation loop demonstrated live (kill a pod, watch the Deployment replace it)
- NodePort Service access via Minikube
- Runtime config injection via ConfigMap

```
basics/
├── pod.yaml          # Raw Pod manifest (learning exercise)
├── deployment.yaml   # Deployment with 3 replicas
├── service.yaml      # NodePort Service on port 30080
└── configmap.yaml    # Runtime environment config
```

---

## 2. Raw manifests (`fleet-k8s/`)

The Fleet Platform backend, PostgreSQL, and Redis expressed as plain Kubernetes YAML — no Helm abstraction yet, so every object is visible and easy to reason about individually.

```
fleet-k8s/
├── backend.yaml     # Deployment + Service for the Node/Express backend
├── postgres.yaml     # StatefulSet-style Postgres with persistent storage
├── redis.yaml        # Redis Deployment + Service
├── configmap.yaml    # Non-secret runtime config
├── secret.yaml        # Kubernetes Secret (see security note below)
└── ingress.yaml       # Ingress routing rules
```

This stage exists to understand what Helm actually templates before letting Helm do it. Deploy with:

```bash
kubectl apply -f fleet-k8s/
```

---

## 3. Helm chart + EKS deployment (`fleet-platform/`)

The same application packaged as a proper Helm chart, with a separate values file for a real AWS EKS cluster.

```
fleet-platform/
├── Chart.yaml
├── values.yaml           # Local/default values
├── values-eks.yaml       # EKS-specific overrides
└── templates/
    ├── backend.yaml
    ├── postgres.yaml
    ├── redis.yaml
    ├── configmap.yaml
    ├── secret.yaml
    └── ingress.yaml       # ALB Ingress (className: alb)
```

### What the EKS deployment actually included

- **Amazon EKS cluster** running the Fleet Platform backend, PostgreSQL, and Redis
- **AWS Load Balancer Controller** installed via IRSA (IAM Roles for Service Accounts) — `iam_policy.json` in this repo is the controller's IAM policy, granting exactly the EC2/ELB describe and ALB management permissions it needs, nothing broader
- **ALB Ingress** (`className: alb`) routing external traffic into the cluster
- **EBS CSI driver** for PostgreSQL's persistent volume (`storageClassName: gp2`)
- Backend images pulled from the same ECR repository used by the EC2 deployment (`956867427400.dkr.ecr.us-east-1.amazonaws.com/fleet-platform`)

Deploy to EKS:

```bash
helm install fleet-platform ./fleet-platform -f fleet-platform/values-eks.yaml -n dev --create-namespace
```

### Security note on this repo

`values-eks.yaml` currently stores `dbPassword`, `jwtSecret`, and `jwtRefreshSecret` as base64 strings directly in the file. **Base64 is encoding, not encryption** — anyone can decode these in one command. This works for a local learning cluster but is not how the values file should look going forward:

- Rotate the three values above before treating this repo as a live reference.
- Use `kubectl create secret generic ... --from-literal=` to create secrets out-of-band instead of committing them, or
- Pull secrets from AWS Secrets Manager / SSM Parameter Store via a tool like External Secrets Operator, so nothing sensitive lives in git at all.

### Why Helm over raw manifests once the object count grew

Once Postgres, Redis, backend, ConfigMap, Secret, and Ingress were all wired together with dependencies between them, templating with variables (`values.yaml` vs `values-eks.yaml`) made it possible to deploy the identical chart to a local cluster and to EKS with only environment-specific values changing — same templates, different inputs.

### Cluster status

The EKS cluster was torn down after confirming the full stack worked end-to-end (Load Balancer Controller routing traffic, EBS-backed Postgres persisting data, Redis available to the backend) — this repo is the record of that deployment, not a currently-running cluster.

---

## Related projects

[`fleet-platform`](https://github.com/Reich-imperial/fleet-platform) — the same application's Docker Compose / EC2 / Terraform / CI-CD deployment path. This repo is the Kubernetes track for the identical app.

---

## Author

Samson Olanipekun — DevOps / Cloud Engineering
GitHub: [github.com/Reich-imperial](https://github.com/Reich-imperial)
LinkedIn: [linkedin.com/in/samson-olanipekun-devops](https://linkedin.com/in/samson-olanipekun-devops)
