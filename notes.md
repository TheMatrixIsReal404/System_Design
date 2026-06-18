# Week 1 — System Design Notes (Beginner's Guide)
 
> **Goal:** Build a solid foundation in System Design thinking using Zomato as a real-world case study.
> **Timeline:** Day 1 → Day 6
 
---
 
## Table of Contents
 
1. [Day 1 — What is System Design?](#day-1--what-is-system-design)
2. [Day 2 — Core Concepts Every Engineer Must Know](#day-2--core-concepts-every-engineer-must-know)
3. [Day 3 — Building Your First Architecture (Zomato)](#day-3--building-your-first-architecture-zomato)
4. [Day 4 — Reliability & Failure Scenarios](#day-4--reliability--failure-scenarios)
5. [Day 5 — Functional vs Non-Functional Requirements](#day-5--functional-vs-non-functional-requirements)
6. [Day 6 — Finalizing the Architecture + Week 1 Wrap-Up](#day-6--finalizing-the-architecture--week-1-wrap-up)
---
 
## Day 1 — What is System Design?
 
### What Does "System Design" Actually Mean?
 
System Design is the process of **defining the architecture, components, and data flow** of a software system to meet specific requirements.
 
Think of it like designing a city:
- Roads = **Network / APIs**
- Traffic signals = **Load Balancers**
- Buildings = **Servers**
- Water pipes = **Databases / Data flow**
- Power grid = **Infrastructure**
When you build a food delivery app like Zomato, you're not just writing code — you're deciding *how millions of users, restaurants, and delivery partners interact with each other at scale*.
 
### Why Should a Beginner Learn System Design?
 
| Reason | What It Means For You |
|---|---|
| ✅ Cracking interviews | FAANG-style companies ask system design questions |
| ✅ Writing better code | You'll understand *why* architecture decisions are made |
| ✅ Working in teams | You'll communicate better with senior engineers |
| ✅ Building side projects | You'll make smarter technology choices |
 
### The Big Picture: What Happens When You Open Zomato?
 
```
You tap "Zomato" on your phone
        ↓
Your phone sends a request via the Internet
        ↓
DNS translates "zomato.com" → IP address
        ↓
Load Balancer receives the request and routes it
        ↓
One of many App Servers handles your request
        ↓
App Server checks Redis Cache (fast!) for your data
        ↓
If not in cache → Queries the Database
        ↓
Data is returned → Response sent back to your phone
        ↓
You see the Zomato homepage in ~200ms
```
 
Every single step above is a **system design decision**. This week, we'll understand each one.
 
---
 
### Resources for Day 1
 
- **Watch:** Gaurav Sen — System Design Introduction (~15 min)
- **Read:** System Design Primer on GitHub — Introduction section
---
 
## Day 2 — Core Concepts Every Engineer Must Know
 
Before drawing any architecture, you need a shared vocabulary. These are the **6 foundational concepts** that come up in every system design discussion.
 
---
 
### 1. Scalability
 
> **One-line definition:** The system handles growing load without a full redesign.
 
**What it means for a beginner:**
Imagine Zomato starts with 100 users. It works fine. Then it gets 10 million users. If the system crashes or slows down, it wasn't scalable.
 
**Two types of scaling:**
 
| Type | What You Do | Example |
|---|---|---|
| **Vertical Scaling** (Scale Up) | Add more power to one machine (more RAM, faster CPU) | Upgrade your server from 8GB RAM to 64GB RAM |
| **Horizontal Scaling** (Scale Out) | Add more machines | Add 10 more servers instead of upgrading one |
 
> Beginner Tip: Vertical scaling has limits (you can't keep adding RAM forever). Horizontal scaling is preferred for large systems.
 
---
 
### 2. Reliability
 
> **One-line definition:** The system keeps working even when things fail.
 
**What it means for a beginner:**
Hardware breaks. Networks go down. Bugs happen. A *reliable* system is designed assuming failure will happen, and has plans to handle it.
 
**Real example:**
If Zomato's database server crashes during dinner rush, does the app go down completely? Or does it automatically switch to a backup? That's reliability.
 
**Key ideas:**
- **Redundancy** — Have backups for critical components
- **Replication** — Copy your data to multiple places
- **Failover** — Automatic switch to a backup when the primary fails
---
 
### 3. Maintainability
 
> **One-line definition:** Engineers can change and operate the system without pain.
 
**What it means for a beginner:**
A system you built today will need to be updated next year. If the code is spaghetti, if there are no logs, if only one person understands it — it has poor maintainability.
 
**Key ideas:**
- **Operability** — Easy to monitor, manage, and deploy
- **Simplicity** — Avoid unnecessary complexity
- **Evolvability** — Easy to add new features or change existing ones
> Beginner Tip: Code is read far more than it's written. Design for the engineer who will maintain it 6 months from now (it might be you!).
 
---
 
### 4. Availability
 
> **One-line definition:** The percentage of time the system is up and serving requests.
 
**What it means for a beginner:**
Availability is usually expressed as a percentage, called "nines":
 
| Availability | Downtime per Year | Example Use Case |
|---|---|---|
| 99% | ~3.65 days | Internal tools |
| 99.9% | ~8.7 hours | Most web apps (e.g., Zomato) |
| 99.99% | ~52 minutes | Banking, payments |
| 99.999% | ~5 minutes | Hospital systems, aviation |
 
**Formula:**
```
Availability = Uptime / (Uptime + Downtime) × 100%
```
 
> Reliability vs Availability: A system can be *available* (technically online) but *unreliable* (frequently erroring). You want both!
 
---
 
### 5. Latency
 
> **One-line definition:** The time it takes for one request to complete.
 
**What it means for a beginner:**
When you search for "biryani" on Zomato, the time between pressing Enter and seeing results appear = **latency**.
 
**Measured as:**
- **p50 (median):** 50% of requests are faster than this
- **p95:** 95% of requests are faster than this
- **p99:** 99% of requests are faster than this (this is what SLAs use)
**Example:**
"Search results return in < 200ms (p99)" means 99 out of 100 users see results within 200ms.
 
> Beginner Tip: Always ask "latency for *which* operation?" — reading from cache is ~1ms, reading from a database could be ~10-100ms, reading from disk could be ~100ms+.
 
---
 
### 6. Throughput
 
> **One-line definition:** The number of requests the system can handle per second.
 
**What it means for a beginner:**
If Zomato can handle 50,000 requests per second (RPS) — that's its throughput. If 100,000 people order at the same time, the system gets overwhelmed.
 
**Throughput vs Latency:**
 
| Concept | Question It Answers | Unit |
|---|---|---|
| Latency | "How fast does ONE request complete?" | milliseconds (ms) |
| Throughput | "How MANY requests can we handle?" | requests/second (RPS) |
 
> Analogy: A highway with 10 lanes has high **throughput** (many cars). The speed limit determines **latency** (how fast one car travels).
 
---
 
### Concepts Summary Table
 
| Concept | One-Line Definition |
|---|---|
| **Scalability** | System handles growing load without redesign |
| **Reliability** | System keeps working even when things fail |
| **Maintainability** | Engineers can change and operate it without pain |
| **Availability** | % of time the system is up and serving requests |
| **Latency** | Time for one request to complete |
| **Throughput** | Number of requests handled per second |
 
---
 
## Day 3 — Building Your First Architecture (Zomato)
 
### What is Software Architecture?
 
Architecture is the **blueprint** of your system. It shows:
- What **components** exist
- How they **connect** to each other
- What **data flows** where
### The Zomato Architecture — Component by Component
 
Let's build the architecture step-by-step, starting from the user.
 
```
User Browser / App
         │
         ▼
      DNS
         │
         ▼
   Load Balancer
    /      |      \
   ▼       ▼       ▼
App      App      App
Server1  Server2  Server3
   \       |       /
    ▼      ▼      ▼
Database Primary    Redis Cache
         │
         ▼
    Read Replica
```
 
---
 
#### Component 1: DNS (Domain Name System)
 
**What it does:**
DNS is like a phone book for the internet. When you type `zomato.com`, DNS translates that human-readable name into an IP address (like `103.121.122.189`) that computers understand.
 
**Why it matters:**
Without DNS, users would have to remember IP addresses. With DNS, you can also route users to the *nearest* data center (for lower latency).
 
**Key terms:**
- **DNS Record:** Maps domain to IP address
- **TTL (Time to Live):** How long DNS results are cached
---
 
#### Component 2: Load Balancer
 
**What it does:**
A load balancer sits in front of your app servers and **distributes incoming requests evenly** across them. It's the traffic cop of your architecture.
 
**Why you need it:**
If you have 3 app servers and all requests go to Server 1, it will crash. The load balancer spreads the load so no single server is overwhelmed.
 
**Common algorithms:**
 
| Algorithm | How It Works |
|---|---|
| **Round Robin** | Sends requests to servers in rotation (1, 2, 3, 1, 2, 3...) |
| **Least Connections** | Sends to whichever server has fewest active connections |
| **IP Hash** | Same user always goes to same server (useful for sessions) |
 
**Real tools:** Nginx, HAProxy, AWS ALB (Application Load Balancer)
 
---
 
#### Component 3: App Servers
 
**What they do:**
App servers contain your **business logic** — the actual code that processes requests. When you place an order on Zomato:
1. App server receives your request
2. Validates your input
3. Checks the cache for restaurant data
4. Queries the database if needed
5. Returns the result
**Why multiple servers?**
- **Fault tolerance:** If one crashes, others keep serving
- **Scalability:** Add more servers when traffic grows
- **Zero downtime deploys:** Take one down for updates, keep others running
**Real tools:** Node.js, Flask (Python), Spring Boot (Java)
 
---
 
#### Component 4: Database (Primary)
 
**What it does:**
The database stores all **persistent data** — users, restaurants, orders, menus. The Primary database handles all **write operations** (creating, updating, deleting data).
 
**Why "Primary"?**
In a Primary-Replica setup:
- **Primary:** Accepts reads AND writes
- **Read Replica:** Only accepts reads (copies data from Primary)
This separation allows you to handle heavy read traffic without overloading the Primary.
 
**Real tools:** PostgreSQL, MySQL, MongoDB
 
---
 
#### Component 5: Redis Cache
 
**What it does:**
Redis is an **in-memory data store** — it keeps frequently accessed data in RAM so it can be retrieved in microseconds instead of milliseconds.
 
**Why caching matters:**
Without cache:
```
User requests "restaurants near me"
  → App Server queries Database
  → Database scans disk
  → Returns in ~50ms (repeated 1,000,000 times = massive DB load!)
```
 
With cache:
```
First request → hits DB → stores result in Redis
All future requests → hits Redis (RAM) → ~1ms response
```
 
**Real tools:** Redis, Memcached
 
**Cache strategies:**
 
| Strategy | What It Means |
|---|---|
| **Cache Hit** | Data found in cache — fast! |
| **Cache Miss** | Data not in cache — goes to DB, then stores in cache |
| **Cache Eviction** | Removing old data when cache is full (LRU = Least Recently Used) |
 
---
 
#### Component 6: Read Replica
 
**What it does:**
A Read Replica is a **copy of your Primary database** that is kept in sync via **replication**. It only handles SELECT (read) queries.
 
**Why it helps:**
- 80-90% of database operations in most apps are reads (checking menus, browsing restaurants)
- Offload reads to the replica — Primary database has capacity for critical writes
- If Primary fails — replica can be promoted to Primary
---
 
### Full Architecture Flow
 
```
1. User opens Zomato app
2. DNS resolves zomato.com → IP of Load Balancer
3. HTTPS request hits Load Balancer (Nginx / AWS ALB)
4. Load Balancer routes to App Server 2 (round-robin)
5. App Server checks Redis Cache:
   - HIT: Returns restaurant data immediately
   - MISS: Queries Read Replica (for reads) or Primary DB (for writes)
6. Read Replica returns data → App Server caches it in Redis
7. Response returned to user in ~150ms
```
 
---
 
## Day 4 — Reliability & Failure Scenarios
 
### "Design for Failure"
 
One of the most important principles in system design is: **Assume everything will fail, eventually.**
 
- Hard drives fail
- Network cables get cut
- Cloud regions go down (it happens!)
- Code has bugs
A well-designed system **anticipates failure** and has mitigations so users barely notice.
 
### Failure Analysis: Each Component
 
Go through each component of the Zomato architecture and ask:
1. **What can go wrong?**
2. **What's the impact?**
3. **How do we mitigate it?**
---
 
#### DNS Failure
 
| | |
|---|---|
| **Failure Scenario** | DNS server goes down |
| **Impact** | Users can't resolve `zomato.com` — entire app is unreachable |
| **Mitigation** | Use **multiple DNS providers** (e.g., Route53 + Cloudflare). If one goes down, the other takes over. |
 
> Insight: DNS failures are rare but catastrophic because they prevent *any* traffic from reaching your servers. Always use at least 2 DNS providers.
 
---
 
#### Load Balancer Failure
 
| | |
|---|---|
| **Failure Scenario** | The load balancer itself crashes |
| **Impact** | ALL traffic stops — no requests reach app servers |
| **Mitigation** | **Active-Passive LB pair:** Two LBs run side by side. Active one handles traffic. Passive one monitors and takes over instantly if active crashes. |
 
> Insight: This is called "eliminating a Single Point of Failure (SPOF)." Any component where failure = total outage is a SPOF and needs redundancy.
 
---
 
#### App Server Failure
 
| | |
|---|---|
| **Failure Scenario** | One app server crashes mid-request |
| **Impact** | Some users get errors, but others (on other servers) are fine |
| **Mitigation** | **Health checks + auto-replace:** Load balancer pings each server every few seconds. If a server doesn't respond, it's removed from rotation. A new server is automatically spun up (auto-scaling). |
 
> Insight: Because you have multiple app servers, losing one is not catastrophic — only the requests being handled by that server are affected. This is horizontal scaling in action.
 
---
 
#### Database Failure
 
| | |
|---|---|
| **Failure Scenario** | Disk full / data corruption on Primary DB |
| **Impact** | Writes fail; data may be lost |
| **Mitigation** | **Read replicas + automated backups:** Replicas have a copy of the data. Backups allow point-in-time recovery. Replicas can be promoted to Primary. |
 
> Insight: The database is often the hardest component to scale and the most critical to protect. Always have automated backups and test your restore process!
 
---
 
#### Cache Failure
 
| | |
|---|---|
| **Failure Scenario** | Redis memory exhausted |
| **Impact** | Cache becomes full, new items can't be stored → more DB hits → higher latency, possible DB overload |
| **Mitigation** | **Eviction policy (LRU) + memory limits:** LRU (Least Recently Used) automatically removes the oldest unused data when memory is full. Set memory limits so Redis never consumes all system RAM. |
 
> Insight: Redis losing data is *not catastrophic* like a database going down — it just means a cache miss and the data will be re-fetched from the DB. But if *all* cache data is evicted at once (called a "thundering herd"), the DB gets hit by a huge spike. This is why proper eviction policies matter.
 
---
 
### Reliability Summary Table
 
| Component | Failure Scenario | Mitigation |
|---|---|---|
| **DNS** | DNS server goes down | Multiple DNS providers (Route53 + Cloudflare) |
| **Load Balancer** | LB itself crashes | Active-passive LB pair |
| **App Servers** | One crashes mid-request | Health checks + auto-replace |
| **Database** | Disk full / corruption | Read replicas + automated backups |
| **Cache** | Redis memory exhausted | Eviction policy (LRU) + memory limits |
 
---
 
### Key Reliability Principles
 
1. **Eliminate Single Points of Failure (SPOFs)** — Every critical component should have redundancy
2. **Fail fast, recover fast** — Detect failures quickly and route around them
3. **Degrade gracefully** — If cache is down, app still works (just slower). If one region is down, route to another
4. **Test your failures** — Netflix's "Chaos Monkey" randomly kills servers in production to ensure resilience
---
 
## Day 5 — Functional vs Non-Functional Requirements
 
### What Are Requirements?
 
Before designing any system, you must understand **what the system needs to do** and **how well it needs to do it**.
 
Requirements are split into two types:
 
---
 
### Functional Requirements (FR)
 
**Definition:** What the system *does* — the features and behaviours visible to users.
 
These are the **"what"** questions:
- What can users do?
- What can admins do?
- What data is stored?
- What outputs does the system produce?
**Zomato Functional Requirements:**
 
| # | Requirement |
|---|---|
| FR1 | User can search restaurants by location |
| FR2 | User can place an order and track it in real-time |
| FR3 | Restaurant owner can update their menu |
 
---
 
### Non-Functional Requirements (NFR)
 
**Definition:** How well the system performs its functions — quality attributes like speed, scale, and uptime.
 
These are the **"how well"** questions:
- How fast must it respond?
- How many users must it support?
- How often can it be down?
- How secure must it be?
**Zomato Non-Functional Requirements:**
 
| # | Requirement |
|---|---|
| NFR1 | Search results return in < 200ms (p99) |
| NFR2 | System handles 500,000 concurrent users |
| NFR3 | 99.9% uptime (~8.7 hours downtime/year max) |
 
---
 
### How NFRs Drive Architecture Decisions
 
This is the **most important concept of Week 1**:
 
> Non-Functional Requirements don't just describe performance — they force every architecture decision.
 
Let's trace how each NFR becomes an architectural component:
 
| NFR | "Because of this NFR, we need..." |
|---|---|
| 500,000 concurrent users | **Load Balancer** + multiple **App Servers** (can't handle this with 1 server) |
| Search results < 200ms (p99) | **Redis Cache** (DB queries alone would take longer) |
| 99.9% uptime | **Database Replication** + Active-Passive **LB pair** (can't afford single points of failure) |
 
> Critical Insight: In a system design interview, **ALWAYS clarify NFRs before drawing a single box.** The architecture looks completely different for a system serving 1,000 users vs 10,000,000 users.
 
---
 
### FR vs NFR: Quick Test
 
For each requirement below, identify if it's FR or NFR:
 
1. "Users can reset their password" → **FR** (it's a feature)
2. "Password reset email arrives within 10 seconds" → **NFR** (it's performance)
3. "Restaurant owners can upload a menu PDF" → **FR** (it's a feature)
4. "PDF upload handles files up to 50MB" → **NFR** (it's a constraint)
5. "99.99% uptime during weekends" → **NFR** (it's availability)
---
 
### Writing Requirements: Best Practices
 
| Bad NFR | Good NFR |
|---|---|
| "The system should be fast" | "API response time < 200ms at p99" |
| "Handle lots of users" | "Support 500,000 concurrent users" |
| "Should rarely go down" | "99.9% uptime (8.7 hours downtime/year max)" |
| "Should be scalable" | "Handle 2x traffic spike during IPL without manual intervention" |
 
> Rule: Good NFRs are **measurable**. If you can't measure it, you can't enforce it.
 
---
 
## Day 6 — Finalizing the Architecture + Week 1 Wrap-Up
 
### Annotating the Architecture
 
A great architecture diagram doesn't just show *what* the components are — it shows:
- **Protocols** (HTTPS, TCP, etc.)
- **Technologies** (Nginx, PostgreSQL, Redis, Node.js)
- **Data flow labels** (Read/Write, Replication, etc.)
- **Key metadata** (Sessions, Hot Data, Primary, Replica)
### The Final Annotated Zomato Architecture
 
```
         User
            │
         HTTPS
            │
            ▼
   Load Balancer
   (Nginx / AWS ALB)
    /               \
   ▼                 ▼
App Server 1      App Server 2
(Flask / Node)    (Flask / Node)
    \        \      /        /
     \    Read/Write        /
      │        │            │
      │        ▼            │
      │  PostgreSQL          │
      │  (Primary)          │
      │        │            │
      │   Replication       │
      │        │            │
      │        ▼            │
      │  Read Replica        │
      │                     │
      └──────────►──────────┘
              ▼
       Redis Cache
       (Sessions + Hot Data)
```
 
---
 
### Technology Choices Explained
 
#### Why Nginx / AWS ALB as Load Balancer?
 
| Tool | Best For |
|---|---|
| **Nginx** | Self-hosted, highly configurable, also serves static files |
| **AWS ALB** | Cloud-native, auto-scales, integrates with AWS ecosystem |
 
Both support HTTPS termination (handling SSL/TLS so your app servers don't need to).
 
---
 
#### Why Flask / Node.js as App Server?
 
| Framework | Language | Best For |
|---|---|---|
| **Flask** | Python | Simple APIs, ML-backed services, rapid prototyping |
| **Node.js** | JavaScript | High-concurrency I/O, real-time features (chat, tracking) |
 
For Zomato's real-time order tracking → Node.js is great. For ML-based recommendations → Flask is preferred.
 
---
 
#### Why PostgreSQL as the Database?
 
PostgreSQL is a **relational database** that:
- Supports complex queries (JOIN multiple tables)
- Has ACID transactions (data integrity for order payments)
- Handles structured data well (users, orders, menus have clear schemas)
- Scales with read replicas
**When would you use NoSQL instead?**
- User activity logs → **Cassandra** (write-heavy, time-series)
- Product catalog search → **Elasticsearch**
- User sessions → **Redis** (already covered!)
---
 
#### Why Redis for Cache?
 
Redis stores data in **RAM** (memory), which is orders of magnitude faster than disk:
 
| Storage Type | Speed | Typical Size |
|---|---|---|
| CPU Cache | ~1ns | KB |
| RAM (Redis) | ~100ns | GB |
| SSD (PostgreSQL) | ~100μs | TB |
| HDD | ~10ms | TB |
 
Redis is perfect for:
- **Sessions** — "Is this user logged in?"
- **Hot data** — "Top 10 restaurants near me"
- **Rate limiting** — "This user made 100 requests in 1 second, block them"
- **Pub/Sub** — Real-time notifications
---
 
### Database Replication: How It Works
 
```
App Server WRITES  ──────────►  PostgreSQL PRIMARY
                                        │
                                 WAL Logs (Write-Ahead Log)
                                        │
                                        ▼
                                 Read Replica (copy)
                                        ▲
App Server READS   ──────────►  Read Replica
```
 
**WAL (Write-Ahead Log):** Every change to the Primary is written to a log first. The Replica reads this log and applies the same changes to stay in sync. This is called **streaming replication**.
 
**Replication lag:** There's always a tiny delay between a write hitting Primary and appearing in the Replica. For most reads (browsing menus, checking restaurant lists), this is acceptable. For critical reads (checking if your order was confirmed), always read from Primary.
 
---
 
### Week 1 Complete Notes Template
 
```markdown
## Week 1 — System Design Notes
 
### Key Concepts
- **Scalability** — System handles growing load without redesign
- **Reliability** — System keeps working even when things fail
- **Maintainability** — Engineers can change and operate it without pain
- **Availability** — % of time the system is up and serving requests
- **Latency** — Time for one request to complete
- **Throughput** — Number of requests handled per second
 
### Zomato Architecture (see week1-architecture.png)
- Components: DNS → Load Balancer → App Servers → DB → Cache
- Reliability concern for each component:
  - DNS: Use multiple DNS providers (Route53 + Cloudflare)
  - Load Balancer: Active-passive LB pair
  - App Servers: Health checks + auto-replace
  - Database: Read replicas + automated backups
  - Cache: Eviction policy (LRU) + memory limits
 
### Functional vs Non-Functional Requirements (Zomato)
**Functional:**
- User can search restaurants by location
- User can place an order and track it in real-time
- Restaurant owner can update their menu
 
**Non-Functional:**
- Search results return in < 200ms (p99)
- System handles 500,000 concurrent users
- 99.9% uptime (~8.7 hours downtime/year max)
 
### Technology Choices
- Load Balancer: Nginx / AWS ALB
- App Server: Flask / Node.js
- Database: PostgreSQL (Primary + Read Replica)
- Cache: Redis (Sessions + Hot Data)
 
### Resources Consumed
- [ ] Gaurav Sen — System Design Introduction (YouTube)
- [ ] System Design Primer (GitHub) — Introduction
```
 
---
 
## Week 1 Resources
 
| Resource | Type |
|---|---|
| Gaurav Sen — System Design Introduction | YouTube (~15 min) |
| System Design Primer — Introduction | GitHub |
| draw.io | Architecture diagramming tool |
 
---
 
## Week 1 Self-Assessment
 
Test yourself before moving to Week 2:
 
**Q1. What is the difference between Latency and Throughput?**
> Latency = how fast one request completes; Throughput = how many requests per second the system handles
 
**Q2. Why do we put a Load Balancer in front of App Servers?**
> To distribute traffic evenly so no single server is overwhelmed, and to route around failed servers
 
**Q3. What is a Read Replica and why do we use one?**
> A copy of the Primary database that only handles reads, to offload read traffic and improve scalability
 
**Q4. Give one example of a Functional Requirement and one Non-Functional Requirement for Zomato.**
> FR: User can place an order; NFR: 99.9% uptime
 
**Q5. If Redis cache goes down, what happens? Is it catastrophic?**
> No — cache misses mean more DB queries and higher latency, but the system still works. The concern is a "thundering herd" if the entire cache is cold at once.
 
**Q6. What does 99.9% availability mean in hours per year?**
> About 8.7 hours of allowed downtime per year
 
---
 
## What's Coming in Week 2
 
| Topic | Preview |
|---|---|
| **CAP Theorem** | Why you can't have Consistency, Availability, and Partition Tolerance all at once |
| **SQL vs NoSQL** | When to use relational databases vs document/column stores |
| **API Design** | REST vs GraphQL vs gRPC |
| **Message Queues** | Why Zomato uses Kafka for order events |
| **CDN** | How static assets (images, CSS) are served globally |
 
---
 
*Notes compiled: Week 1, System Design Fundamentals*  
*Case study: Zomato food delivery architecture*  
*Level: Beginner → Intermediate*



# 🏗️ Week 2 — System Design: Scaling, Databases & Caching
 
---
 
## 🏗️ System Design — Day 8: Horizontal vs Vertical Scaling & Load Balancers
 
> ### ✏️ Today's SD Goal
> Understand *when* and *why* to scale horizontally vs vertically, and how load balancers distribute traffic.
 
---
 
### 📖 Concepts to Learn
 
- [ ] **Vertical Scaling (Scale Up)** — Add more CPU/RAM to a single machine; simpler but has a hard ceiling
- [ ] **Horizontal Scaling (Scale Out)** — Add more machines; no hard ceiling, but introduces complexity (state, coordination)
- [ ] **Stateless vs Stateful servers** — Horizontal scaling *requires* stateless app servers (session state must live outside)
- [ ] **Load Balancer role** — Distributes incoming requests across N servers; single entry point for clients
- [ ] **Load Balancing algorithms** — Round Robin, Least Connections, IP Hash, Weighted Round Robin
- [ ] **Health checks** — LB pings servers; removes unhealthy nodes from the pool automatically
- [ ] **Layer 4 vs Layer 7 LB** — L4 routes on TCP/IP; L7 routes on HTTP (URL, headers, cookies)
- [ ] **Single Point of Failure (SPOF)** — The LB itself must be redundant (Active-Passive or Active-Active pair)
---
 
### 📐 Mental Model
 
```
          ┌─────────────────┐
Clients → │   Load Balancer  │ ← Health Checks
          └────────┬────────┘
         ┌─────────┼─────────┐
         ▼         ▼         ▼
      Server 1  Server 2  Server 3
         │         │         │
         └────┬────┘         │
              ▼              │
        Shared State    (Redis / DB)
```
 
**Key insight:** App servers are *cheap to clone*; shared state (DB, cache, sessions) is the *hard part*.
 
---
 
### ⚖️ Vertical vs Horizontal — Quick Decision Table
 
| Factor              | Vertical (Scale Up)   | Horizontal (Scale Out)  |
|---------------------|-----------------------|-------------------------|
| Cost                | Exponential beyond ~96 cores | Linear with commodity hardware |
| Downtime required   | Yes (resize instance) | No (add nodes live)     |
| Complexity          | Low                   | High (need LB, stateless design) |
| Failure tolerance   | None (single machine) | High (N-1 redundancy)   |
| When to use         | Early stage, < 100K users | At scale, or >1 AZ needed |
 
---
 
### 📝 Practice Calculation
 
```
Instagram-scale example:
- 1B DAU, 500M photo views/day
- Read QPS: 500M ÷ 86,400 ≈ 5,800 QPS  → Need ~6 app servers (each handles ~1K QPS)
- Peak Read QPS: 5,800 × 2.5 ≈ 14,500 QPS → Need ~15 app servers at peak
 
Vertical limit: A 192-core server might serve 10K QPS max
  → Still need multiple machines + LB beyond that
  → Horizontal wins for sustained internet-scale traffic
```
 
- [ ] Draw the above architecture (clients → LB → app servers → DB) from memory
- [ ] Name one real product that uses each LB algorithm (hint: IP Hash → sticky sessions for chat)
---
 
### 📚 Resources
 
- [ ] Read: **System Design Primer — Load Balancing** (GitHub: donnemartin/system-design-primer)
- [ ] Watch: **Gaurav Sen — Horizontal vs Vertical Scaling** (YouTube)
- [ ] Read: **AWS ELB docs — ALB vs NLB** (Layer 7 vs Layer 4 real-world example)
---
 
### 📋 Create a reference note: `SD-Reference-Scaling-and-LB.md`
 
---
---
 
## 🏗️ System Design — Day 9: Databases — SQL vs NoSQL, Replication & Sharding
 
> ### ✏️ Today's SD Goal
> Know *which database type to pick* for a given problem, and explain replication + sharding to an interviewer confidently.
 
---
 
### 📖 Framework to Memorize
 
- [ ] **SQL (Relational)** — Tables, schemas, ACID transactions, JOINs; strong consistency
- [ ] **NoSQL types** — Key-Value (Redis, DynamoDB), Document (MongoDB), Column (Cassandra), Graph (Neo4j)
- [ ] **CAP Theorem** — A distributed DB can only guarantee 2 of 3: Consistency, Availability, Partition Tolerance
- [ ] **ACID vs BASE** — SQL is ACID (strong guarantees); NoSQL tends toward BASE (eventually consistent)
- [ ] **Read Replica** — Copy of DB that handles reads only; primary handles writes → reduces read load
- [ ] **Replication lag** — Read replica may be milliseconds behind primary (stale reads possible)
- [ ] **Sharding (Horizontal Partitioning)** — Split rows across multiple DBs by a *shard key* (e.g. user_id % N)
- [ ] **Shard key choice** — Bad key = hotspot (e.g. timestamp); Good key = evenly distributed (e.g. hashed user_id)
- [ ] **Consistent Hashing** — Minimises data re-distribution when adding/removing shards
---
 
### 📐 Mental Model — Replication + Sharding Together
 
```
                     ┌──────────────────────┐
Writes  →            │   Primary DB (Shard A) │ ─── Replica A1 (read)
(user_id 0-499K)     └──────────────────────┘ ─── Replica A2 (read)
 
                     ┌──────────────────────┐
Writes  →            │   Primary DB (Shard B) │ ─── Replica B1 (read)
(user_id 500K-1M)    └──────────────────────┘ ─── Replica B2 (read)
```
 
**Rule of thumb:**
- **Replication** → improves *read throughput* and *availability*
- **Sharding** → improves *write throughput* and handles *data volume*
---
 
### 🗂️ When to Pick What
 
| Scenario                              | Choose              | Reason                                      |
|---------------------------------------|---------------------|---------------------------------------------|
| User profiles, financial records      | PostgreSQL / MySQL  | ACID, relational queries, structured schema |
| Session store, rate limiting          | Redis               | Sub-ms latency, TTL support, Key-Value      |
| Social graph (who follows whom)       | Neo4j / DynamoDB    | Graph traversal or flexible schema          |
| Logs, time-series events              | Cassandra           | Write-heavy, wide-column, linear scale      |
| E-commerce product catalog            | MongoDB             | Flexible nested docs (variants, specs)      |
| Chat messages (high write, simple key)| DynamoDB / Cassandra| Partition by conversation_id, no JOINs      |
 
---
 
### 📝 Practice Calculation
 
```
Twitter-like system, choosing DB strategy:
- 300M tweets/day (write-heavy)
- Each tweet: 300 bytes
- Storage/day: 300M × 300B = ~90 GB/day
 
Sharding decision:
- Single MySQL max write throughput: ~5K writes/sec
- Tweet writes/sec: 300M ÷ 86,400 ≈ 3,472 writes/sec
  → Fits on 1 primary! But in 2 years at 2× growth → need sharding
 
Shard key choice:
- ❌ created_at (hotspot — all new tweets hit same shard)
- ✅ hash(user_id) — evenly distributed across shards
```
 
- [ ] Re-derive the shard count needed for WhatsApp (20B messages/day, 5K writes/sec per primary)
- [ ] Note: Does your choice of DB change if you need full-text search? (hint: → Elasticsearch layer)
---
 
### 📚 Resources
 
- [ ] Read: **ByteByteGo — SQL vs NoSQL** blog post
- [ ] Read: **Designing Data-Intensive Applications (Kleppmann)** — Chapter 5: Replication (key chapter)
- [ ] Watch: **Gaurav Sen — Database Sharding** (YouTube)
---
 
### 📋 Create a reference note: `SD-Reference-Databases-Replication-Sharding.md`
 
---
---
 
## 🏗️ System Design — Day 10: Caching — Strategies, Redis & CDN
 
> ### ✏️ Today's SD Goal
> Explain the 4 cache write strategies and *where* to place a cache in a system (client, CDN, server, DB) — without looking at notes.
 
---
 
### 📖 Framework to Memorize
 
- [ ] **Cache hit ratio** — (hits ÷ total requests); aim for >95% in a well-tuned cache
- [ ] **Cache-Aside (Lazy Loading)** — App checks cache first; on miss, fetches from DB, then writes to cache
- [ ] **Write-Through** — Every write goes to cache AND DB simultaneously; cache always fresh, but write latency doubles
- [ ] **Write-Back (Write-Behind)** — Write to cache only, async flush to DB; fast writes, risk of data loss on crash
- [ ] **Read-Through** — Cache sits in front of DB; cache library handles DB fetch on miss transparently
- [ ] **Cache eviction policies** — LRU (Least Recently Used), LFU (Least Frequently Used), TTL expiry
- [ ] **Cache stampede / Thundering Herd** — Many cache misses at once all hit DB; mitigation: mutex lock or probabilistic early expiry
- [ ] **CDN (Content Delivery Network)** — Edge servers geographically close to users; caches static assets (images, JS, CSS, video)
- [ ] **CDN push vs pull** — Push: you upload content to CDN proactively; Pull: CDN fetches from origin on first request
---
 
### 📐 Where to Cache — Layer by Layer
 
```
 [Client Browser]    → Cache: HTTP headers (Cache-Control, ETag), localStorage
        ↓
 [CDN Edge Node]     → Cache: static assets, entire HTML pages (for SSG sites)
        ↓
 [Load Balancer]     → Cache: TLS session resumption (minor)
        ↓
 [App Server]        → Cache: in-memory (local), or Redis (shared across servers)
        ↓
 [Database]          → Cache: DB query cache, buffer pool (InnoDB), materialized views
```
 
**Rule:** Cache as *close to the user* as possible. CDN first, Redis second, DB last.
 
---
 
### 🗂️ Cache Strategy Decision Table
 
| Use Case                             | Strategy          | Why                                                    |
|--------------------------------------|-------------------|--------------------------------------------------------|
| User profile (read-heavy, rare write)| Cache-Aside       | Simple, only cache what's requested                    |
| Product inventory (must never be stale) | Write-Through  | Keeps cache consistent with every purchase             |
| Analytics counters, leaderboards     | Write-Back        | Burst write performance; acceptable eventual flush     |
| Static assets (images, videos)       | CDN + pull        | Geographic proximity, offloads origin bandwidth        |
| Session tokens                       | Redis + TTL       | Fast lookup, automatic expiry without manual cleanup   |
 
---
 
### 📝 Practice Calculation
 
```
YouTube homepage feed — caching impact:
 
Without cache:
- DAU: 2B users, 5 feed loads/day = 10B DB reads/day
- Read QPS: 10B ÷ 86,400 ≈ 115,000 QPS
- At ~500 QPS/DB replica → need ~230 DB replicas ❌ (too expensive)
 
With Redis cache (95% hit rate):
- Cache QPS: 115,000 × 0.95 = 109,250 QPS served from Redis ✅
- DB QPS after cache: 115,000 × 0.05 = 5,750 QPS
- At ~500 QPS/DB replica → need only ~12 replicas ✅
 
Redis sizing:
- Avg feed payload: 2 KB
- Unique cached feeds: 50M (not all 2B users are active at once)
- Cache RAM needed: 50M × 2 KB = ~100 GB → 3× Redis nodes (32 GB each) with replication
```
 
- [ ] Re-derive the Redis node count for a Twitter-like timeline (100M active users, 1 KB avg payload)
- [ ] Note: does cache invalidation strategy change when a user *posts* a new tweet? (hint: write-through on their followers' feeds → fan-out problem)
---
 
### 📚 Resources
 
- [ ] Read: **ByteByteGo — A Crash Course in Caching** blog post
- [ ] Read: **AWS ElastiCache — Redis vs Memcached** (docs.aws.amazon.com)
- [ ] Watch: **Exponent — Caching in System Design Interviews** (YouTube)
- [ ] Read: **Cloudflare — How CDNs Work** (cloudflare.com/learning)
---
 
### 📋 Create a reference note: `SD-Reference-Caching-Strategies.md`

---

# 🏛️ Day 11 — System Design: Storage & Bandwidth Estimation
 
> **Today's Goal:** Learn how to estimate storage requirements and bandwidth for a large-scale system — a critical skill in every system design interview.
 
---
 
## 🧠 Why Does This Matter?
 
When you design a large-scale system (like WhatsApp, Instagram, or YouTube), interviewers **always** ask:
- "How much storage will you need?"
- "What's your expected bandwidth?"
These are called **back-of-the-envelope calculations** — rough estimates that show you understand how systems scale. You don't need to be exactly right; you need to be *reasonably close* and show your reasoning.
 
---
 
## 📚 Framework — The 5 Building Blocks
 
### 1. 🗄️ Storage Estimation
 
**Formula:**
```
Storage = writes/day × avg object size × retention period (years)
```
 
**What each part means:**
| Term | Meaning | Example |
|------|---------|---------|
| `writes/day` | How many new items are created daily | 20 billion messages/day |
| `avg object size` | How big is each item on average | 100 bytes per text message |
| `retention period` | How long do you keep the data | 5 years |
 
**Step-by-step:**
1. Figure out how many items are written **per day**
2. Multiply by the **average size** of one item
3. Multiply by the **number of years** you store data
4. Always account for **replication** (see point 4 below)
---
 
### 2. 📥 Bandwidth — Ingress (Data Coming IN)
 
**Formula:**
```
Ingress Bandwidth = write QPS × avg request size
```
 
**What this means:**
- **QPS** = Queries Per Second (how many requests hit your server every second)
- **Ingress** = data flowing *into* your system (uploads, writes, sends)
**Converting writes/day → QPS:**
```
QPS = writes/day ÷ 86,400 seconds/day
```
> 💡 Quick trick: divide by ~100,000 for a rough estimate
 
**Example:**
```
20 billion msgs/day ÷ 86,400 ≈ 231,000 QPS
231K QPS × 100 bytes = ~23 MB/s ingress
```
 
---
 
### 3. 📤 Bandwidth — Egress (Data Going OUT)
 
**Formula:**
```
Egress Bandwidth = read QPS × avg response size
```
 
**What this means:**
- **Egress** = data flowing *out* of your system (downloads, reads, views)
- Usually **higher** than ingress because users read more than they write
- A common assumption: reads are **2× to 10×** more frequent than writes
**Example:**
```
If ingress = 23 MB/s and reads are 2× writes:
Egress = 23 MB/s × 2 = ~46 MB/s
```
 
---
 
### 4. 🔁 Replication Factor
 
**Rule of thumb:** Multiply your raw storage by **3×**
 
**Why?**
- Real systems store **3 copies** of your data across different servers/data centers
- If one server crashes, your data still exists on the other two
- This is the industry standard (used by Google, AWS, Facebook, etc.)
```
Replicated Storage = raw storage × 3
```
 
**Example:**
```
Raw storage needed: 2 TB/day
With replication: 2 TB × 3 = 6 TB/day actual storage consumed
```
 
---
 
### 5. 🗜️ Compression
 
**Rule of thumb:** Text data can be compressed to **0.3–0.5×** of its original size
 
**What this means:**
- Compression reduces storage needs significantly for text
- A 100-byte message might compress to 30–50 bytes
- Images/videos are already compressed and don't benefit much
```
Compressed Storage = raw storage × compression ratio (0.3 to 0.5)
```
 
> ⚠️ In interviews, mention compression as an optimization but clarify that your base estimate doesn't include it unless asked to.
 
---
 
## 🔢 Key Numbers to Memorize
 
These are essential conversions you'll use constantly:
 
| Unit | Value |
|------|-------|
| 1 KB | 1,000 bytes (or 1,024 — use 1,000 for simplicity) |
| 1 MB | 1,000 KB = 10⁶ bytes |
| 1 GB | 1,000 MB = 10⁹ bytes |
| 1 TB | 1,000 GB = 10¹² bytes |
| 1 PB | 1,000 TB = 10¹⁵ bytes |
| Seconds/day | 86,400 ≈ 100,000 |
| Million | 10⁶ |
| Billion | 10⁹ |
 
---
 
## 📝 Practice: WhatsApp Full Estimation
 
The framework above is applied in full detail to WhatsApp — the gold standard practice problem for capacity estimation.
 
👉 **[View WhatsApp-Capacity-Estimate.md](./WhatsApp-Capacity-Estimate.md)**
 
It covers all assumptions, step-by-step storage and bandwidth calculations, a final summary table, and optimizations worth mentioning in an interview.
 
---
 
## 🧩 How to Approach Estimation in an Interview
 
Follow this structured approach every time:
 
```
1. CLARIFY — Ask about scale, users, retention
   "How many DAU? What's the read/write ratio? How long do we store data?"
 
2. ESTIMATE base numbers
   "Let's assume 1B DAU, each sending 20 messages..."
 
3. CALCULATE storage
   writes/day × object size × years × replication
 
4. CALCULATE bandwidth
   ingress = write QPS × request size
   egress = read QPS × response size
 
5. MENTION optimizations
   "We could apply compression to reduce text storage by ~50%"
   "CDN caching would significantly reduce egress costs for popular content"
```
 
---
 
## ⚠️ Common Beginner Mistakes
 
| Mistake | Fix |
|---------|-----|
| Forgetting replication | Always multiply by 3× unless told otherwise |
| Using bytes when you mean KB | Double-check your units at every step |
| Assuming reads = writes | Reads are almost always 2x–10x more than writes |
| Only estimating one type of data | Account for text, images, videos, metadata separately |
| Not stating assumptions | Always say "I'm assuming X…" before calculating |
 
---
 
## 🔑 Key Takeaways
 
1. **Storage = writes/day × object size × retention × replication**
2. **Ingress = write QPS × request size**
3. **Egress = read QPS × response size** (usually 2×–10× ingress)
4. Always multiply by **3× for replication**
5. Compression gives **0.3–0.5× reduction** for text data
6. Convert writes/day to QPS by dividing by **86,400** (≈ 100,000)
7. State your **assumptions clearly** before every calculation
---
 
## ✅ Today's Checklist
 
- [ ] Work through the WhatsApp numbers in your notes manually (without looking)
- [ ] Commit [`WhatsApp-Capacity-Estimate.md`](./WhatsApp-Capacity-Estimate.md) to GitHub
- [ ] Try estimating storage & bandwidth for **Twitter** (as extra practice)
- [ ] Memorize the unit conversion table (KB → MB → GB → TB → PB)
- [ ] Practice converting writes/day to QPS (divide by 86,400)




## 📝 Day 12 — WhatsApp Capacity Estimation


[Whatsapp Estimation](/Day_12_WhatsApp-Capacity-Estimate.md)

## 📝 Day 13 — Twitter Infrastructure Estimation


[Twitter Infrastructure Estimation](/Twitter-Infrastructure-Estimate.md)


# 🛡️ Day 15 — System Design: TCP vs UDP Fundamentals

## 🔗 Today's SD Goal

Understand the trade-offs between TCP and UDP and *why* each protocol exists.

## ✅ Today's Checklist

- [ ] TCP — connection-oriented, 3-way handshake (SYN → SYN-ACK → ACK), guaranteed ordered delivery, congestion control
- [ ] UDP — connectionless, no handshake, no delivery guarantee, lower overhead
- [ ] When to use which — TCP for HTTP/file transfer/email; UDP for DNS, video streaming, gaming, VoIP
- [ ] Header overhead — TCP header (~20 bytes) vs UDP header (8 bytes)
- [ ] Watch: Hussein Nasser — TCP vs UDP Crash Course (YouTube)

## 📝 Reference Note

[TCP vs UDP Reference](/SD-Reference-TCP-vs-UDP.md)


# 🏗️ Day 16 — HTTP/HTTPS Protocols Deep Dive
 
> 🎯 **Today's SD Goal**
> Be able to draw the anatomy of an HTTP request/response and explain the TLS handshake from memory.
 
---
 
## 1. HTTP Request Anatomy
 
An HTTP request is just **structured plain text** sent over a TCP connection. Every request has four parts:
 
```
METHOD  /path?query=string  HTTP/1.1        <-- Request line
Host: api.example.com                       <-- Headers
Authorization: Bearer abc123
Content-Type: application/json
Content-Length: 27
 
{"name": "ankit", "city": "ldh"}             <-- Body (optional)
```
 
| Part | What it is | Beginner intuition |
|---|---|---|
| **Method** | GET, POST, PUT, PATCH, DELETE | "What do you want me to do?" |
| **Path** | `/users/42` | "Which resource?" |
| **Headers** | Key-value metadata | "Extra context about the request" |
| **Body** | Raw data (JSON, form, file) | "The actual payload" — only present in POST/PUT/PATCH usually |
 
**Common methods cheat sheet:**
 
| Method | Idempotent? | Has body? | Use case |
|---|---|---|---|
| GET | ✅ Yes | ❌ No | Fetch data |
| POST | ❌ No | ✅ Yes | Create new resource |
| PUT | ✅ Yes | ✅ Yes | Replace entire resource |
| PATCH | ❌ No | ✅ Yes | Partially update resource |
| DELETE | ✅ Yes | ❌ Usually no | Remove resource |
 
*Idempotent = calling it 5 times has the same effect as calling it once. GET-ing a user 5 times doesn't change anything; POST-ing "create user" 5 times creates 5 users.*
 
---
 
## 2. HTTP Response Anatomy
 
```
HTTP/1.1 200 OK                              <-- Status line
Content-Type: application/json               <-- Headers
Content-Length: 58
Cache-Control: max-age=3600
 
{"id": 42, "name": "ankit", "city": "ldh"}    <-- Body
```
 
| Part | What it is |
|---|---|
| **Status line** | Protocol version + status code + short text reason |
| **Headers** | Caching rules, content type, cookies, security headers |
| **Body** | The actual data being returned |
 
**Status code families — memorize the *first digit*, not every code:**
 
| Range | Family | Meaning | Beginner intuition |
|---|---|---|---|
| 1xx | Informational | "Hold on, still working" | Rarely seen directly (e.g. 101 Switching Protocols for WebSockets) |
| 2xx | Success | "Got it, all good" | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirection | "Go look over there instead" | 301 Moved Permanently, 304 Not Modified (cache hit) |
| 4xx | Client error | "You messed up" | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| 5xx | Server error | "I messed up" | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |
 
> 💡 **401 vs 403, the classic confusion**
> 401 = "I don't know who you are" (missing/invalid auth)
> 403 = "I know who you are, but you're not allowed" (valid auth, no permission)
 
---
 
## 3. Statelessness — Why Cookies/Sessions/Tokens Exist
 
HTTP is **stateless**: the server treats every request as if it's the first time it has ever seen you. It has no built-in memory of your previous request.
 
```
Request 1: "Log me in as ankit"        → Server: "OK ✅"
Request 2: "Show me my orders"         → Server: "...who are you? 🤷"
```
 
This is a problem because most apps need to know *who you are* across multiple requests (logged-in state, shopping cart, etc). Three solutions evolved:
 
```
┌─────────────────────────────────────────────────────────┐
│  COOKIES        → small key-value pairs stored in browser │
│                    automatically sent with every request  │
│                                                             │
│  SESSIONS        → server keeps state in memory/Redis,     │
│                    cookie holds only a session_id pointer  │
│                                                             │
│  TOKENS (JWT)    → server encodes state INTO the token     │
│                    itself, no server-side storage needed   │
└─────────────────────────────────────────────────────────┘
```
 
| Approach | Where state lives | Scales horizontally? | Revocation |
|---|---|---|---|
| Session (cookie + server store) | Server (Redis/DB) | Needs shared store across servers | Easy — just delete the session |
| JWT (token-based) | Encoded in the token itself | Yes, stateless servers | Hard — must wait for expiry or maintain a blocklist |
 
> This connects back to Day 1–6: a stateless server is what lets you put 10 identical servers behind a load balancer without worrying about "sticky sessions."
 
---
 
## 4. TLS Handshake Walkthrough
 
**Why it exists:** HTTP sends plain text — anyone on the network can read it. HTTPS = HTTP + TLS, which encrypts the data in transit.
 
**The core problem TLS solves:** two strangers (browser + server) need to agree on a *secret encryption key*, over a network that an attacker might be listening on.
 
### The 4-step handshake (simplified, TLS 1.2 style)
 
```
 CLIENT                                          SERVER
   │                                                │
   │  1. "Hello, here are the ciphers I support"    │
   │ ─────────────────────────────────────────────► │
   │                                                │
   │  2. "Here's my certificate + public key"       │
   │ ◄───────────────────────────────────────────── │
   │                                                │
   │  3. Client verifies cert, generates a          │
   │     symmetric session key, encrypts it with    │
   │     server's PUBLIC key, sends it over          │
   │ ─────────────────────────────────────────────► │
   │                                                │
   │  4. Server decrypts with its PRIVATE key,       │
   │     both sides now share the same symmetric key │
   │ ◄═══════════ encrypted traffic flows ══════════►│
```
 
Step-by-step in plain English:
 
1. **Client Hello** — browser says "I want to talk securely, here's a list of encryption methods I know."
2. **Server Hello + Certificate** — server replies with its TLS certificate (issued by a trusted Certificate Authority like Let's Encrypt/DigiCert) containing its **public key**.
3. **Certificate validation + key exchange** — the browser checks the cert is signed by a CA it trusts (chain of trust) and isn't expired/revoked. It then generates a random symmetric key, encrypts it using the server's public key, and sends it.
4. **Session key established** — only the server's matching **private key** can decrypt that message. Now both sides know the same symmetric key and switch to symmetric encryption for the rest of the conversation.
### Why switch from asymmetric → symmetric mid-handshake?
 
| | Asymmetric (RSA/ECDHE) | Symmetric (AES) |
|---|---|---|
| Speed | Slow (heavy math) | Fast (~1000x faster) |
| Key count | 2 keys (public/private) | 1 shared key |
| Used for | Solving the "how do two strangers agree on a secret" problem | Bulk encrypting actual data |
 
**Intuition:** asymmetric crypto is great for *exchanging a secret safely* but too slow to encrypt every byte of every page load. So it's used once, just to securely hand over a symmetric key — then the fast algorithm takes over for the actual data transfer. This is called a **hybrid encryption** approach.
 
> 🔐 Note: Modern TLS 1.3 reduces this to a 1-round-trip handshake (faster), but the *core idea* — asymmetric for key exchange, symmetric for bulk data — stays the same.
 
---
 
## 5. HTTP/1.1 vs HTTP/2 — Head-of-Line Blocking
 
**The problem with HTTP/1.1:** each TCP connection can only have one request "in flight" waiting for its response at a time (without pipelining hacks). If request #1 is slow, request #2 has to wait behind it on that connection — even though they're unrelated.
 
```
HTTP/1.1 (one connection, requests queue up):
 
Conn 1:  [Req A.........] [Req B...] [Req C.....]
                ▲
        Req B and C are BLOCKED until A finishes
        (head-of-line blocking)
 
Workaround: browsers open 6 parallel TCP connections per domain
```
 
**HTTP/2's fix: multiplexing.** Multiple requests/responses share *one* TCP connection, broken into small interleaved "frames," tagged with a stream ID so they can be reassembled in any order.
 
```
HTTP/2 (one connection, multiplexed streams):
 
Conn 1:  [A1][B1][C1][A2][C2][B2][A3]...
          ▲  ▲   ▲
       Different streams interleaved on the SAME connection
       No request blocks another at the HTTP layer
```
 
| | HTTP/1.1 | HTTP/2 |
|---|---|---|
| Connections needed | Multiple (browser opens ~6) | One per origin |
| Head-of-line blocking | Yes, at HTTP layer | No, at HTTP layer (but can still happen at TCP layer — fixed by HTTP/3/QUIC) |
| Header overhead | Repeated raw text each request | Compressed (HPACK) |
 
---
 
## 🔗 Related Notes
- Practice problems for today: see `day16-practice.md`
- Builds on: Day 1–6 (load balancers, stateless servers)
- Connects to: Day 11–13 (bandwidth estimation — TLS adds ~1-2 extra round trips of latency, relevant when estimating P99 latency budgets)
## ✅ Concepts Checklist
- [ ] HTTP request anatomy — method, path, headers, body
- [ ] HTTP response anatomy — status line, headers, body, status code families
- [ ] Statelessness — why cookies/sessions/tokens exist
- [ ] TLS handshake — cert validation, asymmetric → symmetric session key
- [ ] HTTP/2 multiplexing vs HTTP/1.1 head-of-line blocking


# Day 17 — REST APIs and API Design Principles
 
## 🎯 Today's Goal
Internalize what makes an API "RESTful" — this is the foundation for Day 19's hands-on design task.
 
---
 
## 1. Resources & URIs — Nouns, Not Verbs
 
REST (REpresentational State Transfer) models everything in your system as a **resource** — a noun — identified by a unique URI. The HTTP verb (GET/POST/PUT/PATCH/DELETE) is what tells the server what to *do* to that resource. The action should never live in the URL itself.
 
```
✅ GOOD (noun-based, verb describes action)
GET    /users/123        → read user 123
POST   /users             → create a new user
DELETE /users/123         → remove user 123
 
❌ BAD (verb baked into the path)
GET /getUser?id=123
POST /createUser
POST /deleteUser?id=123
```
 
**Why this matters:** once verbs leak into the path, you've quietly reinvented RPC (Remote Procedure Call) and lost most of REST's benefits — predictable URLs, cacheability, and the ability to reuse the same resource path across different HTTP methods.
 
### Naming conventions
| Rule | Good | Bad |
|---|---|---|
| Use plural nouns | `/users` | `/user` |
| Lowercase, hyphenated | `/order-items` | `/orderItems` |
| Nest to show relationships | `/users/123/orders` | `/getOrdersForUser?id=123` |
| Don't nest too deep (max ~2-3 levels) | `/users/123/orders/456` | `/users/123/orders/456/items/789/reviews/1` |
| No trailing verbs | `/orders/456/cancel` is a borderline exception (see note below) | `/cancelOrder/456` |
 
> **Note on action-like endpoints:** Sometimes a pure CRUD mapping doesn't fit reality (e.g., "cancel an order" isn't really "delete"). The pragmatic RESTful approach is to model the action as a sub-resource or state change: `POST /orders/456/cancel` or `PATCH /orders/456` with `{"status": "cancelled"}`. Both are debated in practice — know the tradeoff, don't treat REST as religious law.
 
### Resource hierarchy (mental model)
 
```
/users
  └── /users/123                    (a specific user)
        └── /users/123/orders        (that user's orders — collection)
              └── /users/123/orders/456   (one specific order)
```
 
- A path ending in a **collection name** (`/users`, `/orders`) → operates on the *set*.
- A path ending in an **ID** (`/users/123`) → operates on *one item*.
---
 
## 2. HTTP Verbs as CRUD
 
| HTTP Verb | CRUD op | Idempotent? | Safe?* | Typical body | Example |
|---|---|---|---|---|---|
| `GET` | Read | ✅ Yes | ✅ Yes | None | `GET /users/123` |
| `POST` | Create | ❌ No | ❌ No | New resource data | `POST /users` |
| `PUT` | Update (full replace) | ✅ Yes | ❌ No | Entire resource | `PUT /users/123` |
| `PATCH` | Update (partial) | ⚠️ Not guaranteed | ❌ No | Only changed fields | `PATCH /users/123` |
| `DELETE` | Remove | ✅ Yes | ❌ No | None (usually) | `DELETE /users/123` |
 
*\*"Safe" means the request doesn't change server state — it's read-only.*
 
### Idempotency, explained simply
**Idempotent** = calling it once or calling it 100 times in a row produces the *same end state* on the server.
 
```
PUT /users/123 {"name": "Ankit"}     → user 123's name is "Ankit"
PUT /users/123 {"name": "Ankit"}     → still "Ankit" (no change, same result)
                                         ✅ idempotent
 
POST /users {"name": "Ankit"}        → creates user #501
POST /users {"name": "Ankit"}        → creates ANOTHER user #502
                                         ❌ not idempotent — each call adds a new record
```
 
### PUT vs PATCH — the question that trips up beginners
| | `PUT` | `PATCH` |
|---|---|---|
| Sends | The **whole** resource | Only the **fields that changed** |
| If you omit a field | It gets wiped/reset | Untouched fields stay as-is |
| Example | `PUT /users/123 {"name":"Ankit","email":"a@x.com"}` | `PATCH /users/123 {"email":"a@x.com"}` |
 
Rule of thumb: if your client only has *part* of the data to update, use `PATCH`. If it has the complete object, `PUT` is cleaner and safer (no risk of accidentally dropping fields).
 
---
 
## 3. Statelessness
 
**Stateless** means the server stores **no memory of previous requests** from a client between calls. Every single request must carry everything the server needs to process it — auth tokens, headers, parameters — as if it were the very first request ever made.
 
### Stateless vs Stateful — side by side
 
```
STATEFUL (session-based, NOT RESTful)
─────────────────────────────────────
Client → "login"        → Server creates session, stores it in memory on Server A
Client → "get my orders" → MUST hit Server A again (it's the only one that remembers the session)
                            ❌ breaks if Server A goes down, hard to load-balance
 
 
STATELESS (RESTful)
─────────────────────────────────────
Client → "get my orders" + Authorization: Bearer <token>  → Server A handles it, no memory needed
Client → "get my orders" + Authorization: Bearer <token>  → Server B handles it just as well
                            ✅ any server can handle any request
```
 
### Why this matters for scaling (ties back to Day 8-10 notes)
Because no single server "owns" a client's session, requests can be load-balanced across *any* server in the fleet. This is what makes horizontal scaling clean — you can add or remove servers without worrying about which one "remembers" a given user. Session data, if needed, lives in the request itself (e.g., a JWT token) or in a shared external store (e.g., Redis) — never in one server's local memory.
 
---
 
## 4. Status Codes
 
Status codes are grouped by their leading digit. You don't need to memorize all of them — just internalize the *ranges* and the handful of codes you'll see constantly.
 
| Range | Meaning | You'll use these constantly |
|---|---|---|
| `2xx` | Success | `200`, `201`, `204` |
| `4xx` | Client made a mistake | `400`, `401`, `403`, `404` |
| `5xx` | Server made a mistake | `500`, `503` |
 
### The essential codes
 
| Code | Name | When to use it |
|---|---|---|
| `200` | OK | Generic success — `GET`/`PUT`/`PATCH` request succeeded, body returned |
| `201` | Created | A `POST` successfully created a new resource (return the new resource + its URI in `Location` header) |
| `204` | No Content | Request succeeded but there's nothing to return (e.g., a successful `DELETE`) |
| `400` | Bad Request | Malformed request — invalid JSON, missing required field |
| `401` | Unauthorized | "I don't know who you are" — missing/invalid credentials |
| `403` | Forbidden | "I know who you are, but you're not allowed to do this" |
| `404` | Not Found | Resource doesn't exist |
| `500` | Internal Server Error | Something broke on the server, unhandled |
| `503` | Service Unavailable | Server temporarily can't handle the request (overloaded, maintenance, down dependency) |
 
### The classic confusion: 401 vs 403
```
401 Unauthorized  →  "Who ARE you?"        (authentication problem — no/bad token)
403 Forbidden      →  "I know you, but NO."  (authorization problem — valid token, insufficient permission)
```
 
---
 
## 5. Versioning, Pagination & Idempotency (applied)
 
### Versioning — letting your API evolve without breaking clients
| Approach | Example | Pros | Cons |
|---|---|---|---|
| URI versioning | `/v1/users` | Simple, visible, easy to route/cache | Pollutes the URL, "not very RESTful" purists complain |
| Header versioning | `Accept: application/vnd.myapi.v1+json` | Keeps URLs clean | Harder to test in a browser, less discoverable |
| Query param | `/users?version=1` | Easy to default | Easy to forget/omit accidentally |
 
Most real-world public APIs (Stripe, Twitter/X, GitHub) use **URI versioning** (`/v1/...`) because it's the most explicit and cache-friendly, even though it's technically the "least pure" REST approach.
 
### Pagination — don't return 10 million rows in one response
| Style | Example | Pros | Cons |
|---|---|---|---|
| Offset-based | `/orders?page=2&limit=20` | Simple, supports "jump to page N" | Slow on large offsets, breaks if rows are added/removed mid-scroll |
| Cursor-based | `/orders?cursor=eyJpZCI6NDU2fQ&limit=20` | Stable even with inserts/deletes, fast at scale | No "jump to page 50", slightly more complex to implement |
 
```
Offset-based flow:
  page=1, limit=20  → rows 1-20
  page=2, limit=20  → rows 21-40   (breaks if row 5 got deleted in between!)
 
Cursor-based flow:
  cursor=null        → first 20 rows + nextCursor="abc"
  cursor="abc"        → next 20 rows + nextCursor="def"   (always correct, regardless of inserts/deletes)
```
 
### Idempotency keys — applying idempotency to POST
`POST` is naturally *not* idempotent (each call creates a new thing). But what if a client's network drops right after sending a "charge $50" request, and it retries? You don't want to charge the customer twice.
 
**Solution:** the client generates a unique `Idempotency-Key` header and sends it with the request. The server remembers keys it has already processed (briefly, e.g., 24h) and returns the *original* result instead of repeating the side effect.
 
```
POST /payments
Idempotency-Key: 8f14e45f-ceea-467e-9a83-...
Body: {"amount": 50, "currency": "USD"}
 
→ First call: charges $50, stores result against this key
→ Retry with same key: server sees it's already processed, returns the SAME response, does NOT charge again
```
 
This is exactly how Stripe's API works in production — a great real-world example to look at for Day 17's practice exercise.
 
---
 
## ✅ Concept Checklist
- [ ] Resources & URIs — nouns, not verbs (`/users/123` not `/getUser`)
- [ ] HTTP verbs as CRUD — `GET` read, `POST` create, `PUT`/`PATCH` update, `DELETE` remove
- [ ] Statelessness — every request carries everything the server needs
- [ ] Status codes — `2xx`/`4xx`/`5xx` ranges, and the 401 vs 403 distinction
- [ ] Versioning, pagination & idempotency — `/v1/...`, `?page=2&limit=20`, idempotent `PUT` vs non-idempotent `POST`
---
 
## 📝 Practice (fill this in yourself — use as your answer key check, not a shortcut)
 
Pick one real API you've used (GitHub, Stripe, Twitter/X, Reddit, etc.) and fill in:
 
| Question | Your notes |
|---|---|
| Which API did you pick? | |
| List 3-4 resources and their URIs | |
| Which HTTP verbs does it use, and where? | |
| Which status codes have you actually seen it return? | |
| Is it URI-versioned, header-versioned, or unversioned? | |
| Offset or cursor pagination? | |
 
**3 things it does well:**
1.
2.
3.
 
**1 thing you'd design differently:**
1.
 
---
 
## 🔗 Related Notes
- `SD-Reference-Scaling-and-LB.md` — why statelessness enables horizontal scaling
- `SD-Reference-TCP-vs-UDP.md` — Day 15, the transport layer underneath every HTTP call
- Day 19 — hands-on API design task (builds directly on today)
---
 
### Suggested commit
```
docs(day17): add REST API and API design principles notes
```