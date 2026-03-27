# Part 06 - EJS Template Engine
# From Your notes.txt and airbnb notes

## 1. What is EJS?

EJS = Embedded JavaScript. It lets you put JavaScript code inside HTML templates.

```
Regular HTML:        <h1>Welcome to Beach House</h1>
                       <p>Price: $150</p>

EJS Template:        <h1>Welcome to <%= home.name %></h1>
                       <p>Price: $<%= home.price %></p>
```

---

## 2. Setup EJS (From airbnb notes)

```javascript
// Step 1: Install
npm install ejs

// Step 2: Configure in app.js
app.set('view engine', 'ejs');      // Use EJS as template engine
app.set('views', 'views');          // Where to find .ejs files

// Step 3: Create views in views/ folder
// views/home.ejs
```

---

## 3. EJS Tags (From Your Notes)

```
<% %>          → Control flow (if, for, while) - NO output
<%= %>         → Output (escapes HTML)
<%- %>         → Output (raw HTML, NOT escaped)
<%# %>         → Comment (not rendered)
```

### Examples from your notes.txt:
```ejs
<%# This is a comment %>

<% if (user.isAdmin) { %>
    <%= user.name %>
<% } %>

<% homes.forEach(home => { %>
    <h1><%= home.name %></h1>
<% }); %>
```

---

## 4. Your app.js Setup (From airbnb/app.js)

```javascript
// Set view engine
app.set('view engine', 'ejs');

// Set views directory (where to find .ejs files)
app.set('views', 'views');
```

---

## 5. Render a View (From Your Controllers)

```javascript
// Controller renders a view with data
res.render('store/home', {
    homes: homes,              // Variable 'homes' available in EJS
    pageTitle: 'All Homes',  // Variable 'pageTitle' available in EJS
    path: '/'                  // Variable 'path' available in EJS
});
```

**What happens:**
```
1. Express looks for: views/store/home.ejs
2. Processes EJS tags with data
3. Sends final HTML to browser
```

---

## 6. EJS Syntax Examples

### Output Variable
```ejs
<h1><%= pageTitle %></h1>
<!-- If pageTitle = "Home", outputs: <h1>Home</h1> -->
```

### Loop Through Array
```ejs
<h1>All Homes</h1>
<% homes.forEach(home => { %>
    <div class="home-card">
        <h2><%= home.name %></h2>
        <p>Price: $<%= home.price %></p>
        <p>Location: <%= home.location %></p>
        <a href="/home-detail/<%= home.id %>">View Details</a>
    </div>
<% }); %>
```

### Conditional
```ejs
<% if (user.isLoggedIn) { %>
    <a href="/logout">Logout</a>
<% } else { %>
    <a href="/login">Login</a>
<% } %>
```

### Using Data from Controller
```javascript
// Your storeController.js
exports.getHomes = async (req, res) => {
    const homes = await Home.fetchAll();
    res.render('store/home', {
        homes: homes,
        pageTitle: 'All Homes',
        path: '/'
    });
};
```

```ejs
<!-- views/store/home.ejs -->
<!DOCTYPE html>
<html>
<head>
    <title><%= pageTitle %></title>
</head>
<body>
    <nav>
        <a href="/" class="<%= path === '/' ? 'active' : '' %>">Home</a>
        <a href="/home-list">List View</a>
    </nav>

    <h1><%= pageTitle %></h1>

    <% homes.forEach(home => { %>
        <div class="card">
            <img src="<%= home.imageUrl %>" alt="<%= home.name %>">
            <h2><%= home.name %></h2>
            <p>$<%= home.price %> / night</p>
            <p><%= home.location %></p>
            <a href="/home-detail/<%= home.id %>">View Details</a>
        </div>
    <% }); %>
</body>
</html>
```

---

## 7. Layouts with Partials

Partials are reusable template pieces (header, footer, nav).

### Your Partial Structure (from airbnb notes):
```
views/
├── partials/
│   ├── header.ejs
│   ├── footer.ejs
│   └── nav.ejs
├── store/
│   ├── home.ejs
│   └── home-detail.ejs
└── host/
    └── add-home.ejs
```

### header.ejs
```ejs
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title><%= pageTitle %></title>
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    <nav>
        <a href="/">Home</a>
        <a href="/host/home-list">My Homes</a>
    </nav>
```

### footer.ejs
```ejs
    <footer>
        <p>&copy; 2024 Airbnb Clone</p>
    </footer>
</body>
</html>
```

### Using Partials
```ejs
<!-- views/store/home.ejs -->
<%- include('partials/header') %>

<main>
    <h1><%= pageTitle %></h1>
    <% homes.forEach(home => { %>
        <p><%= home.name %></p>
    <% }); %>
</main>

<%- include('partials/footer') %>
```

---

## 8. EJS vs Static HTML

### Static HTML (no dynamic data)
```html
<h1>Beach House</h1>
<p>$150 / night</p>
```

### EJS Template (dynamic data)
```ejs
<h1><%= home.name %></h1>
<p>$<%= home.price %> / night</p>
```

---

## 9. Common Mistakes

### ❌ Wrong: Using = inside control flow
```ejs
<% if (homes.length = 0) { %>  <!-- WRONG: = instead of === -->
```

### ❌ Wrong: Forgetting to close braces
```ejs
<% homes.forEach(home => { %>
    <p><%= home.name %></p>
<!-- Missing %> -->
```

### ✅ Correct
```ejs
<% if (homes.length === 0) { %>
    <p>No homes found</p>
<% } %>

<% homes.forEach(home => { %>
    <p><%= home.name %></p>
<% }); %>
```

---

## 10. Data Flow with EJS

```
Browser Request
       │
       ▼
┌─────────────────┐
│   Router        │
│  /              │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Controller     │
│  getHomes()    │
│  - fetches DB  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Model         │
│  Home.fetchAll()│
│  returns array  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  res.render()   │
│  with data:     │
│  { homes: [...] }│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   EJS Template  │
│  home.ejs       │
│  processes <%= %>│
└────────┬────────┘
         │
         ▼
    Final HTML
         │
         ▼
    Browser Response
```

---

## 11. Accessing Nested Data

```javascript
// Controller passes:
res.render('home', {
    home: {
        name: 'Beach House',
        price: 150,
        owner: {
            name: 'Sachin',
            email: 'sachin@email.com'
        }
    }
});
```

```ejs
<!-- Simple property -->
<p><%= home.name %></p>                    <!-- Beach House -->

<!-- Nested property -->
<p>Owner: <%= home.owner.name %></p>       <!-- Sachin -->

<!-- Array in object -->
<p>Price: $<%= home.price %></p>           <!-- 150 -->
```

---

## Interview Questions

### Q: What is EJS and why use it?
EJS (Embedded JavaScript) is a template engine that lets you embed JavaScript code inside HTML. It allows you to render dynamic content from your backend (database data) into HTML pages.

### Q: What is the difference between <%= and <%- ?
<%= outputs escaped HTML (safe for user data, prevents XSS). <%- outputs raw unescaped HTML (use only for trusted HTML you control).

### Q: How do you pass data from controller to view?
Use res.render('template', { key: value }). The data object becomes available as variables in the EJS template.

### Q: What are partials?
Partials are reusable template fragments like header, footer, navigation. Instead of repeating HTML in every page, you include them: <%- include('partials/header') %>.
