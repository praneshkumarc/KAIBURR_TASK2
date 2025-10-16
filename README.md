# KAIBURR_TASK2
# KAIBURR Task Runner – Kubernetes Manifests

This folder contains all Kubernetes artifacts needed to deploy the Task Runner stack.

> **Note:** Inside the repository every top-level entry **other than** `k8s/` belongs to the Spring Boot backend module. Treat those files and folders (`src/`, `target/`, `pom.xml`, `Dockerfile`, etc.) as a single backend project; `k8s/` can be consumed independently when you are ready to deploy to a cluster.

## Contents

| File | Purpose |
|------|---------|
| `namespace.yaml` | Dedicated namespace for the workload. |
| `rbac.yaml` | Service account and RBAC bindings used by the backend. |
| `mongo-statefulset.yaml` | MongoDB StatefulSet, headless service, and persistent volume claims. |
| `app-deployment.yaml` | Deployment and ClusterIP service for the Task Runner backend. |
| `app-service-nodeport.yaml` | Optional NodePort exposure for on-prem access. |
| `app-service-loadbalancer.yaml` | Optional LoadBalancer service for managed clouds. |

## Prerequisites

- A Kubernetes cluster (v1.26+) and `kubectl` configured to talk to it.
- Container image for the backend pushed to a registry accessible by the cluster. Update `app-deployment.yaml` with the correct image reference before applying.

## How to Deploy

```bash
kubectl apply -f namespace.yaml
kubectl apply -f rbac.yaml
kubectl apply -f mongo-statefulset.yaml
kubectl apply -f app-deployment.yaml
kubectl apply -f app-service-nodeport.yaml      # or use app-service-loadbalancer.yaml
```

Monitor the rollout:

```bash
kubectl get pods -n task-runner
kubectl logs deployment/task-runner -n task-runner
```

## Tearing Everything Down

```bash
kubectl delete -f .
```

This removes all resources created from the manifests in this directory.

## Tips

- Adjust resource requests/limits and MongoDB storage sizes before deploying to production.
- If you use the LoadBalancer service, confirm your cloud provider provisions external IPs automatically; otherwise fall back to the NodePort service and manage ingress manually.
