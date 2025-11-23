---
title: System Design 101 - The Architecture of Scale
date:
description: The fundamental principles of building scalable, reliable, and maintainable distributed systems.
tags:
  - system-design
  - architecture
  - scalability
  - database
  - interview
  - LLM
---

> [!abstract] The Golden Rule
> **"It Depends."**
> Every architectural decision is a trade-off. You trade consistency for availability. You trade latency for throughput. You trade development speed for performance.

## 1. Core Concepts (The Non-Negotiables)

### Scalability
The ability of a system to cope with increased load (users, data, requests).
* **Vertical Scaling (Scale Up):** Buying a bigger machine (More RAM, better CPU).
    * *Pros:* Simple. No code changes.
    * *Cons:* Hard limit (Hardware cap). Single Point of Failure (SPOF).
* **Horizontal Scaling (Scale Out):** Buying *more* machines.
    * *Pros:* Infinite theoretical scale. Resilient.
    * *Cons:* Complexity. Distributed data consistency issues.



### Reliability
The probability that the system performs correctly during a specific time.
* **Availability:** $Availability = \frac{Uptime}{Uptime + Downtime}$
    * **99.9% (3 nines):** ~8.7 hours downtime/year.
    * **99.999% (5 nines):** ~5 minutes downtime/year (The Holy Grail).

---

## 2. Load Balancing (The Traffic Cop)
A Load Balancer (LB) sits between the client and the server farm. It distributes requests to prevent any single server from crashing.



| Type | Layer (OSI) | Description | Example |
| :--- | :--- | :--- | :--- |
| **L4 LB** | Transport | Routes based on IP + Port. "Dumb" but ultra-fast. | HAProxy (TCP mode) |
| **L7 LB** | Application | Routes based on URL, Headers, Cookies. Smart logic. | NGINX, AWS ALB |

> [!tip] Algorithms
> * **Round Robin:** Circular order (1 -> 2 -> 3 -> 1).
> * **Least Connections:** Send to the server with fewest active users.
> * **Consistent Hashing:** Critical for Distributed Caching. Ensures user X always goes to Server Y.

---

## 3. Databases: The Storage Layer 💾
The biggest bottleneck in 99% of systems.

### SQL vs. NoSQL
* **Relational (SQL):** Structured, strict schema, ACID transactions.
    * *Use case:* Banking, Inventory, User Auth. (`PostgreSQL`, `MySQL`)
* **Non-Relational (NoSQL):** Unstructured, flexible, high throughput.
    * *Use case:* Social feeds, Analytics, IoT logs. (`MongoDB`, `Cassandra`, `Redis`)

### The CAP Theorem
In a distributed data store, you can only pick **two** of these three:
1.  **C**onsistency: Every read receives the most recent write or an error.
2.  **A**vailability: Every request receives a (non-error) response, without the guarantee that it contains the most recent write.
3.  **P**artition Tolerance: The system continues to operate despite an arbitrary number of messages being dropped/delayed by the network between nodes.

> [!danger] Reality Check
> In a distributed system (like the Cloud), **Partition Tolerance (P)** is unavoidable. Networks fail.
> So, you really only have a choice between **CP** (Banking systems - fail if network breaks) or **AP** (Facebook likes - show old data if network breaks).



[Image of CAP Theorem diagram]


---

## 4. Caching (The Speed Cheat Code) ⚡
Caching involves saving the result of an expensive computation/query in fast memory (RAM) closer to the user.

* **Client Caching:** Browser Cache, Cookies.
* **CDN (Content Delivery Network):** Caches static assets (images, CSS) at the "Edge" (servers physically close to the user). Cloudflare.
* **Server Caching:** `Redis` or `Memcached`. Stores DB query results.

**Caching Strategies:**
1.  **Cache-Aside:** Application checks Cache first. If empty, check DB, then update Cache. (Most common).
2.  **Write-Through:** Write to Cache and DB simultaneously. (Data consistency is high, write is slow).

---

## 5. Communication Protocols

| Protocol | Type | Use Case |
| :--- | :--- | :--- |
| **HTTP/REST** | Request/Response | Standard web APIs. Human readable. |
| **gRPC** | RPC (Binary) | Microservice-to-Microservice. Ultra-fast (Protobufs). |
| **WebSocket** | Full Duplex | Real-time chat, Live Dashboards (Socket.IO). |
| **GraphQL** | Query Language | Frontend needs specific data shape. Prevents over-fetching. |

---

## 6. Asynchronous Processing (Decoupling)
If a user uploads a video, do not make them wait for the transcoding to finish.
**Use a Message Queue.**

1.  **Producer:** User uploads video. API sends message to Queue. Returns "Upload Successful".
2.  **Queue:** `Kafka` or `RabbitMQ` holds the message.
3.  **Consumer:** A worker server picks up the message, processes the video, and updates the DB hours later.

*Benefit:* If the traffic spikes, the Queue buffers the requests so your servers don't crash.

---

## 7. The "Twitter" Thought Experiment
*Design a generic social media feed.*

1.  **User posts a tweet:**
    * Write to DB (SQL for user data, NoSQL for the tweet text).
    * Fan-out: Push the tweet ID to the "Feed Cache" (Redis) of every follower.
2.  **User views feed:**
    * **Do not** query the DB: `SELECT * FROM tweets WHERE user_id IN (following_list)`. This is $O(N^2)$ and kills the DB.
    * **Do** query the Redis Cache: `GET user_feed_123`. This is $O(1)$.
3.  **Celebrity Problem (Justin Bieber):**
    * Justin has 100M followers. You cannot update 100M Redis caches instantly.
    * *Solution:* Hybrid approach. For celebrities, pull their tweets at *read time* and merge with the cached feed.

---

## Linked Notes
* [[Kubernetes-Basics]] - Orchestrating these distributed components.
* [[Docker-Ultimate-Guide]] - Packaging the microservices.
* [[SQL-Injection-Methodology]] - Securing the database layer.