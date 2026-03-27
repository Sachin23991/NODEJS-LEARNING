# Complete Node.js & Express.js Interview Guide
# By Sachin - Built from your own code

## How to use this guide
This guide is organized from BASICS to ADVANCED. Start from Part 01 if you're
fresh, or jump to any part you need to brush up on.

Each part builds on the previous one - just like your learning journey!

---

## Quick Jump Links

### Level 1 - Fundamentals (What You Already Know ✅)
- [Part 01 - JavaScript Refresher](01-JAVASCRIPT-REFRESHER.md)
- [Part 02 - Node.js Core](02-NODE-JS-CORE.md)

### Level 2 - Express Framework (Your Strong Foundation ✅)
- [Part 03 - Express.js Basics](03-EXPRESS-BASICS.md)
- [Part 04 - Express Router & MVC](04-EXPRESS-ROUTER-MVC.md)
- [Part 05 - Middleware Deep Dive](05-MIDDLEWARE-DEEP-DIVE.md)
- [Part 06 - EJS Template Engine](06-EJS-TEMPLATE-ENGINE.md)

### Level 3 - Database (Your MongoDB Atlas Experience ✅)
- [Part 07 - MongoDB Deep Dive](07-MONGODB-DEEP-DIVE.md)
- [Part 08 - Mongoose ODM](08-MONGOOSE-ODM.md)

### Level 4 - Advanced Concepts (For Interview Success)
- [Part 09 - REST API Design](09-REST-API-DESIGN.md)
- [Part 10 - Authentication & Security](10-AUTHENTICATION-SECURITY.md)
- [Part 11 - Error Handling](11-ERROR-HANDLING.md)
- [Part 12 - Performance & Best Practices](12-PERFORMANCE-BEST-PRACTICES.md)

### Level 5 - Real Interview Prep
- [Part 13 - Interview Questions & Answers](13-INTERVIEW-QUESTIONS.md)
- [Part 14 - Architecture Patterns](14-ARCHITECTURE-PATTERNS.md)
- [Part 15 - System Design Basics](15-SYSTEM-DESIGN-BASICS.md)

### Quick References
- [Cheat Sheet](CHEATSHEET.md)
- [Code Patterns from Your Files](CODE-PATTERNS.md)
- [Glossary](GLOSSARY.md)

---

## What You Already Built (Your Portfolio)
```
airbnb-updated/
├── app.js              → Main entry point (MongoDB connected)
├── router/
│   ├── hostRouter.js   → /host/* routes
│   └── userRouter.js   → /* routes
├── controllers/
│   ├── hostController.js → Host actions (CRUD homes)
│   └── storeController.js → User actions (favourites)
├── models/
│   ├── home.js         → MongoDB homes collection
│   └── favourite.js    → MongoDB favourites collection
├── utils/
│   ├── database.js     → MongoDB Atlas connection
│   └── path.js        → Root directory helper
└── views/              → EJS templates (store/, host/, partials/)
```

---

## Architecture Flow (Your MVC Pattern)

```
┌─────────────────────────────────────────────────────────────────┐
│                         REQUEST                                 │
│                    http://localhost:3030/                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     app.js (Main Entry)                          │
│  • Creates Express app                                          │
│  • Sets view engine (EJS)                                       │
│  • Connects MongoDB via mongoConnect()                          │
│  • Mounts routers                                               │
│  • app.listen() starts server                                    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
    ┌─────────────────┐       ┌─────────────────┐
    │  hostRouter    │       │  userRouter     │
    │  /host/*       │       │  /*             │
    └────────┬───────┘       └────────┬────────┘
             │                        │
             ▼                        ▼
    ┌─────────────────┐       ┌─────────────────┐
    │ hostController │       │ storeController │
    │ • getAddHome   │       │ • getHomes      │
    │ • postAddHome  │       │ • getHomeDetail │
    │ • getHostHome  │       │ • postFavourite │
    │ • getEditHome  │       │ • getFavList   │
    │ • deleteHome   │       │                 │
    └────────┬───────┘       └────────┬────────┘
             │                        │
             ▼                        ▼
    ┌─────────────────────────────────────────────────┐
    │                      MODELS                      │
    │  Home.fetchAll()    → MongoDB 'homes' col     │
    │  Home.save()        → insertOne()             │
    │  Home.findById()     → findOne()              │
    │  Home.update()       → updateOne()            │
    │  Home.deleteById()   → deleteOne()            │
    │                                                   │
    │  Favourite.getFavourites()  → find().toArray() │
    │  Favourite.addFavourites()  → updateOne(upsert)│
    │  Favourite.deleteFromFav()  → deleteOne()      │
    └───────────────────────┬─────────────────────────┘
                            │
                            ▼
    ┌─────────────────────────────────────────────────┐
    │              MONGODB ATLAS CLOUD                │
    │  • Connection via mongoConnect()               │
    │  • Database: airbnb-updated                   │
    │  • Collections: homes, favourites            │
    └─────────────────────────────────────────────────┘
                            │
                            ▼
    ┌─────────────────────────────────────────────────┐
    │              VIEW (EJS Templates)               │
    │  res.render('store/home', { homes })          │
    │  <% homes.forEach(home => { %>                │
    │    <%= home.name %>                            │
    │  <% }); %>                                     │
    └─────────────────────────────────────────────────┘
```

---

## Key Concepts You MUST Know for Interview

### 1. Difference between app.get() and app.use()
┌──────────────────────────────────────────────────────────────────┐
│ app.get()              │ app.use()                               │
├───────────────────────┼──────────────────────────────────────────┤
│ Handles SPECIFIC      │ Handles ANY request (middleware)        │
│ HTTP GET requests     │ or prefix-matched routes                │
│ Exact path match      │ Prefix match (all /user/*)              │
│ res.send/res.render   │ Usually calls next()                    │
└───────────────────────┴──────────────────────────────────────────┘

### 2. Middleware Order Matters!
```javascript
app.use(middleware1);  // Runs FIRST for every request
app.use(middleware2);  // Runs SECOND
app.get('/path', handler);  // Routes AFTER middleware
app.use404(handler);  // LAST - catches unmatched
```

### 3. CRUD → REST Methods
┌──────────────────────────────────────────────────────────────────┐
│ CRUD Operation  │ HTTP Method │ Route Example                    │
├───────────────┼──────────────┼──────────────────────────────────┤
│ Create        │ POST         │ POST /homes                      │
│ Read One      │ GET          │ GET /homes/:id                   │
│ Read All      │ GET          │ GET /homes                       │
│ Update        │ PUT/PATCH    │ PUT /homes/:id                  │
│ Delete        │ DELETE       │ DELETE /homes/:id               │
└───────────────┴──────────────┴──────────────────────────────────┘

### 4. MongoDB vs MySQL
┌──────────────────────────────────────────────────────────────────┐
│ MongoDB (NoSQL)        │ MySQL (SQL)                             │
├────────────────────────┼────────────────────────────────────────┤
│ Collections            │ Tables                                 │
│ Documents (JSON-like)  │ Rows                                   │
│ No fixed schema        │ Fixed schema                           │
│ Flexible documents     │ Rigid structure                        │
│ Embedded documents     │ Related tables                          │
│ _id: ObjectId         │ id: INT AUTO_INCREMENT                 │
│ db.homes.find()       │ SELECT * FROM homes                    │
│ db.homes.insertOne()  │ INSERT INTO homes                      │
└────────────────────────┴────────────────────────────────────────┘

### 5. Your MongoDB Connection Pattern
```javascript
// database.js
let _db;  // Shared connection
const mongoConnect = callback => {
    MongoClient.connect(MONGODB_URL)
        .then(client => {
            _db = client.db();  // Store connection once
            callback(client);   // Start server AFTER connect
        });
};
const getDb = () => _db;  // Reuse same connection

// app.js
mongoConnect(client => {
    app.listen(PORT, () => console.log('Server started'));
});
```

### 6. Express Router Pattern
```javascript
// router/userRouter.js
const router = express.Router();
router.get('/', controller.getHomes);
router.get('/:id', controller.getHomeDetail);
module.exports = router;

// app.js
app.use('/', userRouter);  // All routes prefixed with /
```

---

## Interview One-Liners

**What is Node.js?**
"Node.js is a JavaScript runtime built on Chrome's V8 engine that lets you run JavaScript on the server. It uses an event-driven, non-blocking I/O model that makes it great for real-time apps."

**What is Express.js?**
"Express is a minimal, flexible Node.js web application framework that provides features for web and mobile applications. It handles routing, middleware, and templating."

**What is Middleware?**
"Middleware functions are functions that have access to the request object, response object, and the next function. They execute in order, can modify req/res, and must either end the cycle or call next()."

**What is the difference between PUT and PATCH?**
"PUT replaces the entire resource - if you send {name: 'A'} to /homes/1 with PUT, it sets the whole object to {name: 'A'}. PATCH only updates the fields you send - {name: 'A'} only changes the name field."

**What is the PRG pattern?**
"Post-Redirect-Get. After a POST request (form submit), the server redirects to a GET page. This prevents duplicate submissions when user refreshes."

**Why use MongoDB Atlas?**
"MongoDB Atlas is a fully managed cloud database service. It handles scaling, backups, security, and connectivity so we don't manage our own database server."
