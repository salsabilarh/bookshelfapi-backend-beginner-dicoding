# 📚 Bookshelf API — RESTful Book Management Service

![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Hapi.js](https://img.shields.io/badge/Hapi.js-FF6600?style=for-the-badge)
![nanoid](https://img.shields.io/badge/nanoid-ID_Generation-blueviolet?style=for-the-badge)
![ESLint](https://img.shields.io/badge/ESLint-Dicoding_Style-4B32C3?style=for-the-badge&logo=eslint&logoColor=white)
![Dicoding](https://img.shields.io/badge/Dicoding-Submission-4285F4?style=for-the-badge)
![Score](https://img.shields.io/badge/Score-5%2F5_%E2%98%85-FFD700?style=for-the-badge)
![Learning Path](https://img.shields.io/badge/Backend_JavaScript-Learning_Path_Stage_4-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)

> ⭐ **Final submission for "Belajar Back-End Pemula dengan JavaScript"** — Stage 3 of the *Backend JavaScript Learning Path* at Dicoding.
> A fully functional RESTful API for managing a personal book collection, built with Hapi.js and in-memory storage. **Perfect score 5/5** from the Dicoding review team.

---

## 📖 Table of Contents

- [Executive Summary](#executive-summary)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [API Endpoints](#api-endpoints)
- [Request & Response Reference](#request--response-reference)
- [Architecture & Technical Approach](#architecture--technical-approach)
- [Installation & Local Usage](#installation--local-usage)
- [What I Learned](#what-i-learned)
- [License](#license)

---

## Executive Summary

Bookshelf API is a production-structured RESTful service that exposes full **CRUD operations** for a book collection over HTTP. Every endpoint is designed around a consistent response contract, multi-parameter query filtering, and layered input validation — all without a database or ORM, demonstrating that clean API design is about architecture and discipline, not tooling.

This project is the first point in the learning path where HTTP, routing, request lifecycle, and response design come together as a unified system rather than isolated concepts.

---

## Key Features

- **Full CRUD over HTTP** — Create, Read (collection + single), Update, and Delete operations across five dedicated endpoints
- **Derived State Computation** — The `finished` field is never stored directly; it is computed automatically as `pageCount === readPage` on every write, eliminating the possibility of an inconsistent finished/readPage state
- **Multi-Parameter Query Filtering** — `GET /books` supports independent, composable filters by `name` (case-insensitive partial match), `reading` status, and `finished` status
- **Layered Input Validation** — Two distinct validation rules enforced before any data is written: presence of `name` (required field) and logical consistency of `readPage ≤ pageCount`
- **Collision-Resistant ID Generation** — `nanoid(16)` generates a cryptographically random 16-character URL-safe identifier for every new resource, with a post-insert existence check as an integrity guard
- **Immutable Timestamps** — `insertedAt` is set once at creation and preserved through all subsequent updates; `updatedAt` is refreshed on every PUT operation
- **CORS Enabled** — `origin: ['*']` configured at the server level, making the API immediately consumable from any frontend client without preflight issues
- **Enforced Code Style** — ESLint with `eslint-config-dicodingacademy` applied across the entire codebase, ensuring consistent formatting and catching common runtime errors before execution

---

## Tech Stack

```
Runtime         Node.js (CommonJS modules)
Framework       @hapi/hapi v21
ID Generation   nanoid v3 (URL-safe, 16-char)
Linting         ESLint + eslint-config-dicodingacademy
Dev Server      nodemon (file-watch auto-restart)
Storage         In-memory array (no external database)
Transport       HTTP/1.1 — JSON request & response bodies
```

---

## API Endpoints

| Method | Endpoint | Description | Success Code |
|--------|----------|-------------|:---:|
| `POST` | `/books` | Add a new book to the collection | `201` |
| `GET` | `/books` | Retrieve all books (supports query filters) | `200` |
| `GET` | `/books/{id}` | Retrieve a single book by ID | `200` |
| `PUT` | `/books/{id}` | Update all fields of an existing book | `200` |
| `DELETE` | `/books/{id}` | Remove a book from the collection | `200` |

### Query Parameters — `GET /books`

| Parameter | Type | Value | Description |
|-----------|------|-------|-------------|
| `name` | `string` | Any partial string | Case-insensitive title search |
| `reading` | `string` | `1` / `0` | Filter by currently-reading status |
| `finished` | `string` | `1` / `0` | Filter by finished-reading status |

---

## Request & Response Reference

### `POST /books` — Add a Book

**Request body:**
```json
{
  "name": "Clean Architecture",
  "year": 2017,
  "author": "Robert C. Martin",
  "summary": "A guide to software structure and design.",
  "publisher": "Prentice Hall",
  "pageCount": 432,
  "readPage": 100,
  "reading": true
}
```

**Success `201`:**
```json
{
  "status": "success",
  "message": "Buku berhasil ditambahkan",
  "data": {
    "bookId": "Qbax5Oy7L8WKf74l"
  }
}
```

**Failure `400` — missing name:**
```json
{
  "status": "fail",
  "message": "Gagal menambahkan buku. Mohon isi nama buku"
}
```

**Failure `400` — readPage > pageCount:**
```json
{
  "status": "fail",
  "message": "Gagal menambahkan buku. readPage tidak boleh lebih besar dari pageCount"
}
```

---

### `GET /books` — Get All Books

**Success `200`:**
```json
{
  "status": "success",
  "data": {
    "books": [
      {
        "id": "Qbax5Oy7L8WKf74l",
        "name": "Clean Architecture",
        "publisher": "Prentice Hall"
      }
    ]
  }
}
```

> Only `id`, `name`, and `publisher` are returned in the collection view. Full detail is available via `GET /books/{id}`.

---

### `GET /books/{id}` — Get Book by ID

**Success `200`:** returns the complete book object including all fields.

**Failure `404`:**
```json
{
  "status": "fail",
  "message": "Buku tidak ditemukan"
}
```

---

### `PUT /books/{id}` — Update a Book

Same request body structure as `POST`. All fields are replaced; `insertedAt` is preserved, `updatedAt` is refreshed.

**Success `200`:**
```json
{
  "status": "success",
  "message": "Buku berhasil diperbarui"
}
```

---

### `DELETE /books/{id}` — Delete a Book

**Success `200`:**
```json
{
  "status": "success",
  "message": "Buku berhasil dihapus"
}
```

**Failure `404`:**
```json
{
  "status": "fail",
  "message": "Buku gagal dihapus. Id tidak ditemukan"
}
```

---

## Architecture & Technical Approach

### Project Structure

```
bookshelfapi-backend-beginner-dicoding/
├── src/
│   ├── server.js     ← Hapi server initialization, CORS config, startup
│   ├── routes.js     ← Route definitions — maps HTTP method + path → handler
│   ├── handler.js    ← All business logic — validation, computation, response
│   └── books.js      ← Shared in-memory data store (exported array)
├── package.json
├── eslint.config.mjs
└── .eslintrc.json
```

### Request Lifecycle

```
HTTP Request
     │
     ▼
 server.js          Hapi server receives the request on port 9000
     │
     ▼
 routes.js          Matches method + path → dispatches to the correct handler
     │
     ▼
 handler.js         Validates input → mutates/reads books[] → builds response
     │
     ▼
 books.js           Shared in-memory array acting as the data store
     │
     ▼
HTTP Response       Consistent JSON envelope: { status, message?, data? }
```

### Separation of Concerns

The codebase is split across exactly four files, each with a single responsibility:

- **`server.js`** owns infrastructure — port, host, CORS policy, and the startup sequence. It has no awareness of what the routes do.
- **`routes.js`** owns routing — it maps HTTP verbs and path patterns to handler functions. It contains no business logic.
- **`handler.js`** owns business logic — validation, ID generation, state computation, and all response construction live here.
- **`books.js`** owns state — a single exported array that is the sole source of truth for all data in the process.

### The `finished` Field: Derived State, Never Stored Directly

```javascript
const finished = pageCount === readPage; // computed on every write
```

`finished` is never accepted from the request body. It is always derived at write time. This design decision means `finished` can never fall out of sync with `readPage` and `pageCount` — the invariant is enforced by the system, not by the caller.

---

## Installation & Local Usage

**Prerequisites:** Node.js v18+

```bash
# Clone the repository
git clone https://github.com/salsabilarh/bookshelfapi-backend-beginner-dicoding.git
cd bookshelfapi-backend-beginner-dicoding

# Install dependencies
npm install

# Start in development mode (auto-restart on file change)
npm run start-dev

# Start in production mode
npm start
```

The server starts at: **`http://localhost:9000`**

**Run linting:**
```bash
npm run lint
```

**Test the API with curl:**
```bash
# Add a book
curl -X POST http://localhost:9000/books \
  -H "Content-Type: application/json" \
  -d '{"name":"Clean Code","year":2008,"author":"Robert C. Martin","summary":"Craftsmanship in code.","publisher":"Prentice Hall","pageCount":431,"readPage":100,"reading":true}'

# Get all books
curl http://localhost:9000/books

# Filter by name (partial, case-insensitive)
curl "http://localhost:9000/books?name=clean"

# Filter by reading status
curl "http://localhost:9000/books?reading=1"
```

Or import the collection into **Postman** and test all five endpoints interactively.

---

## What I Learned

### 1. HTTP Is a Contract, Not Just a Transport

Every design decision in this project — status codes, response envelope structure, field naming — is a commitment to the API's callers. A `201` on create and a `404` on missing resource are not arbitrary numbers; they are a machine-readable communication layer that any client can act on without reading documentation.

### 2. Derived State Prevents an Entire Class of Bugs

Computing `finished = pageCount === readPage` at write time instead of accepting it from the client means that a caller can never submit `{ readPage: 200, pageCount: 100, finished: false }` and have the API believe it. The invariant is owned by the server. This is the same principle that drives computed fields in databases and derived state in frontend frameworks.

### 3. Validation Is the First Line of Defense

Validating `name` presence and `readPage ≤ pageCount` before touching the data store means that the data store never contains invalid records. This ordering — validate first, write second — is the foundation of data integrity in any system at any scale.

### 4. In-Memory Storage Teaches What a Database Actually Solves

Building a working API on a plain JavaScript array makes the limitations viscerally clear: no persistence across restarts, no concurrent-write safety, no indexing. Every feature a database provides exists to solve a problem this implementation exposes. Understanding the problem first makes the solution meaningful.

---

## License

This project was built as a Dicoding course submission and is open for educational use.

---

*Submission: Belajar Back-End Pemula dengan JavaScript — Stage 3 of Backend JavaScript Learning Path, Dicoding 2026*
*Reviewed Score: ⭐ 5/5 — Perfect*
