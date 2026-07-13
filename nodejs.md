# Node.js Interview Guide

---

# Beginner

## 1. What is Node.js?

Node.js is an **open-source JavaScript runtime** built on Google's **V8 JavaScript engine**. It allows JavaScript to run **outside the browser**, mainly on servers.

Instead of using JavaScript only for frontend development, Node.js lets you build:

* REST APIs
* GraphQL APIs
* Real-time chat applications
* Streaming services
* CLI tools
* Backend services
* Microservices

### Why Node.js is popular

* Very fast (uses V8 engine)
* Non-blocking I/O
* Event-driven architecture
* Huge npm ecosystem
* Great for APIs and real-time applications

Example:

```js
const http = require("http");

const server = http.createServer((req, res) => {
    res.end("Hello World");
});

server.listen(3000);
```

---

## 2. Explain the Event Loop

The Event Loop is what allows Node.js to perform many operations simultaneously using a **single thread**.

Although JavaScript runs on one thread, operations like:

* File reading
* Database queries
* HTTP requests
* Timers

are delegated to the operating system or libuv.

When they're finished, callbacks are placed into queues.

The Event Loop continuously checks:

```
Call Stack empty?
        ↓
Yes
        ↓
Take next callback from queue
        ↓
Execute it
```

Example:

```js
console.log("Start");

setTimeout(() => {
    console.log("Timer");
}, 0);

console.log("End");
```

Output

```
Start
End
Timer
```

Even though timeout is 0ms, it waits until the call stack becomes empty.

### Event Loop Phases

1. Timers
2. Pending callbacks
3. Idle/Prepare
4. Poll
5. Check (`setImmediate`)
6. Close callbacks

---

## 3. Difference between synchronous and asynchronous code

### Synchronous

Runs one after another.

```
Task A
↓

Task B
↓

Task C
```

Example

```js
console.log("A");
console.log("B");
console.log("C");
```

Output

```
A
B
C
```

---

### Asynchronous

Doesn't wait for slow operations.

Example

```js
console.log("A");

setTimeout(() => {
    console.log("B");
}, 1000);

console.log("C");
```

Output

```
A
C
B
```

The program continues while waiting.

---

## 4. What is non-blocking I/O?

I/O means Input/Output:

* Reading files
* Database access
* Network requests

Blocking I/O:

```js
const data = fs.readFileSync("file.txt");
```

Everything stops until reading finishes.

Non-blocking:

```js
fs.readFile("file.txt", (err, data) => {
    console.log(data);
});
```

Node immediately continues doing other work.

That's why one server can handle thousands of requests.

---

## 5. What is npm?

npm stands for **Node Package Manager**.

It is:

* package manager
* dependency manager
* package registry

Example

```
npm install express
```

Creates:

```
node_modules/
package.json
package-lock.json
```

Useful commands

```
npm install

npm update

npm uninstall

npm run build

npm run dev

npm test
```

---

## 6. Difference between CommonJS and ES Modules

### CommonJS

Older Node module system.

```js
const express = require("express");

module.exports = app;
```

---

### ES Modules

Modern JavaScript standard.

```js
import express from "express";

export default app;
```

Comparison

| CommonJS            | ES Modules      |
| ------------------- | --------------- |
| require()           | import          |
| module.exports      | export          |
| Synchronous loading | Static loading  |
| Older Node          | Modern standard |

Modern projects generally use **ES Modules**.

---

# Intermediate

## 7. Explain Streams

Streams process data **piece by piece** instead of loading everything into memory.

Without streams:

```
2GB File
↓

Load entire file
↓

Memory explosion
```

With streams:

```
Chunk
↓

Chunk
↓

Chunk
```

Example

```js
const fs = require("fs");

const stream = fs.createReadStream("movie.mp4");

stream.on("data", chunk => {
    console.log(chunk.length);
});
```

Types of streams

* Readable
* Writable
* Duplex
* Transform

Examples

* Video streaming
* File uploads
* HTTP responses
* Compression

---

## 8. Worker Threads vs Child Processes

### Worker Threads

* Same process
* Different thread
* Shared memory

Good for:

* image processing
* calculations
* encryption

---

### Child Process

Starts a completely new Node process.

Good for:

* running another application
* executing shell commands
* isolation

Comparison

| Worker               | Child Process    |
| -------------------- | ---------------- |
| Same process         | Separate process |
| Shared memory        | Separate memory  |
| Faster communication | More isolated    |

---

## 9. What causes memory leaks?

Memory leak = memory that is no longer needed but cannot be freed.

Common causes

### Global variables

```js
global.users = [];
```

---

### Unremoved event listeners

```js
emitter.on("event", handler);
```

Never removed.

---

### Timers

```js
setInterval(...)
```

Forgot to clear.

---

### Large cache

```js
const cache = {};
```

Never cleaned.

---

### Closures

Holding references to old objects.

---

## 10. What is clustering?

Node uses one CPU core by default.

Cluster allows multiple Node processes.

```
Load Balancer
      |
-----------------------
|     |      |      |
P1    P2     P3     P4
```

One process per CPU core.

Benefits

* Better CPU usage
* Higher throughput
* Fault tolerance

---

## 11. Explain Buffer

A Buffer stores binary data.

Useful for

* images
* PDFs
* videos
* TCP
* file uploads

Example

```js
const buf = Buffer.from("Hello");

console.log(buf);
```

Output

```
<Buffer 48 65 6c 6c 6f>
```

---

## 12. How does garbage collection work?

Node uses V8's Garbage Collector.

Objects that are no longer reachable are automatically removed.

Example

```js
let user = {
    name: "John"
};

user = null;
```

Object becomes unreachable.

Later:

```
GC removes it
```

Modern V8 uses:

* Mark-and-Sweep
* Generational Garbage Collection

Young objects are collected frequently.

Old objects less often.

---

# Senior

## 13. How would you scale Node.js to millions of users?

Scaling requires both **vertical** and **horizontal** strategies.

Typical architecture:

```
Internet
      ↓
Load Balancer
      ↓
API Servers (multiple Node instances)
      ↓
Redis Cache
      ↓
Message Queue
      ↓
Database
```

Key techniques:

* Use clustering or a process manager like PM2.
* Run multiple instances behind a load balancer (e.g., Nginx or a cloud load balancer).
* Containerize with Docker and orchestrate with Kubernetes.
* Cache frequently accessed data with Redis.
* Use a CDN for static assets.
* Offload long-running work to background workers using queues (BullMQ, RabbitMQ, Kafka).
* Optimize database queries and add read replicas if needed.
* Design stateless services so any instance can handle any request.

---

## 14. CPU-intensive work in Node?

Node's event loop is single-threaded, so CPU-heavy tasks block all incoming requests.

Example:

* Video encoding
* Image resizing
* Large data processing
* Encryption
* Machine learning inference

Avoid blocking the event loop by:

* Using **Worker Threads** for CPU-bound JavaScript tasks.
* Spawning **Child Processes** for isolated workloads or external programs.
* Offloading work to background workers through message queues.
* Moving heavy computation to dedicated microservices if necessary.

**Interview tip:** Never perform expensive loops or synchronous CPU-heavy work directly in a request handler.

---

## 15. Design a high-throughput API

A high-throughput API should maximize requests per second while minimizing latency.

Example architecture:

```
Client
   ↓
Load Balancer
   ↓
Node.js API
   ↓
Redis Cache
   ↓
Message Queue (async jobs)
   ↓
Database
```

Best practices:

* Keep endpoints stateless.
* Use connection pooling for databases.
* Cache frequently requested data.
* Paginate large datasets.
* Compress responses (gzip/Brotli).
* Stream large files instead of loading them entirely into memory.
* Use asynchronous I/O throughout.
* Apply rate limiting to protect the service.
* Optimize database indexes and avoid N+1 queries.
* Use background jobs for emails, notifications, reports, and other slow operations.

---

## 16. How do you monitor Node performance?

Common metrics to monitor include:

* CPU usage
* Memory usage
* Event loop delay/lag
* Request latency
* Throughput (requests per second)
* Error rate
* Database query times

Useful tools:

* **Node.js Inspector** for debugging.
* **Chrome DevTools** for profiling.
* **clinic.js** (Doctor, Flame, Bubbleprof) for performance analysis.
* **0x** for flame graphs.
* **PM2** for process monitoring.
* **Prometheus + Grafana** for metrics dashboards.
* **OpenTelemetry** for distributed tracing.
* Application monitoring tools like Datadog, New Relic, or Elastic APM.

In production, combine logs, metrics, and traces to quickly identify bottlenecks.

---

## 17. How would you debug production memory leaks?

A systematic approach is important:

1. **Confirm the leak**

   * Watch memory usage over time. If heap usage continually grows and never stabilizes, a leak is likely.

2. **Capture heap snapshots**

   * Compare snapshots to identify objects that continue to grow.

3. **Look for common causes**

   * Global variables.
   * Large caches without eviction policies.
   * Uncleared timers.
   * Event listeners that aren't removed.
   * Closures retaining references.
   * Unclosed streams or database connections.

4. **Profile the application**

   * Use Chrome DevTools, Node Inspector, or clinic.js to inspect memory allocation.

5. **Monitor in production**

   * Export metrics (heap size, GC duration, event loop lag) to Prometheus/Grafana or an APM solution.

**Example interview answer:**

> "I'd first verify that memory usage is continuously increasing rather than fluctuating normally with garbage collection. I'd collect heap snapshots, compare retained objects, identify what's preventing garbage collection, and check for common issues like global caches, lingering event listeners, timers, or closures. I'd use tools such as Chrome DevTools, clinic.js, and Prometheus/Grafana to monitor the application, then fix the root cause and validate that heap usage stabilizes after deployment."

---

## Common Node.js Interview Questions

* Why is Node.js single-threaded?
* What is the Event Loop and how does it work?
* What is libuv?
* What is the difference between `process.nextTick()`, `setImmediate()`, and `setTimeout()`?
* When should you use Worker Threads instead of Child Processes?
* What are streams and why are they memory-efficient?
* How does Node.js handle thousands of concurrent connections?
* How do you prevent blocking the event loop?
* What causes memory leaks, and how do you detect them?
* How would you build a scalable, fault-tolerant Node.js backend?


---

# Common Node.js Interview Questions (With Detailed Answers)

---

# 1. Why is Node.js single-threaded?

### Short Answer

Node.js executes JavaScript on a **single main thread**, meaning only one JavaScript task is executed at a time. However, it can still handle thousands of concurrent operations because I/O tasks are delegated to the operating system and **libuv**.

---

### Why was it designed this way?

JavaScript was originally designed for browsers, where a single thread avoids issues like race conditions and makes programming simpler.

Node.js kept this model because:

* It's easier to write asynchronous code.
* There's no need for locks or mutexes in most cases.
* Context switching between threads is minimized.
* It works extremely well for I/O-heavy applications.

---

### Does single-threaded mean it can only handle one request?

**No.** This is a common misconception.

When a request involves waiting (e.g., reading a file or querying a database), Node.js doesn't sit idle. It hands the work off to libuv or the operating system and continues processing other requests.

For example:

```js
app.get("/users", async (req, res) => {
  const users = await db.getUsers();
  res.json(users);
});
```

While the database query is running, Node.js can process many other incoming requests.

---

### When is this a problem?

CPU-intensive tasks can block the single thread.

For example:

```js
app.get("/calculate", (req, res) => {
  // Huge computation
  for (let i = 0; i < 10_000_000_000; i++) {}

  res.send("Done");
});
```

During that loop, **every other request waits**, because the event loop is blocked.

---

### Interview Answer

> "Node.js runs JavaScript on a single thread to simplify concurrency and reduce thread management overhead. It achieves high scalability by delegating asynchronous I/O operations to libuv and the operating system, allowing the event loop to continue processing other requests. CPU-heavy tasks should be moved to Worker Threads or background services."

---

# 2. What is the Event Loop and how does it work?

The Event Loop is the mechanism that allows Node.js to perform asynchronous operations without blocking the main thread.

Think of it as a scheduler.

```
Incoming Request
        │
        ▼
 Execute JavaScript
        │
        ▼
Needs Database?
        │
       Yes
        │
        ▼
libuv handles it
        │
        ▼
Continue processing other requests
        │
        ▼
Database finishes
        │
        ▼
Callback returns to Event Loop
        │
        ▼
Execute callback
```

---

### Event Loop Phases

```
Timers
↓

Pending Callbacks
↓

Idle / Prepare
↓

Poll
↓

Check
↓

Close Callbacks
```

---

### Example

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");
```

Output:

```
A
C
B
```

Even with `0ms`, the callback waits until the call stack is empty.

---

### Interview Answer

> "The Event Loop continuously checks whether the JavaScript call stack is empty. If it is, it takes completed asynchronous callbacks from their respective queues and executes them. This enables Node.js to handle many concurrent I/O operations without creating a thread for each request."

---

# 3. What is libuv?

libuv is the **C library** that powers Node.js's asynchronous behavior.

Without libuv, Node.js would simply execute JavaScript.

libuv provides:

* Event Loop
* Thread Pool
* File System operations
* DNS lookups
* TCP networking
* Timers

---

### Example

```js
fs.readFile("data.txt", callback);
```

What actually happens:

```
JavaScript
      │
      ▼
libuv
      │
      ▼
Operating System
      │
      ▼
File Read
      │
      ▼
Callback returned
```

---

### Thread Pool

libuv has a default thread pool of **4 threads**.

It's used for operations like:

* File system access
* DNS lookups
* Compression
* Cryptography

Network sockets generally use the operating system's asynchronous networking instead of the thread pool.

---

### Interview Answer

> "libuv is a cross-platform C library that implements Node.js's event loop and asynchronous I/O. It provides a thread pool for operations such as file system access and cryptography, enabling Node.js to perform non-blocking operations."

---

# 4. What is the difference between `process.nextTick()`, `setImmediate()`, and `setTimeout()`?

These all schedule work, but at different times.

---

### `process.nextTick()`

Runs **immediately after the current operation**, before the event loop continues.

```js
console.log("A");

process.nextTick(() => {
  console.log("B");
});

console.log("C");
```

Output:

```
A
C
B
```

It has the **highest priority**.

Too many nested `nextTick()` calls can starve the event loop.

---

### `setTimeout(fn, 0)`

Runs after at least the specified delay and during the **Timers** phase.

```js
setTimeout(() => {
  console.log("Timeout");
}, 0);
```

`0` doesn't mean immediate—it means "run on a future timers phase."

---

### `setImmediate()`

Runs during the **Check** phase, usually after the Poll phase.

```js
setImmediate(() => {
  console.log("Immediate");
});
```

It's often used after I/O callbacks.

---

### Execution Priority

```
Current Code

↓

process.nextTick()

↓

Promise microtasks

↓

Event Loop

↓

setTimeout()

↓

setImmediate()
```

*(The relative order of `setTimeout(0)` and `setImmediate()` can vary depending on context, especially outside I/O callbacks.)*

---

### Interview Answer

> "`process.nextTick()` runs before the event loop continues and has the highest priority. `setTimeout()` schedules execution in the timers phase after at least the specified delay. `setImmediate()` executes in the check phase, typically after I/O callbacks."

---

# 5. When should you use Worker Threads instead of Child Processes?

## Worker Threads

Use when you need CPU-intensive JavaScript work.

Examples:

* Image processing
* Video processing
* Encryption
* Data analysis
* AI inference

Advantages:

* Shared memory
* Lower overhead
* Faster communication

---

## Child Processes

Use when you need:

* Complete isolation
* Run another Node.js app
* Execute Python scripts
* Run shell commands

Example:

```js
exec("python train_model.py");
```

---

### Comparison

| Worker Threads         | Child Processes                           |
| ---------------------- | ----------------------------------------- |
| Same process           | Separate process                          |
| Shared memory          | Separate memory                           |
| Lower overhead         | Higher overhead                           |
| Great for CPU-bound JS | Great for external programs and isolation |

---

### Interview Answer

> "Use Worker Threads for CPU-bound JavaScript tasks because they share the same process and communicate efficiently. Use Child Processes when you need process isolation or to execute external applications or scripts."

---

# 6. What are streams and why are they memory-efficient?

Streams process data **incrementally** instead of loading it all into memory.

Without streams:

```
2GB File

↓

Load Entire File

↓

High Memory Usage
```

With streams:

```
Chunk 1

↓

Chunk 2

↓

Chunk 3

↓

Done
```

---

### Example

```js
const fs = require("fs");

const stream = fs.createReadStream("movie.mp4");

stream.on("data", chunk => {
  console.log(chunk.length);
});
```

---

### Types of Streams

* Readable
* Writable
* Duplex
* Transform

---

### Why are they memory-efficient?

Only small chunks are kept in memory at any time.

This makes them ideal for:

* Video streaming
* File uploads
* Large downloads
* Log processing

---

### Interview Answer

> "Streams process data in chunks rather than loading entire files into memory. This significantly reduces memory usage and enables efficient processing of large files or continuous data sources."

---

# 7. How does Node.js handle thousands of concurrent connections?

Node.js does **not** create one thread per request.

Instead:

```
1000 Requests

↓

Event Loop

↓

libuv

↓

Operating System

↓

Callbacks
```

While one request is waiting for I/O:

* Database
* Redis
* File system
* HTTP

the event loop continues serving others.

This is why Node.js scales well for:

* APIs
* Chat apps
* Streaming
* WebSockets

---

### Interview Answer

> "Node.js handles concurrency through its event-driven, non-blocking architecture. Asynchronous I/O operations are delegated to libuv or the operating system, allowing the event loop to continue serving other requests instead of waiting."

---

# 8. How do you prevent blocking the event loop?

Blocking the event loop makes **all** requests wait.

Avoid:

* Large synchronous loops
* `fs.readFileSync()`
* Expensive computations in request handlers

Instead:

* Use asynchronous APIs.
* Process large datasets in batches.
* Stream large files.
* Move CPU-heavy work to Worker Threads.
* Offload long-running jobs to queues (e.g., BullMQ).

---

### Bad Example

```js
const file = fs.readFileSync("big.txt");
```

---

### Better

```js
fs.readFile("big.txt", (err, data) => {
  // handle file
});
```

---

### Interview Answer

> "I avoid synchronous APIs in request handlers, stream large data, use asynchronous I/O, and move CPU-intensive work to Worker Threads or background jobs to keep the event loop responsive."

---

# 9. What causes memory leaks, and how do you detect them?

### Common Causes

* Global variables that keep growing.
* Large caches with no eviction policy.
* Uncleared `setInterval()` or `setTimeout()`.
* Event listeners that aren't removed.
* Closures retaining references.
* Unclosed streams or database connections.

---

### Detecting Leaks

Monitor:

* Heap memory
* Garbage collection frequency
* Heap snapshots over time

Useful tools:

* Chrome DevTools
* Node Inspector
* `clinic.js`
* `heapdump`
* Prometheus + Grafana
* Datadog / New Relic

---

### Interview Answer

> "I monitor heap usage over time, capture and compare heap snapshots, and inspect retained objects to identify what's preventing garbage collection. Common causes include global variables, caches, event listeners, timers, and lingering references."

---

# 10. How would you build a scalable, fault-tolerant Node.js backend?

A production-ready architecture might look like this:

```
                Internet
                    │
             Load Balancer
                    │
      ┌─────────────┴─────────────┐
      │                           │
 Node.js API 1              Node.js API 2
      │                           │
      └─────────────┬─────────────┘
                    │
               Redis Cache
                    │
            Message Queue
          (BullMQ / RabbitMQ)
                    │
          Background Workers
                    │
              PostgreSQL
              (Primary + Replica)
```

### Key Practices

* Run multiple stateless Node.js instances behind a load balancer.
* Use Redis for caching and session storage.
* Offload slow work (emails, reports, notifications) to background workers.
* Use connection pooling for the database.
* Add retries, circuit breakers, and graceful shutdowns.
* Monitor logs, metrics, and traces with tools like Prometheus, Grafana, and OpenTelemetry.
* Deploy with Docker and Kubernetes (or a managed cloud platform) for scaling and self-healing.

### Interview Answer

> "I'd build stateless Node.js services behind a load balancer, use Redis for caching, a message queue for asynchronous jobs, and a reliable database with connection pooling and replicas. I'd containerize the application, deploy multiple instances, implement health checks and graceful shutdowns, and use centralized logging, metrics, and tracing to ensure scalability, observability, and fault tolerance."

