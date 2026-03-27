# Part 04 - Express Router & MVC Architecture
# From Your Code Files: router files, controller files, models

## 1. What is Express Router?

Express Router lets you split your routes into separate files. Instead of putting all routes in app.js, you create multiple routers.

**Why Use Router?**
```
Without Router (in app.js):
app.get('/', ...)       // 50 routes
app.get('/users', ...)
app.get('/products', ...)
...

With Router (separate files):
app.js → uses userRouter.js → 20 user routes
app.js → uses adminRouter.js → 15 admin routes
app.js → uses apiRouter.js → 30 API routes
```

---

## 2. Your Router Setup (From airbnb project)

### app.js
```javascript
const express = require('express');
const app = express();

// Import routers
const hostRouter = require('./router/hostRouter');
const userRouter = require('./router/userRouter');

// Mount routers with prefixes
app.use('/host', hostRouter);   // All /host/* routes go to hostRouter
app.use('/', userRouter);       // All /* routes go to userRouter
```

### router/hostRouter.js
```javascript
const express = require('express');
const hostRouter = express.Router();

hostRouter.get('/add-home', hostController.getAddHome);
hostRouter.post('/add-home', hostController.postAddHome);
hostRouter.get('/home-list', hostController.getHostHomeList);

module.exports = hostRouter;
```

### router/userRouter.js
```javascript
const express = require('express');
const userRouter = express.Router();

userRouter.get('/', storeController.getHomes);
userRouter.get('/home-list', storeController.getHomeList);
userRouter.get('/home-detail/:id', storeController.getHomeDetail);
userRouter.post('/favourites', storeController.postAddToFavourites);

module.exports = userRouter;
```

---

## 3. Router Path Prefixing

```javascript
// In app.js:
app.use('/host', hostRouter);

// In hostRouter.js:
router.get('/add-home', ...);   // Actual URL: /host/add-home
router.get('/home-list', ...);  // Actual URL: /host/home-list
```

**Note:** The `/host` prefix is added in app.js, so routes in hostRouter.js don't need to include it.

---

## 4. MVC Architecture (Model-View-Controller)

MVC separates your code into 3 layers:

```
┌──────────────────────────────────────────────────────────────────┐
│                      MVC ARCHITECTURE                             │
│                                                                   │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐        │
│  │    MODEL    │◀───▶│ CONTROLLER │◀───▶│    VIEW     │        │
│  │             │     │             │     │             │        │
│  │  • MongoDB  │     │ • Logic    │     │  • EJS      │        │
│  │  • JSON    │     │ • req/res  │     │  • HTML    │        │
│  │  • Data    │     │ • Calls    │     │  • Template │        │
│  │  Operations│     │   Models   │     │  Rendering │        │
│  └─────────────┘     └─────────────┘     └─────────────┘        │
│                                                                   │
│  ROUTER → CONTROLLER → MODEL → DATABASE                         │
│  ROUTER → CONTROLLER → VIEW ← (EJS Template)                    │
└───────────────────────────────────────────────────────────────────┘
```

### Model (Data Layer)
- Handles ALL data operations
- Connects to database (MongoDB, MySQL, etc.)
- Your `models/home.js`, `models/favourite.js`

### View (Presentation Layer)
- EJS templates that render HTML
- Receives data from controller
- Your `views/` folder

### Controller (Business Logic)
- Receives requests, processes data
- Calls models for data
- Sends response (renders view or redirects)
- Your `controllers/hostController.js`, `controllers/storeController.js`

---

## 5. Your Models (From airbnb-updated)

### models/home.js
```javascript
const mongodb = require('mongodb');
const ObjectId = mongodb.ObjectId;
const getDb = require('../utils/database').getDb;

module.exports = class Home {
    constructor(name, imageUrl, price, location, id = null) {
        this.id = id;
        this.name = name;
        this.imageUrl = imageUrl;
        this.price = price;
        this.location = location;
    }

    async save() {
        const db = getDb();
        const result = await db.collection('homes').insertOne({
            housename: this.name,
            price: this.price,
            location: this.location,
            housepictureurl: this.imageUrl
        });
        this.id = result.insertedId.toString();
        return this;
    }

    static async fetchAll() {
        const db = getDb();
        const rows = await db.collection('homes').find().sort({ _id: -1 }).toArray();
        return rows.map(row => ({
            id: row._id.toString(),
            name: row.housename,
            price: row.price,
            location: row.location,
            imageUrl: row.housepictureurl
        }));
    }

    static async findById(id) {
        if (!ObjectId.isValid(id)) return undefined;
        const db = getDb();
        const row = await db.collection('homes').findOne({ _id: new ObjectId(id) });
        if (!row) return undefined;
        return {
            id: row._id.toString(),
            name: row.housename,
            price: row.price,
            location: row.location,
            imageUrl: row.housepictureurl
        };
    }

    static async deleteById(id) {
        if (!ObjectId.isValid(id)) return { deletedCount: 0 };
        const db = getDb();
        return db.collection('homes').deleteOne({ _id: new ObjectId(id) });
    }

    static async update(id, updatedData) {
        if (!ObjectId.isValid(id)) return { modifiedCount: 0 };
        const db = getDb();
        return db.collection('homes').updateOne(
            { _id: new ObjectId(id) },
            { $set: {
                housename: updatedData.name,
                price: updatedData.price,
                location: updatedData.location,
                housepictureurl: updatedData.imageUrl
            }}
        );
    }
};
```

---

## 6. Your Controllers (From airbnb-updated)

### controllers/hostController.js
```javascript
const Home = require('../models/home');

exports.getAddHome = (req, res) => {
    res.render('host/add-home', {
        pageTitle: 'Add Home',
        path: '/host/add-home'
    });
};

exports.postAddHome = async (req, res) => {
    // Destructure form data
    const { name, imageUrl, price, location } = req.body;

    // Create and save home
    const home = new Home(name, imageUrl, price, location);
    await home.save();

    // Redirect (PRG pattern)
    res.redirect('/host/home-list');
};

exports.getHostHomeList = async (req, res) => {
    const homes = await Home.fetchAll();
    res.render('host/host-home-list', {
        homes: homes,
        pageTitle: 'My Homes',
        path: '/host/home-list'
    });
};

exports.getEditHome = async (req, res) => {
    const homeId = req.params.id;
    const home = await Home.findById(homeId);
    if (!home) return res.redirect('/host/home-list');
    res.render('host/edit-home', {
        home: home,
        pageTitle: 'Edit Home',
        path: '/host/edit-home'
    });
};

exports.postEditHome = async (req, res) => {
    const { homeId, name, imageUrl, price, location } = req.body;
    await Home.update(homeId, { name, imageUrl, price, location });
    res.redirect('/host/home-list');
};

exports.deleteHome = async (req, res) => {
    const homeId = req.body.homeId;
    await Home.deleteById(homeId);
    res.redirect('/host/home-list');
};
```

---

## 7. RESTful Routes Pattern

REST = Representational State Transfer. It's a convention for structuring APIs.

```
┌──────────────────────────────────────────────────────────────────┐
│              RESTful Routes for "Homes"                          │
├──────────────────────────────────────────────────────────────────┤
│ Method   │ URL                  │ Controller Function          │
├──────────┼───────────────────────┼──────────────────────────────┤
│ GET      │ /homes               │ getAllHomes                 │
│ GET      │ /homes/:id           │ getHomeById                 │
│ POST     │ /homes               │ createHome                  │
│ PUT      │ /homes/:id           │ updateHome                  │
│ DELETE   │ /homes/:id           │ deleteHome                  │
└──────────┴───────────────────────┴──────────────────────────────┘
```

### Your Project Routes:
```
HOST Routes (/host prefix):
GET    /host/add-home      → getAddHome      → Show form
POST   /host/add-home      → postAddHome    → Create home
GET    /host/home-list     → getHostHomeList → List homes
GET    /host/edit-home/:id → getEditHome    → Show edit form
POST   /host/edit-home     → postEditHome   → Update home
POST   /host/delete-home   → deleteHome    → Delete home

USER Routes (/ prefix):
GET    /                   → getHomes       → Home page
GET    /home-list          → getHomeList    → List view
GET    /home-detail/:id    → getHomeDetail → Detail view
POST   /favourites         → postAddToFav  → Add favorite
GET    /favourite-list     → getFavList    → List favorites
```

---

## 8. PRG Pattern (Post-Redirect-Get)

**Problem:** After form submit, if user refreshes, form submits AGAIN!

**Solution:** PRG Pattern
```
1. User submits form → POST /host/add-home
2. Server processes data, saves to DB
3. Server sends REDIRECT (302) to /host/home-list
4. Browser loads /host/home-list (GET request)
5. User refreshes → only GET request, no duplicate submission!
```

**From your code:**
```javascript
exports.postAddHome = async (req, res) => {
    const home = new Home(...);
    await home.save();
    res.redirect('/host/home-list');  // REDIRECT after POST
};
```

---

## 9. Static vs Instance Methods

```javascript
class Home {
    constructor(...) { }  // Instance method - called on object

    save() { }            // Instance method - new Home().save()

    static fetchAll() { }  // Static method - Home.fetchAll()
    static findById() { }  // Static method - Home.findById()
}
```

**Static vs Instance:**
```javascript
// Instance method
const home = new Home("Beach House", 150, "Miami");
home.save();  // Saves THIS specific home

// Static method
Home.fetchAll();  // Gets ALL homes, no specific instance
```

---

## 10. Separation of Concerns

```
┌──────────────────────────────────────────────────────────────┐
│                        app.js                                │
│  • Creates Express app                                      │
│  • Configures middleware                                    │
│  • Mounts routers                                          │
│  • Connects to database                                    │
│  • Starts server                                          │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                      ROUTERS                                 │
│  hostRouter.js  → /host/*                                  │
│  userRouter.js  → /*                                        │
│  • Define routes                                            │
│  • Match URL to controller                                  │
│  • NO business logic here                                   │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                    CONTROLLERS                               │
│  hostController.js  → Host actions                          │
│  storeController.js → User actions                          │
│  • Receive request                                           │
│  • Extract data from req                                    │
│  • Call model methods                                       │
│  • Send response (render/redirect)                          │
│  • ALL business logic here                                  │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                       MODELS                                 │
│  home.js           → Home CRUD                              │
│  favourite.js      → Favourites CRUD                        │
│  • Database operations                                       │
│  • No HTTP knowledge                                        │
│  • Pure data logic                                          │
└──────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

### Q: What is the MVC pattern?
MVC separates code into Model (data), View (presentation), and Controller (logic). The Router directs requests to Controllers, which call Models for data and render Views.

### Q: What is the difference between app.use() and app.get()?
app.use() matches ANY HTTP method and runs as middleware. app.get() specifically matches GET requests with exact or prefix paths.

### Q: Why do we use separate router files?
Routers organize code by feature or domain. A host router handles host-related routes, a user router handles user routes. This makes code maintainable as the app grows.

### Q: What is the PRG pattern and why use it?
Post-Redirect-Get prevents duplicate form submissions. After POST, redirect to a GET page so refresh doesn't resubmit the form.
