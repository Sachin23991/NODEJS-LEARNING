# Part 05 - Middleware Deep Dive
# From Your Code Files: airbnb/app.js

## 1. What is Middleware?

Middleware is a function that processes requests before they reach the route handler. Every request passes through middleware in order.

```
REQUEST → [Middleware 1] → [Middleware 2] → [Route Handler] → RESPONSE
              next()          next()
```

---

## 2. Your First Middleware (From expresslearning/app.js)

```javascript
// MIDDLEWARE PATTERN: req → res → next()

app.use((req, res, next) => {
    res.send('This is the starting page');
    console.log("This is the first middleware", req.url, req.method);
    next();  // Pass to next middleware/route
});
```

**Key Points:**
- `req` - request object
- `res` - response object
- `next()` - MUST call to pass to next middleware

---

## 3. Middleware Types

### A) Application-Level Middleware
Applied to entire app:

```javascript
// Your logger middleware (from airbnb/app.js)
app.use((req, res, next) => {
    console.log("first Middleware:", req.url, req.method);
    next();
});
```

### B) Router-Level Middleware
Applied to specific router:

```javascript
// In router file
router.get('/secret', authMiddleware, (req, res) => {
    // authMiddleware must pass (call next())
    // before this handler runs
});
```

### C) Built-in Middleware
```javascript
app.use(express.json());           // Parse JSON bodies
app.use(express.urlencoded({ extended: true }));  // Parse form data
app.use(express.static('public')); // Serve static files
```

### D) Error-Handling Middleware
```javascript
// MUST have 4 parameters (err, req, res, next)
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).send('Something broke!');
});
```

---

## 4. Order Matters - Your app.js Example

```javascript
// 1. Parse bodies FIRST (needed by routes)
app.use(bodyParser.urlencoded({ extended: false }));

// 2. Serve static files (CSS, images, JS)
app.use(express.static(path.join(__dirname, 'public')));

// 3. Logger middleware (see what comes in)
app.use((req, res, next) => {
    console.log("first Middleware:", req.url, req.method);
    next();
});

// 4. Mount routers (handle routes)
app.use('/host', hostRouter);
app.use('/', userRouter);

// 5. 404 Handler (catch unmatched)
app.use((req, res, next) => {
    res.status(404).sendFile(path.join(rootdir, 'views', '404.html'));
});
```

---

## 5. Middleware That Modifies Request

```javascript
// Add current time to every request
app.use((req, res, next) => {
    req.requestTime = new Date().toISOString();
    next();
});

// Now in route handler:
app.get('/', (req, res) => {
    res.send(`Page visited at: ${req.requestTime}`);
});
```

---

## 6. Real-World Middleware Examples

### Logger Middleware
```javascript
app.use((req, res, next) => {
    const start = Date.now();
    res.on('finish', () => {  // When response is sent
        const duration = Date.now() - start;
        console.log(`${req.method} ${req.url} - ${duration}ms`);
    });
    next();
});
```

### Authentication Middleware
```javascript
const isAuthenticated = (req, res, next) => {
    if (req.session && req.session.userId) {
        next();  // Allow access
    } else {
        res.redirect('/login');  // Deny access
    }
};

router.get('/dashboard', isAuthenticated, (req, res) => {
    // Only runs if isAuthenticated calls next()
    res.send('Welcome to your dashboard!');
});
```

### Error-Handling Middleware
```javascript
// MUST be last middleware (4 parameters!)
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({
        error: 'Something went wrong!',
        message: err.message
    });
});
```

---

## 7. Middleware with Parameters

```javascript
// Factory function that returns middleware
const authorize = (role) => {
    return (req, res, next) => {
        if (req.user && req.user.role === role) {
            next();
        } else {
            res.status(403).send('Forbidden');
        }
    };
};

// Usage:
router.get('/admin', authorize('admin'), (req, res) => {
    res.send('Admin area');
});

router.get('/editor', authorize('editor'), (req, res) => {
    res.send('Editor area');
});
```

---

## 8. Third-Party Middleware

```bash
npm install cors          # Enable Cross-Origin requests
npm install helmet       # Security headers
npm install morgan       # HTTP logger
npm install compression   # Compress responses
```

```javascript
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

app.use(cors());           // Enable CORS
app.use(helmet());         // Security headers
app.use(morgan('combined')); // HTTP logging
app.use(compression());    // Compress responses
```

---

## 9. Your airbnb Middleware Setup

```javascript
// Body parser (parse form data)
app.use(bodyParser.urlencoded({ extended: false }));

// Static files (CSS, JS, images)
app.use(express.static(path.join(__dirname, 'public')));

// Logger
app.use((req, res, next) => {
    console.log("first Middleware:", req.url, req.method);
    next();
});

// Routers
app.use('/host', hostRouter);
app.use('/', userRouter);

// 404 Handler
app.use((req, res, next) => {
    res.status(404).sendFile(path.join(rootdir, 'views', '404.html'));
});
```

---

## 10. Common Middleware Mistakes

### ❌ BAD: Forgetting to call next()
```javascript
// This will hang forever!
app.use((req, res) => {
    // Does something but never calls next()
    res.send('Done');
});
```

### ❌ BAD: next() after response sent
```javascript
// Don't call next() after res.send()
app.use((req, res, next) => {
    res.send('Hello');
    next();  // Unnecessary after sending response
});
```

### ❌ BAD: Middleware order wrong
```javascript
// WRONG: Routes before body parser
app.get('/', handler);              // Can't read req.body here!
app.use(express.json());           // Too late!

// CORRECT: Body parser first
app.use(express.json());           // Parse bodies first
app.get('/', handler);              // Now req.body works
```

---

## 11. Middleware Flow Diagram

```
Request
   │
   ▼
┌─────────────────────────────────────────────┐
│  Middleware 1: body-parser                  │
│  - Parses form data into req.body          │
│  - Calls next()                            │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  Middleware 2: express.static               │
│  - Serves static files if found              │
│  - Calls next() if not                      │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  Middleware 3: Logger                       │
│  - Logs request details                     │
│  - Calls next()                            │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  Router: /host/* or /*                      │
│  - Matches route                           │
│  - Calls controller                        │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  Controller                                 │
│  - Business logic                          │
│  - Calls model                             │
│  - Sends response                          │
└─────────────────────────────────────────────┘
                  │
                  ▼
Response
```

---

## Interview Questions

### Q: What is middleware and how does it work?
Middleware are functions that process requests between the incoming request and the route handler. They receive (req, res, next), can modify them, and must either send a response or call next() to pass control.

### Q: What is the difference between app.use() and app.get()?
app.use() is middleware that runs for ALL HTTP methods (or specified prefix). app.get() only matches GET requests with exact paths. app.use() doesn't require a path - it runs for every request if no path specified.

### Q: How do you handle errors in Express?
Use error-handling middleware with 4 parameters (err, req, res, next). It must be the LAST middleware registered. When you want to trigger it, call next(error) from any middleware or route.

### Q: Why does middleware order matter?
Express processes middleware in registration order. If body-parser runs AFTER routes, req.body will be undefined. Static file middleware must run before routes so CSS/JS files are served.

### Q: How do you create custom middleware with parameters?
Use a factory function that returns the middleware function:
```javascript
const authorize = (role) => (req, res, next) => {
    if (req.user.role === role) next();
    else res.status(403).send('Forbidden');
};
```
