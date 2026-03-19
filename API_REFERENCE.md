# API Reference & Development Guide

## Base URL

- **Local Development**: `http://localhost:3000`
- **Docker Compose**: `http://localhost:3000`
- **Kubernetes**: `http://notes-backend:3000` (internal) or via port-forward

## Content-Type

All requests and responses use `application/json`

---

## Note Endpoints

### 1. Get All Notes

Retrieve all notes from the database.

**Endpoint:**
```
GET /allNotes
```

**Request Example:**
```bash
curl http://localhost:3000/allNotes
```

**Response (200 OK):**
```json
[
  {
    "_id": "507f1f77bcf86cd799439011",
    "title": "First Note",
    "content": "This is the content of the first note",
    "createdAt": "2026-03-19T10:30:00.000Z",
    "updatedAt": "2026-03-19T10:30:00.000Z"
  },
  {
    "_id": "507f1f77bcf86cd799439012",
    "title": "Second Note",
    "content": "This is the content of the second note",
    "createdAt": "2026-03-19T11:15:00.000Z",
    "updatedAt": "2026-03-19T11:15:00.000Z"
  }
]
```

**Response (Empty):**
```json
[]
```

**Status Codes:**
- `200 OK` - Successfully retrieved notes
- `500 Internal Server Error` - Server error

---

### 2. Create Note

Create a new note.

**Endpoint:**
```
POST /addNote
```

**Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "My New Note",
  "content": "The content of my note goes here"
}
```

**Request Example:**
```bash
curl -X POST http://localhost:3000/addNote \
  -H "Content-Type: application/json" \
  -d '{"title":"My Note","content":"Note content"}'
```

**Response (201 Created):**
```json
{
  "_id": "507f1f77bcf86cd799439013",
  "title": "My New Note",
  "content": "The content of my note goes here",
  "createdAt": "2026-03-19T12:00:00.000Z",
  "updatedAt": "2026-03-19T12:00:00.000Z"
}
```

**Validation Rules:**
- `title`: Required, string
- `content`: Required, string

**Status Codes:**
- `201 Created` - Note successfully created
- `400 Bad Request` - Missing or invalid fields
- `500 Internal Server Error` - Server error

---

### 3. Get Note Details

Retrieve a specific note by ID.

**Endpoint:**
```
GET /noteDetails/:id
```

**URL Parameters:**
- `id` (required): MongoDB ObjectId of the note

**Request Example:**
```bash
curl http://localhost:3000/noteDetails/507f1f77bcf86cd799439011
```

**Response (200 OK):**
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "title": "First Note",
  "content": "Detailed content of the first note with more information",
  "createdAt": "2026-03-19T10:30:00.000Z",
  "updatedAt": "2026-03-19T10:45:00.000Z"
}
```

**Status Codes:**
- `200 OK` - Note found and returned
- `404 Not Found` - Note with given ID not found
- `500 Internal Server Error` - Server error

---

### 4. Update Note

Update an existing note.

**Endpoint:**
```
PATCH /updateNote/:id
```

**URL Parameters:**
- `id` (required): MongoDB ObjectId of the note

**Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "Updated Title",
  "content": "Updated content here"
}
```

**Request Example:**
```bash
curl -X PATCH http://localhost:3000/updateNote/507f1f77bcf86cd799439011 \
  -H "Content-Type: application/json" \
  -d '{"title":"Updated","content":"New content"}'
```

**Response (200 OK):**
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "title": "Updated Title",
  "content": "Updated content here",
  "createdAt": "2026-03-19T10:30:00.000Z",
  "updatedAt": "2026-03-19T13:00:00.000Z"
}
```

**Partial Updates:**
Only include fields you want to update:
```json
{ "title": "New Title" }
```

**Status Codes:**
- `200 OK` - Note successfully updated
- `400 Bad Request` - Invalid ID format
- `404 Not Found` - Note not found
- `500 Internal Server Error` - Server error

---

### 5. Delete Note

Delete a note.

**Endpoint:**
```
DELETE /deleteNote/:id
```

**URL Parameters:**
- `id` (required): MongoDB ObjectId of the note

**Request Example:**
```bash
curl -X DELETE http://localhost:3000/deleteNote/507f1f77bcf86cd799439011
```

**Response (200 OK):**
```json
{
  "message": "Note deleted successfully"
}
```

**Status Codes:**
- `200 OK` - Note successfully deleted
- `404 Not Found` - Note not found
- `500 Internal Server Error` - Server error

---

## Feedback Endpoints

### 1. Submit Feedback

Submit user feedback.

**Endpoint:**
```
POST /submitFeedback
```

**Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "message": "Great application! I have a suggestion..."
}
```

**Request Example:**
```bash
curl -X POST http://localhost:3000/submitFeedback \
  -H "Content-Type: application/json" \
  -d '{
    "name":"John Doe",
    "email":"john@example.com",
    "message":"Feedback message"
  }'
```

**Response (201 Created):**
```json
{
  "_id": "507f1f77bcf86cd799439014",
  "name": "John Doe",
  "email": "john@example.com",
  "message": "Great application! I have a suggestion...",
  "createdAt": "2026-03-19T14:00:00.000Z"
}
```

**Validation Rules:**
- `name`: Required, string
- `email`: Required, email format
- `message`: Required, string

**Status Codes:**
- `201 Created` - Feedback submitted successfully
- `400 Bad Request` - Missing or invalid fields
- `500 Internal Server Error` - Server error

---

## Error Handling

### Standard Error Response

```json
{
  "error": "Error message describing what went wrong"
}
```

### Common Errors

**Invalid MongoDB ObjectId:**
```json
{
  "error": "Invalid note ID format"
}
```

**Note Not Found:**
```json
{
  "error": "Note not found"
}
```

**Validation Error:**
```json
{
  "error": "Title and content are required"
}
```

**Server Error:**
```json
{
  "error": "Internal server error"
}
```

---

## Request Example with JavaScript (Fetch API)

```javascript
// Get all notes
fetch('http://localhost:3000/allNotes')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Create note
fetch('http://localhost:3000/addNote', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    title: 'New Note',
    content: 'Note content'
  })
})
.then(res => res.json())
.then(data => console.log(data))
.catch(err => console.error(err));

// Update note
fetch('http://localhost:3000/updateNote/507f1f77bcf86cd799439011', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    title: 'Updated Title'
  })
})
.then(res => res.json())
.then(data => console.log(data))
.catch(err => console.error(err));

// Delete note
fetch('http://localhost:3000/deleteNote/507f1f77bcf86cd799439011', {
  method: 'DELETE'
})
.then(res => res.json())
.then(data => console.log(data))
.catch(err => console.error(err));
```

---

## Request Example with Axios

```javascript
import axios from 'axios';

const API = 'http://localhost:3000';

// Get all notes
axios.get(`${API}/allNotes`)
  .then(res => console.log(res.data))
  .catch(err => console.error(err));

// Create note
axios.post(`${API}/addNote`, {
  title: 'New Note',
  content: 'Note content'
})
.then(res => console.log(res.data))
.catch(err => console.error(err));

// Update note
axios.patch(`${API}/updateNote/507f1f77bcf86cd799439011`, {
  title: 'Updated Title'
})
.then(res => console.log(res.data))
.catch(err => console.error(err));

// Delete note
axios.delete(`${API}/deleteNote/507f1f77bcf86cd799439011`)
  .then(res => console.log(res.data))
  .catch(err => console.error(err));
```

---

## Request Example with cURL

```bash
# Get all notes
curl http://localhost:3000/allNotes

# Create note
curl -X POST http://localhost:3000/addNote \
  -H "Content-Type: application/json" \
  -d '{"title":"Note","content":"Content"}'

# Get note details
curl http://localhost:3000/noteDetails/ID_HERE

# Update note
curl -X PATCH http://localhost:3000/updateNote/ID_HERE \
  -H "Content-Type: application/json" \
  -d '{"title":"Updated"}'

# Delete note
curl -X DELETE http://localhost:3000/deleteNote/ID_HERE
```

---

## Testing with Postman

### Setup

1. **Create Collection**: "MERN Notes App"
2. **Set Base URL**: `http://localhost:3000`

### Requests

All requests in one collection:

| Method | Endpoint | Body |
|--------|----------|------|
| GET | /allNotes | - |
| POST | /addNote | `{title, content}` |
| GET | /noteDetails/:id | - |
| PATCH | /updateNote/:id | `{title?, content?}` |
| DELETE | /deleteNote/:id | - |
| POST | /submitFeedback | `{name, email, message}` |

---

## Performance Considerations

### Pagination (Future Enhancement)

```javascript
// Add offset and limit query parameters
GET /allNotes?offset=0&limit=10
GET /allNotes?page=1&pageSize=10
```

### Filtering (Future Enhancement)

```javascript
// Filter by title
GET /allNotes?title=my

// Filter by date range
GET /allNotes?from=2026-03-01&to=2026-03-31
```

### Sorting (Future Enhancement)

```javascript
// Sort by creation date
GET /allNotes?sort=createdAt&order=desc
```

---

## Database Schema

### Note Schema

```javascript
{
  _id: ObjectId,
  title: String (required),
  content: String (required),
  createdAt: Date (auto-generated),
  updatedAt: Date (auto-updated)
}
```

### Message Schema

```javascript
{
  _id: ObjectId,
  name: String (required),
  email: String (required),
  message: String (required),
  createdAt: Date (auto-generated)
}
```

---

## Status Code Reference

| Code | Meaning | Common Cause |
|------|---------|--------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Invalid input or missing fields |
| 404 | Not Found | Resource not found |
| 500 | Server Error | Internal server error |

---

## CORS Configuration

Current configuration allows all origins:
```javascript
app.use(cors({ origin: "*" }))
```

For production, restrict to specific origins:
```javascript
app.use(cors({
  origin: "https://yourdomain.com",
  credentials: true,
  methods: ['GET', 'POST', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type']
}))
```

---

## Rate Limiting (Future Enhancement)

Consider adding rate limiting for production:
```javascript
const rateLimit = require("express-rate-limit");

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

app.use(limiter);
```

---

## Authentication (Future Enhancement)

Add JWT authentication:
```javascript
const jwt = require("jsonwebtoken");

// Create token
const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET);

// Verify token (middleware)
const auth = (req, res, next) => {
  const token = req.header("Authorization")?.split(" ")[1];
  if (!token) return res.status(401).json({ error: "No token" });
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = decoded.userId;
    next();
  } catch (err) {
    res.status(401).json({ error: "Invalid token" });
  }
};
```

---

## Analytics (Future Enhancement)

Track API usage:
```javascript
// Track request count
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
  next();
});
```

---

**API Documentation Last Updated:** March 19, 2026
