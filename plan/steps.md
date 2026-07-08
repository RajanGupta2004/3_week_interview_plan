# MERN Stack Developer (2 YOE) — Interview Preparation Plan

### Focus: Backend (Node.js, Express, MongoDB) + System Design + Full-Stack Integration

This is a **3-week, day-wise plan**. At 2 years of experience, interviewers expect you to go **beyond syntax** — into _why_ decisions are made (architecture, performance, security, trade-offs). Adjust pace based on how much time/day you have (plan assumes 2–4 hrs/day).

---

## How the 3 Weeks Are Split

- **Week 1** — Node.js & Express internals + JavaScript/Async mastery
- **Week 2** — MongoDB, Mongoose, Database design, Auth & Security
- **Week 3** — System design, API design, Testing/DevOps, Mock interviews & Behavioral

---

## WEEK 1 — Node.js, Express & JavaScript Fundamentals

### Day 1: JavaScript Core (the part interviewers actually probe at 2 YOE)

- Closures, hoisting, `this` binding (call/apply/bind), prototypal inheritance
- Event loop: call stack, microtask vs macrotask queue, `process.nextTick` vs `setImmediate` vs `setTimeout`
- `var` vs `let`/`const`, temporal dead zone
- Deep vs shallow copy, object references
- **Practice Qs:**
  - Explain the event loop with a code example that prints in an unexpected order.
  - Difference between `process.nextTick()`, `Promise.resolve().then()`, and `setImmediate()`.
  - What happens when you `await` inside a `forEach` loop? Why doesn't it work as expected?

### Day 2: Asynchronous JavaScript Deep Dive

- Callbacks → Promises → async/await evolution
- Promise combinators: `Promise.all`, `allSettled`, `race`, `any`
- Error handling in async/await (try/catch, unhandled promise rejections)
- Streams and Buffers in Node.js (Readable, Writable, Transform, Duplex)
- **Practice Qs:**
  - How would you handle 1000 API calls with a concurrency limit of 5?
  - Explain backpressure in Node.js streams.
  - Write a function that retries a failed async operation 3 times with exponential backoff.

### Day 3: Node.js Runtime Internals

- Single-threaded event loop + libuv thread pool
- How Node handles CPU-intensive tasks (worker_threads, child_process, cluster module)
- Module systems: CommonJS vs ES Modules, `require` caching
- Global objects, `global`, `process`, environment variables (`dotenv`)
- Memory management: garbage collection basics, memory leaks (detached closures, global variable accumulation)
- **Practice Qs:**
  - Node.js is single-threaded — how does it handle thousands of concurrent connections?
  - When would you use `cluster` vs `worker_threads`?
  - How do you detect and fix a memory leak in a Node app? (mention `--inspect`, heap snapshots)

### Day 4: Express.js Deep Dive

- Middleware architecture: request/response cycle, `next()`, error-handling middleware (4-arg signature)
- Router-level vs application-level middleware
- Built-in vs third-party middleware (`express.json()`, `cors`, `helmet`, `morgan`)
- Custom middleware: logging, request validation, rate limiting
- Error handling patterns (centralized error handler, async error wrapper)
- **Practice Qs:**
  - How does Express know to skip to the error handler?
  - Write a global error-handling middleware that differentiates operational vs programming errors.
  - How would you implement request validation middleware using `Joi` or `express-validator`?

### Day 5: REST API Design Principles

- Resource naming, HTTP verbs, idempotency (PUT vs PATCH vs POST)
- Status codes — not just 200/404, but 201, 204, 400, 401, 403, 409, 422, 429, 500, 503
- Pagination (offset vs cursor-based), filtering, sorting
- Versioning strategies (URI, header-based)
- HATEOAS (conceptual awareness, not always used in practice)
- **Practice Qs:**
  - Design REST endpoints for an e-commerce order system.
  - Why prefer cursor-based pagination over offset for large datasets?
  - When do you return 409 vs 422?

### Day 6: Practice Problems (Coding)

- Build a mini rate limiter (token bucket / sliding window) in Node.js
- Implement debounce/throttle in JS
- Write a custom Promise implementation (`Promise.myAll`)
- Implement an LRU cache
- Basic DSA revision: arrays, strings, hashmaps, two-pointer, recursion (30–45 min daily from here on)

### Day 7: Revision + Mock Round 1

- Revise Day 1–6 flashcard-style
- Do a self-timed mock: 5 conceptual Qs + 1 coding problem (45 min)

---

## WEEK 2 — MongoDB, Database Design, Auth & Security

### Day 8: MongoDB Fundamentals

- Document model, BSON vs JSON, collections vs tables
- CRUD operations, query operators (`$gt`, `$in`, `$regex`, `$elemMatch`)
- Indexes: single-field, compound, multikey, text, TTL indexes
- `explain()` and query performance analysis
- **Practice Qs:**
  - How does MongoDB decide which index to use?
  - What's a covered query and why does it matter for performance?
  - Difference between `$lookup` and application-side joins — when to use which?

### Day 9: Mongoose ODM

- Schemas, models, validation, virtuals, middleware (pre/post hooks)
- Population (`populate()`) — performance implications, alternatives
- Schema design patterns: embedding vs referencing (1:1, 1:N, N:N)
- Transactions in MongoDB (multi-document ACID transactions, sessions)
- **Practice Qs:**
  - When would you embed vs reference documents? Give a concrete example (e.g., comments on a blog post).
  - How do you handle transactions across multiple collections in Mongoose?
  - What are the risks of deeply nested `populate()` chains?

### Day 10: Database Design & Aggregation Framework

- Aggregation pipeline: `$match`, `$group`, `$project`, `$unwind`, `$lookup`, `$facet`
- Schema design for scale: denormalization trade-offs, read-heavy vs write-heavy design
- Sharding concepts (shard key selection), replication (replica sets, read preference, write concern)
- **Practice Qs:**
  - Write an aggregation pipeline to get monthly revenue grouped by product category.
  - How would you shard a "users" collection with 100M+ documents?
  - Explain write concern `{w: "majority"}` vs `{w: 1}` — when does it matter?

### Day 11: Authentication & Authorization

- Session-based auth vs JWT — trade-offs (statelessness, revocation, storage)
- JWT structure (header/payload/signature), access token + refresh token flow
- Secure storage of tokens (httpOnly cookies vs localStorage — XSS/CSRF trade-offs)
- Role-based access control (RBAC) middleware design
- OAuth2.0 flow (conceptual — Google/GitHub login)
- **Practice Qs:**
  - Why is storing JWT in localStorage risky? What's the mitigation?
  - How do you implement token refresh without forcing re-login?
  - How would you invalidate a JWT before its expiry (e.g., on logout)?

### Day 12: Backend Security Essentials

- OWASP Top 10 relevant to Node/Express: NoSQL injection, XSS, CSRF, broken auth
- `helmet`, CORS configuration, rate limiting (`express-rate-limit`), input sanitization
- Password hashing (`bcrypt`, salt rounds) — never store plaintext, never use MD5/SHA1 alone
- Environment variable/secrets management
- **Practice Qs:**
  - How does a NoSQL injection attack work in MongoDB, and how do you prevent it?
  - Explain CSRF and how SameSite cookies help mitigate it.
  - How many bcrypt salt rounds would you use in production, and why not more?

### Day 13: Caching & Performance

- Redis basics: caching strategies (cache-aside, write-through), TTL, cache invalidation
- Where to cache in a MERN app (DB query results, session store, rate limiter store)
- Connection pooling (MongoDB driver pool size), N+1 query problem and fixes
- Load testing basics (concept of `autocannon`/`k6`)
- **Practice Qs:**
  - How would you cache a frequently-read, rarely-updated endpoint (e.g., product catalog)?
  - What's the N+1 query problem and how do you solve it in Mongoose?

### Day 14: Revision + Mock Round 2

- Revise Day 8–13
- Mock: design a schema for a real app (e.g., food delivery) + write 2 aggregation queries + explain your auth flow choice

---

## WEEK 3 — System Design, Testing, DevOps & Behavioral

### Day 15: System Design Basics (Backend-Heavy)

- Client-server architecture, load balancers, horizontal vs vertical scaling
- Microservices vs monolith trade-offs (you'll likely be asked why/when to split)
- Message queues (RabbitMQ/Kafka basics — decoupling services, async processing)
- API Gateway concept, rate limiting at gateway level
- **Practice Design Qs (2 YOE level — keep it practical, not FAANG-scale):**
  - Design a URL shortener (schema, endpoints, collision handling, redirect flow)
  - Design a notification system (email/SMS/push) using a queue
  - Design the backend for a simple chat app (REST + WebSocket, message persistence)

### Day 16: Real-Time & File Handling

- WebSockets vs polling vs Server-Sent Events — when to use which
- `Socket.io` basics: rooms, namespaces, broadcasting
- File uploads: `multer`, streaming large files, storing in S3/Cloud storage vs DB
- **Practice Qs:**
  - How would you scale Socket.io across multiple server instances? (Redis adapter)
  - How do you handle large file uploads without blocking the event loop?

### Day 17: Testing

- Unit testing with Jest/Mocha: mocking DB calls, dependency injection for testability
- Integration testing APIs with `supertest`
- Test pyramid concept (unit > integration > e2e)
- **Practice Qs:**
  - Write a Jest test for an Express route that creates a user (mock the Mongoose model).
  - How do you test middleware in isolation?

### Day 18: DevOps & Deployment Awareness

- Environment configs (dev/staging/prod), `.env` management
- Docker basics: Dockerfile for a Node app, docker-compose with Mongo
- CI/CD conceptual flow (GitHub Actions: lint → test → build → deploy)
- Logging & monitoring: `winston`/`pino`, centralized logging, health-check endpoints
- Basic understanding of hosting (EC2/Render/Vercel for frontend, backend server considerations)
- **Practice Qs:**
  - Write a basic Dockerfile for an Express + MongoDB app.
  - What would a `/health` endpoint check in production?

### Day 19: Full-Stack Integration & MERN-Specific Questions

- How React talks to Express (Axios/Fetch, CORS config, proxy in dev)
- State management touchpoints with backend (React Query/Redux + API calls, optimistic updates)
- Handling file/image uploads from React to Express to storage
- Environment separation for frontend/backend API URLs
- **Practice Qs:**
  - How do you handle a slow backend response gracefully on the frontend? (loading states, timeouts, retries)
  - Walk through the full request lifecycle: user clicks submit on a React form → data reaches MongoDB.

### Day 20: Behavioral + Project Deep-Dive Prep

- Prepare STAR-format answers for:
  - A bug you debugged that was hard to trace (shows backend depth)
  - A time you optimized a slow API/query
  - A disagreement with a teammate/lead on architecture
  - A production incident and how you handled it
- Be ready to **defend every design decision in your resume's projects** — interviewers will drill into "why MongoDB and not SQL here?", "why did you structure the API this way?"
- Prepare 2–3 smart questions to ask the interviewer (team structure, deployment process, code review culture)

### Day 21: Final Mock Interview + Weak-Area Revision

- Full 45–60 min mock: 1 JS/Node conceptual round + 1 DB design round + 1 system design mini-question + resume-based questions
- Revisit your weakest 2 topics from the past 3 weeks
- Sleep well — confidence matters as much as knowledge at this stage

---

## Quick-Reference: Most Commonly Asked Backend Questions at 2 YOE

1. Explain the Node.js event loop in detail.
2. Difference between SQL and NoSQL — when would you _not_ use MongoDB?
3. How do you design a schema for embedding vs referencing?
4. Explain JWT-based auth end-to-end, including refresh tokens.
5. How do you prevent NoSQL injection?
6. What is middleware in Express, and how does error-handling middleware differ?
7. How would you scale a Node.js app handling 10K concurrent users?
8. Explain indexing in MongoDB and how you'd debug a slow query.
9. How do you handle transactions across multiple documents/collections?
10. Difference between `process.nextTick`, microtasks, and macrotasks.
11. How do you secure REST APIs (auth, rate limiting, input validation, CORS)?
12. Walk through what happens when 100 requests hit your API simultaneously.

---

## Resources

- **Practice coding:** LeetCode (Easy/Medium — arrays, strings, hashmaps), or take-home style challenges
- **System design (light, practical):** "System Design Primer" (GitHub), ByteByteGo YouTube channel
- **MongoDB:** MongoDB University free courses (M001, M121 for aggregation)
- **Node internals:** Node.js official docs on Event Loop, Streams
- **Mock interviews:** Pramp, or practice explaining answers out loud/recording yourself

---

## Daily Habit for All 3 Weeks

- 30–45 min DSA practice (consistency > volume)
- Explain one concept out loud daily as if teaching someone — this is the single best way to expose gaps at the 2 YOE "explain your reasoning" level.
