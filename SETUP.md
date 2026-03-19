# Local Development Setup Guide

## Prerequisites

Before setting up the project, ensure you have:

- **Node.js 18+** - Download from [nodejs.org](https://nodejs.org/)
- **MongoDB 5+** - Download from [mongodb.com](https://www.mongodb.com/try/download/community)
- **Git** - Download from [git-scm.com](https://git-scm.com/)
- **Code Editor** - VS Code recommended
- **Terminal/Command Line** - PowerShell, bash, or zsh

### Verify Installation

```bash
# Check Node.js version
node --version        # Should be v18.0.0 or higher

# Check npm version
npm --version        # Should be v9.0.0 or higher

# Check MongoDB version (if installed locally)
mongod --version     # Should be MongoDB v5.0 or higher

# Check Git version
git --version        # Should be git version 2.x or higher
```

---

## Backend Setup

### Step 1: Install Dependencies

Navigate to the server directory and install Node packages:

```bash
# From project root
cd server

# Install dependencies
npm install

# Verify installation (should see node_modules folder)
ls node_modules
```

### Step 2: Configure Environment Variables

Create `.env` file in the `server/` directory:

```env
# Database Connection String
# Local MongoDB:
dbURL=mongodb://localhost:27017/notes

# Server Port
PORT=3000

# Environment
NODE_ENV=development
```

**Explanation:**
- `dbURL`: MongoDB connection string (local or cloud)
- `PORT`: Port where Node.js server listens
- `NODE_ENV`: Development mode enables hot reload with nodemon

### Step 3: Start MongoDB

**Option A: Using Local MongoDB**

```bash
# Windows
mongod

# macOS/Linux
brew services start mongodb-community

# Verify MongoDB is running
mongo --version
```

**Option B: Using Docker**

```bash
docker run -d -p 27017:27017 --name notes-mongo mongo:6
```

**Option C: Using MongoDB Atlas (Cloud)**

1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Create a free cluster
3. Get connection string
4. Update `.env`:
```env
dbURL=mongodb+srv://username:password@cluster.mongodb.net/notes
```

### Step 4: Start Backend Server

```bash
# From server directory
npm run dev

# Expected output:
# Connection to the Database was established!
# Server running on port 3000
```

**Test the API:**
```bash
# Open new terminal
curl http://localhost:3000/allNotes

# Should return empty array or existing notes:
# []
```

---

## Frontend Setup

### Step 1: Install Dependencies

Navigate to the frontend directory:

```bash
# From project root
cd frontend

# Install dependencies
npm install

# Verify
ls node_modules
```

### Step 2: Configure Vite

The `vite.config.js` is pre-configured. No changes needed unless:

```javascript
// frontend/vite.config.js (default configuration)
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
```

### Step 3: Start Development Server

```bash
# From frontend directory
npm run dev

# Expected output:
# VITE v4.4.5  ready in xxx ms
# ➜  Local:   http://localhost:5173/
# ➜  press h to show help
```

### Step 4: Access Application

Open browser and navigate to:
```
http://localhost:5173
```

---

## Connecting Frontend to Backend

### During Development

The frontend needs to know the backend URL. Update in `frontend/src/` files:

**Check current configuration in components:**
```javascript
// Look for axios calls like:
axios.get('http://localhost:3000/allNotes')
```

**Verify it points to:**
```
Backend URL: http://localhost:3000
```

### Environment Variables (Optional)

Create `frontend/.env` for easier configuration:

```env
VITE_API_URL=http://localhost:3000
```

Then update components:
```javascript
const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:3000';
axios.get(`${API_URL}/allNotes`)
```

---

## Common Development Tasks

### Adding a Node Package

```bash
# Frontend
cd frontend
npm install <package-name>

# Backend
cd server
npm install <package-name>
```

### Debugging

**Backend Debugging:**
```bash
# Use VS Code debugger
# 1. Add breakpoint in code
# 2. Run: npm run dev
# 3. Attach debugger in VS Code

# Or use console logs
console.log('Debug info:', variable)
```

**Frontend Debugging:**
```bash
# Use browser DevTools (F12)
# 1. Open DevTools in browser
# 2. Console tab for logs
# 3. Network tab for API calls
# 4. Elements tab for HTML/CSS
```

**React DevTools Extension:**
- Chrome: [React Developer Tools](https://chrome.google.com/webstore)
- Firefox: [React Developer Tools](https://addons.mozilla.org/firefox/)

### Running Tests

```bash
# Backend (if configured)
npm test

# Frontend (if configured)
npm test
```

### Building for Production

**Frontend Build:**
```bash
# From frontend directory
npm run build

# Output in dist/ folder
# Can be served with any web server
```

**Backend:**
```bash
# Just run with NODE_ENV=production
NODE_ENV=production PORT=3000 npm start
```

---

## Project-Specific Setup

### Database Initialization

The backend automatically creates collections when first run.

**To pre-populate with sample data:**

```bash
# Connect to MongoDB
mongosh

# Switch to notes database
use notes

# Insert sample note
db.notes.insertOne({
  title: "Sample Note",
  content: "This is a sample note",
  createdAt: new Date(),
  updatedAt: new Date()
})

# Verify
db.notes.find()

# Exit
exit
```

### Resetting Development Database

```bash
# Delete all data
mongosh
use notes
db.notes.deleteMany({})
exit

# Or delete entire database
db.dropDatabase()
```

---

## Troubleshooting Development Setup

### Problem: Port 3000 already in use

```bash
# Windows - Find and kill process
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# macOS/Linux
lsof -i :3000
kill -9 <PID>

# Or use different port
PORT=3001 npm run dev
```

### Problem: Port 5173 already in use

```bash
# Let Vite use next available port
npm run dev -- --port 5174
```

### Problem: Cannot connect to MongoDB

```bash
# Check if MongoDB is running
# Windows
tasklist | findstr mongod

# macOS/Linux
ps aux | grep mongod

# If not running, start it
mongod

# Test connection
mongo mongodb://localhost:27017/notes
```

### Problem: Dependencies installation fails

```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and package-lock
rm -rf node_modules package-lock.json

# Install fresh
npm install
```

### Problem: Vite dev server not refreshing

```bash
# Kill the process and restart
Ctrl+C

npm run dev

# If persists, clear .vite cache
rm -rf .vite
npm run dev
```

### Problem: API calls failing with CORS error

**Check Backend CORS Configuration:**
```javascript
// In server.js
app.use(cors({
  origin: "*",  // Allow all origins
}))
```

For production, restrict to specific origins:
```javascript
app.use(cors({
  origin: "https://your-frontend-domain.com",
  credentials: true
}))
```

---

## IDE Setup Recommendations

### VS Code Extensions

Install these for better development experience:

1. **ES7+ React/Redux/React-Native snippets**
   - `dsznajder.es7-react-js-snippets`

2. **MongoDB for VS Code**
   - `mongodb.mongodb-vscode`

3. **Prettier - Code formatter**
   - `esbenp.prettier-vscode`

4. **ESLint**
   - `dbaeumer.vscode-eslint`

5. **Postman**
   - For testing API endpoints

### VS Code Settings

Create `.vscode/settings.json`:
```json
{
  "editor.formatOnSave": true,
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

---

## Performance Tips

### Frontend Performance
- Use React DevTools Profiler to identify slow renders
- Implement code splitting with React.lazy()
- Optimize images and assets
- Use CSS-in-JS efficiently

### Backend Performance
- Index MongoDB collections for faster queries
- Implement caching strategies
- Use pagination for large datasets
- Monitor server logs and CPU usage

### General
- Keep dependencies updated: `npm outdated`
- Remove unused packages: `npm prune`
- Monitor bundle size: `npm run build` and check `dist/` size
- Use Network tab in DevTools to see response times

---

## Useful Commands Reference

```bash
# Backend Development
cd server
npm install              # Install dependencies
npm run dev             # Start with nodemon (hot reload)
npm start               # Start production server
npm test                # Run tests

# Frontend Development
cd frontend
npm install             # Install dependencies
npm run dev             # Start Vite dev server
npm run build           # Build for production
npm run preview         # Preview production build
npm run lint            # Run ESLint

# MongoDB
mongosh                 # Connect to MongoDB
use notes               # Switch to database
db.notes.find()         # View all notes
db.dropDatabase()       # Delete database

# General
git status              # Check git status
git add .               # Stage all changes
git commit -m "msg"     # Commit changes
npm cache clean --force # Clear npm cache
```

---

## Next Steps

1. **Understand the codebase:**
   - Read through `frontend/src/App.jsx`
   - Explore backend routes in `server/routes/routes.js`

2. **Run the application:**
   - Start backend: `cd server && npm run dev`
   - Start frontend: `cd frontend && npm run dev`
   - Open frontend in browser

3. **Test functionality:**
   - Create a new note
   - Edit existing note
   - Delete a note
   - Submit feedback

4. **Modify and extend:**
   - Add new routes in backend
   - Add new components in frontend
   - Test changes immediately

5. **Learn more:**
   - Read DOCUMENTATION.md for full project overview
   - Check HPA_DEPLOYMENT_SUMMARY.md for deployment info
   - Explore k8s/ folder for cloud deployment

---

## Getting Help

- **Node.js Issues**: Check [Node.js docs](https://nodejs.org/docs/)
- **MongoDB Issues**: Visit [MongoDB documentation](https://docs.mongodb.com/)
- **React Issues**: Check [React documentation](https://react.dev/)
- **Express Issues**: See [Express.js docs](https://expressjs.com/)

---

**Happy coding! 🚀**
