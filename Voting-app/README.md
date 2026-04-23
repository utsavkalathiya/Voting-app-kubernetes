# Kubernetes Voting Application

A distributed voting application running on Kubernetes, demonstrating microservices architecture with multiple components working together.

## 📋 Overview

This application allows users to vote between two options and view real-time results. It consists of five components:

- **Voting App** - Frontend web application for casting votes (Python)
- **Redis** - In-memory data store for collecting new votes
- **Worker** - Background service that processes votes from Redis and stores them in PostgreSQL
- **PostgreSQL** - Database for storing votes
- **Result App** - Web application that displays voting results in real-time (Node.js)

## 🏗️ Architecture

```
┌─────────────┐      ┌─────────┐      ┌────────┐      ┌──────────────┐      ┌──────────────┐
│   Voting    │────▶│  Redis  │─────▶│ Worker │────▶│  PostgreSQL  │◀─────│    Result    │
│     App     │      │         │      │        │      │              │      │     App      │
│  (Python)   │      │ (Cache) │      │ (.NET) │      │  (Database)  │      │  (Node.js)   │
└─────────────┘      └─────────┘      └────────┘      └──────────────┘      └──────────────┘
      ▲                                                                              ▲
      │                                                                              │
   Port 31000                                                                    Port 31001
   (NodePort)                                                                    (NodePort)
```

## 🚀 Components

### Pods

| Component | Image | Container Port |
|-----------|-------|----------------|
| Voting App | `dockersamples/examplevotingapp_vote` | 80 |
| Result App | `dockersamples/examplevotingapp_result` | 80 |
| Worker | `dockersamples/examplevotingapp_worker` | - |
| Redis | `redis:alpine` | 6379 |
| PostgreSQL | `postgres:15-alpine` | 5432 |

### Services

| Service | Type | Port | TargetPort | NodePort | Purpose |
|---------|------|------|------------|----------|---------|
| vote | NodePort | 8080 | 80 | 31000 | External access to voting interface |
| result | NodePort | 8080 | 80 | 31001 | External access to results interface |
| redis | ClusterIP | 6379 | 6379 | - | Internal Redis access |
| db | ClusterIP | 5432 | 5432 | - | Internal PostgreSQL access |

## 📦 Prerequisites

- Kubernetes cluster (v1.19+)
- `kubectl` configured to communicate with your cluster
- Sufficient cluster resources (minimum 2 CPU cores, 4GB RAM recommended)

## 🛠️ Installation

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd Voting-app
```

### 2. Deploy all components

Deploy the application in the following order:

```bash
# Deploy data layer
kubectl apply -f redis-pod.yaml
kubectl apply -f redis-service.yaml
kubectl apply -f db-pod.yaml
kubectl apply -f db-service.yaml

# Deploy worker
kubectl apply -f worker-pod.yaml

# Deploy frontend applications
kubectl apply -f voting-app-pod.yaml
kubectl apply -f vote-service.yaml
kubectl apply -f result-app-pod.yaml
kubectl apply -f result-service.yaml
```

Or deploy everything at once:

```bash
kubectl apply -f .
```

### 3. Verify deployment

Check that all pods are running:

```bash
kubectl get pods
```

Expected output:
```
NAME     READY   STATUS    RESTARTS   AGE
vote     1/1     Running   0          1m
result   1/1     Running   0          1m
worker   1/1     Running   0          1m
redis    1/1     Running   0          1m
db       1/1     Running   0          1m
```

Check services:

```bash
kubectl get services
```

## 🌐 Access the Application

### Voting Interface
```
http://<node-ip>:31000
```

Users can vote between two options (typically "Cats" vs "Dogs").

### Results Interface
```
http://<node-ip>:31001
```

Real-time display of voting results.

**Note:** Replace `<node-ip>` with your Kubernetes node's IP address. For Minikube, use:
```bash
minikube ip
```

## 🔍 Monitoring

### View logs

```bash
# Voting app logs
kubectl logs vote

# Result app logs
kubectl logs result

# Worker logs
kubectl logs worker

# Redis logs
kubectl logs redis

# PostgreSQL logs
kubectl logs db
```

### Describe resources

```bash
kubectl describe pod <pod-name>
kubectl describe service <service-name>
```

## 🧹 Cleanup

Remove all resources:

```bash
kubectl delete -f .
```

Or delete individual components:

```bash
kubectl delete pod vote result worker redis db
kubectl delete service vote result redis db
```

## 🔧 Configuration

### Database Credentials

The PostgreSQL database uses default credentials defined in `db-pod.yaml`:
- **Username:** `postgres`
- **Password:** `postgres`

> ⚠️ **Security Note:** For production use, store credentials in Kubernetes Secrets instead of plain environment variables.

### Port Configuration

You can modify the NodePort values in the service files:
- Voting app: `vote-service.yaml` (default: 31000)
- Result app: `result-service.yaml` (default: 31001)

NodePort range must be between 30000-32767.

## 📝 File Structure

```
.
├── voting-app-pod.yaml      # Voting frontend pod
├── vote-service.yaml        # Voting app NodePort service
├── result-app-pod.yaml      # Results frontend pod
├── result-service.yaml      # Results app NodePort service
├── worker-pod.yaml          # Vote processing worker
├── redis-pod.yaml           # Redis cache pod
├── redis-service.yaml       # Redis ClusterIP service
├── db-pod.yaml              # PostgreSQL database pod
└── db-service.yaml          # PostgreSQL ClusterIP service
```

## 🐛 Troubleshooting

### Pods not starting

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Cannot access voting/result interface

1. Verify services are running: `kubectl get svc`
2. Check firewall rules allow ports 31000-31001
3. Ensure you're using the correct node IP address

### Worker not processing votes

1. Check worker logs: `kubectl logs worker`
2. Verify Redis and PostgreSQL are running
3. Ensure worker can connect to both services

### Database connection issues

1. Verify PostgreSQL pod is running: `kubectl get pod db`
2. Check credentials in `db-pod.yaml`
3. Verify db service exists: `kubectl get svc db`

## 🚀 Production Considerations

For production deployment, consider:

1. **Use Deployments instead of Pods** - Provides replica management, rolling updates, and self-healing
2. **Implement Secrets** - Store database credentials securely
3. **Add Resource Limits** - Define CPU and memory constraints
4. **Configure Persistent Volumes** - Ensure data persists across pod restarts
5. **Set up Health Checks** - Implement liveness and readiness probes
6. **Use LoadBalancer or Ingress** - Instead of NodePort for external access
7. **Enable TLS** - Secure communication between components
8. **Implement Network Policies** - Restrict pod-to-pod communication

## 📚 References

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [kubectl Command Reference](https://kubernetes.io/docs/reference/kubectl/)
- [Docker Samples - Voting App](https://github.com/dockersamples/example-voting-app)

## 📄 License

[Add your license information here]

## 👥 Contributing

[Add contribution guidelines here]

## 📧 Contact

[Add your contact information here]
