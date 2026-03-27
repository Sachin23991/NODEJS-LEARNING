# Part 14 - Architecture Patterns

## 1. MVC Pattern (Your Current Architecture)

Request Flow:
```
Browser -> app.js middleware -> Router -> Controller -> Model -> Database
                                    |
                                    v
                                res.render()
                                    |
                                    v
                            EJS Template -> HTML
```

---

## 2. Your Architecture Structure

```
app.js           → Config + Server Start + MongoDB Connection
router/         → Routes + URL Mapping
controllers/    → Logic + Data Transformation
models/         → Database Operations
utils/          → Database Connection + Path Helpers
views/          → EJS Templates
```

---

## 3. Request-Response Cycle (Your Project)

```
1. Browser: GET /home-detail/abc123

2. app.js middleware (in order):
   bodyParser -> static files -> logger

3. Router matches: userRouter.get('/home-detail/:id')

4. Controller: storeController.getHomeDetail(req, res)

5. Model: Home.findById('abc123')

6. MongoDB: db.collection('homes').findOne({ _id: ObjectId('abc123') })

7. MongoDB returns document

8. Controller renders: res.render('store/home-detail', { homes: home })

9. EJS template processed

10. HTML sent to browser
```

---

## 4. Layered Architecture

```
PRESENTATION LAYER
Router -> Controller -> View (EJS)
Handles HTTP requests/responses

BUSINESS LOGIC LAYER
Controller -> Services
Processes data, applies rules

DATA ACCESS LAYER
Model -> Database
Database operations only
```

---

## 5. Separation of Concerns

| Layer | Responsibility | Should NOT have |
|-------|---------------|----------------|
| Router | URL mapping | Business logic |
| Controller | Logic + calling models | Direct DB calls |
| Model | Database operations | HTTP knowledge |
| View/EJS | Display data | DB queries |

---

## 6. MVC Request Flow Diagram

```
REQUEST
  |
  v
[Middleware: body-parser, static, logger]
  |
  v
[Router matches URL and method]
  |
  v
[Controller function called]
  |
  v
[Controller calls Model]
  |
  v
[Model queries MongoDB]
  |
  v
[Data returned to Controller]
  |
  v
[res.render() with data]
  |
  v
[EJS Template rendered]
  |
  v
RESPONSE (HTML)
```

---

## 7. Your Current Pattern Summary

app.js:
- Creates Express app
- Sets view engine (EJS)
- Connects MongoDB
- Registers middleware
- Mounts routers
- Starts server

Routers:
- Define URL routes
- Map to controller functions
- No business logic

Controllers:
- Receive request
- Call model methods
- Handle response (render/redirect)

Models:
- Database CRUD operations
- Return data to controllers

Views (EJS):
- HTML with embedded JavaScript
- Receives data from controller
- Renders final HTML

---

## Interview Questions

### Q: What architecture does your project use?
Your airbnb project uses MVC (Model-View-Controller) pattern with Express Router for route organization.

### Q: Explain your request flow?
Browser sends request -> Middleware processes -> Router matches -> Controller handles logic -> Model queries MongoDB -> Controller renders EJS view -> HTML sent to browser.

### Q: Why separate concerns?
Separation of concerns makes code maintainable, testable, and organized. Each layer has a specific job.
