# Week 1 — System Design Notes
 
---
 
## Day 1 — Introduction to System Design
 
### Resources
- 📺 Watch: [Gaurav Sen — System Design Introduction](https://www.youtube.com/watch?v=...) (~15 min)
- 📖 Read: [System Design Primer on GitHub](https://github.com/donnemartin/system-design-primer) — Introduction section
---
 
### Key Concepts
 
| Concept | One-Line Definition |
|---|---|
| **Scalability** | System handles growing load without redesign |
| **Reliability** | System keeps working even when things fail |
| **Maintainability** | Engineers can change and operate it without pain |
| **Availability** | % of time the system is up and serving requests |
| **Latency** | Time for one request to complete |
| **Throughput** | Number of requests handled per second |
 
---
 
## Day 2 — Zomato Architecture Sketch
 
### SD Hands-On Exercise
> Open [draw.io](https://draw.io) and sketch the Zomato architecture.
 
### High-Level Architecture Flow
 
```
User Browser / App
        |
       DNS
        |
   Load Balancer
   /     |     \
App1  App2   App3
  \    |    /      \
  Database Primary   Redis Cache
        |
   Read Replica
```
 
### Component Descriptions
 
| Component | Role |
|---|---|
| **User Browser / App** | Client entry point — web or mobile |
| **DNS** | Resolves domain to IP; first hop in every request |
| **Load Balancer** | Distributes traffic across app servers |
| **App Server 1 / 2 / 3** | Stateless application logic layer |
| **Database Primary** | Single source of truth for all writes |
| **Read Replica** | Offloads read traffic from the primary DB |
| **Redis Cache** | In-memory store for sessions and hot data |
 
---
 
## Day 3 — Reliability Concerns per Component
 
> For each component, write one reliability concern in your notes.
 
### Failure Scenarios & Mitigations
 
| Component | Failure Scenario | Mitigation |
|---|---|---|
| **DNS** | DNS server goes down | Use multiple DNS providers (Route53 + Cloudflare) |
| **Load Balancer** | LB itself crashes | Active-passive LB pair |
| **App Servers** | One crashes mid-request | Health checks + auto-replace |
| **Database** | Disk full / corruption | Read replicas + automated backups |
| **Cache** | Redis memory exhausted | Eviction policy (LRU) + memory limits |
 
### Commit
```
docs: add week1 system design architecture diagram and notes
```
 
---
 
## Day 4 — Functional vs Non-Functional Requirements
 
### Case Study: Zomato
 
#### ✅ Functional Requirements
These define **what the system does**:
 
| # | Requirement |
|---|---|
| 1 | User can search restaurants by location |
| 2 | User can place an order and track it in real-time |
| 3 | Restaurant owner can update their menu |
 
#### ⚡ Non-Functional Requirements (NFRs)
These define **how well the system does it**:
 
| # | Requirement |
|---|---|
| 1 | Search results return in < 200ms (p99) |
| 2 | System handles 500,000 concurrent users |
| 3 | 99.9% uptime (~8.7 hours downtime/year max) |
 
---
 
### 💡 Why Non-Functional Requirements Drive Architecture
 
> NFRs are what force every design decision:
> - **"500k concurrent users"** → you need load balancing
> - **"200ms p99"** → you need caching
> - **"99.9% uptime"** → you need replication
>
> Always clarify NFRs in system design interviews **before drawing a single box.**
 
---
 
## Day 5 — Finalizing draw.io Architecture
 
### Annotated Architecture Diagram
 
#### Layer 1 — Entry & Routing
```
[User] --HTTPS--> [Load Balancer: Nginx / AWS ALB]
                          |              |
               [App Server 1]     [App Server 2]
               Flask / Node       Flask / Node
```
 
#### Layer 2 — Data Layer
```
[App Server 1] --Read/Write--> [PostgreSQL Primary]
[App Server 2] --Read/Write-->        |
                                  Replication
                                      |
                              [Read Replica]
 
[App Server 1] --> [Redis Cache: Sessions + Hot Data]
[App Server 2] --> [Redis Cache: Sessions + Hot Data]
```
 
### Component Annotations
 
| Component | Technology | Notes |
|---|---|---|
| Load Balancer | Nginx / AWS ALB | Routes HTTPS traffic; terminates SSL |
| App Servers | Flask / Node.js | Stateless; horizontally scalable |
| Primary DB | PostgreSQL | All writes go here |
| Read Replica | PostgreSQL | Async replication; handles read traffic |
| Cache | Redis | Sessions + frequently accessed (hot) data |
 
### Export & Commit
- Save PNG export as `system-design-notes/week1-architecture.png`
- Commit with message: `docs: add week1 architecture diagram`
---
 
## Day 6 — Week 1 Complete: Full Notes Finalization
 
### Summary Checklist
 
- [x] Watched: Gaurav Sen — System Design Introduction (YouTube)
- [x] Read: System Design Primer (GitHub) — Introduction
- [x] Documented key concepts (Scalability, Reliability, etc.)
- [x] Sketched Zomato architecture on draw.io
- [x] Noted reliability concerns per component
- [x] Wrote Functional & Non-Functional requirements for Zomato
- [x] Finalized and annotated architecture diagram
- [x] Exported `week1-architecture.png`
### Final Commit
```
docs: finalize week1 system design notes
```
 
---
 
## Week 1 — Quick Revision Card
 
### The 6 Core Concepts at a Glance
 
```
Scalability      → Handle more load
Reliability      → Survive failures
Maintainability  → Easy to change
Availability     → Stay online (99.9%+)
Latency          → Fast per request (<200ms)
Throughput       → High requests/sec
```
 
### The Zomato Stack (Mental Model)
```
User → DNS → LB (Nginx) → App (Flask/Node) → PostgreSQL (Primary + Replica)
                                           ↘ Redis (Cache)
```
 
### NFR → Architecture Decision Map
```
500k users   →  Load Balancer (horizontal scaling)
200ms p99    →  Redis Cache (hot data)
99.9% uptime →  DB Replication + Health Checks
```
 
---