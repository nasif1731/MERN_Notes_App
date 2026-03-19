# MERN Notes App - Complete Documentation

![MERN Stack](https://img.shields.io/badge/MERN-Complete-brightgreen)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Deployed-blue)
![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED)
![Node.js](https://img.shields.io/badge/Node.js-18-green)
![React](https://img.shields.io/badge/React-18-blue)
![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-success)

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [Features](#features)
4. [Project Structure](#project-structure)
5. [Quick Start](#quick-start)
6. [Architecture](#architecture)
7. [Deployment](#deployment)
8. [API Endpoints](#api-endpoints)
9. [Configuration](#configuration)
10. [Troubleshooting](#troubleshooting)
11. [Contributing](#contributing)

---

## 🎯 Project Overview

**MERN Notes App** is a full-stack web application for creating, reading, updating, and deleting (CRUD) notes. It demonstrates modern web development practices with:

- ✅ MongoDB for data persistence
- ✅ Express.js for REST API
- ✅ React for responsive UI
- ✅ Node.js for backend runtime
- ✅ Docker containerization
- ✅ Kubernetes orchestration
- ✅ Horizontal Pod Autoscaling (HPA)
- ✅ Persistent storage for databases

### Key Capabilities
- Create, read, update, delete notes
- User feedback submission system
- Responsive web interface
- RESTful API design
- Cloud-ready deployment
- Production-grade containerization

---

## 🛠️ Technology Stack

### Frontend
| Technology | Version | Purpose |
|-----------|---------|---------|
| React | 18.2.0 | UI framework |
| React Router | 6.14.2 | Client-side routing |
| Axios | 1.4.0 | HTTP client |
| Vite | 4.4.5 | Build tool & dev server |
| Animate.css | 4.1.1 | CSS animations |
| SweetAlert2 | 11.7.26 | User notifications |

### Backend
| Technology | Version | Purpose |
|-----------|---------|---------|
| Node.js | 18 | Runtime environment |
| Express.js | 4.18.2 | Web framework |
| Mongoose | 7.3.1 | MongoDB ODM |
| MongoDB | 6 | Database |
| CORS | 2.8.5 | Cross-origin requests |
| dotenv | 16.3.1 | Environment variables |

### DevOps
| Technology | Version | Purpose |
|-----------|---------|---------|
| Docker | Latest | Containerization |
| Docker Compose | 3.8 | Multi-container orchestration |
| Kubernetes | 1.30.2 | Container orchestration |
| Nginx | Alpine | Reverse proxy & web server |

---

## ⚡ Features

### Note Management
- **Create Notes**: Add new notes with title and content
- **View All Notes**: Browse all notes on home page
- **View Details**: See full note information
- **Edit Notes**: Update existing notes
- **Delete Notes**: Remove unwanted notes
- **Search/Filter**: Find notes quickly (via frontend)

### User Features
- **Responsive Design**: Works on desktop, tablet, mobile
- **Real-time Updates**: Instant UI refresh after operations
- **Error Handling**: User-friendly error messages
- **Feedback System**: Submit feedback directly from app

### Infrastructure Features
- **Containerized**: Docker images for frontend & backend
- **Orchestrated**: Kubernetes deployment with services
- **Auto-scaling**: HPA scales pods based on CPU usage
- **Persistent Storage**: MongoDB with persistent volumes
- **High Availability**: Multiple replicas (3 frontend, 3 backend)
- **Health Checks**: Liveness & readiness probes

---

## 📁 Project Structure

```
MERN_Notes_App/
├── frontend/                    # React Frontend Application
│   ├── src/
│   │   ├── App.jsx             # Main React component
│   │   ├── main.jsx            # Entry point
│   │   ├── index.css           # Global styles
│   │   ├── components/         # Reusable components
│   │   │   ├── Head.jsx        # Header component
│   │   │   ├── NoteCard.jsx    # Note card display
│   │   │   ├── DetailCard.jsx  # Detail view card
│   │   │   └── AddNote.jsx     # Add note form
│   │   └── pages/              # Route pages
│   │       ├── Home.jsx        # Home page
│   │       ├── AddForm.jsx     # Add note page
│   │       ├── EditForm.jsx    # Edit note page
│   │       └── NoteDetails.jsx # Note details page
│   ├── package.json            # Dependencies & scripts
│   ├── vite.config.js          # Vite configuration
│   ├── Dockerfile             # Frontend Docker image
│   ├── nginx.conf             # Nginx configuration
│   └── index.html             # HTML template
│
├── server/                      # Node.js Backend Application
│   ├── controllers/            # Business logic
│   │   ├── noteController.js   # Note operations
│   │   └── messageController.js # Feedback operations
│   ├── models/                 # MongoDB schemas
│   │   ├── note.js             # Note schema
│   │   └── message.js          # Message schema
│   ├── routes/                 # API routes
│   │   └── routes.js           # All API endpoints
│   ├── server.js               # Express app setup
│   ├── package.json            # Dependencies
│   ├── Dockerfile             # Backend Docker image
│   └── .env                    # Environment variables
│
├── k8s/                         # Kubernetes Configuration
│   ├── backend-deployment.yaml   # Backend K8s deployment
│   ├── frontend-deployment.yaml  # Frontend K8s deployment
│   ├── mongo-deployment.yaml     # MongoDB K8s deployment
│   ├── mongo-pv-pvc.yaml        # Persistent storage config
│   ├── services.yaml            # K8s services
│   └── hpa.yaml                 # Horizontal Pod Autoscaler
│
├── docker-compose.yml           # Local Docker Compose setup
├── .dockerignore                # Docker build exclusions
├── .env.docker                  # Docker environment vars
├── README.md                     # This file
├── HPA_DEPLOYMENT_SUMMARY.md    # HPA documentation
└── LICENSE                       # License file
```

---

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Desktop (or Kubernetes cluster)
- kubectl (for Kubernetes)
- Node.js 18+ (for local development)
- MongoDB (or Docker container)

### Option 1: Docker Compose (Fastest ⚡)

```bash
# Clone the repository
git clone <your-repo-url>
cd MERN_Notes_App

# Start all services
docker-compose up --build

# Access the application
# Frontend: http://localhost
# Backend: http://localhost:3000
# MongoDB: localhost:27017
```

**Stop services:**
```bash
docker-compose down
```

### Option 2: Kubernetes (Production 🏭)

```bash
# Build Docker images
docker build -t mern_notes_app-backend:latest ./server
docker build -t mern_notes_app-frontend:latest ./frontend

# Deploy to Kubernetes
kubectl apply -f k8s/

# Check deployment status
kubectl get pods
kubectl get svc

# Access frontend
kubectl port-forward svc/notes-frontend 8080:80
# Visit: http://localhost:8080
```

### Option 3: Local Development (Development 💻)

**Backend setup:**
```bash
cd server
npm install
npm run dev
# Server runs on http://localhost:3000
```

**Frontend setup (in new terminal):**
```bash
cd frontend
npm install
npm run dev
# Frontend runs on http://localhost:5173
```

---

## 🏗️ Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Users                                     │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ↓
        ┌──────────────────────────────────────┐
        │     Nginx Load Balancer (80)         │
        │   (frontend-service in Kubernetes)   │
        └──────────────────┬───────────────────┘
                           │
        ┌──────────────────┴──────────────────┐
        ↓                                      ↓
    ┌─────────────┐                    ┌──────────────┐
    │  Frontend   │                    │  Frontend    │
    │  Pod 1,2,3  │                    │  Pod 1,2,3   │
    │  (React)    │                    │  (React)     │
    └──────┬──────┘                    └───────┬──────┘
           │                                  │
           └──────────────┬───────────────────┘
                          ↓
        ┌──────────────────────────────────────┐
        │   Backend Service (3000)             │
        │ (ClusterIP for internal routing)     │
        └──────────────────┬───────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
    ┌─────────┐       ┌─────────┐       ┌──────────┐
    │Backend  │       │Backend  │       │Backend   │
    │Pod 1    │       │Pod 2    │       │Pod 3     │
    │Express  │       │Express  │       │Express   │
    └────┬────┘       └────┬────┘       └────┬─────┘
         │                 │                 │
         └─────────────────┼─────────────────┘
                           ↓
        ┌──────────────────────────────────────┐
        │   MongoDB Service (27017)            │
        │ (ClusterIP for internal routing)     │
        └──────────────────┬───────────────────┘
                           ↓
        ┌──────────────────────────────────────┐
        │   MongoDB Pod (1 replica)            │
        │        Database                      │
        └──────────────────┬───────────────────┘
                           ↓
        ┌──────────────────────────────────────┐
        │   Persistent Volume (5GB)            │
        │   Data Storage (/data/db)            │
        └──────────────────────────────────────┘
```

### Data Flow

```
User Request
    ↓
Frontend (React)
    ↓ (HTTP/REST)
Backend API (Express)
    ↓
MongoDB
    ↓
Database Storage (Persistent Volume)
    ↓ (Response)
Backend
    ↓ (JSON)
Frontend
    ↓
User Interface Update
```

---

## 📦 Deployment

### Docker Containerization

**Frontend Dockerfile** - Multi-stage build:
- Stage 1: Node.js build environment (builds React app with Vite)
- Stage 2: Nginx production server (serves static files)
- Benefits: Smaller image size, production-optimized

**Backend Dockerfile** - Simple Node.js:
- Alpine base image (lightweight)
- Production dependencies only
- Health checks configured

### Kubernetes Deployment

**Deployment Specs:**
- Backend: 3 replicas, port 3000, ClusterIP service
- Frontend: 3 replicas, port 80, LoadBalancer service
- MongoDB: 1 replica, persistent storage

**Services:**
- Frontend Service: LoadBalancer (external access)
- Backend Service: ClusterIP (internal only)
- MongoDB Service: ClusterIP (internal only)

**Storage:**
- PersistentVolume: 5GB hostPath
- PersistentVolumeClaim: Mounted at /data/db
- Data persists across pod restarts

**Auto-scaling (HPA):**
- Min replicas: 2
- Max replicas: 5
- CPU target: 70%
- Memory target: Available

---

## 🔌 API Endpoints

All endpoints are prefixed with base URL (e.g., `http://localhost:3000`)

### Note Endpoints

#### Get All Notes
```
GET /allNotes
```
**Response:**
```json
[
  {
    "_id": "507f1f77bcf86cd799439011",
    "title": "My Note",
    "content": "Note content here",
    "createdAt": "2026-03-19T16:30:00Z",
    "updatedAt": "2026-03-19T16:30:00Z"
  }
]
```

#### Create Note
```
POST /addNote
Content-Type: application/json

{
  "title": "New Note",
  "content": "Content here"
}
```

#### Get Note Details
```
GET /noteDetails/:id
```
**Parameters:**
- `:id` - MongoDB ObjectId of the note

**Response:**
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "title": "My Note",
  "content": "Detailed content",
  "createdAt": "2026-03-19T16:30:00Z",
  "updatedAt": "2026-03-19T16:30:00Z"
}
```

#### Update Note
```
PATCH /updateNote/:id
Content-Type: application/json

{
  "title": "Updated Title",
  "content": "Updated content"
}
```

#### Delete Note
```
DELETE /deleteNote/:id
```
**Response:**
```json
{
  "success": true,
  "message": "Note deleted successfully"
}
```

### Feedback Endpoint

#### Submit Feedback
```
POST /submitFeedback
Content-Type: application/json

{
  "name": "User Name",
  "email": "user@example.com",
  "message": "Your feedback message"
}
```

---

## ⚙️ Configuration

### Environment Variables

**Backend (.env file in server/)**
```env
# Database Connection
dbURL=mongodb://localhost:27017/notes
# For Docker: mongodb://mongo:27017/notes
# For Production: mongodb+srv://username:password@cluster.mongodb.net/notes

# Server Port
PORT=3000

# Node Environment
NODE_ENV=development
```

**Docker (.env.docker file in root)**
```env
DB_URL=mongodb://mongo:27017/notes
PORT=3000
NODE_ENV=production
```

### Docker Compose Configuration

**Services:**
1. **Frontend** (Port 80)
   - Nginx reverse proxy
   - Routes API requests to backend
   - Serves React static files

2. **Backend** (Port 3000)
   - Node.js Express server
   - Connects to MongoDB
   - Health checks enabled

3. **MongoDB** (Port 27017)
   - Database service
   - Volume: mongo_data
   - Connection string: mongodb://mongo:27017/notes

### Kubernetes Configuration

**Resource Requests/Limits:**
```yaml
Backend:
  Requests: CPU 100m, Memory 128Mi
  Limits: CPU 500m, Memory 512Mi

Frontend:
  Requests: CPU 50m, Memory 64Mi
  Limits: CPU 200m, Memory 256Mi

MongoDB:
  No limits (production-grade)
```

**HPA Configuration:**
```yaml
Backend HPA:
  Min Pods: 2
  Max Pods: 5
  CPU Target: 70%

Frontend HPA:
  Min Pods: 2
  Max Pods: 5
  CPU Target: 70%
```

---

## 🐛 Troubleshooting

### Docker Issues

**Problem: Containers not starting**
```bash
# Check logs
docker-compose logs -f backend
docker-compose logs -f frontend

# Restart containers
docker-compose restart

# Full reset
docker-compose down -v
docker-compose up --build
```

**Problem: Port already in use**
```bash
# Change port in docker-compose.yml
# Or kill process using the port:
# Windows: netstat -ano | findstr :3000
# Mac/Linux: lsof -i :3000
```

### Kubernetes Issues

**Problem: Pods not starting**
```bash
# Check pod status
kubectl get pods

# Get detailed information
kubectl describe pod <pod-name>

# Check logs
kubectl logs <pod-name>

# Check events
kubectl get events
```

**Problem: Unable to connect to service**
```bash
# Verify service exists
kubectl get svc

# Test connection within cluster
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
# Inside pod: wget http://notes-backend:3000

# Port forward for local access
kubectl port-forward svc/notes-backend 3000:3000
```

**Problem: Database connection failed**
```bash
# Check MongoDB pod
kubectl get pods | grep mongo

# Check MongoDB service
kubectl get svc | grep mongo

# Test MongoDB connection
kubectl exec -it <mongo-pod> -- mongosh
```

**Problem: HPA not scaling**
```bash
# Check HPA status
kubectl get hpa
kubectl describe hpa notes-backend-hpa

# Check metrics availability
kubectl top pods
kubectl top nodes

# Verify resource requests are set
kubectl describe pod <backend-pod> | grep -A 5 "Requests"
```

### MongoDB Issues

**Problem: Cannot connect to MongoDB**
```bash
# Verify MongoDB is running
docker ps | grep mongo
# or
kubectl get pods | grep mongo

# Check connection string format
# Local: mongodb://localhost:27017/notes
# Docker: mongodb://mongo:27017/notes
# Cloud: mongodb+srv://user:pass@cluster.mongodb.net/notes
```

**Problem: Data persistence not working**
```bash
# Check persistent volume
kubectl get pv,pvc

# Verify volume mount
kubectl describe pod <mongo-pod> | grep -A 10 "Mounts"

# Check volume data
kubectl exec -it <mongo-pod> -- ls -la /data/db
```

---

## 📝 Development Guide

### Adding a new API endpoint

**1. Create controller method** (`server/controllers/noteController.js`):
```javascript
exports.your_method = async (req, res) => {
  try {
    // Your logic here
    res.json({ success: true, data: result });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

**2. Add route** (`server/routes/routes.js`):
```javascript
router.get("/yourEndpoint", noteController.your_method);
```

**3. Test endpoint:**
```bash
curl http://localhost:3000/yourEndpoint
```

### Adding a new React component

**1. Create component** (`frontend/src/components/YourComponent.jsx`):
```javascript
function YourComponent() {
  return <div>Your component</div>;
}
export default YourComponent;
```

**2. Import in App or page:**
```javascript
import YourComponent from './components/YourComponent';
```

### Building Docker images

```bash
# Backend
cd server
docker build -t mern_notes_app-backend:latest .

# Frontend
cd frontend
docker build -t mern_notes_app-frontend:latest .

# List images
docker images | grep mern_notes_app
```

---

## 📊 Monitoring & Logs

### Docker Logs
```bash
# View logs
docker-compose logs -f

# Specific service
docker-compose logs -f backend
docker-compose logs -f frontend

# MongoDB logs
docker-compose logs -f mongo
```

### Kubernetes Logs
```bash
# Pod logs
kubectl logs <pod-name>

# Follow logs
kubectl logs -f deployment/notes-backend

# Previous pod logs (after crash)
kubectl logs <pod-name> --previous

# All pods in deployment
kubectl logs -l app=notes-backend
```

### Health Checks
```bash
# Kubernetes health check
kubectl get pods

# Test endpoint health
curl http://localhost:3000

# Check service connectivity
kubectl exec -it <pod> -- curl http://notes-backend:3000
```

---

## 🔐 Security Considerations

### Production Checklist
- [ ] Change default MongoDB credentials
- [ ] Use HTTPS/TLS certificates
- [ ] Set strong environment variables
- [ ] Enable CORS only for trusted origins
- [ ] Implement authentication & authorization
- [ ] Use secrets management (HashiCorp Vault, K8s Secrets)
- [ ] Regular backups of MongoDB
- [ ] Monitor and log all API requests
- [ ] Use network policies in Kubernetes
- [ ] Implement rate limiting

### Current Configuration
- CORS enabled for all origins (`*`)
- No authentication required
- MongoDB default credentials used
- HTTP only (no HTTPS)

⚠️ **Not suitable for production without security hardening**

---

## 🚀 Scaling & Performance

### Horizontal Scaling
- Frontend scales 2-5 pods (CPU based)
- Backend scales 2-5 pods (CPU based)
- MongoDB static (1 pod with persistent storage)

### Performance Tips
- Use Kubernetes Dashboard for monitoring
- Enable caching strategies
- Implement database indexing
- Use CDN for frontend assets
- Implement pagination for large datasets
- Optimize API response times

### Load Testing
```bash
# Use Apache Bench
ab -n 1000 -c 10 http://localhost/

# Monitor HPA scaling
kubectl get hpa -w
```

---

## 📚 Additional Resources

### Documentation Files
- [HPA_DEPLOYMENT_SUMMARY.md](HPA_DEPLOYMENT_SUMMARY.md) - Detailed HPA guide
- [docker-compose.yml](docker-compose.yml) - Docker configuration
- [k8s/](k8s/) - Kubernetes manifests

### External Resources
- [MERN Stack Guide](https://www.mongodb.com/mern)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [Express.js Guide](https://expressjs.com/)
- [React Documentation](https://react.dev/)

---

## 📄 License

This project is licensed under the ISC License - see the [LICENSE](LICENSE) file for details.

---

## 👥 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📞 Support

For issues, questions, or suggestions, please:
- Open an issue on GitHub
- Contact the development team
- Check existing documentation

---

## ✅ Checklist for Production Deployment

- [ ] Set strong database credentials
- [ ] Configure HTTPS/TLS
- [ ] Set up monitoring and alerting
- [ ] Configure automated backups
- [ ] Set up CI/CD pipeline
- [ ] Perform security audit
- [ ] Load testing completed
- [ ] Disaster recovery plan in place
- [ ] Documentation updated
- [ ] Team training completed

---

**Last Updated:** March 19, 2026  
**Version:** 1.0.0  
**Maintained By:** Development Team
