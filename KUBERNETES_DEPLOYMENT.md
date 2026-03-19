# Kubernetes & Cloud Deployment Guide

## Overview

This guide covers deploying the MERN Notes App to Kubernetes, either locally or on cloud providers.

---

## Prerequisites

- Docker images built: `mern_notes_app-backend:latest` and `mern_notes_app-frontend:latest`
- kubectl installed and configured
- Kubernetes cluster running (Docker Desktop, Minikube, or cloud)
- Basic understanding of Kubernetes concepts

---

## Local Kubernetes Setup

### Option 1: Docker Desktop (Easiest)

**Enable Kubernetes:**
1. Open Docker Desktop
2. Go to **Settings → Kubernetes**
3. Check "Enable Kubernetes"
4. Click "Apply & Restart"
5. Wait 2-3 minutes for initialization

**Verify:**
```bash
kubectl cluster-info
kubectl get nodes
```

### Option 2: Minikube

**Install Minikube:**
```bash
# Windows (with Chocolatey)
choco install minikube

# macOS
brew install minikube

# Linux
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**Start Cluster:**
```bash
minikube start --cpus=4 --memory=4096

# Verify
minikube status
kubectl cluster-info
```

**Access Dashboard:**
```bash
minikube dashboard
```

---

## Building & Pushing Docker Images

### Local Cluster (Docker Desktop/Minikube)

```bash
# Build images
docker build -t mern_notes_app-backend:latest ./server
docker build -t mern_notes_app-frontend:latest ./frontend

# Verify images exist
docker images | grep mern_notes_app
```

**For Minikube**, use Minikube's Docker daemon:
```bash
eval $(minikube docker-env)

# Then build
docker build -t mern_notes_app-backend:latest ./server
docker build -t mern_notes_app-frontend:latest ./frontend
```

### Cloud Registry (GKE, EKS, AKS)

```bash
# For Google Container Registry (GCR)
docker tag mern_notes_app-backend:latest gcr.io/your-project/mern-backend:latest
docker tag mern_notes_app-frontend:latest gcr.io/your-project/mern-frontend:latest

docker push gcr.io/your-project/mern-backend:latest
docker push gcr.io/your-project/mern-frontend:latest

# Update k8s YAML files to use gcr.io images
```

---

## Kubernetes Manifests Explained

### 1. Persistent Storage (mongo-pv-pvc.yaml)

**PersistentVolume (PV):**
- 5GB storage capacity
- hostPath for local clusters
- ReadWriteOnce access mode

**PersistentVolumeClaim (PVC):**
- Requests 5GB from PV
- Automatically binds to available PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/mongo  # Local storage location
```

**For Cloud Providers:**
```yaml
# AWS EBS
storageClassName: ebs
awsElasticBlockStore:
  volumeID: vol-xxxxx
  fsType: ext4

# Azure Disk
storageClassName: managed-premium
azureDisk:
  diskName: myDisk
  diskURI: /subscriptions/.../resourceGroups/.../providers/.../disks/myDisk

# GCP Persistent Disk
storageClassName: standard
gcePersistentDisk:
  pdName: gce-disk
  fsType: ext4
```

### 2. MongoDB Deployment (mongo-deployment.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:  
  name: notes-mongo
spec:
  replicas: 1  # Single replica for database
  selector:
    matchLabels:
      app: notes-mongo
  template:
    metadata:
      labels:
        app: notes-mongo
    spec:
      containers:
      - name: mongo
        image: mongo:6
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongo-storage
          mountPath: /data/db  # MongoDB data directory
      volumes:
      - name: mongo-storage
        persistentVolumeClaim:
          claimName: mongo-pvc  # Uses PVC created above
```

**Key Points:**
- 1 replica (single MongoDB instance)
- Volume mounted at `/data/db` (standard MongoDB location)
- Service exposes port 27017 internally

### 3. Backend Deployment (backend-deployment.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notes-backend
spec:
  replicas: 3  # 3 pod replicas
  selector:
    matchLabels:
      app: notes-backend
  strategy:
    type: RollingUpdate  # Gradual update strategy
    rollingUpdate:
      maxSurge: 1        # 1 extra pod during update
      maxUnavailable: 0  # No pods down during update
  template:
    metadata:
      labels:
        app: notes-backend
    spec:
      containers:
      - name: backend
        image: mern_notes_app-backend:latest
        imagePullPolicy: Never  # Use local image
        ports:
        - containerPort: 3000
        env:
        - name: dbURL
          value: "mongodb://notes-mongo:27017/notes"
        - name: PORT
          value: "3000"
        resources:
          requests:
            cpu: 100m      # Minimum CPU needed
            memory: 128Mi  # Minimum memory needed
          limits:
            cpu: 500m      # Maximum CPU allowed
            memory: 512Mi  # Maximum memory allowed
        livenessProbe:     # Restart if fails
          httpGet:
            path: /
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:    # Mark ready when passes
          httpGet:
            path: /
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

**Resource Requests:**
- Kubernetes reserves requested resources
- Pod only starts if resources available
- Limits prevent pods from using excessive resources

**Health Checks:**
- **Liveness Probe**: Restarts pod if unhealthy
- **Readiness Probe**: Removes from service if not ready

### 4. Frontend Deployment (frontend-deployment.yaml)

Similar to backend but:
- Image: `mern_notes_app-frontend:latest`
- Port: 80 (Nginx)
- Lower resource requests (50m CPU, 64Mi memory)

### 5. Services (services.yaml)

**Frontend Service (LoadBalancer):**
```yaml
kind: Service
metadata:
  name: notes-frontend
spec:
  type: LoadBalancer    # External access
  selector:
    app: notes-frontend
  ports:
  - port: 80           # External port
    targetPort: 80     # Pod port
```

**Backend Service (ClusterIP):**
```yaml
kind: Service
metadata:
  name: notes-backend
spec:
  type: ClusterIP       # Internal only
  selector:
    app: notes-backend
  ports:
  - port: 3000         # Internal port
    targetPort: 3000   # Pod port
```

**MongoDB Service (ClusterIP):**
```yaml
kind: Service
metadata:
  name: notes-mongo
spec:
  clusterIP: None       # Headless service
  selector:
    app: notes-mongo
  ports:
  - port: 27017
    targetPort: 27017
```

### 6. Horizontal Pod Autoscaler (hpa.yaml)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: notes-backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: notes-backend
  minReplicas: 2        # Always keep 2 pods
  maxReplicas: 5        # Never exceed 5 pods
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Target 70% CPU usage
```

**Scaling Logic:**
- CPU > 70%: Scale UP (add pods)
- CPU < 70% for 5 mins: Scale DOWN (remove pods)
- Never go below minReplicas or above maxReplicas

---

## Deployment Steps

### Step 1: Create Local Storage Directory

For Minikube/local clusters:
```bash
# Create data directory
mkdir -p /mnt/data/mongodb

# Ensure write permissions
chmod 777 /mnt/data/mongodb
```

### Step 2: Build Docker Images

```bash
cd server && docker build -t mern_notes_app-backend:latest .
cd ../frontend && docker build -t mern_notes_app-frontend:latest .
cd ..
```

### Step 3: Deploy to Kubernetes

```bash
# Deploy all resources
kubectl apply -f k8s/

# Or deploy individually
kubectl apply -f k8s/mongo-pv-pvc.yaml
kubectl apply -f k8s/mongo-deployment.yaml
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/services.yaml
kubectl apply -f k8s/hpa.yaml
```

### Step 4: Verify Deployment

```bash
# Check resources created
kubectl get all

# Get specific resources
kubectl get pods          # Check pod status
kubectl get services      # Check services
kubectl get pvc          # Check persistent volumes
kubectl get hpa          # Check autoscaler

# Detailed information
kubectl describe deployment notes-backend
kubectl describe service notes-frontend
```

### Step 5: Access Application

**For Docker Desktop:**
```bash
# localhost:80 should show the frontend
# Go to http://localhost in browser
```

**For Minikube:**
```bash
# Get external URL
minikube service notes-frontend

# Or port-forward
kubectl port-forward svc/notes-frontend 8080:80
# Visit http://localhost:8080
```

**For Cloud (GKE, EKS, AKS):**
```bash
# Get external IP (may take 1-2 minutes)
kubectl get service notes-frontend

# Visit http://<EXTERNAL-IP>
```

---

## Monitoring & Troubleshooting

### View Pod Status

```bash
# Get all pods
kubectl get pods

# Get detailed pod info
kubectl describe pod <pod-name>

# Get pod logs
kubectl logs <pod-name>

# Follow logs (like tail -f)
kubectl logs -f deployment/notes-backend

# Get logs from all pods in deployment
kubectl logs -l app=notes-backend
```

### Check Service Connectivity

```bash
# Test connectivity from within cluster
kubectl run -it --rm debug --image=busybox --restart=Never -- sh

# Inside the debug pod, try:
wget http://notes-backend:3000
wget http://notes-mongo:27017
exit
```

### Monitor HPA

```bash
# Get HPA status
kubectl get hpa

# Detailed HPA info
kubectl describe hpa notes-backend-hpa

# Watch HPA scaling in real-time
kubectl get hpa -w

# Check metrics (requires metrics-server)
kubectl top pods
kubectl top nodes
```

### Database Operations

```bash
# Connect to MongoDB pod
kubectl exec -it deployment/notes-mongo -- mongosh

# Inside MongoDB shell
use notes
db.notes.find()
db.notes.countDocuments()
db.dropDatabase()
exit
```

---

## Scaling Operations

### Manual Scaling

```bash
# Scale to specific number of replicas
kubectl scale deployment notes-backend --replicas=4

# Verify
kubectl get deployment notes-backend
```

### Rolling Updates

```bash
# Update image
kubectl set image deployment/notes-backend \
  backend=mern_notes_app-backend:v2

# Watch update progress
kubectl rollout status deployment/notes-backend

# Rollback if needed
kubectl rollout undo deployment/notes-backend
```

### Zero-Downtime Updates

Configured in deployment:
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # 1 extra pod during update
    maxUnavailable: 0  # No pods down
```

This ensures:
1. New pod starts
2. Traffic gradually shifted to new pod
3. Old pod terminates
4. Load balancer maintains availability

---

## Cleanup

```bash
# Delete all resources
kubectl delete -f k8s/

# Delete specific resource
kubectl delete deployment notes-backend

# Delete service
kubectl delete service notes-frontend

# Delete everything in namespace
kubectl delete all --all
```

---

## Production Deployment Checklist

- [ ] Update image registry (Docker Hub, ECR, GCR, etc.)
- [ ] Configure proper resource requests/limits
- [ ] Set up pod disruption budgets
- [ ] Configure network policies
- [ ] Enable RBAC (Role-Based Access Control)
- [ ] Set up monitoring and logging
- [ ] Configure backup and restore procedures
- [ ] Set up CI/CD pipeline
- [ ] Use StatefulSet for MongoDB (instead of Deployment)
- [ ] Use managed database service (Atlas, RDS, etc.)
- [ ] Configure Ingress for routing (instead of LoadBalancer)
- [ ] Set up SSL/TLS certificates
- [ ] Configure CORS for security
- [ ] Enable Pod Security Policies
- [ ] Set up resource quotas per namespace

---

## Cloud Provider Specific

### Google Cloud (GKE)

```bash
# Create cluster
gcloud container clusters create my-cluster

# Get credentials
gcloud container clusters get-credentials my-cluster

# Deploy
kubectl apply -f k8s/

# View in console
gcloud container clusters list
```

### Amazon AWS (EKS)

```bash
# Create cluster (requires eksctl)
eksctl create cluster --name my-cluster

# Get credentials
aws eks update-kubeconfig --name my-cluster

# Deploy
kubectl apply -f k8s/
```

### Microsoft Azure (AKS)

```bash
# Create resource group
az group create --name mygroup --location eastus

# Create cluster
az aks create --resource-group mygroup --name myaks

# Get credentials
az aks get-credentials --resource-group mygroup --name myaks

# Deploy
kubectl apply -f k8s/
```

---

## Next Steps

1. **Monitor deployment:** Use `kubectl get pods -w`
2. **Check logs:** Use `kubectl logs` for debugging
3. **Test API:** Test backend endpoints
4. **Load test:** Generate load to trigger HPA scaling
5. **Set up monitoring:** Implement Prometheus, Grafana
6. **Configure backups:** Regular MongoDB backups
7. **Plan disaster recovery:** Test restore procedures

---

**Happy deploying! 🚀**
