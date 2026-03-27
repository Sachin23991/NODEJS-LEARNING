# Part 09 - REST API Design

## 1. What is REST?

REST = Representational State Transfer. It's a convention for structuring web APIs.

```
REST API = URL + HTTP Method + Response Format
```

---

## 2. RESTful Routes for "Homes"

| Method | URL | What it does | Controller |
|--------|-----|--------------|------------|
| GET | /api/homes | Get all homes | getAll |
| GET | /api/homes/:id | Get one home | getOne |
| POST | /api/homes | Create home | create |
| PUT | /api/homes/:id | Update home | update |
| DELETE | /api/homes/:id | Delete home | delete |

---

## 3. REST vs Non-REST

### Non-REST (Your current project)
```
GET  /home          → render home page (HTML)
POST /submit-form   → form submission
GET  /show-details  → details page
```

### REST (API-based)
```
GET    /api/homes     → JSON list of homes
POST   /api/homes     → Create new home
GET    /api/homes/123 → JSON one home
PUT    /api/homes/123 → Update home
DELETE /api/homes/123 → Delete home
```

---

## 4. Your Routes vs REST Routes

### Your Current (View-based)
```javascript
// User routes
router.get('/', storeController.getHomes);              // Home page
router.get('/home-detail/:id', storeController.getHomeDetail);  // Detail page
router.post('/favourites', storeController.postAddToFavourites); // Add fav

// Host routes
router.post('/host/add-home', hostController.postAddHome);  // Add home
router.post('/host/delete-home', hostController.deleteHome); // Delete
```

### REST API (JSON-based)
```javascript
// API routes
app.get('/api/homes', apiController.getAll);           // GET all
app.get('/api/homes/:id', apiController.getOne);       // GET one
app.post('/api/homes', apiController.create);          // POST create
app.put('/api/homes/:id', apiController.update);       // PUT update
app.delete('/api/homes/:id', apiController.delete);     // DELETE
```

---

## 5. REST API Controller Example

```javascript
// controllers/apiController.js

// GET /api/homes
exports.getAll = async (req, res) => {
    try {
        const homes = await Home.fetchAll();
        res.status(200).json({
            success: true,
            count: homes.length,
            data: homes
        });
    } catch (err) {
        res.status(500).json({ success: false, error: err.message });
    }
};

// GET /api/homes/:id
exports.getOne = async (req, res) => {
    try {
        const home = await Home.findById(req.params.id);
        if (!home) {
            return res.status(404).json({ success: false, error: 'Not found' });
        }
        res.status(200).json({ success: true, data: home });
    } catch (err) {
        res.status(500).json({ success: false, error: err.message });
    }
};

// POST /api/homes
exports.create = async (req, res) => {
    try {
        const home = new Home(req.body.name, req.body.imageUrl, req.body.price, req.body.location);
        await home.save();
        res.status(201).json({ success: true, data: home });
    } catch (err) {
        res.status(400).json({ success: false, error: err.message });
    }
};

// PUT /api/homes/:id
exports.update = async (req, res) => {
    try {
        await Home.update(req.params.id, req.body);
        const home = await Home.findById(req.params.id);
        res.status(200).json({ success: true, data: home });
    } catch (err) {
        res.status(400).json({ success: false, error: err.message });
    }
};

// DELETE /api/homes/:id
exports.delete = async (req, res) => {
    try {
        await Home.deleteById(req.params.id);
        res.status(200).json({ success: true, message: 'Deleted' });
    } catch (err) {
        res.status(400).json({ success: false, error: err.message });
    }
};
```

---

## 6. REST Response Format

```javascript
// Success response
{
    success: true,
    data: { ... },
    message: 'Home created successfully'
}

// Error response
{
    success: false,
    error: 'Error message here'
}

// List response
{
    success: true,
    count: 10,
    data: [ ... ]
}
```

---

## 7. HTTP Status Codes for REST

| Code | Meaning | When to use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Not logged in |
| 403 | Forbidden | Logged but no permission |
| 404 | Not Found | Resource doesn't exist |
| 500 | Server Error | Database/API error |

---

## 8. REST Best Practices

### Use nouns, not verbs
```
❌ GET /getUsers
❌ GET /fetchHome
✅ GET /users
✅ GET /homes
```

### Use plural nouns
```
❌ GET /user
❌ GET /home
✅ GET /users
✅ GET /homes
```

### Use kebab-case for URLs
```
❌ GET /userProfile
✅ GET /user-profile
```

### Version your API
```
/api/v1/homes
/api/v2/homes
```

### Handle errors consistently
```javascript
{
    success: false,
    error: 'Invalid home ID format'
}
```

---

## 9. Query Parameters

```
GET /api/homes                    → All homes
GET /api/homes?price=150          → Filter by price
GET /api/homes?sort=price         → Sort by price
GET /api/homes?page=2&limit=10     → Pagination
GET /api/homes?location=Miami     → Filter by location
```

```javascript
exports.getAll = async (req, res) => {
    const { price, sort, page = 1, limit = 10 } = req.query;

    let query = {};
    if (price) query.price = price;
    if (location) query.location = location;

    const homes = await Home.find(query)
        .sort({ price: sort === 'price' ? 1 : -1 })
        .skip((page - 1) * limit)
        .limit(Number(limit));

    res.json({ success: true, data: homes });
};
```

---

## 10. Authentication for REST API

```javascript
// Simple API key auth
const API_KEY = 'your-api-key';

const authMiddleware = (req, res, next) => {
    const key = req.headers['x-api-key'];
    if (key !== API_KEY) {
        return res.status(401).json({ success: false, error: 'Unauthorized' });
    }
    next();
};

app.use('/api', authMiddleware, apiRouter);
```

---

## Interview Questions

### Q: What is REST?
REST (Representational State Transfer) is an architectural style for web APIs. It uses HTTP methods (GET, POST, PUT, DELETE) and URLs to represent resources.

### Q: What is the difference between PUT and PATCH?
PUT replaces the entire resource. PATCH updates only the fields you send.

### Q: What status code does POST return on success?
201 Created.

### Q: When do you return 204?
When DELETE is successful and there's no content to return.
