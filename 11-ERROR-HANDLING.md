# Part 11 - Error Handling

## 1. Types of Errors

### Syntax Errors
```javascript
// Code that won't parse
if (true {  // Missing )
    console.log('error');
}
```

### Runtime Errors
```javascript
// Code runs but throws
JSON.parse('invalid json');  // SyntaxError
undefinedProperty.name;         // TypeError
```

### Logical Errors
```javascript
// Code runs but wrong output
const discount = price * 0.01;  // Should be 0.10 (10%)
```

---

## 2. Try-Catch

```javascript
try {
    // Code that might throw
    const data = JSON.parse(userInput);
    const result = data.nonexistent.property;
} catch (error) {
    console.log(error.message);  // What went wrong
    console.log(error.name);     // Error type
}
```

---

## 3. Async Error Handling

```javascript
// With Promises
Home.findById(id)
    .then(home => {
        if (!home) throw new Error('Home not found');
        return home;
    })
    .catch(err => console.log(err.message));

// With async/await
async function getHome(id) {
    try {
        const home = await Home.findById(id);
        if (!home) throw new Error('Home not found');
        return home;
    } catch (err) {
        console.log(err.message);
    }
}
```

---

## 4. Express Error Handling

### Error-Handling Middleware (MUST be last)
```javascript
// 4 parameters required!
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({
        success: false,
        error: err.message || 'Something went wrong'
    });
});
```

### Triggering Error Middleware
```javascript
// Call next() with error to pass to error handler
app.get('/home/:id', async (req, res, next) => {
    try {
        const home = await Home.findById(req.params.id);
        if (!home) {
            const error = new Error('Home not found');
            error.status = 404;
            throw error;
        }
        res.json({ success: true, data: home });
    } catch (err) {
        next(err);  // Pass to error middleware
    }
});
```

### Setting Status on Errors
```javascript
app.use((err, req, res, next) => {
    const status = err.status || 500;
    res.status(status).json({
        success: false,
        error: err.message
    });
});
```

---

## 5. Custom Error Classes

```javascript
class AppError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode;
        this.isOperational = true;  // Expected error
        Error.captureStackTrace(this, this.constructor);
    }
}

class NotFoundError extends AppError {
    constructor(message = 'Resource not found') {
        super(message, 404);
    }
}

class ValidationError extends AppError {
    constructor(message = 'Validation failed') {
        super(message, 400);
    }
}

// Usage
if (!home) {
    throw new NotFoundError('Home with this ID not found');
}
```

---

## 6. Async Error Wrapper

```javascript
// Wrap async route handlers to catch errors
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// Usage
app.get('/home/:id', asyncHandler(async (req, res) => {
    const home = await Home.findById(req.params.id);
    if (!home) throw new NotFoundError();
    res.json({ success: true, data: home });
}));
```

---

## 7. Error Handling in Your Project

```javascript
// In controller (current approach)
exports.getHomeDetail = async (req, res) => {
    try {
        const home = await Home.findById(req.params.id);
        if (!home) {
            return res.status(404).render('store/home-detail', {
                homes: null,
                pageTitle: 'Home Not Found'
            });
        }
        res.render('store/home-detail', {
            homes: home,
            pageTitle: 'Home Detail'
        });
    } catch (err) {
        console.log(err);
        res.status(500).send('Server Error');
    }
};
```

---

## 8. Common Error Patterns

### Not Found (404)
```javascript
app.use((req, res) => {
    res.status(404).json({ success: false, error: 'Route not found' });
});
```

### Validation Error (400)
```javascript
if (!name || !price) {
    return res.status(400).json({ success: false, error: 'Name and price required' });
}
```

### Unauthorized (401)
```javascript
if (!req.session.userId) {
    return res.status(401).json({ success: false, error: 'Please login' });
}
```

### Forbidden (403)
```javascript
if (home.ownerId !== req.session.userId) {
    return res.status(403).json({ success: false, error: 'Not authorized' });
}
```

---

## 9. Error Response Format

```javascript
// Always consistent error format
{
    success: false,
    error: 'Error message here',
    code: 'ERROR_CODE'  // Optional machine-readable code
}

// Multiple errors
{
    success: false,
    errors: [
        { field: 'email', message: 'Invalid email format' },
        { field: 'password', message: 'Must be at least 6 characters' }
    ]
}
```

---

## Interview Questions

### Q: How do you handle errors in Express?
Use error-handling middleware with 4 parameters (err, req, res, next). Must be registered LAST. Call next(error) from routes to pass errors to it.

### Q: What is the difference between try-catch and error middleware?
try-catch handles synchronous errors within a function. Error middleware handles async errors passed via next(error) or thrown in async route handlers wrapped with asyncHandler.

### Q: Why is consistent error format important?
Clients can predict the error structure and handle errors uniformly. Makes debugging easier and API more predictable.
