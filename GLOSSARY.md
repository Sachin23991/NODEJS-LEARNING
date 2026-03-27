# Glossary - All Important Terms

## Node.js Terms

**Node.js** - JavaScript runtime that runs on server side, built on Chrome V8 engine

**Event Loop** - Mechanism that allows Node.js to perform non-blocking I/O operations despite being single-threaded

**npm** - Node Package Manager, world's largest software registry

**package.json** - File that contains project metadata and dependencies

**node_modules** - Folder where npm installs packages

**CommonJS** - Node.js module system using require() and module.exports

**ES Modules** - Modern module system using import/export syntax

**Buffer** - Temporary memory storage for binary data in Node.js

**Stream** - Objects for processing data in chunks instead of loading all at once

**EventEmitter** - Node.js class for handling events

**Worker Threads** - Threads for CPU-intensive tasks without blocking event loop

---

## Express.js Terms

**Express** - Minimal web framework for Node.js

**Middleware** - Functions that process requests before route handlers

**Router** - Mini Express app for organizing routes by feature

**Route** - URL path combined with HTTP method

**Handler** - Function that processes a matched route

**app.get()** - Route handler for GET requests

**app.post()** - Route handler for POST requests

**app.use()** - Middleware registration (runs for all requests)

**req.params** - URL parameters like /users/:id

**req.query** - Query string like ?search=term

**req.body** - Parsed request body data

**res.send()** - Send response (text, HTML, JSON)

**res.render()** - Render EJS template with data

**res.redirect()** - Redirect to another URL

**res.json()** - Send JSON response

**res.status()** - Set HTTP status code

---

## MongoDB Terms

**MongoDB** - NoSQL document database storing JSON-like data

**Atlas** - Fully managed MongoDB cloud service

**Collection** - Group of MongoDB documents (like SQL table)

**Document** - Single record in collection (like SQL row)

**BSON** - Binary JSON format MongoDB uses internally

**ObjectId** - MongoDB default primary key type

**CRUD** - Create, Read, Update, Delete operations

**insertOne()** - MongoDB method to insert single document

**find()** - MongoDB method to query documents

**findOne()** - MongoDB method to get single document

**updateOne()** - MongoDB method to update single document

**deleteOne()** - MongoDB method to delete single document

**upsert** - Update if exists, insert if not

**$set** - MongoDB operator to update fields

**$gt, $lt** - Greater than, less than operators

---

## MVC Terms

**Model** - Data layer, handles database operations

**View** - Presentation layer, renders HTML/templates

**Controller** - Business logic, connects Model and View

**Router** - Maps URLs to controller functions

**Static Method** - Method called on class, not instance

**Instance Method** - Method called on object instance

**Destructuring** - Extract values from objects/arrays

**Spread Operator** - Expand array/object with ...

---

## EJS Terms

**EJS** - Embedded JavaScript, template engine for Node.js

**<%= %>** - Output escaped HTML

**<%- %>** - Output raw unescaped HTML

**<% %>** - Control flow (if, for, while) no output

**Partials** - Reusable template components (header, footer)

**include()** - Include partial in template

**Template** - HTML with embedded JavaScript

---

## HTTP Terms

**HTTP** - Protocol for web communication

**GET** - HTTP method to retrieve data

**POST** - HTTP method to create data

**PUT** - HTTP method to replace entire resource

**PATCH** - HTTP method to update specific fields

**DELETE** - HTTP method to remove data

**Status 200** - OK, request succeeded

**Status 201** - Created, resource was created

**Status 301/302** - Redirect codes

**Status 400** - Bad Request, client error

**Status 401** - Unauthorized, not logged in

**Status 403** - Forbidden, no permission

**Status 404** - Not Found

**Status 500** - Internal Server Error

---

## Security Terms

**Authentication** - Verifying user identity (login)

**Authorization** - Checking user permissions (what they can do)

**JWT** - JSON Web Token for authentication

**Session** - Server-side user state storage

**Hashing** - One-way encryption of passwords (bcrypt)

**CORS** - Cross-Origin Resource Sharing

**Helmet** - Express security middleware

**Rate Limiting** - Limit requests per IP/time

**Environment Variables** - Secure config storage (.env)

---

## API Terms

**REST** - Representational State Transfer, API design pattern

**API** - Application Programming Interface

**Endpoint** - URL where API can be accessed

**CRUD API** - Create, Read, Update, Delete endpoints

**RESTful** - Following REST conventions

**JSON** - JavaScript Object Notation, data format

**API Key** - Token for API authentication

---

## Async Terms

**Promise** - Object representing eventual completion of async operation

**async/await** - Syntax for handling Promises

**Callback** - Function passed to be called when async completes

**Event Loop** - Processes async callbacks

**Microtask Queue** - Priority queue for Promise callbacks

**Macrotask Queue** - Queue for setTimeout/setInterval callbacks

**then()** - Handle Promise success

**catch()** - Handle Promise failure

**finally()** - Run after Promise settles (success or failure)

**Promise.all()** - Wait for all promises

**Promise.race()** - Return when first promise settles

---

## Architecture Terms

**MVC** - Model-View-Controller architecture pattern

**Layered Architecture** - Separating app into presentation/business/data layers

**Separation of Concerns** - Each component has single responsibility

**Coupling** - How much components depend on each other

**Cohesion** - How related components are grouped

**Scalability** - Ability to handle more load

---

## Your Project Terms

**airbnb project** - Full-stack rental listing app with MongoDB

**hostRouter** - Routes for hosts to manage listings

**userRouter** - Routes for users to browse homes

**storeController** - Handles user-facing pages

**hostController** - Handles host management pages

**Home model** - CRUD for home listings in MongoDB

**Favourite model** - CRUD for user favorites

**mongoConnect** - Connects to MongoDB Atlas

**getDb()** - Returns shared database connection

**PRG Pattern** - Post-Redirect-Get to prevent duplicate submissions
