# KAIBURR Task Runner - Minikube Guide

This folder contains all Kubernetes manifests required to run the Task Runner stack on a local Minikube cluster.

> **Note:** Everything outside `k8s/` belongs to the Spring Boot backend project. Build/push the backend container image from that module before deploying if you are not building inside Minikube.

## Contents

| File | Purpose |
|------|---------|
| `namespace.yaml` | Dedicated namespace for the workload. |
| `rbac.yaml` | Service account and RBAC bindings used by the backend. |
| `mongo-statefulset.yaml` | MongoDB StatefulSet, headless service, and persistent volume claims. |
| `app-deployment.yaml` | Deployment and ClusterIP service for the Task Runner backend. |
| `app-service-nodeport.yaml` | NodePort exposure of the backend service (used for Minikube). |
| `app-service-loadbalancer.yaml` | Optional LoadBalancer service for managed clouds. |

## 1. Prepare Minikube

```bash
minikube start            # launch or resume the VM
minikube status           # should show kubelet/apiserver/kubeconfig = Running
kubectl config use-context minikube
kubectl cluster-info      # verifies kubectl can reach the API server
```

If `cluster-info` returns `connection refused`, fix Minikube before continuing (restart Docker/Hyper-V, rerun `minikube start`, etc.).

## 2. Build the Backend Image Inside Minikube (optional)

If you want to avoid pushing to an external registry, build using Minikube’s Docker daemon:

```bash
eval "$(minikube docker-env)"
cd ../backend
mvn -q -DskipTests package
docker build -t task-runner-api:podrunner .
cd ../k8s
```

> Run `eval "$(minikube docker-env -u)"` later to restore your host Docker configuration.

If you already have an image in a registry, just update `app-deployment.yaml` with that reference and skip the build.

## 3. Deploy the Manifests

```bash
kubectl apply -f namespace.yaml
kubectl apply -f rbac.yaml
kubectl apply -f mongo-statefulset.yaml
kubectl apply -f app-deployment.yaml
kubectl apply -f app-service-nodeport.yaml      # exposes NodePort 30080
```

## 4. Verify Pods and Services

```bash
kubectl get pods -n task-runner
kubectl get svc  -n task-runner
kubectl logs deployment/task-runner-api -n task-runner
```

Wait until MongoDB and the backend pod are both in `Running` state before exercising the API.

## 5. Call the API from the Host

```bash
BASE=http://$(minikube ip):30080/api
curl -s $BASE/tasks | jq

curl -X PUT "$BASE/tasks" -H 'Content-Type: application/json' -d '{
  "id":"123","name":"Hello","owner":"You","command":"echo hello from busybox"
}'
curl -s -X PUT "$BASE/tasks/123/executions" | jq
```

If the request fails, double-check the NodePort (`kubectl get svc task-runner-api -n task-runner`) and confirm your firewall allows traffic to port `30080`.

## 6. Tear Everything Down

```bash
kubectl delete -f .
```

This command removes the namespace, RBAC, MongoDB StatefulSet, deployment, and services created from this directory.
<img width="1218" height="558" alt="Screenshot 2025-10-19 144627" src="https://github.com/user-attachments/assets/b1a02d9e-ba2f-4984-ad73-b60f866b6bf2" />
<img width="1363" height="595" alt="Screenshot 2025-10-19 144726" src="https://github.com/user-attachments/assets/e5f6b5f1-e66f-46c5-a15f-ddf630560563" />
<img width="1246" height="396" alt="Screenshot 2025-10-16 111105" src="https://github.com/user-attachments/assets/27c70c16-373a-48ab-851e-8036679a7e39" />
<img width="1487" height="179" alt="Screenshot 2025-10-16 111044" src="https://github.com/user-attachments/assets/7d236d2f-9343-4f80-bf31-22f1976fbf9f" />
<img width="1132" height="306" alt="Screenshot 2025-10-18 203934" src="https://github.com/user-attachments/assets/efbbb3e8-c0d5-421e-acc3-08f0c6c9b93d" />


