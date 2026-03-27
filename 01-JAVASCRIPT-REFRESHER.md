# Part 01 - JavaScript Refresher
# From Your Code Files

## 1. Array Methods (from array.js, forloop.js)

### map() - Transform Every Element
```javascript
let arr = [1, 2, 3, 4, 5];
let newarr = arr.map(n => n + 1);
// newarr = [2, 3, 4, 5, 6]
```
**YOUR CODE:** `arr.map(function(n){ return n+=1; })`

---

### filter() - Keep Only What Passes
```javascript
let arr = [1, 2, 3, 4, 5, 6, 7, 8];
let evenarr = arr.filter(n => n % 2 === 0);
// evenarr = [2, 4, 6, 8]
```
**YOUR CODE:** `arr.filter((n) => { if(n % 2 === 0) return true; })`

---

### reduce() - Reduce to Single Value
```javascript
let numbers = [1, 2, 3, 4, 5];
let sum = numbers.reduce((accumulator, current) => accumulator + current, 0);
// sum = 15
```

---

### forEach() - Loop Without Return
```javascript
const numbers = [1, 2, 3, 4, 5];
numbers.forEach((num, index) => {
    console.log(`Index ${index}: ${num}`);
});
```

---

### find() - First Match
```javascript
let found = numbers.find(num => num > 3);
// found = 4 (first number greater than 3)
```

---

### some() / every() - Boolean Tests
```javascript
const hasEven = numbers.some(num => num % 2 === 0);
// true if ANY element passes

const allPositive = numbers.every(num => num > 0);
// true if ALL elements pass
```

---

## 2. Objects (from objectcreation.js)

### Creating Objects
```javascript
let obj = {
    Name: "Sachin",
    age: 25,
    "Relationship status": "Single",  // Key with space - must use quotes
    greet: function() {
        console.log("Hello World");
    }
};

// Access
obj.Name;           // "Sachin"
obj["age"];          // 25
obj["Relationship status"];  // "Single"
obj.greet();         // "Hello World"
```

---

## 3. Closures (from splice_deep_dive.js)

### What is a Closure?
A function that "remembers" variables from its outer scope even after that scope has finished executing.

```javascript
function solve(number) {
    return function(number) {
        return number * number * number * number;
    }
}

let ans = solve(5);  // ans is now a function
console.log(ans(5));  // 625 (5^4)
```

### Real-World Closure Pattern (Like Your Code)
```javascript
// Function returns another function
function createGreeter(greeting) {
    return function(name) {
        console.log(greeting + ", " + name);
    }
}

const sayHello = createGreeter("Hello");
sayHello("Sachin");  // "Hello, Sachin"
sayHello("Rahul");   // "Hello, Rahul"
```

---

## 4. Destructuring

### Object Destructuring
```javascript
const { name, age } = { name: "Sachin", age: 25 };
// name = "Sachin", age = 25

// YOUR CODE - in hostController:
const { name, imageUrl, price, location } = req.body;
```

### Array Destructuring
```javascript
const [first, second] = [1, 2, 3];
// first = 1, second = 2
```

---

## 5. Spread & Rest Operators

### Spread (Expand)
```javascript
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];
// arr2 = [1, 2, 3, 4, 5]

const obj1 = { name: "Sachin" };
const obj2 = { ...obj1, age: 25 };
// obj2 = { name: "Sachin", age: 25 }
```

### Rest (Collect)
```javascript
function sum(...numbers) {
    return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4);  // 10
```

---

## 6. Template Literals

```javascript
const name = "Sachin";
const age = 25;
const message = `My name is ${name} and I am ${age} years old.`;
// "My name is Sachin and I am 25 years old."
```

---

## 7. This Keyword

### In Regular Functions
```javascript
const obj = {
    name: "Sachin",
    greet: function() {
        console.log(this.name);  // "Sachin"
    }
};
```

### Arrow Functions Don't Have Their Own 'this'
```javascript
const obj = {
    name: "Sachin",
    // Arrow function - 'this' refers to outer context (global)
    greet: () => {
        console.log(this.name);  // undefined (in Node) or window.name
    }
};
```

---

## 8. Classes (ES6)

```javascript
class Home {
    constructor(name, price, location) {
        this.name = name;
        this.price = price;
        this.location = location;
    }

    greet() {
        return `Welcome to ${this.name}!`;
    }
}

const home = new Home("Beach House", 150, "Miami");
home.greet();  // "Welcome to Beach House!"
```

### Your MongoDB Model Pattern:
```javascript
// models/home.js
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
};
```

---

## 9. Error Handling

### Try-Catch
```javascript
try {
    const data = JSON.parse(jsonString);
} catch (error) {
    console.log(error.message);
}
```

### Throwing Custom Errors
```javascript
class ValidationError extends Error {
    constructor(message) {
        super(message);
        this.name = "ValidationError";
    }
}

function checkAge(age) {
    if (age < 18) {
        throw new ValidationError("Age must be at least 18");
    }
    return "You can vote";
}
```

---

## 10. ES6 Modules Summary

```javascript
// Named Export
exports.getHomes = async (req, res) => { };

// Named Import
const { getHomes } = require('./controller');

// Default Export
module.exports = class Home { };

// Default Import
const Home = require('./home');
```

---

## Practice Questions

1. What does `[1,2,3].map(x => x * 2)` return?
2. How is a closure different from a normal function?
3. What is the difference between `forEach` and `map`?
4. Why do we use destructuring?
5. What does the spread operator `...` do?
6. What is the difference between `==` and `===`?
7. How does `this` work in arrow functions vs regular functions?
