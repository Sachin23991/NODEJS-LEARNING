# Quick Reference Cheat Sheet

## Express Server Setup
```javascript
const express = require('express');
const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

app.set('view engine', 'ejs');
app.set('views', 'views');

app.listen(PORT, () => console.log(`Running on port ${PORT}`));
```

## Router Setup
```javascript
const router = express.Router();
router.get('/', controller.fn);
router.post('/path', controller.fn);
module.exports = router;

app.use('/prefix', router);
```

## Request Properties
```javascript
req.params.id        // URL parameters
req.query.q          // Query string
req.body             // Parsed body (needs middleware)
req.method           // GET, POST, PUT, DELETE
req.url              // Full path
req.headers          // HTTP headers
```

## Response Methods
```javascript
res.send('text')           // Send text/HTML
res.json({ key: value })  // Send JSON
res.render('view', data)  // Render EJS
res.redirect('/path')     // Redirect
res.sendFile('/path')    // Send file
res.status(404).send()   // Set status
```

## Middleware Order
```javascript
app.use(express.json());        // 1. Parse body
app.use(express.static('pub')); // 2. Static files
app.use((req, res, next) => {  // 3. Logger
    console.log(req.url);
    next();
});
app.use(router);                // 4. Routes
app.use((err, req, res) => {   // 5. Error handler
    res.status(500).send('Error');
});
```

## MongoDB CRUD
```javascript
// Connect
const { mongoConnect, getDb } = require('./utils/database');
mongoConnect(client => { app.listen(PORT); });

// Create
db.collection('homes').insertOne({ field: value });

// Read
db.collection('homes').find().toArray();
db.collection('homes').findOne({ _id: new ObjectId(id) });

// Update
db.collection('homes').updateOne(
    { _id: new ObjectId(id) },
    { $set: { field: newValue } }
);

// Delete
db.collection('homes').deleteOne({ _id: new ObjectId(id) });
```

## MongoDB Operators
```javascript
{ $gt: 100 }      // Greater than
{ $lt: 100 }      // Less than
{ $gte: 100 }     // Greater or equal
{ $lte: 100 }     // Less or equal
{ $in: [a, b] }   // In array
{ $set: {} }      // Update fields
{ $inc: { n: 1 } } // Increment
{ $push: { arr: x } } // Add to array
```

## HTTP Status Codes
```
200 OK
201 Created
301 Redirect
302 Found (temporary redirect)
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
500 Internal Server Error
```

## REST Routes
```
GET    /homes           -> getAll
GET    /homes/:id       -> getOne
POST   /homes           -> create
PUT    /homes/:id       -> update
DELETE /homes/:id       -> delete
```

## EJS Tags
```
<%= variable %>   Output (escaped)
<%- variable %>   Output (raw HTML)
<% code %>        Control flow (no output)
<%# comment %>    Comment
```

## Array Methods
```javascript
.map(x => x * 2)           // Transform
.filter(x => x > 0)         // Filter
.reduce((a, b) => a + b, 0) // Sum
.find(x => x.id === 1)     // Find first
.forEach(x => console.log(x)) // Loop
.some(x => x > 0)         // Any match
.every(x => x > 0)         // All match
```

## Node Modules
```javascript
require('http')           // HTTP server
require('fs')            // File system
require('path')          // Path utilities
require('querystring')   // URL query strings
```

## Path Utilities
```javascript
path.join(__dirname, 'folder', 'file')  // Join paths
path.dirname('/a/b/c.txt')              // Directory name
path.extname('file.txt')               // Extension
__dirname                              // Current file's directory
```

## Async/Await
```javascript
async function fn() {
    try {
        const data = await promise;
        return data;
    } catch (err) {
        console.error(err);
    }
}
```

## Common npm Packages
```
express           // Web framework
mongoose         // MongoDB ODM
mongodb          // MongoDB driver
body-parser      // Parse request bodies
ejs              // Template engine
cors             // Cross-origin requests
helmet           // Security headers
morgan           // HTTP logger
dotenv           // Environment variables
jsonwebtoken     // JWT auth
bcrypt           // Password hashing
```

## Environment Variables
```javascript
// .env file
PORT=3000
DB_URL=mongodb+srv://...
SECRET=mysecretkey

// code
require('dotenv').config();
const PORT = process.env.PORT;
const DB_URL = process.env.DB_URL;
```

## Error Handling
```javascript
// Sync
try {
    JSON.parse(invalid);
} catch (err) {
    console.error(err.message);
}

// Async
try {
    await db.collection('x').findOne();
} catch (err) {
    console.error(err.message);
}

// Throwing
throw new Error('Custom error');
```

## Mongoose Schema Types
```javascript
const schema = new mongoose.Schema({
    name: String,
    age: Number,
    email: { type: String, required: true },
    createdAt: { type: Date, default: Date.now },
    isActive: { type: Boolean, default: true }
});
```

## Session Setup
```javascript
const session = require('express-session');
app.use(session({
    secret: 'secret key',
    resave: false,
    saveUninitialized: false,
    cookie: { secure: false }
}));
```

## JWT Setup
```javascript
const jwt = require('jsonwebtoken');
const token = jwt.sign({ userId: 1 }, 'secret', { expiresIn: '1h' });
const payload = jwt.verify(token, 'secret');
```
