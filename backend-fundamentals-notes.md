# Backend Fundamentals — Notes

## 1. What is a Backend?

The backend is the part of an application that runs on a server, not on the user's device. It handles:
- Business logic (the actual rules and processing of your app)
- Data storage and retrieval (databases)
- Authentication and authorization
- Communicating with other services

The user never sees the backend directly — they interact with a **frontend** (website, mobile app), which talks to the backend behind the scenes.

**Analogy:** Think of a restaurant. The frontend is the dining area and menu (what the customer sees and interacts with). The backend is the kitchen — where the actual food (data/logic) is prepared, out of the customer's sight.

---

## 2. Client vs Server

| | Client | Server |
|---|---|---|
| **What it is** | The device/app requesting something | The machine/program providing something |
| **Examples** | Browser, mobile app, Postman | Node.js app, Django app, database server |
| **Role** | Initiates requests | Listens for requests and responds |

- **Client** = anything that *sends a request* (e.g., a browser loading a webpage).
- **Server** = anything that *receives a request and sends back a response*.
- A single machine can be both — e.g., a server can act as a client when it calls another API.

---

## 3. Request and Response

This is the core communication pattern of the web:

1. **Client sends a Request** → "Hey server, give me the list of users."
2. **Server processes it** → looks up data, runs logic.
3. **Server sends a Response** → "Here's the list of users" (or an error if something went wrong).

Every request typically has:
- A **method** (what action to perform — see HTTP methods below)
- A **URL** (where the resource lives)
- **Headers** (metadata)
- Sometimes a **body** (data being sent)

Every response typically has:
- A **status code** (success/failure indicator)
- **Headers**
- Usually a **body** (the actual data, often JSON)

---

## 4. HTTP (HyperText Transfer Protocol)

HTTP is the **protocol (set of rules)** that clients and servers use to communicate over the web.

Key characteristics:
- **Stateless**: each request is independent; the server doesn't automatically remember previous requests (this is why things like cookies/sessions/tokens exist — to fake "memory").
- **Text-based** (though data is often binary/encoded underneath).
- Runs on top of **TCP/IP**.
- **HTTPS** = HTTP + encryption (via TLS/SSL) — same protocol, but secure.

---

## 5. HTTP Methods

Each method signals the *intent* of the request:

| Method | Purpose | Example |
|---|---|---|
| `GET` | Retrieve data (read-only, no side effects) | Get list of products |
| `POST` | Create a new resource | Add a new user |
| `PUT` | Replace/update an entire resource | Replace a user's full profile |
| `PATCH` | Partially update a resource | Update just a user's email |
| `DELETE` | Remove a resource | Delete a user |

**Rule of thumb:**
- `GET` and `DELETE` usually don't send a body.
- `POST`, `PUT`, `PATCH` usually send a body (the data to create/update).
- `GET` should never change data on the server (it should be "safe").

---

## 6. HTTP Status Codes

Status codes tell the client what happened, grouped by first digit:

| Range | Meaning | Common Examples |
|---|---|---|
| `1xx` | Informational | 100 Continue |
| `2xx` | Success | 200 OK, 201 Created, 204 No Content |
| `3xx` | Redirection | 301 Moved Permanently, 304 Not Modified |
| `4xx` | Client error (you messed up) | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| `5xx` | Server error (server messed up) | 500 Internal Server Error, 503 Service Unavailable |

**Most commonly used in practice:**
- `200 OK` — success
- `201 Created` — resource created (after a POST)
- `204 No Content` — success, but nothing to return (often after DELETE)
- `400 Bad Request` — invalid input from client
- `401 Unauthorized` — not logged in / missing credentials
- `403 Forbidden` — logged in but not allowed
- `404 Not Found` — resource doesn't exist
- `500 Internal Server Error` — something broke on the server

---

## 7. Headers

Headers are **key-value metadata** sent with requests and responses — extra information that isn't the main data itself.

Common examples:
- `Content-Type: application/json` → tells the receiver the body format
- `Authorization: Bearer <token>` → sends auth credentials
- `Accept: application/json` → tells the server what format the client wants back
- `Cache-Control` → controls caching behavior
- `User-Agent` → identifies the client (browser/app)

Think of headers as the "envelope info" while the body is the "letter inside."

---

## 8. JSON (JavaScript Object Notation)

The most common data format used to exchange data between client and server.

```json
{
  "id": 1,
  "name": "Aayush",
  "isActive": true,
  "roles": ["admin", "editor"]
}
```

Why JSON is popular:
- Lightweight and human-readable
- Maps naturally to objects/dictionaries in most languages
- Language-independent (works with JS, Python, Java, etc.)

It's the default body format for most REST APIs (paired with `Content-Type: application/json`).

---

## 9. REST API (Representational State Transfer)

REST is an **architectural style** for designing APIs, built around resources and standard HTTP methods.

Core principles:
- **Resources** are represented by URLs (e.g., `/users`, `/products/5`)
- Standard HTTP methods define actions (`GET`, `POST`, `PUT`, `DELETE`, etc.)
- **Stateless** — each request contains everything the server needs
- Responses typically use JSON
- Uses status codes properly to indicate outcomes

**Example REST design for a "users" resource:**

| Action | Method + URL |
|---|---|
| Get all users | `GET /users` |
| Get one user | `GET /users/5` |
| Create a user | `POST /users` |
| Update a user fully | `PUT /users/5` |
| Update part of a user | `PATCH /users/5` |
| Delete a user | `DELETE /users/5` |

---

## 10. API Endpoints

An **endpoint** is a specific URL where an API can be accessed — a combination of a **path** and an **HTTP method**.

Example:
```
GET  /api/products        → list all products
GET  /api/products/42     → get product with id 42
POST /api/products        → create a new product
```

Each endpoint usually maps to a specific piece of backend logic (a "handler" or "controller function") that processes the request and returns a response.

---

## 11. Middleware

Middleware is code that runs **between** the incoming request and the final response — a processing step in the pipeline.

Common uses:
- **Authentication** — check if the user is logged in before proceeding
- **Logging** — record details about each request
- **Validation** — check that incoming data is well-formed
- **Error handling** — catch errors and format a proper response
- **CORS handling** — control which origins can access the API

**Flow example:**
```
Request → Logger Middleware → Auth Middleware → Route Handler → Response
```

Each middleware can:
- Let the request continue to the next step (`next()`)
- Stop the request and send a response immediately (e.g., reject if not authenticated)

---

## Quick Recap

- **Backend** = server-side logic + data handling
- **Client/Server** = requester and provider
- **Request/Response** = the basic communication cycle
- **HTTP** = the protocol/rules for that communication
- **Methods** = the *intent* of a request (GET/POST/PUT/PATCH/DELETE)
- **Status codes** = the *outcome* of a request
- **Headers** = metadata around the request/response
- **JSON** = the common data format
- **REST API** = a design style combining resources + methods + status codes
- **Endpoints** = specific URL + method combinations
- **Middleware** = reusable steps in the request-handling pipeline
