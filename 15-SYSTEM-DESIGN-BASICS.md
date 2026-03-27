# Part 15 - System Design Basics

## 1. How to Design a System

### Step 1: Understand the Requirements
- What does the app do?
- Who are the users?
- What data needs to be stored?

### Step 2: API Design
- What endpoints do you need?
- GET, POST, PUT, DELETE for each resource
- What data comes in and goes out?

### Step 3: Database Design
- What data to store?
- Collections/documents or tables/rows?
- Relationships between data?

### Step 4: Architecture
- MVC pattern for web apps
- Separate concerns
- Scalability considerations

---

## 2. Designing Your Airbnb Clone

### Requirements
- Users can browse homes
- Hosts can list homes
- Users can favorite homes
- View home details

### API Endpoints
```
GET  /           → Home page with all homes
GET  /home/:id  → Home details
GET  /host/add-home  → Add home form
POST /host/add-home  → Submit new home
GET  /favourites     → User favorites
POST /favourites     → Add to favorites
```

### Database (MongoDB)
```
Collection: homes
{
  _id: ObjectId,
  name: String,
  price: Number,
  location: String,
  imageUrl: String
}

Collection: favourites
{
  _id: ObjectId,
  homeId: String
}
```

### Architecture
```
Browser -> Express Server -> MongoDB Atlas
              |
              v
         Controllers
              |
              v
           Models
```

---

## 3. Scalability Basics

### Vertical Scaling
- Add more RAM/CPU to existing server
- Simple but limited

### Horizontal Scaling
- Add more servers
- Need load balancer

### Caching
- Store frequently accessed data in memory
- Redis for caching

### Database Indexing
- Index frequently queried fields
- Faster queries

---

## 4. Common System Design Questions

### Q: How would you design Instagram?
- Users, Posts, Comments, Likes
- User posts photo
- Others can like and comment
- Feed shows posts from followed users

### Q: How would you design Twitter?
- Users, Tweets, Followers, Following
- User posts tweet
- Others can retweet, like, reply
- Timeline shows tweets from followed users

### Q: How would you design a chat app?
- Users, Messages, Conversations
- Real-time messaging (WebSockets)
- Online/offline status
- Message notifications

---

## 5. Your Project at Scale

### Current (Small Scale)
```
Browser -> Express Server -> MongoDB Atlas
              |
         Single Server
```

### If Millions of Users
```
                    [CDN]
                       |
[Browser] -> [Load Balancer] -> [Server 1]
                              |           [Server 2]
                              |           [Server 3]
                              |
                         [Redis Cache]
                              |
                         [MongoDB Atlas]
                              |
                         [Sharded Cluster]
```

---

## 6. Interview Tips

### Explain Your Thinking
1. Ask clarifying questions
2. Think out loud
3. Explain trade-offs
4. Start simple, then optimize

### Key Points to Mention
- Caching for performance
- Database indexing
- Horizontal vs vertical scaling
- API design
- Error handling
- Security considerations

---

## Quick Answers for System Design

### How do you handle high traffic?
Add more servers, use caching (Redis), optimize database queries with indexes, use CDN for static files.

### How do you secure your API?
Use authentication (JWT/sessions), validate input, use HTTPS, secure headers with Helmet, rate limiting.

### How do you design a URL shortener?
Store original URL and short code in database. When short URL accessed, redirect to original URL.

### How do you design a rate limiter?
Track requests per IP/user in Redis/memory. Allow max N requests per time window.
