# Part 13 - Interview Questions & Answers

## Node.js Questions

### Q1: What is Node.js?
Node.js is a JavaScript runtime built on Chrome's V8 engine that lets you run JavaScript on the server. It uses an event-driven, non-blocking I/O model that makes it efficient for real-time applications.

### Q2: How is Node.js different from browser JavaScript?
| Feature | Node.js | Browser |
|---------|---------|---------|
| DOM | No | Yes (window, document) |
| File System | Yes (fs module) | No (security) |
| Modules | require() | import/export |
| Global | global | window |
| Packages | npm | npm |

### Q3: What is the Event Loop?
The Event Loop allows Node.js to perform non-blocking I/O operations. Even though JavaScript is single-threaded, async operations are offloaded to the system kernel, and callbacks are processed when operations complete.

### Q4: What is the difference between synchronous and asynchronous code?
Synchronous code blocks execution until it completes. Asynchronous code doesn't block - it starts an operation and runs a callback when complete. Example:
```javascript
// Synchronous (blocks)
const data = fs.readFileSync('file.txt');

// Asynchronous (non-blocking)
fs.readFile('file.txt', (err, data) => { });
```

### Q5: What is the V8 engine?
V8 is Google's open-source JavaScript engine (used in Chrome and Node.js). It compiles JavaScript to machine code directly, making it very fast.

### Q6: What are streams?
Streams are objects that let you read/write data in chunks instead of loading everything at once. Types: Readable, Writable, Duplex, Transform.

### Q7: What is the difference between process.nextTick() and setImmediate()?
- process.nextTick() adds callback to the beginning of the next event loop iteration
- setImmediate() adds callback to the 'check' phase of the event loop

```javascript
process.nextTick(() => console.log('nextTick'));
setImmediate(() => console.log('setImmediate'));
// Output: nextTick, then setImmediate
```

### Q8: What is a worker thread?
Worker threads enable parallel execution of JavaScript code. Unlike the main thread which is event-driven, worker threads can perform CPU-intensive tasks without blocking the event loop.

---

## Express.js Questions

### Q9: What is Express.js?
Express is a minimal, flexible Node.js web application framework that provides features for web and mobile applications. It handles routing, middleware, and templating.

### Q10: What is the difference between app.get() and app.use()?
| app.get() | app.use() |
|-----------|-----------|
| Handles specific GET requests | Handles ANY request or prefix |
| Exact path match | Prefix match |
| Route handler | Middleware |
| Usually ends response | Usually calls next() |

### Q11: What is middleware?
Middleware functions have access to req, res, and next(). They can modify req/req, end the cycle, or call next() to pass to the next middleware.

### Q12: How does middleware order work?
Express processes middleware in the order they are defined. Body parser must come before routes. Static files before routes. 404 handler at the end.

```javascript
app.use(bodyParser);    // 1. Parse bodies FIRST
app.use(staticFiles);  // 2. Serve static files
app.use(router);       // 3. Handle routes
app.use(404);          // 4. Catch unmatched
```

### Q13: What is the difference between PUT and PATCH?
- PUT: Replaces the entire resource (all fields)
- PATCH: Updates only the fields you send

```javascript
// PUT - sends complete object, all fields replaced
{ name: 'New Name' }  // price, location gone!

// PATCH - only updates sent fields
{ name: 'New Name' }  // price, location unchanged
```

### Q14: What is the PRG pattern?
Post-Redirect-Get. After a POST request, redirect to a GET page. This prevents duplicate submissions when user refreshes.

```javascript
app.post('/form', (req, res) => {
    saveData(req.body);
    res.redirect('/success');  // PRG pattern
});
```

### Q15: How do you handle errors in Express?
Use error-handling middleware with 4 parameters:
```javascript
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).send('Something broke!');
});
```

### Q16: What is CORS and how do you enable it?
Cross-Origin Resource Sharing. Allows frontend from different origin to access your API.
```javascript
const cors = require('cors');
app.use(cors());
```

---

## MongoDB Questions

### Q17: What is MongoDB Atlas?
Fully managed cloud database service. Handles server provisioning, data replication, backups, security, and automatic scaling.

### Q18: MongoDB vs MySQL?
| MongoDB (NoSQL) | MySQL (SQL) |
|-----------------|-------------|
| Collections | Tables |
| Documents | Rows |
| Flexible schema | Fixed schema |
| No relationships | JOINs |
| _id primary key | AUTO_INCREMENT |

### Q19: What is a Document in MongoDB?
A record in a collection. Stored as BSON (binary JSON). Can have nested objects and arrays.

```javascript
{
    _id: ObjectId("..."),
    name: "Beach House",
    price: 150,
    amenities: ["wifi", "pool"],
    owner: { name: "Sachin", phone: "123" }
}
```

### Q20: What is an ObjectId?
MongoDB's default primary key type. 12-byte value containing: timestamp, machine ID, process ID, and counter.

```javascript
const { ObjectId } = require('mongodb');
const id = new ObjectId();  // Create new
const str = id.toString();   // Convert to string
ObjectId.isValid(id);        // Check validity
```

### Q21: What does upsert mean?
Update + Insert. If document exists, update it. If not, insert new document.

```javascript
db.collection('favourites').updateOne(
    { homeId: id },
    { $setOnInsert: { homeId: id } },
    { upsert: true }
);
```

### Q22: How do you prevent duplicate favourites?
Use $setOnInsert - only sets values if inserting a new document, ignores if document already exists.

---

## MVC Architecture Questions

### Q23: What is MVC?
Model-View-Controller. Separates code into:
- Model: Data operations (database)
- View: Presentation (EJS templates)
- Controller: Business logic (request handling)

### Q24: What is a Router in Express?
A mini Express app that handles a group of related routes. Used to organize routes by feature (user routes, admin routes, API routes).

### Q25: What is the difference between static and instance methods?
```javascript
class Home {
    constructor() { }     // Instance - called on object
    save() { }            // Instance - new Home().save()
    static fetchAll() { }  // Static - Home.fetchAll()
}
```

---

## Security Questions

### Q26: How do you secure an Express app?
```javascript
const helmet = require('helmet');
const cors = require('cors');

app.use(helmet());    // Security headers
app.use(cors());       // Enable CORS
app.use(helmet.hidePoweredBy());  // Hide X-Powered-By
```

### Q27: How do you prevent SQL/NoSQL injection?
- Use parameterized queries (MongoDB driver does this automatically)
- Validate and sanitize user input
- Don't concatenate user input into queries

### Q28: How do you handle authentication?
Use sessions or JWT tokens:
- Sessions: express-session with session cookies
- JWT: jsonwebtoken package for token-based auth

---

## Performance Questions

### Q29: How do you improve Node.js performance?
- Use async/await instead of callbacks
- Use connection pooling for databases
- Cache frequently accessed data
- Use compression middleware
- Serve static files with CDN

### Q30: What is clustering in Node.js?
Allows running multiple Node.js processes to handle load. Uses all CPU cores.

```javascript
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
} else {
    app.listen(PORT);
}
```

---

## Your Project-Specific Questions

### Q31: Explain your airbnb project architecture
See the architecture diagram in the main README.

### Q32: How does MongoDB connection work in your project?
1. app.js calls mongoConnect()
2. MongoClient.connect() connects to Atlas
3. Connection stored in _db global variable
4. getDb() returns _db for all queries
5. Server starts only after successful connection

### Q33: What is the difference between your userRouter and hostRouter?
- userRouter: User-facing pages (browse homes, view details, favourites)
- hostRouter: Host/admin pages (add home, edit home, delete home)

### Q34: How does form submission work in your project?
1. User fills form and submits (POST)
2. express.urlencoded() parses body into req.body
3. Controller extracts data via destructuring
4. Model saves to MongoDB
5. Controller redirects (PRG pattern)

### Q35: What is the flow when a user adds a home?
```
User visits /host/add-home (GET)
    -> hostRouter matches
    -> hostController.getAddHome
    -> renders add-home.ejs form

User submits form (POST /host/add-home)
    -> body-parser parses req.body
    -> hostController.postAddHome
    -> new Home(req.body).save()
    -> MongoDB insertOne()
    -> res.redirect('/host/home-list')
```
