# Quick Reference Guide

Fast lookup for common commands and configurations.

---

## Development Commands

### Backend
```bash
cd server
npm install        # Install dependencies
npm run dev       # Start dev server (port 3000)
npm test          # Run tests (if configured)
npm run build     # Build for production
```

### Frontend
```bash
cd frontend
npm install       # Install dependencies
npm run dev      # Start dev server (port 5173)
npm run build    # Build for production
npm run preview  # Preview production build
```

### Database
```bash
mongosh                              # Connect to MongoDB
use notes                           # Use notes database
db.notes.find()                     # See all notes
db.notes.deleteMany({})             # Clear collection
db.notes.createIndex({ title: 1 })  # Create index
```

---

## Docker Commands

### Build
```bash
# Backend
docker build -t mern_notes_app-backend:latest ./server

# Frontend
docker build -t mern_notes_app-frontend:latest ./frontend

# Both (from root)
docker-compose build
```

### Run
```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# Stop all services
docker-compose down

# Remove volumes (delete data)
docker-compose down -v
```

### Debug
```bash
# View logs
docker-compose logs backend
docker-compose logs frontend
docker-compose logs mongo

# Follow logs
docker-compose logs -f backend

# List images
docker images

# List containers
docker ps

# Execute command in container
docker exec -it backend bash
```

### Cleanup
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Remove everything
docker system prune -a
```

---

## Kubernetes Commands

### Cluster Info
```bash
kubectl version
kubectl cluster-info
kubectl get nodes
kubectl get namespaces
```

### Deployments
```bash
# List deployments
kubectl get deployments

# Check deployment status
kubectl describe deployment notes-backend

# Scale deployment
kubectl scale deployment notes-backend --replicas=5

# Update deployment
kubectl set image deployment/notes-backend notes-backend=mern_notes_app-backend:latest

# Restart deployment
kubectl rollout restart deployment/notes-backend

# View rollout history
kubectl rollout history deployment/notes-backend

# Rollback to previous version
kubectl rollout undo deployment/notes-backend
```

### Pods
```bash
# List pods
kubectl get pods

# Watch pods
kubectl get pods -w

# Detailed pod info
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# Follow logs
kubectl logs -f <pod-name>

# Previous logs (if crashed)
kubectl logs <pod-name> --previous

# Execute command in pod
kubectl exec -it <pod-name> -- bash

# Copy files from pod
kubectl cp <pod-name>:/path/to/file ./local/path
```

### Services
```bash
# List services
kubectl get services

# Get service details
kubectl describe service notes-backend

# Get external IP
kubectl get svc notes-frontend --watch

# Port forward
kubectl port-forward svc/notes-frontend 8080:80
```

### Apply Configuration
```bash
# Apply all files in directory
kubectl apply -f k8s/

# Apply single file
kubectl apply -f k8s/backend-deployment.yaml

# Wait for deployment
kubectl wait --for=condition=available --timeout=300s deployment/notes-backend
```

### HPA (Auto-scaling)
```bash
# List HPA
kubectl get hpa

# Detailed HPA info
kubectl describe hpa notes-backend-hpa

# Watch HPA scaling
kubectl get hpa -w

# View HPA events
kubectl describe hpa notes-backend-hpa | grep Events -A 20
```

### Debugging
```bash
# Get cluster info
kubectl cluster-info dump

# Check resource usage
kubectl top nodes
kubectl top pods

# Get events
kubectl get events

# Enable verbose logging
kubectl get pods -v=8

# Check metrics
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/default/pods
```

### Cleanup
```bash
# Delete deployment
kubectl delete deployment notes-backend

# Delete service
kubectl delete service notes-frontend

# Delete all resources
kubectl delete -f k8s/

# Delete pod (will restart)
kubectl delete pod <pod-name>

# Delete namespace
kubectl delete namespace default
```

---

## Environment Variables

### Backend (.env)
```env
# MongoDB connection
dbURL=mongodb://localhost:27017/notes
dbURL=mongodb://mongo:27017/notes              # Docker
dbURL=mongodb://notes-mongo:27017/notes        # Kubernetes
dbURL=mongodb+srv://user:pass@cluster/notes   # MongoDB Atlas

# Server config
PORT=3000
NODE_ENV=development
```

### Frontend
```env
# API URL - Set via build-time variables
VITE_API_URL=http://localhost:3000             # Development
VITE_API_URL=http://backend:3000               # Docker
VITE_API_URL=http://notes-backend:3000         # Kubernetes
```

---

## Configuration Files Quick Edit

### docker-compose.yml - Change ports
```yaml
services:
  backend:
    ports:
      - "3001:3000"  # Change to 3001 externally

  frontend:
    ports:
      - "8080:80"    # Change to 8080 externally

  mongo:
    ports:
      - "27017:27017"  # Change if needed
```

### k8s/backend-deployment.yaml - Change replicas
```yaml
spec:
  replicas: 5  # Change number of pods
```

### k8s/hpa.yaml - Change scaling config
```yaml
spec:
  minReplicas: 2      # Minimum pods
  maxReplicas: 5      # Maximum pods
  targetCPUUtilizationPercentage: 70  # CPU threshold
```

---

## API Endpoints

### Base URL
```
Development: http://localhost:3000
Docker: http://backend:3000
Kubernetes: http://notes-backend:3000
Production: Your domain
```

### Endpoints

**Get all notes**
```bash
GET /allNotes

curl http://localhost:3000/allNotes
```

**Add note**
```bash
POST /addNote
Content-Type: application/json

{
  "title": "My Note",
  "content": "Note content"
}

curl -X POST http://localhost:3000/addNote \
  -H "Content-Type: application/json" \
  -d '{"title":"Test","content":"Content"}'
```

**Get note details**
```bash
GET /noteDetails/:id

curl http://localhost:3000/noteDetails/507f1f77bcf86cd799439011
```

**Update note**
```bash
PATCH /updateNote/:id
Content-Type: application/json

{
  "title": "Updated Title",
  "content": "Updated Content"
}

curl -X PATCH http://localhost:3000/updateNote/507f1f77bcf86cd799439011 \
  -H "Content-Type: application/json" \
  -d '{"title":"Updated","content":"New content"}'
```

**Delete note**
```bash
DELETE /deleteNote/:id

curl -X DELETE http://localhost:3000/deleteNote/507f1f77bcf86cd799439011
```

**Submit feedback**
```bash
POST /submitFeedback
Content-Type: application/json

{
  "email": "user@example.com",
  "message": "Feedback message"
}

curl -X POST http://localhost:3000/submitFeedback \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","message":"Great app!"}'
```

---

## File Locations

```
MERN_Notes_App/
├── frontend/
│   ├── src/
│   │   ├── App.jsx              # Main component
│   │   ├── main.jsx             # Entry point
│   │   ├── index.css            # Styles
│   │   ├── components/          # Reusable components
│   │   └── pages/               # Page components
│   ├── Dockerfile               # Frontend container
│   ├── nginx.conf               # Nginx config
│   └── package.json             # Dependencies
│
├── server/
│   ├── server.js                # Express app
│   ├── routes/
│   │   └── routes.js            # API routes
│   ├── controllers/             # Route handlers
│   ├── models/                  # Database models
│   ├── Dockerfile               # Backend container
│   ├── .env                     # Environment config
│   └── package.json             # Dependencies
│
├── k8s/
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   ├── mongo-deployment.yaml
│   ├── mongo-pv-pvc.yaml
│   ├── services.yaml
│   └── hpa.yaml
│
├── docker-compose.yml           # Docker orchestration
├── .dockerignore                # Docker build ignore
├── DOCUMENTATION.md             # Main documentation
├── SETUP.md                     # Setup guide
├── KUBERNETES_DEPLOYMENT.md     # K8s guide
├── API_REFERENCE.md             # API documentation
└── TROUBLESHOOTING.md           # Troubleshooting guide
```

---

## Common Errors Quick Fix

| Error | Solution |
|-------|----------|
| `mongodberror: connect econnrefused` | Start MongoDB: `docker-compose up mongo` |
| `Cannot GET /api/...` | Backend not running: `npm run dev` in `server/` |
| `CORS error` | Backend has CORS enabled, restart server |
| `Port 3000 already in use` | Kill process: `lsof -i :3000 \| kill -9` |
| `Module not found` | Install: `npm install` in that directory |
| `Cannot find module 'mongoose'` | Install: `npm install` in `server/` |
| `ImagePullBackOff` in K8s | Build images with correct names: `docker build -t mern_notes_app-backend:latest ./server` |
| `CrashLoopBackOff` | Check logs: `kubectl logs <pod-name>` |
| `Metrics unavailable` in HPA | Install metrics-server: `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml` |
| `PVC pending` | Create directory: `mkdir -p /data/mongo && chmod 777 /data/mongo` |

---

## Useful Tools

- **Postman**: API testing - https://www.postman.com/
- **MongoDB Compass**: Database GUI - https://www.mongodb.com/products/compass
- **Docker Desktop**: Local Docker - https://www.docker.com/products/docker-desktop
- **VS Code**: Code editor - https://code.visualstudio.com/
- **Extensions**:
  - REST Client
  - MongoDB for VS Code
  - Docker
  - Kubernetes

---

## Performance Tips

### Development
- Use `.env` file for configuration
- Run backend and frontend separately for better debugging
- Use Redux or Context API for state management
- Implement pagination for large lists

### Docker
- Use Alpine images for smaller size
- Cache npm dependencies in Dockerfile
- Use `.dockerignore` to exclude files
- Tag images with version numbers

### Kubernetes
- Set resource requests and limits
- Use HPA for auto-scaling
- Implement health checks (liveness/readiness probes)
- Use readiness probe before marking healthy
- Monitor with Prometheus/Grafana
- Use PersistentVolumes for data

### Database
- Create indexes on frequently queried fields
- Use `.lean()` for read-only queries
- Batch database operations
- Archive old data periodically
- Monitor slow queries

---

## DevOps Checklist

- [ ] Docker images build successfully
- [ ] docker-compose up runs all services
- [ ] Frontend accessible on localhost
- [ ] Backend responds on localhost:3000
- [ ] MongoDB has data
- [ ] Kubernetes cluster running
- [ ] Images pushed to registry (if using)
- [ ] Deployments show all pods Running
- [ ] Services created and accessible
- [ ] HPA metrics available
- [ ] PersistentVolume mounted
- [ ] Health checks configured
- [ ] Resource requests set
- [ ] Documentation updated
- [ ] Production checklist reviewed

---

## Useful Links

- **Node.js**: https://nodejs.org/
- **React**: https://react.dev/
- **Express**: https://expressjs.com/
- **MongoDB**: https://www.mongodb.com/
- **Docker**: https://www.docker.com/
- **Kubernetes**: https://kubernetes.io/
- **Postman**: https://www.postman.com/
- **VS Code**: https://code.visualstudio.com/

---

**Last Updated:** March 19, 2026

Print this guide for quick reference!
