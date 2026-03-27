# Part 03 - Express.js Basics
# From Your Code Files: expresslearning/app.js, airbnb/app.js

## 1. What is Express.js?

Express is a minimal web framework for Node.js. It simplifies:
- Routing (handling different URLs)
- Middleware (processing requests)
- Template engines (EJS, Pug)
- Error handling

**Without Express:** You manually check `req.url` and `req.method` for every route.
**With Express:** You use simple methods like `app.get()`, `app.post()`, etc.

---

## 2. Your First Express Server

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.send('This is the home page');
});

app.get('/product', (req, res) => {
    res.send('This is the product page');
});

const PORT = 3001;
app.listen(PORT, () => {
    console.log(`server is running at http://localhost:${PORT}`);
});
```

**Compare to raw Node.js:**
```javascript
// Raw Node.js (from your firstserver.js)
const server = http.createServer((req, res) => {
    if (req.url === '/' && req.method === 'GET') {
        res.end('This is the home page');
    }
    else if (req.url === '/product' && req.method === 'GET') {
        res.end('This is the product page');
    }
});
```

Express is MUCH cleaner!

---

## 3. HTTP Methods in Express

```javascript
app.get('/path', (req, res) => { });    // Read data
app.post('/path', (req, res) => { });   // Create data
app.put('/path', (req, res) => { });    // Replace data
app.patch('/path', (req, res) => { });  // Update data
app.delete('/path', (req, res) => { });  // Delete data
```

**From your code:**
```javascript
// userRouter.js
router.get('/', storeController.getHomes);           // GET /
router.get('/home-detail/:id', storeController.getHomeDetail);  // GET /home-detail/:id
router.post('/favourites', storeController.postAddToFavourites);  // POST /favourites
```

---

## 4. The Request Object (req)

```javascript
app.get('/users/:id', (req, res) => {
    req.params.id;           // Route parameters: /users/123 → "123"
    req.query.search;         // Query string: /search?q=hello → "hello"
    req.body;                 // Request body (needs body-parser!)
    req.headers;              // HTTP headers
    req.method;               // GET, POST, PUT, DELETE
    req.url;                  // Full URL path
});
```

### Route Parameters (Dynamic URLs)
```javascript
// From your userRouter.js:
router.get('/home-detail/:id', storeController.getHomeDetail);
// URL: /home-detail/abc123
// req.params.id = "abc123"

// Multiple parameters:
router.get('/users/:userId/posts/:postId', (req, res) => {
    req.params.userId;   // "123"
    req.params.postId;   // "456"
});
```

### Query String Parameters
```
URL: /search?q=javascript&sort=desc
req.query.q      // "javascript"
req.query.sort   // "desc"
```

---

## 5. The Response Object (res)

```javascript
res.send('Hello');              // Send text/HTML
res.json({ name: 'Sachin' });   // Send JSON
res.render('home', { data });   // Render EJS template
res.redirect('/login');          // Redirect
res.status(404).send('Not Found');  // Set status + send
res.sendFile('/path/to/file');  // Send a file
```

---

## 6. Body Parsing (From Your Code)

**Problem:** When you submit a form, data comes in the request body. Express doesn't parse it automatically.

**Solution:** Use middleware to parse it.

```javascript
// Your airbnb app.js:
const bodyParser = require('body-parser');
app.use(bodyParser.urlencoded({ extended: false }));

// OR in Express 4.16+:
app.use(express.urlencoded({ extended: true }));
app.use(express.json());  // For JSON bodies
```

**What this does:**
```
Form submission: firstName=Sachin&lastName=Rao
                    ↓
            body-parser middleware
                    ↓
req.body = { firstName: 'Sachin', lastName: 'Rao' }
```

**Your hostController receives it like:**
```javascript
exports.postAddHome = async (req, res) => {
    const { name, imageUrl, price, location } = req.body;
    // Destructure directly from req.body!
};
```

---

## 7. App.set() - Configuration

```javascript
app.set('view engine', 'ejs');      // Tell Express to use EJS
app.set('views', 'views');          // Where to find templates
app.set('port', 3000);              // Set port
```

**From your airbnb app.js:**
```javascript
app.set('view engine', 'ejs');
app.set('views', 'views');
```

---

## 8. Static Files

```javascript
app.use(express.static(path.join(__dirname, 'public')));
```

**What this does:**
```
Browser requests: /css/style.css
Express looks in: project_root/public/css/style.css
If found: sends the file
If not found: passes to next middleware
```

**Why needed?**
Without this, your CSS/JS files won't load!

---

## 9. Serving HTML Files

### Using res.sendFile()
```javascript
// From your airbnb/hostRouter.js:
const path = require('path');
const rootdir = require('../utils/path');

router.get('/add-home', (req, res) => {
    res.sendFile(path.join(rootdir, 'views', 'add-home.html'));
});
```

### Path Utilities
```javascript
const path = require('path');

// path.join() - joins paths correctly (handles / and \)
path.join(__dirname, 'views', 'home.html');
// On Windows: C:\project\views\home.html

// path.dirname() - get directory name
path.dirname('/path/to/file.txt');  // "/path/to"

// __dirname - directory of current file (Node.js global)
```

---

## 10. Order of Operations (Very Important!)

Express processes middleware in the ORDER you define them.

```javascript
// ✅ CORRECT ORDER:
app.use(express.urlencoded({ extended: true }));  // 1. Parse body FIRST
app.use((req, res, next) => {                      // 2. Logger
    console.log(req.url);
    next();
});
app.use('/', userRouter);                           // 3. Routes
app.use((req, res) => {                            // 4. 404 handler
    res.status(404).send('Not Found');
});
```

**From your firstexpressproject/app.js:**
```javascript
app.use(express.urlencoded({ extended: true }));  // Parse body
app.use((req, res, next) => {                     // Logger
    console.log(req.url, req.method);
    next();
});
app.use(contactdetail);                            // Router
app.listen(PORT);                                  // Start server
```

---

## 11. Response Methods Summary

| Method | Use Case |
|--------|----------|
| `res.send()` | Send text, HTML, or JSON |
| `res.json()` | Send JSON response |
| `res.render()` | Render EJS template |
| `res.redirect()` | Redirect to another URL |
| `res.sendFile()` | Send a file |
| `res.status()` | Set HTTP status code |
| `res.download()` | Force file download |

---

## 12. Status Codes You Must Know

| Code | Meaning |
|------|---------|
| 200 | OK (default) |
| 201 | Created |
| 301 | Moved Permanently (redirect) |
| 302 | Found (temporary redirect) |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |

---

## Interview Questions

### Q: How does Express routing work?
Express matches routes in the order they are defined. For each incoming request, it checks if any route matches the method and path. If matched, that handler runs. If no match, the 404 handler runs.

### Q: What is the difference between req.params and req.query?
`req.params` captures parts of the URL path (like `/users/:id`). `req.query` captures query string parameters (like `/search?q=term`).

### Q: Why do we need body-parser?
HTTP requests send data in the body, but Express doesn't parse it automatically. Body-parser (or express.json/urlencoded) parses the body and makes it available as req.body.
