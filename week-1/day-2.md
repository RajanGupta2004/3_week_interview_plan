Here's the detailed Q&A for **Day 2: Asynchronous JavaScript Deep Dive**, with explanations and code snippets.

---

## 1. Callbacks → Promises → async/await Evolution

**Q: Walk through why JavaScript moved from callbacks to Promises to async/await.**

**Callbacks (the original approach):**

```javascript
function getUser(id, callback) {
  db.query(`SELECT * FROM users WHERE id=${id}`, (err, user) => {
    if (err) return callback(err);
    getOrders(user.id, (err, orders) => {
      if (err) return callback(err);
      getPayments(orders.id, (err, payments) => {
        if (err) return callback(err);
        callback(null, payments);
      });
    });
  });
}
```

This nesting is called **"callback hell"** — hard to read, hard to handle errors consistently (every level needs its own `if (err)` check), and hard to compose.

**Promises (fixed composability and error handling):**

```javascript
function getUser(id) {
  return db.query(`SELECT * FROM users WHERE id=${id}`);
}

getUser(1)
  .then((user) => getOrders(user.id))
  .then((orders) => getPayments(orders.id))
  .then((payments) => console.log(payments))
  .catch((err) => console.error("Something failed:", err)); // ONE catch for the whole chain
```

**async/await (syntactic sugar over Promises — reads like sync code):**

```javascript
async function getUserPayments(id) {
  try {
    const user = await getUser(id);
    const orders = await getOrders(user.id);
    const payments = await getPayments(orders.id);
    return payments;
  } catch (err) {
    console.error("Something failed:", err);
  }
}
```

**Key point for interviews:** async/await is not a replacement mechanism — it's still Promises underneath. `async function` always returns a Promise, and `await` just pauses execution until that Promise settles.

```javascript
async function foo() {
  return 42;
}
foo().then(console.log); // 42 — foo() returns a Promise that resolves to 42
```

---

## 2. Promise Combinators: `all`, `allSettled`, `race`, `any`

**Q: Explain the difference between `Promise.all`, `allSettled`, `race`, and `any`, with examples.**

```javascript
const p1 = new Promise((res) => setTimeout(() => res("P1 done"), 100));
const p2 = new Promise((res) => setTimeout(() => res("P2 done"), 200));
const p3 = new Promise((_, rej) => setTimeout(() => rej("P3 failed"), 150));
```

### `Promise.all` — fails fast if ANY promise rejects

```javascript
Promise.all([p1, p2]).then((results) => console.log(results)); // ["P1 done", "P2 done"]

Promise.all([p1, p2, p3])
  .then((results) => console.log(results))
  .catch((err) => console.log("Rejected:", err)); // "Rejected: P3 failed" (as soon as p3 rejects, ignores p2)
```

Use when: **all results are required**, and failing one means the whole operation should fail (e.g., fetching user + their permissions + their profile before rendering a page).

### `Promise.allSettled` — waits for all, never short-circuits

```javascript
Promise.allSettled([p1, p2, p3]).then((results) => console.log(results));
/*
[
  { status: 'fulfilled', value: 'P1 done' },
  { status: 'fulfilled', value: 'P2 done' },
  { status: 'rejected', reason: 'P3 failed' }
]
*/
```

Use when: you want the **outcome of every promise regardless of failures** (e.g., sending notifications to 100 users — one failing shouldn't stop the rest, and you want a report of who failed).

### `Promise.race` — settles as soon as the FIRST one settles (resolve or reject)

```javascript
Promise.race([p1, p2, p3]).then(console.log).catch(console.log);
// "P1 done" — p1 settles first at 100ms
```

Use when: implementing **timeouts**.

```javascript
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error("Request timed out")), ms),
  );
  return Promise.race([promise, timeout]);
}

// Usage:
withTimeout(fetch("/api/slow-endpoint"), 3000)
  .then((res) => console.log("Got response in time"))
  .catch((err) => console.log(err.message)); // "Request timed out" if it takes >3s
```

### `Promise.any` — resolves as soon as the FIRST one succeeds; ignores rejections unless ALL fail

```javascript
Promise.any([p3, p2, p1]).then((result) => console.log(result)); // "P1 done" — first successful one, ignores p3's rejection

Promise.any([Promise.reject("fail1"), Promise.reject("fail2")]).catch((err) =>
  console.log(err),
);
// AggregateError: All promises were rejected
```

Use when: you have **multiple redundant sources** (e.g., multiple mirror servers) and just need the fastest successful response.

**Quick summary table (great to say out loud in an interview):**

| Method       | Resolves when              | Rejects when               |
| ------------ | -------------------------- | -------------------------- |
| `all`        | all fulfill                | any one rejects            |
| `allSettled` | all settle (always)        | never                      |
| `race`       | first settles (either way) | first settles as rejection |
| `any`        | first fulfills             | all reject                 |

---

## 3. Error Handling in async/await

**Q: How do you properly handle errors in async/await, and what is an unhandled promise rejection?**

**Basic try/catch:**

```javascript
async function getData() {
  try {
    const data = await fetchData();
    return data;
  } catch (err) {
    console.error("Failed to fetch:", err.message);
    throw err; // re-throw if the caller needs to know
  }
}
```

**The Express-specific trap:** forgetting to catch errors in async route handlers.

```javascript
// BAD — if getUserById throws, Express doesn't catch it automatically,
// the request hangs or crashes the process depending on Node version
app.get("/users/:id", async (req, res) => {
  const user = await getUserById(req.params.id); // unhandled rejection if this throws
  res.json(user);
});
```

**Fix 1 — manual try/catch:**

```javascript
app.get("/users/:id", async (req, res, next) => {
  try {
    const user = await getUserById(req.params.id);
    res.json(user);
  } catch (err) {
    next(err); // forwards to Express error-handling middleware
  }
});
```

**Fix 2 — async wrapper utility (common production pattern):**

```javascript
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get(
  "/users/:id",
  asyncHandler(async (req, res) => {
    const user = await getUserById(req.params.id);
    res.json(user);
  }),
);
```

**Q: What is an unhandled promise rejection, and how do you catch it globally?**

```javascript
Promise.reject(new Error("Oops")); // no .catch() anywhere — unhandled

process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled Rejection:", reason);
  // log it, alert monitoring, and often: process.exit(1) to fail fast
});

process.on("uncaughtException", (err) => {
  console.error("Uncaught Exception:", err);
  process.exit(1); // Node recommends NOT continuing after this
});
```

**Interview point:** Since Node 15+, an unhandled promise rejection **crashes the process by default** (previously just a warning). This is important — it means a single un-caught async error in an old Express route can bring down your whole server.

---

## 4. Streams and Buffers in Node.js

**Q: What is a Buffer, and why does Node need it?**

A `Buffer` is a fixed-size chunk of raw binary data, used because JavaScript strings weren't originally designed to handle binary data (e.g., reading a file, a TCP socket, an image).

```javascript
const buf = Buffer.from("Hello");
console.log(buf); // <Buffer 48 65 6c 6c 6f> — raw bytes
console.log(buf.toString()); // "Hello"
```

**Q: What are Streams, and why use them instead of reading a whole file into memory?**

Streams let you process data **piece by piece** instead of loading everything into memory at once — critical for large files or network data.

```javascript
const fs = require("fs");

// BAD for large files — loads entire file into memory
fs.readFile("large-video.mp4", (err, data) => {
  res.end(data);
});

// GOOD — streams the file in chunks
const stream = fs.createReadStream("large-video.mp4");
stream.pipe(res); // pipes chunks directly to the HTTP response as they're read
```

**Q: Explain the 4 types of streams.**

| Type          | Purpose                                               | Example                                        |
| ------------- | ----------------------------------------------------- | ---------------------------------------------- |
| **Readable**  | Source you read data from                             | `fs.createReadStream()`, incoming HTTP request |
| **Writable**  | Destination you write data to                         | `fs.createWriteStream()`, HTTP response        |
| **Duplex**    | Both readable and writable                            | TCP sockets                                    |
| **Transform** | Duplex stream that modifies data as it passes through | `zlib.createGzip()` (compression)              |

```javascript
const fs = require("fs");
const zlib = require("zlib");

// Real-world example: read a file, compress it, write to disk — all streamed
fs.createReadStream("input.txt")
  .pipe(zlib.createGzip()) // Transform stream
  .pipe(fs.createWriteStream("input.txt.gz")); // Writable stream
```

---

## 5. Backpressure (Practice Q)

**Q: Explain backpressure in Node.js streams.**

Backpressure happens when a **Writable stream can't consume data as fast as the Readable stream produces it** — data would pile up in memory if not handled.

```javascript
const readable = fs.createReadStream("huge-file.txt");
const writable = fs.createWriteStream("copy.txt");

readable.on("data", (chunk) => {
  const canContinue = writable.write(chunk);
  if (!canContinue) {
    readable.pause(); // tell the readable stream to slow down
  }
});

writable.on("drain", () => {
  readable.resume(); // writable has caught up, safe to resume
});
```

In practice, you almost never write this manually — `.pipe()` handles backpressure automatically:

```javascript
readable.pipe(writable); // pause/resume logic is built in
```

**Why interviewers ask this:** it shows you understand that Node.js streams aren't just a convenience API — they exist specifically to prevent memory overload when producer and consumer speeds differ (e.g., streaming a big file to a slow client over a slow network connection).

---

## 6. Concurrency-Limited API Calls (Practice Q)

**Q: How would you handle 1000 API calls with a concurrency limit of 5?**

**Why not `Promise.all` directly:** firing 1000 requests at once can overwhelm the server/API (rate limits) and spike memory.

**Approach — a simple concurrency pool:**

```javascript
async function runWithConcurrencyLimit(tasks, limit) {
  const results = [];
  const executing = [];

  for (const task of tasks) {
    const p = task().then((result) => {
      executing.splice(executing.indexOf(p), 1); // remove from in-flight list when done
      return result;
    });
    results.push(p);
    executing.push(p);

    if (executing.length >= limit) {
      await Promise.race(executing); // wait for the fastest one to finish before adding more
    }
  }

  return Promise.all(results);
}

// Usage:
const urls = Array.from(
  { length: 1000 },
  (_, i) => `https://api.example.com/item/${i}`,
);
const tasks = urls.map((url) => () => fetch(url).then((r) => r.json()));

runWithConcurrencyLimit(tasks, 5).then((allResults) => {
  console.log("All 1000 done, 5 at a time:", allResults.length);
});
```

**Mention in interview:** in real production code, you'd typically just use a well-tested library like `p-limit` instead of hand-rolling this:

```javascript
const pLimit = require("p-limit");
const limit = pLimit(5);

const tasks = urls.map((url) => limit(() => fetch(url).then((r) => r.json())));
const results = await Promise.all(tasks);
```

---

## 7. Retry with Exponential Backoff (Practice Q)

**Q: Write a function that retries a failed async operation 3 times with exponential backoff.**

```javascript
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function retryWithBackoff(fn, retries = 3, baseDelay = 500) {
  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      return await fn(); // success — return immediately
    } catch (err) {
      if (attempt === retries) {
        throw new Error(`Failed after ${retries + 1} attempts: ${err.message}`);
      }
      const waitTime = baseDelay * 2 ** attempt; // 500, 1000, 2000, 4000...
      console.log(
        `Attempt ${attempt + 1} failed. Retrying in ${waitTime}ms...`,
      );
      await delay(waitTime);
    }
  }
}

// Usage:
async function unreliableApiCall() {
  const res = await fetch("https://api.example.com/data");
  if (!res.ok) throw new Error(`Status ${res.status}`);
  return res.json();
}

retryWithBackoff(unreliableApiCall, 3, 500)
  .then((data) => console.log("Success:", data))
  .catch((err) => console.error(err.message));
```

**Bonus talking point:** mention adding **jitter** (small random variation) to the backoff delay in real systems — it prevents the "thundering herd" problem where many clients retry at the exact same moment after a shared failure (e.g., a brief DB outage).

```javascript
const waitTime = baseDelay * 2 ** attempt + Math.random() * 100; // jitter
```

---

### Quick Self-Test

1. Why does `Promise.all` reject immediately when one promise fails, while `allSettled` doesn't?
2. Give a real use case for `Promise.race` and one for `Promise.any`.
3. Why can an uncaught error in an `async` Express route crash your server if you don't handle it?
4. What problem do streams solve that `fs.readFile` doesn't?
5. Why add jitter to exponential backoff instead of a fixed delay sequence?

Want me to continue with **Day 3 (Node.js Runtime Internals)** in the same format next?
