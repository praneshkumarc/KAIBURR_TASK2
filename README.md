# KAIBURR Task Runner - Kubernetes Manifests

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
kubectl logs deployment/task-runner-api -n task-runner
```

## Local Minikube Walkthrough

The following commands capture the full workflow when you build the backend image directly inside a Minikube cluster and exercise the API from your host.

```bash
# Build image inside your local cluster’s Docker (Minikube example)
eval $(minikube docker-env)
cd task-runner-api
mvn -q -DskipTests package
docker build -t task-runner-api:podrunner .

# Apply manifests
cd ../k8s
kubectl apply -f namespace.yaml -f rbac.yaml -f mongo-statefulset.yaml \
              -f app-deployment.yaml -f app-service-nodeport.yaml

# Show running pods + services (screenshot)
kubectl get pods -n task-runner
kubectl get svc  -n task-runner

# Hit an endpoint from host (screenshot)
BASE=http://$(minikube ip):30080/api
curl -s $BASE/tasks | jq

# Create and run (screenshot)
curl -X PUT "$BASE/tasks" -H 'Content-Type: application/json' -d '{
  "id":"123","name":"Hello","owner":"You","command":"echo hello from busybox"
}'
curl -s -X PUT "$BASE/tasks/123/executions" | jq
```

## Tearing Everything Down

```bash
kubectl delete -f .
```

This removes all resources created from the manifests in this directory.

<img width="1246" height="396" alt="Screenshot 2025-10-16 111105" src="https://github.com/user-attachments/assets/27c70c16-373a-48ab-851e-8036679a7e39" />
<img width="1487" height="179" alt="Screenshot 2025-10-16 111044" src="https://github.com/user-attachments/assets/7d236d2f-9343-4f80-bf31-22f1976fbf9f" />
<img width="1132" height="306" alt="Screenshot 2025-10-18 203934" src="https://github.com/user-attachments/assets/efbbb3e8-c0d5-421e-acc3-08f0c6c9b93d" />


