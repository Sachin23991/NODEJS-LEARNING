# Part 02 - Node.js Core
# From Your Code Files: firstserver.js, ParsingRequest.js, module.js

## 1. What is Node.js?

Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine that lets you run JavaScript on the server.

**Key Features:**
- Event-driven (reacts to events, doesn't wait)
- Non-blocking I/O (can handle many connections)
- Single-threaded (one main thread handles requests)
- Cross-platform (Windows, Mac, Linux)

---

## 2. Your First Node.js Server (from firstserver.js)

```javascript
const http = require('http');  // Built-in Node.js module

const server = http.createServer((request, response) => {
    // request = incoming data from client
    // response = what we send back

    response.write("Hello Sachin");
    response.end();
});

const PORT = 3030;
server.listen(PORT, () => {
    console.log(`server is running at http://localhost:${PORT}`);
});
```

**Step by Step:**
1. Import `http` module (built-in)
2. `http.createServer()` creates a server object
3. Pass callback that runs on EVERY request
4. `server.listen(PORT)` starts listening

---

## 3. The Request Object (req)

From your ParsingRequest.js:
```javascript
console.log(request.url);      // URL path: "/", "/product", "/submit"
console.log(request.method);    // HTTP method: "GET", "POST", "PUT", "DELETE"
console.log(request.headers);   // All HTTP headers
```

**Content-Type Headers (from RequestAndResponse.js):**
```javascript
response.setHeader('Content-Type', 'text/html');     // For HTML pages
response.setHeader('Content-Type', 'application/json'); // For JSON data
response.setHeader('Content-Type', 'text/plain');    // For plain text
response.setHeader('Content-Type', 'text/css');     // For CSS files
response.setHeader('Content-Type', 'image/png');    // For images
```

---

## 4. Routing Without Express (from firstserver.js, ParsingRequest.js)

```javascript
const server = http.createServer((req, res) => {
    res.setHeader('Content-Type', 'text/html');

    if (req.url === '/') {
        res.end(`<h1>Home Page</h1>`);
    }
    else if (req.url === '/product') {
        res.end(`<h1>Product Page</h1>`);
    }
    else if (req.url === '/about') {
        res.end(`<h1>About Page</h1>`);
    }
    else {
        res.end(`<h1>404 - Page Not Found</h1>`);
    }
});
```

---

## 5. Handling POST Requests (from ParsingRequest.js)

**WHY POST?** Because forms send data via POST method.

### Step 1: Collect Chunks
Data from forms comes in SMALL PIECES called "chunks":
```javascript
const body = [];  // Array to store chunks

req.on('data', (chunk) => {
    console.log(chunk);  // Buffer object
    body.push(chunk);    // Add each chunk to array
});
```

### Step 2: Process When All Data Received
```javascript
req.on('end', () => {
    // Buffer.concat() merges all chunks into ONE Buffer
    const fullBody = Buffer.concat(body).toString();

    // Data comes as: "firstName=Sachin&lastName=Rao"
    // Convert to object using URLSearchParams
    const param = new URLSearchParams(fullBody);
    const jsonobject = Object.fromEntries(param);

    console.log(jsonobject);
});
```

### Step 3: Redirect After Processing
```javascript
res.statusCode = 302;  // 302 = Redirect
res.setHeader('Location', '/');  // Where to redirect
res.end();
```

---

## 6. Module System (from module.js)

### What is a Module?
A file is a module. Each file in Node.js is isolated - variables defined in one file don't exist in another.

### Your Calculator Example (from caLculator/add.js, requesthandler.js)

**add.js:**
```javascript
function add(a, b) {
    return a + b;
}
module.exports = add;  // Export the function
```

**requesthandler.js:**
```javascript
const add = require('./add');  // Import the function

// Now we can use it:
const result = add(10, 20);  // 30
```

### Multiple Exports
```javascript
// hostRouter.js
exports.hostRouter = hostRouter;
exports.houseregister = houseregister;

// app.js
const { hostRouter, houseregister } = require('./router/hostRouter');
```

---

## 7. Node.js Core Modules (Built-in)

### File System (fs)
```javascript
const fs = require('fs');

// Synchronous (blocks - don't use in production)
const data = fs.readFileSync('file.txt', 'utf8');
fs.writeFileSync('file.txt', 'Hello World');

// Asynchronous (non-blocking - PREFERRED)
fs.readFile('file.txt', 'utf8', (err, data) => {
    if (err) throw err;
    console.log(data);
});

// Async with Promises (modern way)
const fs = require('fs').promises;
const data = await fs.readFile('file.txt', 'utf8');
```

### Path
```javascript
const path = require('path');

// Join paths safely (handles / and \ automatically)
path.join(__dirname, 'views', 'home.html');
// On Windows: C:\project\views\home.html

// Get file extension
path.extname('image.png');  // ".png"

// Get directory name
path.dirname('/path/to/file.txt');  // "/path/to"
```

### Your Path Utility (from airbnb/utils/path.js):
```javascript
const path = require('path');
const rootdir = path.dirname(require.main.filename);
module.exports = rootdir;
// rootdir = absolute path to project root
```

---

## 8. The Event Loop (Key Concept!)

Node.js is SINGLE-THREADED but can handle many connections because of the event loop.

```
┌─────────────────────────────────────────────────────────────┐
│                      EVENT LOOP                             │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│  │ Call Stack  │───▶│  Web APIs    │───▶│  Callback   │   │
│  │ (Sync code) │    │ (setTimeout, │    │  Queue      │   │
│  │ executes    │    │  fs, etc.)   │    │             │   │
│  │ here first  │    │              │    │             │   │
│  └─────────────┘    └─────────────┘    └──────┬──────┘   │
│                                               │            │
│              Once Web API completes ──────────┘            │
│              it adds callback to queue                    │
└─────────────────────────────────────────────────────────────┘
```

### Example: Order of Execution
```javascript
console.log("1 - Synchronous");

setTimeout(() => console.log("2 - setTimeout"), 0);

Promise.resolve().then(() => console.log("3 - Promise"));

console.log("4 - Synchronous");

// Output: 1, 4, 3, 2
// (Promises run before setTimeout even with 0ms!)
```

---

## 9. Streams (Your Form Handling Uses This!)

Streams process data in CHUNKS instead of loading everything at once.

```javascript
// Your form handling (from ParsingRequest.js)
const body = [];
req.on('data', (chunk) => {
    body.push(chunk);  // Process chunk by chunk
});
req.on('end', () => {
    const fullBody = Buffer.concat(body).toString();
});
```

**Types of Streams:**
- Readable (req is a readable stream - data coming IN)
- Writable (res is a writable stream - data going OUT)
- Duplex (both readable and writable)

---

## 10. NPM (Node Package Manager)

### Your notes.txt:
> "nodemon ek asa package hai jiski wajah se server chalta rhata hai agar
> hu change kare file mein vo uske hisab se adapt kar leta hai"

**Nodemon** - Automatically restarts server when files change:
```bash
npm install nodemon
npx nodemon app.js
```

### Key NPM Commands
```bash
npm init                      # Create package.json
npm install express           # Install a package
npm install --save-dev nodemon # Dev dependency
npm install                    # Install all from package.json
npm list                      # Show installed packages
```

### Your package.json (from airbnb project):
```json
{
  "dependencies": {
    "express": "^5.2.1",
    "body-parser": "^1.20.2",
    "ejs": "^3.1.9",
    "mongodb": "^6.3.0"
  }
}
```

---

## 11. Buffers

A Buffer is a temporary storage area in memory for binary data.

```javascript
// Buffer.concat() - combine multiple buffers
const body = [Buffer1, Buffer2, Buffer3];
const fullBody = Buffer.concat(body).toString();
```

---

## 12. CommonJS vs ES Modules

### CommonJS (Node.js default - what you use)
```javascript
const express = require('express');
module.exports = router;
```

### ES Modules (Modern - needs "type": "module" in package.json)
```javascript
import express from 'express';
export default router;
```

---

## Important Interview Questions

### Q: What is the difference between Node.js and browser JavaScript?
| Feature | Node.js | Browser |
|--------|---------|---------|
| DOM | No DOM | Has window, document |
| File System | Yes (fs module) | No (security) |
| Modules | require() | import/export |
| Package Manager | npm | npm/pnpm |
| Global Object | global | window |

### Q: What is the Event Loop?
The event loop is what allows Node.js to perform non-blocking I/O operations. Even though JavaScript is single-threaded, Node.js offloads async operations to the OS/system and processes them via callbacks when complete.

### Q: What is the difference between synchronous and asynchronous code?
Synchronous code blocks execution until it completes. Asynchronous code doesn't block - it starts an operation and continues while waiting, then runs a callback when done.

### Q: Why should you avoid synchronous file operations in production?
`fs.readFileSync()` blocks the entire Node.js event loop while reading the file. No other requests can be processed during that time. Always use async versions: `fs.readFile()` or `fs.promises.readFile()`.
