# Troubleshooting Guide

Comprehensive guide for solving common issues.

---

## Table of Contents

1. [Backend Issues](#backend-issues)
2. [Frontend Issues](#frontend-issues)
3. [Database Issues](#database-issues)
4. [Docker Issues](#docker-issues)
5. [Kubernetes Issues](#kubernetes-issues)
6. [Network Issues](#network-issues)
7. [Performance Issues](#performance-issues)

---

## Backend Issues

### Problem: Server won't start

**Symptoms:**
```
Cannot start server
Port already in use
EADDRINUSE: address already in use
```

**Solutions:**

**1. Port already in use:**
```bash
# Windows - Find process using port 3000
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# macOS/Linux
lsof -i :3000
kill -9 <PID>

# Or use different port
PORT=3001 npm run dev
```

**2. Dependencies not installed:**
```bash
cd server
npm install
npm run dev
```

**3. Wrong Node version:**
```bash
# Check Node version
node --version  # Should be v18+

# If outdated, update Node.js
# Visit: https://nodejs.org/
```

---

### Problem: Cannot connect to MongoDB

**Symptoms:**
```
MongooseError: Cannot connect to MongoDB
Connection refused
ECONNREFUSED 127.0.0.1:27017
```

**Solutions:**

**1. MongoDB not running:**
```bash
# Windows
tasklist | findstr mongod
# If not listed, start MongoDB

# macOS
brew services start mongodb-community

# Linux
sudo systemctl start mongod

# Docker
docker run -d -p 27017:27017 --name notes-mongo mongo:6
```

**2. Wrong connection string:**
```env
# Check .env file
dbURL=mongodb://localhost:27017/notes

# For Docker Compose
dbURL=mongodb://mongo:27017/notes

# For MongoDB Atlas
dbURL=mongodb+srv://username:password@cluster.mongodb.net/notes
```

**3. MongoDB Atlas IP whitelist:**
- Go to MongoDB Atlas console
- Network Access → IP Whitelist
- Add your IP address (or 0.0.0.0/0 for development)

**Test connection:**
```bash
# Using mongosh
mongosh mongodb://localhost:27017/notes

# Or test from Node
node -e "const m=require('mongoose');m.connect('mongodb://localhost:27017/notes').then(()=>console.log('Connected')).catch(e=>console.log('Error:',e.message))"
```

---

### Problem: API endpoint returns 404

**Symptoms:**
```
GET /allNotes → 404 Not Found
POST /addNote → 404 Not Found
```

**Solutions:**

**1. Wrong endpoint name:**
```bash
# Check available endpoints
curl http://localhost:3000/allNotes
curl http://localhost:3000/addNote

# Verify against server/routes/routes.js
```

**2. Server not running:**
```bash
# Check if server is running
curl http://localhost:3000

# Start server if not running
npm run dev
```

**3. Wrong port:**
```bash
# Check configured port
echo $PORT or echo %PORT%

# Default is 3000, check in .env
PORT=3000
```

**Debug:**
```bash
# Test basic connectivity
curl -v http://localhost:3000/allNotes

# Check server logs
npm run dev  # Should show "listening on port 3000"
```

---

### Problem: API returns 500 Server Error

**Symptoms:**
```
POST /addNote → 500 Internal Server Error
GET /noteDetails/... → 500 Internal Server Error
```

**Solutions:**

**1. Check server logs:**
```bash
# Runtime error visible in console
npm run dev

# Look for error stack trace
```

**2. Common causes:**
- Invalid MongoDB document ID
- Missing required fields
- Database connection lost

**3. Enable better error logging:**
```javascript
// Add to server.js
app.use((err, req, res, next) => {
  console.error("Error:", err.stack);
  res.status(500).json({ error: err.message });
});
```

**4. Test with valid data:**
```bash
# Test with curl
curl -X POST http://localhost:3000/addNote \
  -H "Content-Type: application/json" \
  -d '{"title":"Test","content":"Test content"}'
```

---

### Problem: CORS error when calling from frontend

**Symptoms:**
```
Access to XMLHttpRequest has been blocked by CORS policy
No 'Access-Control-Allow-Origin' header
```

**Solutions:**

**1. CORS already enabled (current setup):**
```javascript
// server.js already has:
app.use(cors({ origin: "*" }))
```

**2. If still getting error, restart server:**
```bash
Ctrl+C
npm run dev
```

**3. Check frontend is using correct API URL:**
```javascript
// Frontend should call
axios.get('http://localhost:3000/allNotes')

// NOT
axios.get('localhost:3000/allNotes')
axios.get('/allNotes')  // Only works with same domain
```

**4. For Kubernetes, use service name:**
```javascript
// Inside Kubernetes cluster
axios.get('http://notes-backend:3000/allNotes')
```

---

### Problem: Database operations extremely slow

**Solutions:**

**1. Check database performance:**
```bash
# Connect to MongoDB
mongosh

# Check indexes
db.notes.getIndexes()

# Create index for faster queries
db.notes.createIndex({ title: 1 })
db.notes.createIndex({ createdAt: -1 })
```

**2. Monitor active operations:**
```bash
# In mongosh
db.currentOp()

# Kill slow operation
db.killOp(opid)
```

**3. Check database size:**
```bash
# In mongosh
db.notes.stats()
db.stats()

# If too large, consider archiving old data
```

**4. Enable connection pooling:**
```javascript
// server.js - Already optimized
mongoose.connect(dbURL, {
  maxPoolSize: 10,
  minPoolSize: 5
})
```

---

## Frontend Issues

### Problem: Page blank or won't load

**Symptoms:**
```
Blank page in browser
Nothing rendered
Console shows errors
```

**Solutions:**

**1. Check dev server is running:**
```bash
npm run dev

# Should show:
# VITE v4.4.5 ready in 123ms
# ➜ Local: http://localhost:5173/
```

**2. Check browser console (F12):**
- Look for JavaScript errors
- Check Network tab for failed requests

**3. Clear cache and restart:**
```bash
# Stop dev server (Ctrl+C)
# Clear cache
rm -rf node_modules/.vite
npm cache clean --force

# Restart
npm run dev
```

**4. Check port:**
```bash
# If 5173 in use, try different port
npm run dev -- --port 5174
```

---

### Problem: API calls failing (network errors)

**Symptoms:**
```
POST /addNote → Failed to fetch
GET /allNotes → net::ERR_FAILED
Cannot GET data from server
```

**Solutions:**

**1. Verify backend is running:**
```bash
# Test backend directly
curl http://localhost:3000/allNotes

# If fails, start backend
cd server
npm run dev
```

**2. Check API URL in code:**
```javascript
// Search for API endpoints in components
// Should point to http://localhost:3000

// Check in:
// - frontend/src/pages/Home.jsx
// - frontend/src/components/*.jsx
```

**3. Test with Postman:**
- Download [Postman](https://www.postman.com/downloads/)
- Set URL to `http://localhost:3000/allNotes`
- Send GET request
- If works in Postman but not in app, check frontend code

**4. Check CORS headers:**
```bash
# With curl, check response headers
curl -i http://localhost:3000/allNotes

# Should include:
# Access-Control-Allow-Origin: *
```

---

### Problem: Styles not loading

**Symptoms:**
```
Page loads but looks plain/unstyled
CSS files 404
No colors or formatting
```

**Solutions:**

**1. Check CSS imports:**
```javascript
// frontend/src/App.jsx should have:
import './index.css'
```

**2. Verify CSS file exists:**
```bash
ls -la frontend/src/index.css
```

**3. Clear browser cache:**
```
Ctrl+Shift+Delete (or Cmd+Shift+Delete on Mac)
Select "All time"
Clear cache
```

**4. Rebuild:**
```bash
# Stop dev server (Ctrl+C)
npm run dev
```

---

### Problem: React Router (page navigation) not working

**Symptoms:**
```
Routes not working
Can't navigate between pages
URL changes but content doesn't
```

**Solutions:**

**1. Check Router setup:**
```javascript
// App.jsx should have BrowserRouter
import { BrowserRouter as Router } from 'react-router-dom'

// App must be wrapped:
<Router>
  <Routes>
    <Route path="/" element={<Home />} />
  </Routes>
</Router>
```

**2. Verify routes are defined:**
```bash
grep -r "Route path" frontend/src/
```

**3. Check console for errors:**
- Open DevTools (F12)
- Look for React/Router errors
- Check Network tab for failed requests

**4. Test simple route:**
```javascript
// Try simple component
<Route path="/test" element={<div>Test</div>} />

// Navigate to /test in browser
```

---

### Problem: Form submission not working

**Symptoms:**
```
Submit button doesn't work
Data not sent to API
Form resets but nothing happens
```

**Solutions:**

**1. Check form handler:**
```javascript
// Should have preventDefault
const handleSubmit = (e) => {
  e.preventDefault()  // Required!
  // Send data
}
```

**2. Verify axios call:**
```javascript
// Should be POST or PATCH
axios.post('/addNote', data)
  .then(res => console.log('Success:', res.data))
  .catch(err => console.log('Error:', err))
```

**3. Check API URL:**
```javascript
// Should include full backend URL
axios.post('http://localhost:3000/addNote', data)
```

**4. Test in console:**
```javascript
// Open DevTools console (F12)
fetch('http://localhost:3000/allNotes')
  .then(r => r.json())
  .then(d => console.log(d))
```

---

## Database Issues

### Problem: No data persists after restart

**Symptoms:**
```
Added data → restarted → data gone
MongoDB doesn't save changes
Database appears empty
```

**Solutions:**

**1. Check database connection:**
```bash
mongosh
use notes
db.notes.find()
```

**2. Verify data was actually saved:**
```javascript
// Check before restart
curl http://localhost:3000/allNotes

// Should return data
```

**3. For Docker, use volume:**
```yaml
# docker-compose.yml should have:
services:
  mongo:
    volumes:
      - mongo_data:/data/db  # Named volume

volumes:
  mongo_data:  # Persistent storage
```

**4. Local development:**
```bash
# Data location
/data/mongo  # Linux/Mac
C:\data\mongo  # Windows

# Ensure directory exists and has write permissions
```

---

### Problem: Cannot delete or modify data

**Symptoms:**
```
DELETE request returns error
PATCH request doesn't update
Update shows success but no change
```

**Solutions:**

**1. Verify MongoDB ObjectId format:**
```bash
# Valid ObjectId format (24 hex characters)
507f1f77bcf86cd799439011

# If invalid, will cause error
```

**2. Check document exists:**
```bash
mongosh
use notes
db.notes.findOne({_id: ObjectId("507f1f77bcf86cd799439011")})
```

**3. Check permissions:**
```javascript
// Make sure MongoDB user has write permissions
// Default development setup allows all
```

**4. Test manually:**
```bash
# Delete manually
mongosh
use notes
db.notes.deleteOne({_id: ObjectId("...")})
```

---

### Problem: Database connection pool exhausted

**Symptoms:**
```
Too many connections
Connection pool limit exceeded
Random timeouts
```

**Solutions:**

**1. Check connection settings:**
```javascript
// server.js - Verify pool settings
mongoose.connect(dbURL, {
  maxPoolSize: 10,    // Default
  minPoolSize: 5      // Default
})
```

**2. Increase pool size:**
```javascript
mongoose.connect(dbURL, {
  maxPoolSize: 20,    // Increase if needed
  minPoolSize: 10
})
```

**3. Monitor connections:**
```bash
mongosh
db.serverConnectionStatus()
```

---

## Docker Issues

### Problem: Container won't start

**Symptoms:**
```
docker-compose up → Error
Container exits immediately
Cannot start container
```

**Solutions:**

**1. Check logs:**
```bash
docker-compose logs backend
docker-compose logs frontend
docker-compose logs mongo
```

**2. Verify images exist:**
```bash
docker images | grep mern
```

**3. Rebuild images:**
```bash
docker-compose build --no-cache
docker-compose up
```

**4. Check for syntax errors:**
```bash
docker-compose config  # Validates syntax
```

---

### Problem: Port conflicts

**Symptoms:**
```
Port 3000 already in use
Port 80 already in use
Address already in use
```

**Solutions:**

**1. Change ports in docker-compose.yml:**
```yaml
services:
  backend:
    ports:
      - "3001:3000"  # Change left number (host port)
```

**2. Kill existing processes:**
```bash
# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# macOS/Linux
lsof -i :3000
kill -9 <PID>
```

**3. Verify ports are free:**
```bash
# Should return empty or port not in use
netstat -tuln | grep 3000
```

---

### Problem: Cannot connect between containers

**Symptoms:**
```
Backend cannot reach MongoDB
Frontend cannot reach Backend
Connection refused errors
```

**Solutions:**

**1. Use service names (not localhost):**
```javascript
// Frontend (in Docker)
axios.get('http://backend:3000/allNotes')

// Backend (in Docker)
mongoose.connect('mongodb://mongo:27017/notes')
```

**2. Check networking:**
```bash
docker-compose ps  # See all containers

docker exec -it <container> ping <service>
# Example: docker exec -it backend ping mongo
```

**3. Verify environment variables:**
```bash
docker-compose config  # See final config
docker exec <container> env  # See container env vars
```

---

### Problem: Data loss when container restarts

**Solutions:**

**1. Use volumes:**
```yaml
# docker-compose.yml
services:
  mongo:
    volumes:
      - mongo_data:/data/db  # Persist data

volumes:
  mongo_data:  # Define volume
```

**2. Verify volume is mounted:**
```bash
docker volume ls
docker volume inspect <volume_name>
```

**3. Check logs for startup issues:**
```bash
docker-compose logs -f mongo
```

---

## Kubernetes Issues

### Problem: Pods not starting (ErrImageNeverPull)

**Symptoms:**
```
Pods in ImagePullBackOff status
ErrImageNeverPull error
Cannot find Docker image
```

**Solutions:**

**1. Verify images exist and have correct names:**
```bash
docker images

# Should show:
#mern_notes_app-backend   latest
# mern_notes_app-frontend   latest
```

**2. For Minikube, use Minikube's Docker:**
```bash
eval $(minikube docker-env)

# Then build images
docker build -t mern_notes_app-backend:latest ./server
```

**3. Update k8s YAML with correct image names:**
```yaml
image: mern_notes_app-backend:latest
imagePullPolicy: Never  # Use local image
```

**4. Restart deployment:**
```bash
kubectl delete deployment notes-backend
kubectl apply -f k8s/backend-deployment.yaml
```

---

### Problem: Pods crash (CrashLoopBackOff)

**Symptoms:**
```
Pod keeps restarting
Stuck in CrashLoopBackOff
Container exits immediately
```

**Solutions:**

**1. Check pod logs:**
```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```

**2. Common causes:**
- Database connection failed
- Port already in use
- Missing environment variable
- Application error

**3. Fix and restart:**
```bash
# Update pod env variables or image
kubectl delete pod <pod-name>

# Pod will auto-restart with new config
```

---

### Problem: Service cannot reach backend

**Symptoms:**
```
Frontend cannot connect to backend
"Cannot resolve notes-backend"
Connection refused
```

**Solutions:**

**1. Verify services exist:**
```bash
kubectl get services

# Should show:
# notes-frontend
# notes-backend
# notes-mongo
```

**2. Test connectivity:**
```bash
# From one pod, test connection to another
kubectl exec -it <pod> -- bash
curl http://notes-backend:3000

# Exit
exit
```

**3. Check DNS:**
```bash
# Test DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup notes-backend
```

**4. Check network policies:**
```bash
kubectl get networkpolicies

# If strict policies, may need to adjust
```

---

### Problem: HPA not scaling

**Symptoms:**
```
Pods don't scale up under load
HPA shows "Unknown" metrics
Desired replicas not changing
```

**Solutions:**

**1. Check HPA status:**
```bash
kubectl get hpa
kubectl describe hpa notes-backend-hpa
```

**2. Verify metrics are available:**
```bash
kubectl top pods
kubectl top nodes

# If "Metrics unavailable", metrics-server not working
```

**3. Install metrics-server:**
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Wait for deployment
kubectl wait --for=condition=ready pod -l k8s-app=metrics-server -n kube-system --timeout=120s
```

**4. Verify resource requests are set:**
```bash
# Pods must have resource requests for HPA to work
kubectl describe pod <pod-name> | grep -A 5 "Requests"

# Should show CPU and memory requests
```

---

### Problem: PersistentVolume not mounting

**Symptoms:**
```
PVC pending
Pod cannot mount volume
Data persists not working
```

**Solutions:**

**1. Check PV and PVC status:**
```bash
kubectl get pv,pvc

# Status should be "Bound"
```

**2. Create required directory:**
```bash
# For hostPath volumes, ensure directory exists
mkdir -p /data/mongo
chmod 777 /data/mongo
```

**3. Check pod mount:**
```bash
kubectl describe pod <mongodb-pod> | grep -A 5 "Mounts"

# Should show volume mounted
```

**4. Verify data actually stored:**
```bash
kubectl exec -it <mongodb-pod> -- ls -la /data/db
```

---

## Network Issues

### Problem: Frontend cannot reach backend

**Symptoms:**
```
API calls fail
Connected to http://localhost:3000 from frontend
Browser shows CORS errors
```

**Solutions:**

**1. Frontend URL configuration:**
```bash
# Development:
Backend should be http://localhost:3000

# Docker Compose:
Backend should be http://backend:3000

# Kubernetes (internal):
Backend should be http://notes-backend:3000
```

**2. Check CORS configuration:**
```javascript
// server.js should have CORS enabled
app.use(cors({ origin: "*" }))
```

**3. Test endpoint directly:**
```bash
curl http://localhost:3000/allNotes

# If works, check frontend code
```

---

### Problem: Cannot access frontend from browser

**Symptoms:**
```
localhost:80 refuses connection
Frontend service shows Pending external IP
Cannot reach http://localhost
```

**Solutions:**

**1. For Kubernetes:**
```bash
# Port forward to access
kubectl port-forward svc/notes-frontend 8080:80

# Access at http://localhost:8080
```

**2. For Docker Desktop:**
```bash
# Kubernetes should automatically expose
# Try http://localhost (port 80)
```

**3. For Minikube:**
```bash
# Get service URL
minikube service notes-frontend

# Or port forward
kubectl port-forward svc/notes-frontend 8000:80
```

---

## Performance Issues

### Problem: Slow API responses

**Solutions:**

**1. Check backend resource usage:**
```bash
# Docker
docker stats backend

# Kubernetes
kubectl top pods notes-backend-*
```

**2. Check database performance:**
```bash
mongosh
use notes

# Check slowest operations
db.notes.find().explain("executionStats")

# Create index if needed
db.notes.createIndex({ title: 1 })
```

**3. Enable caching:**
```javascript
// Add response caching middleware
const cache = require('memory-cache');

app.get('/allNotes', (req, res) => {
  const cacheKey = 'notes_all';
  const cachedData = cache.get(cacheKey);
  
  if (cachedData) return res.json(cachedData);
  
  // Fetch data...
  cache.put(cacheKey, data, 60000); // Cache 60 seconds
});
```

---

### Problem: High memory usage

**Solutions:**

**1. Check memory leaks:**
```bash
# Monitor memory growth over time
docker stats --no-stream
```

**2. Limit memory:**
```yaml
# kubernetes deployment
resources:
  limits:
    memory: "512Mi"
    cpu: "500m"
```

**3. Optimize queries:**
```javascript
// Use .lean() for read-only queries
Note.find().lean()

// Use projection to select fields
Note.find().select('title content')
```

---

### Problem: High CPU usage

**Solutions:**

**1. Check what's consuming CPU:**
```bash
# In container
docker stats

# In Kubernetes
kubectl top pods nodes-backend-*
```

**2. Optimize calculations:**
```javascript
// Avoid heavy operations in request handler
// Move to background job if needed

// Don't loop unnecessary queries
// Use aggregation pipelines instead
```

**3. Add caching:**
```javascript
// Cache frequently accessed data
// Use Redis for distributed caching
```

---

## Getting Help

### When stuck, check:

1. **Server logs:**
   ```bash
   npm run dev
   # Watch for error messages
   ```

2. **Docker logs:**
   ```bash
   docker-compose logs -f <service>
   ```

3. **Kubernetes logs:**
   ```bash
   kubectl logs <pod-name>
   kubectl describe pod <pod-name>
   ```

4. **Browser console:**
   ```
   F12 → Console tab
   Look for JavaScript errors
   ```

5. **Network requests:**
   ```
   F12 → Network tab
   Check request/response status
   ```

6. **Database state:**
   ```bash
   mongosh
   use notes
   db.notes.find()
   ```

---

## Quick Checklist

- [ ] Backend running on port 3000?
- [ ] MongoDB running and accessible?
- [ ] Frontend can reach backend URL?
- [ ] CORS enabled on backend?
- [ ] MongoDB data directory exists with write permissions?
- [ ] Docker images built with correct names?
- [ ] Docker containers can communicate?
- [ ] Kubernetes pods in Running state?
- [ ] Services created and exposed?
- [ ] HPA has metrics available?

---

**Last Updated:** March 19, 2026

For additional help, check the other documentation files or open an issue.
