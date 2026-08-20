# Express.js — Notes

## 1. What is Express?

Express is a **minimal web framework for Node.js** that makes it much easier to build servers and APIs.

Recall from the Node.js notes — you *can* build a server with the built-in `http` module alone, but it's verbose: you have to manually parse URLs, handle different methods, parse the request body, etc. Express wraps all of that in a clean, simple API.

```js
// Without Express (raw http module)
const http = require('http');
const server = http.createServer((req, res) => {
  if (req.url === '/api/users' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ message: 'All users' }));
  }
});

// With Express
app.get('/api/users', (req, res) => {
  res.json({ message: 'All users' });
});
```

Express handles routing, middleware, request parsing, and response helpers — so you focus on your app's logic instead of low-level plumbing.

---

## 2. Creating an Express Server

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World');
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**Line by line:**
- `require('express')` — loads the Express library.
- `express()` — calls Express as a function, which creates an **application object** (`app`). This object is the core of your server — you attach routes, middleware, and settings to it.
- `app.get('/', ...)` — defines what happens when someone visits `/` with a GET request.
- `app.listen(PORT, callback)` — starts the server, telling it to listen for incoming requests on that port. The callback runs once the server has successfully started.

---

## 3. Routes

A **route** defines how the app responds to a client request to a specific **URL path** + **HTTP method** combination.

```js
app.get('/api/users', (req, res) => {
  res.json({
    message: "All users"
  });
});
```

**Explaining every part:**
- `app` — the Express application instance.
- `.get(...)` — this method registers a route that only responds to **GET** requests. (There's also `.post()`, `.put()`, `.patch()`, `.delete()` — matching the HTTP methods.)
- `'/api/users'` — the **path**. This route only triggers when a client requests this exact URL.
- `(req, res) => { ... }` — the **route handler** (a callback function). Express automatically calls this function whenever a matching request comes in, and passes it two objects:
  - `req` — represents the incoming **request** (what the client sent).
  - `res` — represents the outgoing **response** (what you send back).
- `res.json({...})` — a helper method that:
  1. Converts the JavaScript object into a JSON string.
  2. Sets the `Content-Type` header to `application/json`.
  3. Sends it back to the client as the response body.
- `{ message: "All users" }` — the actual data being sent back to the client.

So in plain English: *"When someone sends a GET request to `/api/users`, run this function, which sends back a JSON object containing a message."*

You can register routes for other methods the same way:
```js
app.post('/api/users', (req, res) => { ... });     // create
app.put('/api/users/:id', (req, res) => { ... });   // full update
app.patch('/api/users/:id', (req, res) => { ... }); // partial update
app.delete('/api/users/:id', (req, res) => { ... });// delete
```

---

## 4. Route Parameters

Route parameters are **dynamic segments** in the URL path — used when part of the URL represents an ID or identifier that varies.

```js
app.get('/api/users/:id', (req, res) => {
  const userId = req.params.id;
  res.json({ message: `User with ID ${userId}` });
});
```

- `:id` in the path is a **placeholder**. If a client requests `/api/users/42`, Express matches it to this route and captures `"42"` as the value of `id`.
- `req.params` — an object containing all route parameters. So `req.params.id` would be `"42"` (note: always a **string**, even if it looks numeric — convert with `Number()` if needed).

You can have multiple parameters:
```js
app.get('/api/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});
```

---

## 5. Query Parameters

Query parameters are **key-value pairs added to the end of a URL** after a `?`, commonly used for filtering, searching, sorting, or pagination.

```
GET /api/users?role=admin&page=2
```

```js
app.get('/api/users', (req, res) => {
  const role = req.query.role;
  const page = req.query.page;
  res.json({ role, page });
});
```

- `req.query` — an object containing all query parameters as key-value pairs. Here, `req.query` would be `{ role: 'admin', page: '2' }`.
- Unlike route parameters, query parameters are **optional** by nature — the route still works even if they're missing (`req.query.role` would just be `undefined`).

**Route params vs Query params — when to use which:**
| Use case | Type | Example |
|---|---|---|
| Identifying a specific resource | Route param | `/users/42` |
| Filtering, sorting, optional options | Query param | `/users?sort=name&order=asc` |

---

## 6. Request Object (`req`)

The `req` object represents the **incoming HTTP request** and contains everything the client sent.

Commonly used properties/methods:

| Property | Description |
|---|---|
| `req.params` | Route parameters (e.g. `:id`) |
| `req.query` | Query string parameters |
| `req.body` | Data sent in the request body (needs middleware to parse — see below) |
| `req.headers` | All request headers |
| `req.method` | HTTP method used (`GET`, `POST`, etc.) |
| `req.url` | The requested URL path |
| `req.cookies` | Cookies sent by the client (needs `cookie-parser` middleware) |

Example using `req.body`:
```js
app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  res.json({ message: `Created user ${name}` });
});
```
This only works if you've enabled Express's built-in JSON body parser first:
```js
app.use(express.json());
```

---

## 7. Response Object (`res`)

The `res` object represents the **outgoing HTTP response** — what you send back to the client. It has helper methods to make responding easy.

| Method | Description |
|---|---|
| `res.send(data)` | Sends a response (string, HTML, buffer, or object) |
| `res.json(data)` | Sends a JSON response (sets `Content-Type` automatically) |
| `res.status(code)` | Sets the HTTP status code (chainable) |
| `res.redirect(url)` | Redirects the client to another URL |
| `res.set(header, value)` | Sets a custom response header |
| `res.end()` | Ends the response without sending data |

**Chaining example:**
```js
res.status(404).json({ message: "User not found" });
```
Here, `res.status(404)` sets the status code and **returns the `res` object itself**, which is why you can immediately call `.json(...)` on it — this is called **method chaining**.

---

## 8. Middleware

Middleware functions are functions that run **during the request-response cycle**, before the final route handler. Each middleware has access to `req`, `res`, and a special third argument: `next`.

```js
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

- `app.use(...)` — registers middleware that runs on **every** incoming request (unless a specific path is given).
- `(req, res, next) => {...}` — the middleware function.
- `next()` — **crucial**: this tells Express "I'm done, move on to the next middleware or route handler." If you forget to call `next()` (and don't send a response), the request will **hang forever**.

Middleware executes **in the order it's defined**, forming a pipeline:
```
Request → Middleware 1 → Middleware 2 → Route Handler → Response
```

Common built-in/third-party middleware:
```js
app.use(express.json());        // parses JSON request bodies into req.body
app.use(express.urlencoded());  // parses form data
app.use(cors());                // enables cross-origin requests
app.use(morgan('dev'));         // logs requests to the console
```

---

## 9. Custom Middleware

You can write your own middleware for things like authentication, logging, or validation.

```js
function requireAuth(req, res, next) {
  const token = req.headers.authorization;

  if (!token) {
    return res.status(401).json({ message: "Unauthorized" });
  }

  // (token verification logic would go here)
  next();
}

app.get('/api/profile', requireAuth, (req, res) => {
  res.json({ message: "Welcome to your profile" });
});
```

**Explaining this:**
- `requireAuth` is a custom middleware function that checks for an `Authorization` header.
- If there's no token, it responds immediately with `401 Unauthorized` and **does not call `next()`** — this stops the request from reaching the route handler.
- If a token exists, it calls `next()`, allowing the request to continue to the actual route logic.
- Notice the route: `app.get('/api/profile', requireAuth, (req, res) => {...})` — you can pass middleware **directly into a specific route** by listing it before the final handler. Express runs them left to right.

---

## 10. Error-Handling Middleware

A special type of middleware used to catch and handle errors, defined with **4 parameters** instead of 3 (`err, req, res, next`) — this signature is how Express recognizes it as an error handler.

```js
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: "Something went wrong" });
});
```

- Must be defined **after all your routes** — Express runs it only when an error is passed along.
- Triggered when:
  - A synchronous error is thrown inside a route handler, OR
  - You explicitly call `next(err)` somewhere in your code.

Example of triggering it manually:
```js
app.get('/api/risky', (req, res, next) => {
  try {
    throw new Error("Something broke");
  } catch (err) {
    next(err); // passes the error to the error-handling middleware
  }
});
```

---

## 11. Routers

As an app grows, putting every route in one file becomes messy. `express.Router()` lets you group related routes into separate, modular files.

```js
// routes/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ message: "All users" });
});

router.get('/:id', (req, res) => {
  res.json({ message: `User ${req.params.id}` });
});

module.exports = router;
```

```js
// index.js (main app file)
const usersRouter = require('./routes/users');

app.use('/api/users', usersRouter);
```

**Explaining this:**
- `express.Router()` creates a **mini Express app** — it supports its own routes and middleware, but isn't a full server on its own.
- Inside `routes/users.js`, paths are defined **relative** to wherever the router gets mounted.
- `app.use('/api/users', usersRouter)` — mounts the router at the `/api/users` prefix. So a route defined as `router.get('/:id', ...)` actually becomes accessible at `/api/users/:id`.

This keeps route files organized by resource (`users.js`, `products.js`, `orders.js`, etc.) instead of one giant file.

---

## 12. Controllers

While routers define **what URL/method triggers what**, controllers define **what actually happens** — the logic itself. Separating these keeps route files clean and logic reusable/testable.

```js
// controllers/userController.js
const getAllUsers = (req, res) => {
  res.json({ message: "All users" });
};

const getUserById = (req, res) => {
  const { id } = req.params;
  res.json({ message: `User ${id}` });
};

module.exports = { getAllUsers, getUserById };
```

```js
// routes/users.js
const express = require('express');
const router = express.Router();
const { getAllUsers, getUserById } = require('../controllers/userController');

router.get('/', getAllUsers);
router.get('/:id', getUserById);

module.exports = router;
```

**Why this pattern matters:** it separates concerns —
- **Router** = "which URL maps to which function"
- **Controller** = "what that function actually does"

This mirrors a common backend architecture pattern (often extended further into **Routes → Controllers → Services → Models**).

---

## 13. REST API Creation

Putting it together — a simple REST API for a "users" resource, following REST conventions:

```js
const express = require('express');
const app = express();
app.use(express.json());

let users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" }
];

// GET all users
app.get('/api/users', (req, res) => {
  res.status(200).json(users);
});

// GET one user
app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === Number(req.params.id));
  if (!user) return res.status(404).json({ message: "User not found" });
  res.status(200).json(user);
});

// POST create a user
app.post('/api/users', (req, res) => {
  const newUser = { id: users.length + 1, name: req.body.name };
  users.push(newUser);
  res.status(201).json(newUser);
});

// PUT update a user
app.put('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === Number(req.params.id));
  if (!user) return res.status(404).json({ message: "User not found" });
  user.name = req.body.name;
  res.status(200).json(user);
});

// DELETE a user
app.delete('/api/users/:id', (req, res) => {
  users = users.filter(u => u.id !== Number(req.params.id));
  res.status(204).send();
});

app.listen(3000, () => console.log("Server running on port 3000"));
```

This follows the standard REST pattern from the earlier notes: resource-based URLs (`/api/users`), correct HTTP methods for each action, and appropriate status codes for each outcome.

---

## 14. Status Codes (in Express)

Set explicitly using `res.status(code)`, chained with the response method:

```js
res.status(200).json({ message: "OK" });          // success
res.status(201).json({ message: "Created" });      // resource created
res.status(204).send();                             // success, no body
res.status(400).json({ message: "Bad request" });   // client sent bad data
res.status(401).json({ message: "Unauthorized" });  // not authenticated
res.status(404).json({ message: "Not found" });     // resource doesn't exist
res.status(500).json({ message: "Server error" });  // something broke
```

If you never call `.status()`, Express defaults to **200 OK** for successful responses.

---

## 15. Sending JSON Responses

`res.json(data)` is the standard way to send data back from a REST API.

```js
app.get("/api/users", (req, res) => {
    res.json({
        message: "All users"
    });
});
```

**Breaking this down one final time, piece by piece:**
- `app.get("/api/users", ...)` — registers a handler that runs only for **GET** requests to `/api/users`.
- `(req, res) => { ... }` — the callback Express invokes with the request and response objects.
- `res.json({ message: "All users" })`:
  - Takes the JavaScript object `{ message: "All users" }`.
  - Serializes it into a JSON string: `'{"message":"All users"}'`.
  - Sets the response header `Content-Type: application/json`.
  - Sends this as the response body back to the client.
  - Also implicitly sends status `200 OK` (since none was set manually).

So the full behavior: *a client sends a GET request to `/api/users`, and the server replies with a JSON body containing `{ "message": "All users" }` and a 200 status.*

You'd typically combine this with a status code for clarity/best practice:
```js
res.status(200).json({ message: "All users" });
```

---

## Quick Recap

- **Express** = a framework on top of Node's `http` module that simplifies building servers/APIs
- **Routes** = path + method combinations mapped to handler functions
- **`req.params`** = route parameters (`/users/:id`) · **`req.query`** = query string (`?role=admin`)
- **`req`** = incoming request data · **`res`** = tools to send a response
- **Middleware** = functions that run in the request pipeline, calling `next()` to continue
- **Error-handling middleware** = special 4-argument middleware (`err, req, res, next`)
- **Routers** = modularize routes by resource · **Controllers** = hold the actual logic
- **REST API** = resource-based routes + correct methods + correct status codes
- **`res.json()`** = the standard way to send structured data back to the client
