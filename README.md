# Flask App: Hello World

A minimal Flask web application packaged in a lightweight Docker container using `python:3.11-alpine`, ready for production deployment on Kubernetes with Ingress support.

## Features

- Small Docker image (~74MB) using Alpine Linux
- Flask 3.1.2 (latest) with async and typing support
- Kubernetes manifests for Deployment, Service, and Ingress
- Liveness and readiness probes for zero-downtime deployments
- 3 replicas for high availability
- Resource limits for autoscaling and cost control
- Kubernetes-native networking and Ingress routing

## Requirements Compliance

### Introduce into existing K8s cluster with multiple web apps

- Uses `ClusterIP` Service (no port conflicts with other apps)
- Has unique labels (`app: flask-app`) (safe for coexistence)
- Exposed via its own Ingress subdomain (`flask-app.local`)
- No hardcoded values that would cause namespace/route collisions

### Do not affix pod to static host port

- Uses `containerPort: 5000` internally (for Flask)
- Exposed via internal Service (port 80 → 5000)
- No use of `hostPort` or `NodePort` (no binding to physical nodes)

### Reproducibility & Cloud Cost Efficiency (for 20+ apps)

- Uses `python:3.11-alpine` — extremely small and secure base image
- Stateless design — trivial to replicate or scale horizontally
- Liveness/readiness probes ensure HA and fast rollout/rollback
- Resource `requests` and `limits` allow:
  - Efficient pod bin-packing
  - Cluster autoscaler integration
  - Cost forecasting and budgeting
- Shared Ingress controller supports many apps on a single LoadBalancer

## Testing with Minikube

Follow these steps to deploy and verify the app using Minikube:

```bash
minikube start
minikube addons enable ingress
docker build -t flask-app:alpine .
minikube image load flask-app:alpine
kubectl apply -f k8s/
kubectl get pods
kubectl get svc
kubectl get ingress
kubectl run -it --rm --image=busybox:1.28 dns-test --restart=Never -- sh
# Inside the pod
wget -qO- http://flask-app-service.default.svc.cluster.local:80
# Returns: Hello World!
```

### Note

The application responds on the / route. I’ve verified this within the cluster using a test pod and also configured ingress to expose it at http://flask-app.local/. While minikube tunnel was running and DNS was configured, external curl requests still hung, likely due to a Minikube networking issue. In a cloud environment with managed ingress, this would be resolvable.