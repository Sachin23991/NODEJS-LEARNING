# Code Patterns Extracted From Your Files

## Pattern 1: Express Server Setup
```javascript
// airbnb/app.js
const express = require('express');
const app = express();

app.set('view engine', 'ejs');
app.set('views', 'views');

app.use(bodyParser.urlencoded({ extended: false }));
app.use(express.static(path.join(__dirname, 'public')));

app.use('/host', hostRouter);
app.use('/', userRouter);

app.use((req, res) => {
    res.status(404).sendFile(path.join(rootdir, 'views', '404.html'));
});

mongoConnect(client => {
    app.listen(PORT, () => console.log(`Running on port ${PORT}`));
});
```

## Pattern 2: Router Definition
```javascript
// airbnb/router/userRouter.js
const express = require('express');
const router = express.Router();

router.get('/', storeController.getHomes);
router.get('/home-detail/:id', storeController.getHomeDetail);
router.post('/favourites', storeController.postAddToFavourites);

module.exports = router;
```

## Pattern 3: Controller with MongoDB
```javascript
// airbnb-updated/controllers/hostController.js
const Home = require('../models/home');

exports.postAddHome = async (req, res) => {
    const { name, imageUrl, price, location } = req.body;
    const home = new Home(name, imageUrl, price, location);
    await home.save();
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
```

## Pattern 4: MongoDB Model
```javascript
// airbnb-updated/models/home.js
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
};
```

## Pattern 5: MongoDB Connection
```javascript
// airbnb-updated/utils/database.js
const mongodb = require('mongodb');
const MongoClient = mongodb.MongoClient;

const MONGODB_URL = 'mongodb+srv://sachin:rao9350@cluster0...';

let _db;

const mongoConnect = callback => {
    MongoClient.connect(MONGODB_URL)
        .then(client => {
            _db = client.db();
            callback(client);
        })
        .catch(err => {
            console.log('MongoDB connection error:', err.message);
            throw err;
        });
};

const getDb = () => {
    if (!_db) throw new Error('Database not initialized');
    return _db;
};

module.exports = { mongoConnect, getDb };
```

## Pattern 6: Path Utility
```javascript
// airbnb/utils/path.js
const path = require('path');
const rootdir = path.dirname(require.main.filename);
module.exports = rootdir;
```

## Pattern 7: EJS Template
```ejs
<!-- views/store/home.ejs -->
<%- include('partials/header') %>

<h1><%= pageTitle %></h1>

<% homes.forEach(home => { %>
    <div class="card">
        <h2><%= home.name %></h2>
        <p>$<%= home.price %></p>
        <a href="/home-detail/<%= home.id %>">View</a>
    </div>
<% }); %>

<%- include('partials/footer') %>
```

## Pattern 8: Form Submission
```javascript
// Controller - receives POST
exports.postAddHome = async (req, res) => {
    const { name, imageUrl, price, location } = req.body;
    const home = new Home(name, imageUrl, price, location);
    await home.save();
    res.redirect('/host/home-list');  // PRG pattern
};
```

## Pattern 9: Dynamic Route
```javascript
// Router
router.get('/home-detail/:id', storeController.getHomeDetail);

// Controller
exports.getHomeDetail = async (req, res) => {
    const homeId = req.params.id;
    const home = await Home.findById(homeId);
    if (!home) return res.redirect('/');
    res.render('store/home-detail', { home, ... });
};
```

## Pattern 10: 404 Handler
```javascript
// Always LAST
app.use((req, res, next) => {
    res.status(404).sendFile(path.join(rootdir, 'views', '404.html'));
});
```

## Pattern 11: Middleware with next()
```javascript
app.use((req, res, next) => {
    console.log(req.url, req.method);
    next();  // MUST call next
});
```

## Pattern 12: Destructuring req.body
```javascript
// Instead of:
const name = req.body.name;
const price = req.body.price;
const location = req.body.location;

// Use destructuring:
const { name, imageUrl, price, location } = req.body;
```
