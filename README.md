A containerized simple application deployed on Kubernetes with a three-tier architecture including frontend (Nginx), backend (Node.js), and database (MongoDB) components.

## Architecture Overview

This project demonstrates a complete e-commerce application stack deployed on Kubernetes with:

- **Frontend**: Nginx web server serving static content
- **Backend**: Node.js application handling API requests  
- **Database**: MongoDB for data persistence
- **Monitoring**: Node monitoring with DaemonSet
- **Auto-scaling**: Both horizontal and vertical pod autoscaling

## Project Structure

```
k8s-ecom/
├── README.md
├── config.yml                 # Kind cluster configuration
├── namespace.yml              # Namespace definitions
├── frontend/
│   ├── deployment.yml         # Frontend deployment
│   └── service.yml           # Frontend service (NodePort)
├── backend/
│   ├── deployment.yml        # Backend deployment
│   ├── service.yml          # Backend service (ClusterIP)
│   ├── configmap.yml        # Backend configuration
│   └── secret.yml           # Backend secrets (empty)
├── database/
│   ├── statefulset.yml      # MongoDB StatefulSet
│   ├── service.yml          # MongoDB headless service
│   ├── pv.yml               # Persistent Volume
│   └── pvc.yml              # Persistent Volume Claim
├── autoscaling/
│   ├── hpa.yml              # Horizontal Pod Autoscaler
│   └── vpa.yml              # Vertical Pod Autoscaler
└── monitoring/
    └── daemonset.yml        # Node monitoring DaemonSet
```

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) (Kubernetes in Docker)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Quick Start

### 1. Create Kind Cluster

```bash
# Create cluster with custom configuration
kind create cluster --config=config.yml --name=ecom-cluster
```

### 2. Set up Namespaces

```bash
kubectl apply -f namespace.yml
```

### 3. Deploy Database Layer

```bash
# Apply database components
kubectl apply -f database/pv.yml
kubectl apply -f database/pvc.yml
kubectl apply -f database/statefulset.yml
kubectl apply -f database/service.yml
```

### 4. Deploy Backend Layer

```bash
# Create backend secret (add your secrets to secret.yml first)
kubectl apply -f backend/secret.yml
kubectl apply -f backend/configmap.yml
kubectl apply -f backend/deployment.yml
kubectl apply -f backend/service.yml
```

### 5. Deploy Frontend Layer

```bash
kubectl apply -f frontend/deployment.yml
kubectl apply -f frontend/service.yml
```

### 6. Set up Autoscaling

```bash
kubectl apply -f autoscaling/hpa.yml
kubectl apply -f autoscaling/vpa.yml
```

### 7. Deploy Monitoring

```bash
kubectl apply -f monitoring/daemonset.yml
```

## Access the Application

The frontend is exposed via NodePort on port 30000:

```bash
# Get node IP
kubectl get nodes -o wide

# Access application at http://<node-ip>:30000
```

For Kind clusters, you can access via:
```bash
# Port forward to access locally
kubectl port-forward -n frontend service/frontend-svc 8080:80

# Then access at http://localhost:8080
```

## Configuration

### Database Configuration

- **MongoDB**: Deployed as StatefulSet with persistent storage
- **Storage**: 1Gi persistent volume using hostPath
- **Service**: Headless service for StatefulSet communication

### Backend Configuration

- **Image**: `rakshitmalik136/nodejs-demo-app`
- **Replicas**: 2 (minimum for HPA)
- **Database Connection**: Configured via ConfigMap to connect to MongoDB
- **Secrets**: Store sensitive data in `backend/secret.yml`

### Frontend Configuration

- **Image**: `nginx:latest`
- **Replicas**: 2
- **Exposure**: NodePort service on port 30000

## Auto-scaling

### Horizontal Pod Autoscaler (HPA)
- **Target**: Backend deployment
- **CPU Threshold**: 50% utilization
- **Replica Range**: 2-5 pods

### Vertical Pod Autoscaler (VPA)
- **Target**: Backend deployment
- **Update Mode**: Auto
- **Function**: Automatically adjusts resource requests/limits

## Monitoring

- **DaemonSet**: Deploys monitoring container on every node
- **Purpose**: Node-level monitoring and logging
- **Image**: BusyBox with simple monitoring script

## Environment Variables

### Backend ConfigMap
- `DB_HOST`: mongo.database.svc.cluster.local
- `DB_PORT`: 27017

### Backend Secrets
Configure the following in `backend/secret.yml`:
- Database credentials
- JWT secrets
- API keys

## Troubleshooting

### Check Pod Status
```bash
kubectl get pods --all-namespaces
```

### View Logs
```bash
# Backend logs
kubectl logs -n backend -l app=backend

# Database logs
kubectl logs -n database -l app=mongo

# Frontend logs
kubectl logs -n frontend -l app=frontend
```

### Check Services
```bash
kubectl get services --all-namespaces
```

### Check Persistent Volumes
```bash
kubectl get pv,pvc --all-namespaces
```

## Development

### Updating Backend Image
```bash
kubectl set image deployment/backend-deploy backend=your-new-image:tag -n backend
```

### Scaling Manually
```bash
# Scale frontend
kubectl scale deployment frontend-deploy --replicas=3 -n frontend

# Scale backend (if HPA is disabled)
kubectl scale deployment backend-deploy --replicas=4 -n backend
```

## Cleanup

```bash
# Delete all resources
kubectl delete -f ./ -R

# Delete cluster
kind delete cluster --name=ecom-cluster
```

## Next Steps

- [ ] Add Ingress controller for better routing
- [ ] Implement SSL/TLS certificates
- [ ] Add comprehensive monitoring with Prometheus/Grafana
- [ ] Set up CI/CD pipeline
- [ ] Add health checks and readiness probes
- [ ] Implement network policies
- [ ] Add resource quotas and limits

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test in Kind cluster
5. Submit a pull request

---
🧑🏻‍💻 Rakshit Malik

## License

This project is licensed under the MIT License.
