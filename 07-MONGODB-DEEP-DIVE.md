# Part 07 - MongoDB Deep Dive
# From Your Code Files: utils/database.js, models/home.js, models/favourite.js

## 1. What is MongoDB?

MongoDB is a NoSQL document database. It stores data as JSON-like documents (BSON).

```
MySQL (SQL):                    MongoDB (NoSQL):
Database                  →      Database
  Table                  →        Collection
    Row                 →          Document
    Column              →          Field
```

---

## 2. Your MongoDB Connection (From airbnb-updated/utils/database.js)

```javascript
const mongodb = require('mongodb');
const MongoClient = mongodb.MongoClient;

const MONGODB_URL =


let _db;  // Store connection globally

const mongoConnect = callback => {
    MongoClient.connect(MONGODB_URL)
        .then(client => {
            _db = client.db();  // Store database handle
            callback(client);    // Start server after connect
        })
        .catch(err => {
            console.log('MongoDB connection error:', err.message);
            throw err;
        });
};

const getDb = () => {
    if (!_db) {
        throw new Error('Database not initialized. Call mongoConnect() first.');
    }
    return _db;  // Return same connection for all queries
};

module.exports = { mongoConnect, getDb };
```

---

## 3. URL Breakdown

```

protocol      user    password              server                     database
```

Query parameters:
- retryWrites=true - retry failed writes
- w=majority - wait for replica confirmation
- appName=Cluster0 - name in Atlas dashboard

Note: @ symbol in password must be encoded as %40 in URL

---

## 4. MongoDB CRUD Operations

### Create (Insert)
```javascript
const db = getDb();
const result = await db.collection('homes').insertOne({
    housename: 'Beach House',
    price: 150,
    location: 'Miami'
});
console.log(result.insertedId);  // ObjectId
```

### Read (Query)
```javascript
// Find All
const allHomes = await db.collection('homes').find().toArray();

// Find with filter
const affordable = await db.collection('homes')
    .find({ price: { $lt: 100 } })
    .toArray();

// Find One
const home = await db.collection('homes').findOne({ _id: new ObjectId(id) });
```

### Update
```javascript
await db.collection('homes').updateOne(
    { _id: new ObjectId(id) },
    { $set: { price: 200 } }
);
```

### Delete
```javascript
await db.collection('homes').deleteOne({ _id: new ObjectId(id) });
```

---

## 5. MongoDB Operators

```javascript
// Comparison
{ price: { $gt: 100 } }   // price > 100
{ price: { $lt: 100 } }   // price < 100
{ price: { $in: [50, 100, 150] } }  // price in array

// Update Operators
{ $set: { field: value } }    // Set field
{ $inc: { price: 10 } }         // Increment
{ $push: { tags: 'beach' } }   // Add to array

// Logical
{ $and: [{ price: { $gt: 50 } }, { price: { $lt: 200 } }] }
{ $or: [{ name: 'A' }, { name: 'B' }] }
```

---

## 6. Your Home Model (From airbnb-updated)

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

## 7. Your Favourite Model (From airbnb-updated)

```javascript
const getDb = require('../utils/database').getDb;

module.exports = class Favourite {
    static async getFavourites() {
        const db = getDb();
        const favouriteDocs = await db.collection('favourites').find().toArray();
        return favouriteDocs.map(doc => String(doc.homeId));
    }

    static async addFavourites(homeId) {
        const db = getDb();
        const normalizedHomeId = String(homeId);
        await db.collection('favourites').updateOne(
            { homeId: normalizedHomeId },
            { $setOnInsert: { homeId: normalizedHomeId } },
            { upsert: true }
        );
    }

    static async deleteFromFavourites(homeId) {
        const db = getDb();
        await db.collection('favourites').deleteOne({ homeId: String(homeId) });
    }
};
```

---

## 8. ObjectId Handling

```javascript
const { ObjectId } = require('mongodb');

// Check validity before querying
if (!ObjectId.isValid(id)) return undefined;

// Convert string to ObjectId
const objectId = new ObjectId(id);

// Convert ObjectId to string
const str = objectId.toString();
```

---

## 9. App Startup Flow (From airbnb-updated/app.js)

```javascript
const { mongoConnect } = require('./utils/database');

mongoConnect(client => {
    console.log('Mongo client ready:', !!client);
    app.listen(PORT, () => {
        console.log(`Server is running on http://localhost:${PORT}`);
    });
});
```

Flow: mongoConnect connects to Atlas -> stores connection in _db -> calls callback -> app.listen starts server

---

## 10. MongoDB vs SQL Comparison

| MongoDB | SQL |
|---------|-----|
| Database | Database |
| Collection | Table |
| Document | Row |
| Field | Column |
| _id | PRIMARY KEY |
| .find() | SELECT |
| .insertOne() | INSERT |
| .updateOne() | UPDATE |
| .deleteOne() | DELETE |
| $set | SET |
| $gt | > |
| $lt | < |

---

## Interview Questions

### Q: What is MongoDB Atlas?
Fully managed cloud database service. Handles server provisioning, backups, security, and scaling automatically.

### Q: MongoDB vs MySQL?
MongoDB is NoSQL - stores JSON-like documents without fixed schema. MySQL is SQL - stores data in tables with fixed structure.

### Q: What is ObjectId?
MongoDB default primary key. 12-byte value with timestamp, machine ID, process ID, and counter.

### Q: What does upsert mean?
Update + Insert. If document exists, update it. If not, insert new one.

### Q: Why store connection globally?
Creating new connection per request is slow. Connect once, store in _db, reuse via getDb().
