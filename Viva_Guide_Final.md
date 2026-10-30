# PrepVault — Final Evaluation Master Viva Guide 🎓

Welcome to the definitive Viva Guide for your **PrepVault** final evaluation. This document bridges the entire course syllabus (Lectures 1–60) directly to your project codebase. It is designed to help you explain *why* and *how* every part of your project works, pointing your evaluators directly to the code files.

---

## 🗺️ Project-to-Syllabus Mapping Matrix

| Lecture Block | Syllabus Topic | Where is it in PrepVault? (File References) |
|---|---|---|
| **1-4** | Client-Server Architecture & Request Handling | [prepvault-frontend/src/api/index.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-frontend/src/api/index.js), [prepvault-backend/server.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/server.js) |
| **5-8** | Node.js Setup, `fs` module, Environment Setup | [prepvault-backend/.env](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/.env), [loggerMiddleware.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/middleware/loggerMiddleware.js) |
| **9-12** | Node.js Architecture (Event Loop, Single-Threaded) | Explained below (Performance, Pros/Cons) |
| **13-16** | Core HTTP, NPM Packages, require/import | [prepvault-backend/package.json](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/package.json), [server.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/server.js) |
| **17-20** | Frameworks: Express-JS Introduction | [server.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/server.js) |
| **21-24** | Static Files, Express Routing, Response Methods | [routes/experienceRoutes.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/routes/experienceRoutes.js), [server.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/server.js) |
| **25-28** | Middleware (Lifecycle, Types, Body Parser) | [middleware/authMiddleware.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/middleware/authMiddleware.js), [middleware/errorMiddleware.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/middleware/errorMiddleware.js) |
| **29-32** | SSR vs CSR, EJS Template Engine | [views/report.ejs](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/views/report.ejs), [server.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/server.js) |
| **33-36** | SQL vs NoSQL, MongoDB Connection, Mongoose ODM | [models/User.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/models/User.js), [models/Experience.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/models/Experience.js) |
| **37-40** | Session Management vs Stateless Cookies/JWT | [middleware/authMiddleware.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/middleware/authMiddleware.js) |
| **41-44** | Authentication: Bcrypt & JWT | [controllers/authController.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/controllers/authController.js) |
| **45-48** | Full-Duplex Comms: Socket.IO | [socket.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/socket.js), [Explore.jsx](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-frontend/src/pages/Explore.jsx) |
| **49-52** | Relational DB (PostgreSQL) & Prisma ORM Migration | Schema Design & Migration Code detailed below |
| **53-56** | Multipart Form-Data, Multer, Cloudinary Storage | [routes/experienceRoutes.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/routes/experienceRoutes.js) |
| **57-60** | Unit/Smoke Testing & Deployment | [testing/api_smoke_test.js](file:///d:/01_Projects/PrepVault/PrepVault/testing/api_smoke_test.js), Build Configurations |

---

## 🛠️ Topic-by-Topic Viva Guide (Lectures 1–60)

---

### 🌐 Lectures 1-4: Client-Server Architecture & Server Request Handling

#### 💡 Concept Summary
In a web application, the **Client** (Frontend) requests resources, and the **Server** (Backend) processes those requests and returns responses. Modern web apps separate these into a User Interface layer (React SPA) and an Application Program Interface layer (Express REST API).

#### 🏗️ PrepVault Implementation
*   **Client**: A React Single Page Application (SPA) run on port `5173`.
*   **Server**: Node.js + Express backend run on port `5000` connected to MongoDB.
*   **Vite Proxy**: Configured in `vite.config.js` to automatically redirect requests from `/api/*` to `http://localhost:5000/api/*`.
*   **Request Lifecycle**:
    1.  User clicks "Submit Experience" in `Explore.jsx`.
    2.  Axios in [api/index.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-frontend/src/api/index.js) sends a `POST` request to `/api/experiences`.
    3.  Express in [server.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/server.js) intercepts the request on port `5000`.
    4.  The request passes through parsing middlewares, validation, and is processed by `createExperience()` in `experienceController.js`.
    5.  The controller writes to MongoDB and sends a `201 Created` JSON payload back to React.

#### ❓ Viva Q&A
1.  **Q: Why do we separate the frontend and backend onto different ports?**
    *   **A:** It decouples the UI representation from the business logic. It allows us to scale them independently, deploy them to different services (e.g., Vercel for React, Render for Express), and serve other clients (like a mobile app) using the same backend API.
2.  **Q: Explain the request-response lifecycle when loading experiences in PrepVault.**
    *   **A:** The frontend triggers `getExperiences()` via Axios. The browser sends a `GET` request to `/api/experiences`. The Express server matches this path in `experienceRoutes.js`, executes `getAllExperiences()` controller, queries MongoDB using Mongoose `Experience.find({ flagged: { $ne: true } })`, and responds with a JSON array and HTTP code `200 OK`.
3.  **Q: What is the purpose of the Vite Proxy configuration?**
    *   **A:** During development, the React app runs on port `5173` and the backend on `5000`. Direct requests would trigger CORS errors (Cross-Origin Resource Sharing). The proxy intercepts `/api` calls on port `5173` and forwards them to port `5000` behind the scenes, avoiding CORS issues during local development.
4.  **Q: What HTTP methods are used in your REST API and what do they represent?**
    *   **A:** `GET` for retrieving data (`/api/experiences`), `POST` for creating resources (`/api/experiences` or `/api/auth/register`), `PUT` for updating a specific resource (`/api/experiences/:id`), and `DELETE` for removing a resource (`/api/experiences/:id`).
5.  **Q: How does the server determine that a request is successfully handled?**
    *   **A:** By sending an appropriate HTTP Status Code (e.g., `200 OK` for successful fetches, `201 Created` for resource creations, `400 Bad Request` for invalid input, and `500 Internal Server Error` for exceptions).

---

### 📂 Lectures 5-8: Node.js Environment & File Handling (`fs`)

#### 💡 Concept Summary
Node.js environment setup relies on configuration files (`package.json`, `.env`). The built-in `fs` (File System) module allows asynchronous reading and writing of files on the server storage without blocking runtime execution.

#### 🏗️ PrepVault Implementation
*   **Environment Variables**: Stored in `prepvault-backend/.env` (including `MONGO_URI`, `JWT_SECRET`, and `PORT`). Loaded in `server.js` using `require('dotenv').config()`.
*   **Asynchronous Logging**: Implemented in [loggerMiddleware.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/middleware/loggerMiddleware.js) using the native `fs/promises` module. It creates a logs folder (`logs/`) and logs request metadata to `logs/requests.log` with `fs.appendFile()`.
*   **Upload Dir Initialization**: Implemented in [experienceRoutes.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/routes/experienceRoutes.js) to guarantee the attachments uploads folder exists:
    ```javascript
    fs.mkdir(uploadsDir, { recursive: true }).catch(...)
    ```

#### ❓ Viva Q&A
1.  **Q: Why did you use `fs/promises` (async) instead of `fs` synchronous methods (like `writeFileSync`)?**
    *   **A:** Node.js runs on a single main thread. If we use synchronous file operations, the server freezes and cannot handle incoming requests while the disk is writing. `fs/promises` delegates disk operations to the Libuv thread pool, keeping the main thread free to handle network requests.
2.  **Q: Why do we exclude the `.env` file from git repository using `.gitignore`?**
    *   **A:** The `.env` file contains sensitive secrets like database connection strings (with credentials) and JWT signing keys. Committing these to a public or shared repository exposes the project to security breaches. We only commit `.env.example` as a template.
3.  **Q: Explain how your logger middleware writes to the log file dynamically.**
    *   **A:** Every HTTP request triggers `logger(req, res, next)`. It formats a log string with the current timestamp, HTTP method, and URL. It then calls `fs.appendFile(logPath, log)` to append this to `requests.log` asynchronously, then immediately triggers `next()` to pass execution to the route handler.
4.  **Q: What does `{ recursive: true }` do in `fs.mkdir`?**
    *   **A:** It ensures that if the nested parent directories do not exist, Node will create them step-by-step instead of throwing an error. It acts like the terminal command `mkdir -p`.
5.  **Q: How does `dotenv` package load environment variables into the application?**
    *   **A:** It reads the key-value pairs from the `.env` file and appends them to Node’s global runtime object `process.env`. These can then be accessed anywhere, e.g. `process.env.JWT_SECRET`.

---

### ⚡ Lectures 9-12: Node.js Architecture (Event Loop & Performance)

#### 💡 Concept Summary
Node.js uses an **Event-Driven, Non-Blocking I/O model** powered by the V8 JavaScript engine and Libuv. It uses a single thread to execute JavaScript, handing off heavy tasks (network, disk, crypto) to the operating system or worker threads.

#### 🏗️ PrepVault Implementation
*   Our backend relies heavily on non-blocking asynchronous operations: database queries (MongoDB/Mongoose `await mongoose.connect()`, `await doc.save()`), password hashing (`await bcrypt.hash()`), and file writing (`await fs.appendFile()`).
*   This ensures that while the database is saving a new experience, the server can still receive login requests or push real-time updates via WebSockets.

#### ❓ Viva Q&A
1.  **Q: If Node.js is single-threaded, how does it handle thousands of concurrent requests?**
    *   **A:** It uses the **Event Loop**. When a request involves I/O (e.g., querying MongoDB), Node delegates the task to the database engine and continues executing other code. Once the DB returns the data, an event is placed in the task queue, and the Event Loop pushes the callback onto the call stack when it is empty.
2.  **Q: What is the difference between CPU-bound and I/O-bound tasks? Which one is Node.js good for?**
    *   **A:** CPU-bound tasks require heavy mathematical calculations (e.g., video editing, machine learning). I/O-bound tasks spend time waiting for external systems (e.g., database queries, reading disk files, network requests). Node.js is excellent for **I/O-bound** tasks because of its asynchronous nature, but poor for CPU-bound tasks since they block the single execution thread.
3.  **Q: What is the Libuv library, and why is it important in Node.js?**
    *   **A:** Libuv is the C++ library that Node.js uses under the hood to handle asynchronous operations. It provides the Event Loop, the system thread pool (handling file systems, DNS, and cryptography), and abstracts OS-specific asynchronous behaviors (like epoll on Linux or IOCP on Windows).
4.  **Q: What is the "Callback Hell" and how does our codebase resolve it?**
    *   **A:** Callback Hell occurs when nesting multiple asynchronous callback functions inside each other, making code unreadable and hard to debug. In PrepVault, we use **Promises and async/await syntax**, which makes asynchronous code look and behave like synchronous code.
5.  **Q: What are the main limitations of Node.js?**
    *   **A:** Single-threaded execution makes it unsuitable for heavy computational work. It lacks compile-time type-safety (unless TypeScript is used), and has an ecosystem (NPM) where package vulnerability management requires constant attention.

---

### 📦 Lectures 13-16: HTTP Module, NPM, & Modules

#### 💡 Concept Summary
Node.js offers a low-level native `http` module. Modern frameworks wrap this module to simplify development. Dependency management is handled by NPM (Node Package Manager) through `package.json`, which tracks scripts and package versions. Node.js supports CommonJS (`require`) and ES Modules (`import/export`).

#### 🏗️ PrepVault Implementation
*   **HTTP Server Wrapper**: We use the native `http` module in [server.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/server.js) to wrap our Express app so Socket.IO can listen on the same server instance:
    ```javascript
    const http = require('http')
    const server = http.createServer(app)
    ```
*   **Dependency Management**: Backend packages (`express`, `mongoose`, `jsonwebtoken`, `bcryptjs`, `socket.io`, `ejs`, `multer`) are tracked in [prepvault-backend/package.json](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/package.json).
*   **Module Systems**:
    *   **Backend**: Uses **CommonJS** (`const module = require('path')` / `module.exports`).
    *   **Frontend**: Uses **ES Modules** (`import React from 'react'` / `export default Component`).

#### ❓ Viva Q&A
1.  **Q: Why did you create an HTTP server using `http.createServer(app)` instead of directly running `app.listen()`?**
    *   **A:** Socket.IO requires direct access to an HTTP server instance to handle the initial WebSocket handshake protocol. `app.listen()` in Express creates an HTTP server implicitly, but does not expose the instance directly. By creating it explicitly, we can attach the Socket.IO server to the exact same HTTP instance.
2.  **Q: What is the difference between `package.json` and `package-lock.json`?**
    *   **A:** `package.json` lists the direct project dependencies with semantic versioning ranges (e.g. `^4.0.0`). `package-lock.json` locks down the exact, nested dependency tree with precise versions and integrity hashes, ensuring that every developer and server installs the identical dependency versions.
3.  **Q: Compare CommonJS (`require`) and ES Modules (`import/export`).**
    *   **A:** CommonJS is synchronous, loads modules at runtime, and uses `module.exports` and `require()`. ES Modules (ESM) are static (analyzable at compile time), support tree-shaking, load asynchronously, and use `import` and `export` statements.
4.  **Q: What are the scripts in `package.json` and how do we run them?**
    *   **A:** They are shortcut commands defined under the `"scripts"` key. In our backend, `"dev": "nodemon server.js"` runs the dev server with hot-reloading using `npm run dev`. `"start": "node server.js"` starts the server in production.
5.  **Q: What does the caret symbol (`^`) before dependency versions in `package.json` mean?**
    *   **A:** It tells NPM to install compatible newer minor/patch versions of that package (e.g., `^1.2.3` permits updates to `1.3.0` but restricts major upgrades like `2.0.0` which could introduce breaking changes).

---

### 🛤️ Lectures 17-20: Express Framework & MVC Architecture

#### 💡 Concept Summary
Express.js is a minimalist, unopinionated web framework for Node.js. It simplifies routing, middleware, and request/response manipulation. The **MVC (Model-View-Controller)** pattern organizes applications by separating data structure (Model), UI presentation (View), and business logic (Controller).

#### 🏗️ PrepVault Architecture
PrepVault follows a robust MVC/Layered setup:
```
[React App / EJS Templates] (View)
           │
           ▼ (HTTP Requests)
[Routes: authRoutes.js, experienceRoutes.js]
           │
           ▼ (Interpreted)
[Controllers: authController.js, experienceController.js]
           │
           ▼ (Schema & Database Operations)
[Models: User.js, Experience.js] (Model)
```
*   **Model**: [models/User.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/models/User.js) & [models/Experience.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/models/Experience.js) (defines DB schemas).
*   **View**: React Pages on frontend (Dynamic CSR) + EJS Admin files (Static SSR snapshot).
*   **Controller**: Handles logic (e.g., `controllers/authController.js` handles encryption, signups).
*   **Routes**: Declared in `/routes` folder; maps endpoints to controllers.

#### ❓ Viva Q&A
1.  **Q: Why do we use Express instead of writing raw Node.js HTTP servers?**
    *   **A:** Writing raw Node requires manual parsing of URLs, body streams, query parameters, cookies, CORS, and manually mapping handlers. Express abstracts this work with clean APIs, robust routing support, and modular middleware integration.
2.  **Q: How does the MVC pattern benefit the PrepVault codebase?**
    *   **A:** It isolates concerns. If the layout of the experience card changes, we modify the View (React). If we want to change password verification rules, we edit the Controller (`authController.js`). If we add a field like "flagged" to the database, we update the Model (`Experience.js`). This makes the codebase maintainable and scalable.
3.  **Q: What is a Router in Express, and why do we use `express.Router()`?**
    *   **A:** `express.Router()` creates isolated, modular route handlers. In PrepVault, instead of writing all routes in a single `server.js`, we group them logically (e.g. `experienceRoutes.js`, `authRoutes.js`) and mount them as middleware in `server.js` using `app.use('/api/experiences', experienceRoutes)`.
4.  **Q: How does routing match incoming requests in Express?**
    *   **A:** Express matches requests top-to-bottom based on the HTTP verb (GET, POST, etc.) and matching path pattern, executing the registered route handler function.
5.  **Q: Where does business logic reside in your project?**
    *   **A:** Inside the **Controllers** directory (e.g., `authController.js`, `experienceController.js`). This keeps route registration files clean and focused strictly on mounting middleware and endpoint paths.

---

### 📡 Lectures 21-24: Routing, Response Methods, & Static Files

#### 💡 Concept Summary
Express offers routing tools that support route parameters, query parameters, static file serving, and advanced response handlers (`res.download`, `res.json`).

#### 🏗️ PrepVault Implementation
*   **Static Assets**: Backend serves public static assets (like uploads) from the `/public` folder:
    ```javascript
    app.use(express.static('public'))
    ```
*   **Route Parameters**: Used to fetch or modify items by ID:
    ```javascript
    router.put('/:id', authMiddleware, updateExperience)
    ```
*   **Response Methods**:
    *   `res.json()`: Sends JSON payloads (used across controllers).
    *   `res.download()`: Used in [server.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/server.js) `/uploads/:filename` route to allow downloading user attachments securely:
        ```javascript
        res.download(filepath, filename, (err) => { ... })
        ```
*   **Directory Traversal Prevention**: Safely normalizes filepaths to block hackers:
    ```javascript
    const filepath = path.join(__dirname, 'public/uploads', filename)
    const normalizedPath = path.normalize(filepath)
    if (!normalizedPath.startsWith(path.join(__dirname, 'public/uploads'))) {
      return res.status(403).json(...)
    }
    ```

#### ❓ Viva Q&A
1.  **Q: What is the difference between Route Parameters and Query Parameters?**
    *   **A:** Route parameters are parts of the URL path used to identify specific resources (e.g. `/api/experiences/:id` accessed via `req.params.id`). Query parameters are key-value pairs appended at the end of a URL after `?` used for sorting, filtering, or options (e.g. `/api/experiences?includeFlagged=true` accessed via `req.query.includeFlagged`).
2.  **Q: Explain how `res.download()` differs from `res.sendFile()`.**
    *   **A:** `res.sendFile()` instructs the browser to open and display the file in the window (e.g. images, PDFs). `res.download()` sets the `Content-Disposition` header to `attachment`, forcing the browser to prompt the user to save the file locally.
3.  **Q: What is directory traversal and how does PrepVault prevent it?**
    *   **A:** Directory traversal occurs when attackers input relative path elements (like `../../windows/win.ini` or `/etc/passwd`) in route parameters to read arbitrary system files. We neutralize this by using `path.normalize(filepath)` and verifying that the final path strictly starts with our allowed upload folder path.
4.  **Q: How does `app.use(express.static('public'))` work?**
    *   **A:** It is a built-in middleware that serves static files (HTML, CSS, JS, images) from the specified root directory. If a file is requested at `http://localhost:5000/uploads/resume.pdf`, it looks inside the `./public/uploads/resume.pdf` folder and returns it.
5.  **Q: How are unhandled exceptions caught in routes?**
    *   **A:** Any error thrown inside our controllers or middlewares is forwarded automatically (or caught with try-catch and passed to `next(err)`) to our centralized error handler middleware defined in [errorMiddleware.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/middleware/errorMiddleware.js).

---

### 🛡️ Lectures 25-28: Middleware Lifecycle & Request Flow

#### 💡 Concept Summary
Middlewares are intermediate functions that intercept requests before they execute route handlers. They have access to the `req` (Request) object, `res` (Response) object, and the `next` function in the application's request-response cycle.

#### 🏗️ PrepVault Implementation
*   **Request Lifecycle in PrepVault**:
    ```
    Incoming Request
          │
          ▼
    1. loggerMiddleware.js (Logs method/URL to requests.log)
          │
          ▼
    2. express.json() (Parses incoming payload stream to req.body)
          │
          ▼
    3. authMiddleware.js (Checks JWT Authorization headers; attaches req.user)
          │
          ▼
    4. multer.array('attachments') (Only on file submits; processes form files)
          │
          ▼
    5. validationMiddleware.js (Checks fields logic validity)
          │
          ▼
    6. Route Controller (Saves to DB; emits Socket event)
          │
          ▼
    7. errorMiddleware.js (Executes only if an error is thrown)
    ```

#### ❓ Viva Q&A
1.  **Q: What are the four types of Express middlewares? How do they differ?**
    *   **A:**
        1.  **Application-level**: Bound to `app` (e.g. `app.use(express.json())`). Runs on all incoming requests.
        2.  **Router-level**: Bound to a router instance (e.g. `router.get('/', authMiddleware, handler)`).
        3.  **Error-handling**: Middleware defined with exactly 4 parameters `(err, req, res, next)`.
        4.  **Third-party**: Installed packages (e.g. `multer` for processing file uploads).
2.  **Q: What is the purpose of `express.json()` middleware? What happens if it's missing?**
    *   **A:** Node receives request payloads as raw stream buffers. `express.json()` reads this stream, parses it as a JSON object, and attaches it to `req.body`. Without it, `req.body` is `undefined`, and the server cannot read incoming payload fields.
3.  **Q: How does Express recognize an Error-Handling middleware?**
    *   **A:** Express checks the function signature. If it accepts **exactly 4 arguments** (`err, req, res, next`), Express registers it as an error-handling middleware. When `next(error)` is called, Express skips all standard routes and jumps straight to this handler.
4.  **Q: What is blocking vs non-blocking middleware? Give an example.**
    *   **A:** Non-blocking middleware returns promises or executes asynchronously without halting execution. For example, our logger calls `fs.appendFile` but does not block other requests. A blocking middleware would use synchronous executions (like `fs.writeFileSync`) which pauses the entire process.
5.  **Q: Why must `errorMiddleware` be mounted as the last middleware in `server.js`?**
    *   **A:** Express executes middlewares sequentially. If we mount the error middleware before defining the routes, routes throwing errors won't trigger the error handler because the route definitions appear *after* the error listener in the execution chain.

---

### 🖥️ Lectures 29-32: CSR vs SSR & EJS Template Engines

#### 💡 Concept Summary
*   **CSR (Client-Side Rendering)**: Server sends a bare skeleton HTML file. The browser downloads JS bundles (React) which dynamically renders the components and fetches data asynchronously.
*   **SSR (Server-Side Rendering)**: Server compiles templates (EJS, HBS) with actual database values on demand, rendering a fully populated HTML page to the browser.

#### 🏗️ PrepVault Implementation
*   **CSR Layout**: Main application (Explore, Profiles) is built on React SPA (Vite). Highly interactive, fast page transitions.
*   **SSR Dashboard**: For quick administrative views, we registered an EJS rendering engine in `server.js` and designed [views/report.ejs](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/views/report.ejs).
    *   Mounting Engine:
        ```javascript
        app.set('view engine', 'ejs')
        app.set('views', path.join(__dirname, 'views'))
        ```
    *   Controller handler (`server.js` line 67):
        ```javascript
        const exps = await Experience.find().sort({ createdAt: -1 }).limit(20).lean()
        res.render('report', { stats: { expCount, userCount, exps } })
        ```

#### ❓ Viva Q&A
1.  **Q: Compare SSR and CSR. When should you choose which?**
    *   **A:**
        *   **CSR (React)**: Fast interactions, rich animations, reduces server compute load since the browser renders. Poor SEO because initial HTML is empty. Best for interactive dashboards.
        *   **SSR (EJS/NextJS)**: Excellent SEO because crawler gets complete HTML, fast first-paint. High load on server, full page reload required on navigation. Best for public marketing landing pages and blog directories.
2.  **Q: How does EJS parse dynamic variables in our admin snapshot?**
    *   **A:** EJS evaluates custom scriptlet tags:
        *   `<%= stats.expCount %>` displays escaped HTML content.
        *   `<% for (let e of exps) { %>` runs raw javascript statements (loops, conditions) without outputting HTML directly.
3.  **Q: Why does the admin dashboard EJS view not trigger CORS issues?**
    *   **A:** Because EJS templates are fully rendered on the server side. The client makes a direct request to the server, and the server responds directly with complete HTML text. There are no asynchronous API cross-domain fetches involved.
4.  **Q: How did you configure Express to use EJS?**
    *   **A:** By calling `app.set('view engine', 'ejs')` to specify the template engine, and `app.set('views', path.join(__dirname, 'views'))` to define the folder directory containing our `.ejs` template files.
5.  **Q: Can a single backend application serve both a CSR React bundle and SSR EJS pages?**
    *   **A:** Yes, as demonstrated in PrepVault. Express serves static React static assets for frontend paths, and exposes dedicated endpoints like `/admin/report` which render and return EJS views.

---

### 🗄️ Lectures 33-36: Database Connections, MongoDB, & Mongoose ODM

#### 💡 Concept Summary
*   **SQL (Relational)**: Structured schemas, tables, relationships using foreign keys (e.g. PostgreSQL).
*   **NoSQL (Non-Relational)**: Schema-less documents, nested structures, collections (e.g. MongoDB).
*   **ODM (Object Document Mapper)**: Abstracts database queries into JavaScript objects and structures validations.

#### 🏗️ PrepVault Implementation
*   **Database**: MongoDB Atlas cloud cluster.
*   **Mongoose Schemas**:
    *   **User Schema** ([models/User.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/models/User.js)): Holds name, unique email, encrypted password hash, and a referenced array of favorite experiences:
        ```javascript
        savedFavorites: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Experience' }]
        ```
    *   **Experience Schema** ([models/Experience.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/models/Experience.js)): Holds experience metadata including complex nested arrays:
        *   `comments: [{ user: String, text: String, createdAt: Date }]` (Subdocument nesting)
        *   `attachments: [{ filename: String, url: String, mime: String }]`
        *   `checklist: [{ text: String, done: Boolean }]`

#### ❓ Viva Q&A
1.  **Q: Why did you choose MongoDB (NoSQL) over PostgreSQL (SQL) for PrepVault?**
    *   **A:** Interview experiences are unstructured documents. Different companies have different round checklists, attachment formats, and varying numbers of questions. MongoDB's flexible JSON-like documents allow us to nest variables (comments, checklists, questions) inside a single document easily, without writing complex JOIN queries across multiple relational tables.
2.  **Q: Explain the difference between schema referencing and nesting in Mongoose.**
    *   **A:**
        *   **Referencing (Normalized)**: Stores only the Object ID of another document (e.g. `User.savedFavorites` refers to `Experience`). Best for independent collections that scale.
        *   **Nesting (Embedded)**: Stores subdocuments directly inside the parent document (e.g. `Experience.comments`). Best for dependent data that is always fetched together with the parent.
3.  **Q: What is Mongoose validation and where did we use it?**
    *   **A:** Validation checks that data conforms to specific rules before saving to MongoDB. In [Experience.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/models/Experience.js), we set `required: true` on fields like `company` and `role`, and enforced limits using `min: 1, max: 5` on the `difficulty` field.
4.  **Q: How do you establish connection to MongoDB in Express?**
    *   **A:** By using `mongoose.connect(MONGO_URI)` inside `server.js` and awaiting it. If the connection fails, we catch the exception and shut down the process to prevent the server from running without database connectivity.
5.  **Q: What is a Mongoose `.lean()` method and why is it used?**
    *   **A:** By default, Mongoose queries return fat documents with internal methods (like `$save`, virtuals). Calling `.lean()` returns raw, plain JavaScript objects instead. This reduces memory usage and speeds up server execution significantly. We used it in `getAllExperiences()` to optimize API speeds.

---

### 🍪 Lectures 37-40: Session Management vs Cookies vs JWT

#### 💡 Concept Summary
Stateless authentication uses JWTs sent in HTTP headers, requiring no storage on the server. Stateful session management stores user sessions on the server (memory/Redis) and maps them to users via session ID cookies.

#### 🏗️ PrepVault Design Rationale
PrepVault implements **JWT-based stateless authentication**:
*   **Tokens** are stored on the client side in `localStorage` (`prepvault_token`).
*   **Authorization Interceptor**: Configured in [api/index.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-frontend/src/api/index.js) to attach the JWT automatically to every API request header:
    ```javascript
    api.interceptors.request.use((config) => {
      const token = localStorage.getItem(TOKEN_KEY)
      if (token) config.headers.Authorization = `Bearer ${token}`
      return config
    })
    ```

#### ❓ Viva Q&A
1.  **Q: Compare Stateful Sessions and Stateless JWTs.**
    *   **A:**
        *   **Sessions**: Server stores session objects. The browser gets a session ID in cookies. Good for immediate session revocation, but hard to scale because different backend instances need access to a shared store (like Redis).
        *   **JWT**: Server signs a token containing payload claims. The server does not store the token. Good for scalability, microservices, and mobile apps. However, tokens cannot be easily revoked before they expire.
2.  **Q: What is the `HttpOnly` flag on a cookie, and why is it important?**
    *   **A:** `HttpOnly` prevents client-side JavaScript from accessing the cookie. This makes it immune to Cross-Site Scripting (XSS) attacks where attackers try to steal authentication tokens using malicious scripts. (Storing JWT in `localStorage` makes it vulnerable to XSS).
3.  **Q: How would we implement session-based auth in our Express app?**
    *   **A:** We would install `express-session` and configure it with a session store (like MongoDB Session Store). We would mount it as middleware: `app.use(session({ secret: 'key', store: new MongoStore() }))`. On login, we would write `req.session.userId = user._id`.
4.  **Q: What is Cross-Site Request Forgery (CSRF)? How are cookies vs local storage affected?**
    *   **A:** CSRF occurs when a malicious site forces a browser to send a request to a site where the user is authenticated. Cookies are vulnerable to CSRF because browsers attach them automatically. Storing tokens in `localStorage` protects against CSRF because JavaScript must attach the token manually, but it is vulnerable to XSS.
5.  **Q: How does the server verify a stateless JWT if it doesn't store it?**
    *   **A:** It decodes the header and payload, hashes them using the secret key from `.env`, and compares the result to the signature part of the token. If they match, the server trusts the payload contents.

---

### 🔑 Lectures 41-44: Authentication (Bcrypt & JWT) & Passport.js

#### 💡 Concept Summary
*   **Bcrypt**: A password hashing algorithm designed to resist brute-force cracking using salting and cryptographic stretching.
*   **JWT (JSON Web Token)**: Compact token divided into: **Header** (metadata), **Payload** (claims), and **Signature** (verifies authenticity).
*   **Passport.js**: Authentication middleware providing strategies for local, OAuth, and social sign-ins.

#### 🏗️ PrepVault Implementation
*   **Password Salting & Hashing**: Handled in [authController.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/controllers/authController.js) registration:
    ```javascript
    const salt = await bcrypt.genSalt(10)
    const hash = await bcrypt.hash(password, salt)
    ```
*   **JWT Signature**: Signed with a 6-hour expiration limit:
    ```javascript
    jwt.sign({ id: user._id, email: user.email }, JWT_SECRET, { expiresIn: '6h' })
    ```
*   **Middleware Guard**: Verified in [authMiddleware.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/middleware/authMiddleware.js):
    ```javascript
    const payload = jwt.verify(token, JWT_SECRET)
    req.user = await User.findById(payload.id).select('-passwordHash')
    ```

#### ❓ Viva Q&A
1.  **Q: Explain Salting and Hashing. Why not use simple MD5 or SHA256?**
    *   **A:** Hashing is a one-way mathematical function. MD5 and SHA256 are fast, making them vulnerable to brute-force attacks and precomputed lookup lookup tables (Rainbow Tables). Bcrypt solves this by adding a "Salt" (random characters) and running the hashing function thousands of times (work factor), making brute-forcing computationally expensive.
2.  **Q: What is the structure of a JWT token?**
    *   **A:** A JWT is a string split into 3 parts separated by dots:
        1.  **Header**: Base64 encoded JSON specifying token type and hash algorithm (e.g. HS256).
        2.  **Payload**: Base64 encoded JSON containing claims (e.g., user ID, token expiration `exp`).
        3.  **Signature**: Generated by hashing the encoded Header, Payload, and a server-side secret key.
3.  **Q: How does `authMiddleware` protect routes like experience submission?**
    *   **A:** It checks for the `Authorization` header. If missing or invalid, it returns `401 Unauthorized`. If present, it validates the signature using `jwt.verify()`. On success, it fetches the user details (excluding password hash) and attaches them to `req.user`, then calls `next()`.
4.  **Q: What is Passport.js and how does it compare to our custom JWT authentication?**
    *   **A:** Passport.js is an authentication library that abstracts different authentication patterns into "strategies". While we wrote our JWT parsing logic manually, Passport.js could do this with `passport-jwt`. Passport.js is useful for scaling auth options (e.g. adding Google/GitHub Login).
5.  **Q: What is token expiration and rotation? Why are they used?**
    *   **A:** Expired tokens block hijackers from keeping access indefinitely. In production, we use short-lived access tokens (e.g. 15 minutes) and a long-lived refresh token stored in an `HttpOnly` cookie. Rotation replaces the refresh token with a new one on every request, neutralizing replayed tokens.

---

### 🔌 Lectures 45-48: Full-Duplex Real-Time Communication (Socket.IO)

#### 💡 Concept Summary
WebSockets provide a persistent, bi-directional, full-duplex communication channel over a single TCP connection, eliminating the overhead of HTTP request polling. Socket.IO is a library that abstracts WebSockets, providing features like rooms, namespaces, and auto-reconnections.

```
[Explore Page React] (Client)             [Express App Socket.IO] (Server)
         │                                               │
         ├─────────── Establish WebSocket ──────────────>│ (Persistent)
         │                                               │
         │                        [experienceController.js creates post]
         │                                               │
         │<────────── Emit 'new-experience' ─────────────┤ (Server Push)
         ▼ (React State updates instantly)               ▼
```

#### 🏗️ PrepVault Implementation
*   **Socket Hub**: Configured in [socket.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/socket.js):
    ```javascript
    ioInstance = new Server(server, { cors: { origin: 'http://localhost:5173' } })
    ```
*   **Emit on Post**: In `createExperience()` in `experienceController.js`:
    ```javascript
    socketHelper.getIO().emit('new-experience', doc)
    ```
*   **React Listener**: In [Explore.jsx](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-frontend/src/pages/Explore.jsx):
    ```javascript
    useEffect(() => {
      socket.on('new-experience', (newExp) => {
        setExperiences((prev) => [newExp, ...prev])
      })
      return () => socket.off('new-experience')
    }, [])
    ```

#### ❓ Viva Q&A
1.  **Q: What is full-duplex communication? Compare WebSockets and HTTP.**
    *   **A:** Full-duplex communication means both client and server can send data simultaneously. HTTP is half-duplex and client-initiated: the server cannot push updates without a client request. WebSockets establish a single persistent connection, allowing the server to push updates instantly.
2.  **Q: Why does the Socket.IO setup require a CORS origin configuration?**
    *   **A:** Browsers restrict WebSockets from connecting to different domains by default. Since our React frontend runs on port `5173` and the socket backend runs on `5000`, Socket.IO rejects the handshake unless we explicitly allow the origin `http://localhost:5173`.
3.  **Q: What is the purpose of cleaning up the socket listener in React's `useEffect`?**
    *   **A:** By calling `socket.off('new-experience')` in the cleanup function, we remove the event listener when the component unmounts. If we omit this, every time the user navigates back to the Explore page, a new duplicate listener is registered, causing memory leaks and redundant state updates.
4.  **Q: How does Socket.IO handle browser clients that do not support WebSockets?**
    *   **A:** It uses HTTP Long Polling first, then upgrades the connection to WebSockets if the client and network support it.
5.  **Q: Explain how real-time comments syncing is implemented in PrepVault.**
    *   **A:** When a user comments, `addComment` controller saves the subdocument to MongoDB, then broadcasts the updated experience object via `io.emit('experience-updated', exp)`. Other clients listen for this event and update their local React state, reflecting the comments in the UI in real time.

---

### 💾 Lectures 49-52: Relational Databases (PostgreSQL) & Prisma ORM

#### 💡 Concept Summary
Relational databases (PostgreSQL) store data in structured tables with foreign key constraints. Prisma is a type-safe ORM (Object-Relational Mapper) that replaces raw SQL or traditional query builders, using a schema file to generate clients and database migrations.

#### 🏗️ PrepVault Database Migration Plan
If we migrated PrepVault from MongoDB/Mongoose to PostgreSQL/Prisma, here is how the schema and code would change:

##### 1. The Prisma Schema File (`schema.prisma`)
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id           String       @id @default(uuid())
  name         String?
  email        String       @unique
  passwordHash String
  favorites    Favorite[]
  createdAt    DateTime     @default(now())
}

model Experience {
  id          String       @id @default(uuid())
  company     String
  role        String
  rounds      Int          @default(1)
  difficulty  Int
  questions   String[]
  tags        String[]
  tips        String?
  submittedBy String       @default("Anonymous")
  favorites   Favorite[]
  comments    Comment[]
  createdAt   DateTime     @default(now())
}

model Favorite {
  id           String     @id @default(uuid())
  userId       String
  user         User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  experienceId String
  experience   Experience @relation(fields: [experienceId], references: [id], onDelete: Cascade)
  createdAt    DateTime   @default(now())

  @@unique([userId, experienceId])
}

model Comment {
  id           String     @id @default(uuid())
  experienceId String
  experience   Experience @relation(fields: [experienceId], references: [id], onDelete: Cascade)
  user         String
  text         String
  createdAt    DateTime   @default(now())
}
```

##### 2. Controller Migration Code Comparison

**Mongoose (MongoDB - Current):**
```javascript
// Fetch experiences
const data = await Experience.find({ flagged: { $ne: true } }).sort({ createdAt: -1 })
```

**Prisma (PostgreSQL - Migrated):**
```javascript
const { PrismaClient } = require('@prisma/client')
const prisma = new PrismaClient()

// Fetch experiences with comments and favorites count
const data = await prisma.experience.findMany({
  where: { flagged: false },
  orderBy: { createdAt: 'desc' },
  include: {
    comments: true,
    _count: { select: { favorites: true } }
  }
})
```

#### ❓ Viva Q&A
1.  **Q: What are the main differences between Mongoose and Prisma?**
    *   **A:** Mongoose is an ODM for MongoDB (document store), where schemas are defined in JavaScript code. Prisma is an ORM for relational databases (like PostgreSQL), where schemas are defined in a custom declarative language (`.prisma`). Prisma automatically generates TypeScript types directly from your database schema, ensuring type safety.
2.  **Q: Explain how the User-Experience "Favorites" relationship changes when migrating to PostgreSQL.**
    *   **A:** In MongoDB, we store an array of Object IDs directly inside the User document (`User.savedFavorites`). In PostgreSQL, storing arrays of relations is not recommended. We create a join table (e.g. `Favorite` model) with foreign keys referencing both `User.id` and `Experience.id` to establish a many-to-many relationship.
3.  **Q: What is a database migration in Prisma?**
    *   **A:** A migration is a SQL file that updates the database schema structure (e.g., creating tables, adding columns) to match changes in the `schema.prisma` file. Running `npx prisma migrate dev` generates and applies these SQL scripts, tracking the database version history.
4.  **Q: How does Prisma prevent SQL Injection?**
    *   **A:** Prisma Client automatically sanitizes all inputs and uses parameterized queries under the hood, ensuring user inputs are treated as data values rather than executable SQL commands.
5.  **Q: Why does relational data require multiple tables compared to MongoDB collections?**
    *   **A:** Relational databases enforce normalization to avoid data redundancy. Nested structures (like comments inside experiences) are normalized into their own table (`Comment`) and linked back using foreign keys, whereas MongoDB can embed them directly.

---

### 📂 Lectures 53-56: Form-Data, Multer, & Cloud Storage (Cloudinary)

#### 💡 Concept Summary
Standard HTTP requests send payloads as JSON (`application/json`). However, binary file uploads must be sent as `multipart/form-data`. **Multer** is Express middleware that parses multipart request streams and saves files to disk or memory. **Cloudinary** is a cloud storage service used to store media assets and serve them via a global CDN.

#### 🏗️ PrepVault Implementation
*   **Multer Disk Storage**: Defined in [routes/experienceRoutes.js](file:///d:/01_Projects/PrepVault/PrepVault/prepvault-backend/routes/experienceRoutes.js). It preserves the uploaded file names and saves them to the server's filesystem:
    ```javascript
    const storage = multer.diskStorage({
      destination: (req, file, cb) => { cb(null, uploadsDir) },
      filename: (req, file, cb) => { cb(null, file.originalname) }
    })
    const upload = multer({ storage })
    ```
*   **Route Hook**: Mounted on the experience submission route:
    ```javascript
    router.post('/', authMiddleware, upload.array('attachments', 6), validateExperience, createExperience)
    ```
*   **DB Attribution**: The file metadata is stored in the database, and files are served locally at `/uploads/:filename`.

#### ❓ Viva Q&A
1.  **Q: Why can't we use standard JSON requests to upload files?**
    *   **A:** JSON is text-based and not optimized for raw binary data. File uploads require `multipart/form-data` encoding, which splits the HTTP request body into separate boundaries for text fields and binary file streams.
2.  **Q: What is the difference between `multer.diskStorage` and `multer.memoryStorage`?**
    *   **A:** `diskStorage` writes the uploaded files directly to server disk space (like our `public/uploads` folder). `memoryStorage` keeps files in system RAM as Buffer objects, which is useful when we want to process the file (e.g. encrypt it, or upload it to Cloudinary) without writing it to disk.
3.  **Q: How would we integrate Cloudinary into PrepVault instead of local disk storage?**
    *   **A:** We would switch Multer to `memoryStorage`. In the controller, we would stream the file buffer from RAM to Cloudinary using the Cloudinary Node SDK:
        ```javascript
        const cloudinary = require('cloudinary').v2
        cloudinary.uploader.upload_stream({ folder: 'prepvault' }, (err, result) => {
          const fileUrl = result.secure_url // Store this URL in MongoDB
        }).end(file.buffer)
        ```
4.  **Q: Why is local disk storage not recommended for production deployments?**
    *   **A:** Cloud platforms (like Heroku or Render) use ephemeral filesystems. Every time the server restarts or redeploys, all locally uploaded files are deleted. Using a cloud storage provider like Cloudinary ensures files are persisted and served via CDNs.
5.  **Q: How do we handle file array limits in Multer?**
    *   **A:** By using `upload.array('attachments', 6)`. The second argument sets a limit, automatically rejecting requests that exceed 6 files.

---

### 🧪 Lectures 57-60: Testing & Deployment

#### 💡 Concept Summary
*   **Smoke Testing**: High-level, rapid integration testing to ensure critical path features (like server startup, registration, database connections) are working before a build is deployed.
*   **Unit Testing**: Isolated testing of individual modules (e.g., checking helper functions) without system dependencies.
*   **Deployment**: Configuring platforms (Vite/Vercel for Frontend, Render/AWS for Backend) with environment keys.

#### 🏗️ PrepVault Implementation
*   **Integration Smoke Test**: Built-in test script in [testing/api_smoke_test.js](file:///d:/01_Projects/PrepVault/PrepVault/testing/api_smoke_test.js).
    *   It tests the critical request path sequentially: Register User ➔ Login ➔ Create Experience ➔ Get Experience List ➔ Get Profile.
    *   Run command:
        ```bash
        node testing/api_smoke_test.js
        ```
*   **Deployment Configuration**:
    *   **Backend (Render/AWS)**: Binds to dynamic port `process.env.PORT`. Needs environment variables configured in the provider's settings dashboard.
    *   **Frontend (Vercel)**: Compiles React using Vite build assets and proxies dynamic API endpoints.

#### ❓ Viva Q&A
1.  **Q: What is a Smoke Test? How does it differ from a Unit Test?**
    *   **A:** A smoke test verifies that the critical components of an application run without crashing (e.g., verifying database connection and that registration works). Unit testing isolates a single function in a mock environment (using tools like Jest) to test individual logic.
2.  **Q: How would you test your Express controllers using Jest and Supertest?**
    *   **A:** We would export our Express `app` without calling `server.listen()`. In our tests, we would pass the `app` instance to `supertest`, which simulates HTTP requests programmatically without starting a physical server:
        ```javascript
        const request = require('supertest')
        const app = require('../app')
        const res = await request(app).get('/api/experiences')
        expect(res.statusCode).toEqual(200)
        ```
3.  **Q: What is the significance of the `PORT` binding configuration for cloud hosting?**
    *   **A:** Cloud hosting providers dynamically assign ports to running applications at runtime. If we hardcode port `5000` on the server, the deployment will fail because the host can't bind to it. We must read the port dynamically: `const PORT = process.env.PORT || 5000`.
4.  **Q: How does Vercel optimize frontends compared to a standard web server?**
    *   **A:** Vercel builds the React app into static files (HTML, CSS, JS) and serves them from Edge network locations (CDN nodes close to users), reducing page load times.
5.  **Q: What is CI/CD, and how does it help development?**
    *   **A:** Continuous Integration and Continuous Deployment. It automates running tests (like our smoke test) whenever code is pushed to GitHub, and automatically deploys the updated app to production (Vercel/Render) once tests pass.

---

## 🎯 Final Project Defense: Suggested Demo Script

Use this step-by-step plan to guide your evaluators through the application during your viva:

1.  **Introduce the Project Overview**:
    *   "PrepVault is a community-driven repository where users share interview experience details, rounds, questions, and reference attachments."
2.  **Demonstrate the Core Web App (Client-Side CSR)**:
    *   Open `http://localhost:5173/explore`. Explain that this is a React SPA communicating asynchronously with the backend API via Axios.
3.  **Register a New User**:
    *   Register an account. Explain that the password is encrypted using **Bcrypt** (salting + hashing) in `authController.js` and stored in MongoDB.
4.  **Show the Authentication State (JWT)**:
    *   Open browser developer tools (F12) ➔ Application ➔ Local Storage.
    *   Show the `prepvault_token` JWT. Explain how the client-side interceptor attaches this header to all subsequent API requests.
5.  **Submit an Experience**:
    *   Fill out the modal, attach a dummy document, and click Submit.
    *   Explain that the form uses `multipart/form-data` parsed by **Multer**, saving the file to `public/uploads` with directory traversal guards.
6.  **Highlight Real-Time Sync (Socket.IO)**:
    *   Show that the new post appeared instantly on the dashboard. If possible, have another browser tab open in Incognito to show that the new post appeared in the other tab without manual refreshes via WebSockets.
7.  **Demonstrate the Server-Side Rendered View (SSR)**:
    *   Navigate to `http://localhost:5000/admin/report`.
    *   Explain: "For administrators, we built a server-side rendered snapshot page using **EJS**. Unlike the React frontend, this page is compiled on the server and returned as HTML, allowing admins to moderate, flag, or delete posts."
8.  **Run the Smoke Test**:
    *   Open your terminal and run `node testing/api_smoke_test.js`.
    *   Explain: "We also built an integration smoke testing script that programmatically tests the full API pipeline (Register ➔ Login ➔ Post ➔ Fetch) to verify system stability."
