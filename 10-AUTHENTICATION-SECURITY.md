# Part 10 - Authentication & Security

## 1. Authentication vs Authorization

```
Authentication = Who are you? (login)
Authorization = What can you do? (permissions)
```

---

## 2. Password Hashing

Never store passwords as plain text!

```javascript
const bcrypt = require('bcrypt');

async function hashPassword(password) {
    const salt = await bcrypt.genSalt(10);
    return await bcrypt.hash(password, salt);
}

async function comparePassword(password, hash) {
    return await bcrypt.compare(password, hash);
}

// Usage
const hashedPassword = await hashPassword('mypassword123');
const isMatch = await comparePassword('mypassword123', hashedPassword);
```

---

## 3. JWT (JSON Web Tokens)

JWT lets you authenticate users without sessions.

```javascript
const jwt = require('jsonwebtoken');
const SECRET = 'your-secret-key';

// Create token
function createToken(user) {
    return jwt.sign(
        { userId: user.id, email: user.email },
        SECRET,
        { expiresIn: '1h' }
    );
}

// Verify token
function verifyToken(token) {
    try {
        return jwt.verify(token, SECRET);
    } catch (err) {
        return null;
    }
}

// Usage in route
app.post('/login', async (req, res) => {
    const user = await User.findOne({ email: req.body.email });
    if (!user) return res.status(401).send('Invalid credentials');

    const isMatch = await bcrypt.compare(req.body.password, user.password);
    if (!isMatch) return res.status(401).send('Invalid credentials');

    const token = createToken(user);
    res.json({ token });
});
```

---

## 4. JWT Middleware

```javascript
function authMiddleware(req, res, next) {
    const token = req.headers['authorization'];

    if (!token) {
        return res.status(401).json({ error: 'No token provided' });
    }

    const decoded = verifyToken(token);
    if (!decoded) {
        return res.status(401).json({ error: 'Invalid token' });
    }

    req.userId = decoded.userId;  // Add user ID to request
    next();
}

// Usage
app.get('/profile', authMiddleware, (req, res) => {
    res.json({ userId: req.userId });
});
```

---

## 5. Session-Based Auth (express-session)

```javascript
const session = require('express-session');

app.use(session({
    secret: 'your-secret-key',
    resave: false,
    saveUninitialized: false,
    cookie: { secure: false }  // true in production with HTTPS
}));

// Login
app.post('/login', async (req, res) => {
    const user = await User.findOne({ email: req.body.email });
    if (user && await bcrypt.compare(req.body.password, user.password)) {
        req.session.userId = user.id;  // Store in session
        res.json({ message: 'Logged in' });
    } else {
        res.status(401).json({ error: 'Invalid credentials' });
    }
});

// Logout
app.post('/logout', (req, res) => {
    req.session.destroy();
    res.json({ message: 'Logged out' });
});

// Check if logged in
app.get('/profile', (req, res) => {
    if (!req.session.userId) {
        return res.status(401).json({ error: 'Not logged in' });
    }
    res.json({ userId: req.session.userId });
});
```

---

## 6. Security Headers (helmet)

```javascript
const helmet = require('helmet');
app.use(helmet());

// What helmet does:
- Hides X-Powered-By header
- Sets Content-Security-Policy
- Prevents clickjacking
- Prevents MIME type sniffing
```

---

## 7. CORS (Cross-Origin Resource Sharing)

```javascript
const cors = require('cors');

// Allow all origins (development)
app.use(cors());

// Allow specific origin (production)
app.use(cors({
    origin: 'http://localhost:3000',
    credentials: true
}));
```

---

## 8. Environment Variables (.env)

Never hardcode secrets!

```bash
# .env file
PORT=3000
DB_URL=mongodb+srv://...
SECRET=myverysecretkey123
```

```javascript
// code
require('dotenv').config();

const PORT = process.env.PORT;
const DB_URL = process.env.DB_URL;
const SECRET = process.env.SECRET;
```

---

## 9. Input Validation

```javascript
// Simple validation
function validateInput(data) {
    const errors = [];

    if (!data.email || !data.email.includes('@')) {
        errors.push('Invalid email');
    }

    if (!data.password || data.password.length < 6) {
        errors.push('Password must be at least 6 characters');
    }

    return errors;
}

// Usage
app.post('/register', async (req, res) => {
    const errors = validateInput(req.body);
    if (errors.length > 0) {
        return res.status(400).json({ errors });
    }
    // Continue with registration
});
```

---

## 10. Rate Limiting

Prevent too many requests from same IP.

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,  // 15 minutes
    max: 100  // 100 requests per window
});

app.use(limiter);

// Stricter for login
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5  // Only 5 login attempts
});

app.post('/login', loginLimiter, (req, res) => {
    // Login logic
});
```

---

## Interview Questions

### Q: What is JWT and how does it work?
JWT (JSON Web Token) is a token format for authentication. The server signs a token containing user info. Client sends this token with each request. Server verifies the signature to authenticate.

### Q: What is the difference between authentication and authorization?
Authentication is verifying who the user is (login). Authorization is checking what the user is allowed to do (permissions/roles).

### Q: Why should you never store passwords as plain text?
If the database is compromised, all passwords are exposed. Hashing (with bcrypt) one-way encrypts passwords so they cannot be reversed.

### Q: What is CORS?
Cross-Origin Resource Sharing. Browsers block requests from one origin to another unless the server allows it. CORS headers tell browsers which origins can access your API.

### Q: How do you secure an Express app?
Use helmet for security headers, cors for cross-origin requests, bcrypt for password hashing, JWT or sessions for authentication, rate limiting, and environment variables for secrets.
