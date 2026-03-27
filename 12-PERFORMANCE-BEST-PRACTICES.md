# Part 12 - Performance & Best Practices

## 1. Async/Await Best Practices

### Always await or catch errors
```javascript
// ❌ BAD - Unhandled promise rejection
async function bad() {
    await db.collection('homes').insertOne({});
}

// ✅ GOOD - Handle errors
async function good() {
    try {
        await db.collection('homes').insertOne({});
    } catch (err) {
        console.error(err);
    }
}
```

### Parallel vs Sequential
```javascript
// ❌ SLOW - Sequential (waits for each)
const user = await getUser(id);
const posts = await getPosts(id);
const friends = await getFriends(id);

// ✅ FAST - Parallel (all start at once)
const [user, posts, friends] = await Promise.all([
    getUser(id),
    getPosts(id),
    getFriends(id)
]);
```

---

## 2. Database Best Practices

### Connection Pooling
```javascript
// MongoDB driver handles this, but don't create new connection each request
const getDb = () => {
    if (!_db) throw new Error('DB not initialized');
    return _db;  // Reuse same connection
};
```

### Indexing
```javascript
// Create indexes for frequently queried fields
db.collection('homes').createIndex({ location: 1 });
db.collection('homes').createIndex({ price: 1 });
db.collection('homes').createIndex({ _id: 1 });  // Primary key is auto-indexed
```

### Projection (only get needed fields)
```javascript
// ❌ Gets all fields
await db.collection('homes').find().toArray();

// ✅ Gets only needed fields
await db.collection('homes').find({}, { projection: { name: 1, price: 1 } }).toArray();
```

---

## 3. Express Best Practices

### Don't use synchronous functions
```javascript
// ❌ BAD - blocks event loop
const fs = require('fs');
const data = fs.readFileSync('file.txt');

// ✅ GOOD - non-blocking
const fs = require('fs').promises;
const data = await fs.readFile('file.txt');
```

### Use compression
```javascript
const compression = require('compression');
app.use(compression());  // Compress all responses
```

### Security headers
```javascript
const helmet = require('helmet');
app.use(helmet());
```

---

## 4. Caching

### Simple in-memory cache
```javascript
const cache = {};
const CACHE_DURATION = 5 * 60 * 1000;  // 5 minutes

async function getHomes() {
    const cached = cache['homes'];
    if (cached && Date.now() - cached.timestamp < CACHE_DURATION) {
        return cached.data;
    }
    
    const homes = await Home.fetchAll();
    cache['homes'] = { data: homes, timestamp: Date.now() };
    return homes;
}
```

---

## 5. Pagination

```javascript
// ✅ CORRECT pagination
app.get('/api/homes', async (req, res) => {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const skip = (page - 1) * limit;
    
    const homes = await db.collection('homes')
        .find()
        .skip(skip)
        .limit(limit)
        .toArray();
    
    const total = await db.collection('homes').countDocuments();
    
    res.json({
        success: true,
        data: homes,
        page,
        totalPages: Math.ceil(total / limit)
    });
});
```

---

## 6. Environment Variables

```bash
# .env file - NEVER commit this!
PORT=3000
DB_URL=mongodb+srv://...
NODE_ENV=production
```

```javascript
require('dotenv').config();

const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || 'development';
```

---

## 7. Project Structure Best Practice

```
project/
├── app.js              # Entry point
├── router/             # Route definitions
│   ├── userRouter.js
│   └── adminRouter.js
├── controllers/        # Business logic
│   ├── userController.js
│   └── adminController.js
├── models/            # Database operations
│   ├── User.js
│   └── Home.js
├── views/             # EJS templates
│   ├── home.ejs
│   └── user/
├── public/            # Static files
│   ├── css/
│   ├── js/
│   └── images/
├── utils/             # Helper functions
│   ├── database.js
│   └── path.js
├── .env              # Environment variables
├── .gitignore        # Don't commit node_modules, .env
└── package.json
```

---

## 8. .gitignore

```
node_modules/
.env
.DS_Store
*.log
npm-debug.log*
```

---

## 9. Logging Best Practices

```javascript
// Use morgan for HTTP logging
const morgan = require('morgan');
app.use(morgan('combined'));  // Apache format

// Or custom logger
const logger = {
    info: (msg) => console.log(`[INFO] ${new Date()} ${msg}`),
    error: (msg) => console.error(`[ERROR] ${new Date()} ${msg}`)
};

app.use((req, res, next) => {
    logger.info(`${req.method} ${req.url}`);
    next();
});
```

---

## 10. General Best Practices

### Use const over let
```javascript
const name = 'Sachin';  // ✅ Can't be reassigned
let count = 0;  // Only use let when reassigning
```

### Early returns
```javascript
// ❌ Nested conditionals
if (user) {
    if (user.isAdmin) {
        if (home) {
            // do something
        }
    }
}

// ✅ Early return
if (!user || !user.isAdmin) return res.status(403).send('Forbidden');
if (!home) return res.status(404).send('Not found');
// do something
```

### Meaningful variable names
```javascript
// ❌ Bad
const x = users.filter(u => u.a > 18);

// ✅ Good
const adults = users.filter(user => user.age > 18);
```

---

## Interview Questions

### Q: How do you improve Node.js performance?
Use async/await, parallel queries with Promise.all, caching, pagination, compression middleware, security headers, database indexing, and connection pooling.

### Q: What is the event loop in Node.js?
The event loop allows Node.js to perform non-blocking I/O. Even though JavaScript is single-threaded, async operations are offloaded and callbacks processed when complete.

### Q: How do you prevent blocking the event loop?
Never use synchronous file operations (fs.readFileSync), keep heavy computation off the main thread, use worker threads for CPU-intensive tasks.
