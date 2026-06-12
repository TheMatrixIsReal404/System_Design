# 🗄️ SD Reference — Databases, Replication & Sharding

> Quick-review cheat sheet. Read this before any system design interview.

---

## 🔀 SQL vs NoSQL — Decision Framework

**Pick SQL (PostgreSQL / MySQL) when:**
- You need ACID transactions (payments, inventory, bookings)
- Data is relational with complex JOINs
- Schema is well-defined and stable
- Strong consistency is non-negotiable

**Pick NoSQL when:**
- Schema is flexible or evolving rapidly
- You need massive horizontal write scale
- Data access patterns are simple (no JOINs)
- Eventual consistency is acceptable

---

## 🗂️ NoSQL Type Cheat Sheet

| Type | Examples | Data Model | Best For |
|---|---|---|---|
| **Key-Value** | Redis, DynamoDB | `key → value` | Sessions, caching, rate limiting, leaderboards |
| **Document** | MongoDB, Firestore | JSON-like docs | Product catalogs, user profiles, CMS |
| **Wide-Column** | Cassandra, HBase | Rows + dynamic columns | Time-series, logs, IoT, chat messages |
| **Graph** | Neo4j, Amazon Neptune | Nodes + edges | Social graphs, recommendations, fraud detection |
| **Search** | Elasticsearch | Inverted index | Full-text search, log analysis |

---

## 📐 CAP Theorem — 3 Guarantees, Pick 2

```
         Consistency
              △
              │
    CA ───────┼─────── CP
              │
    (no       │    (consistent
  partition)  │     under split)
              │
    AP ───────┴─────────────
  (available       Partition
   under split)    Tolerance
```

| Combination | Real Systems | Trade-off |
|---|---|---|
| **CP** | HBase, Zookeeper, etcd | Returns error during partition instead of stale data |
| **AP** | Cassandra, CouchDB, DynamoDB | Returns stale data during partition instead of error |
| **CA** | Traditional RDBMS (single node) | Assumes no network partition (only valid on one machine) |

> **In practice:** Network partitions always happen in distributed systems. You are always choosing between **C** and **A** when a partition occurs.

---

## 🔁 Replication — Key Facts

```
Writes →  [Primary DB]  ──async replication──→  [Read Replica 1]
                        ──async replication──→  [Read Replica 2]
Reads  →  [Read Replica 1 or 2]
```

| Term | Meaning |
|---|---|
| **Synchronous replication** | Primary waits for replica to confirm write → zero data loss, higher latency |
| **Asynchronous replication** | Primary writes immediately, replica catches up → possible replication lag |
| **Replication lag** | Replica may be ms–seconds behind; stale reads possible |
| **Read-your-writes consistency** | After a user writes, route their reads to primary (not replica) |
| **Failover** | If primary dies, promote a replica to primary (manual or automatic) |

**When to add read replicas:**
- Read:Write ratio is 10:1 or higher
- Primary CPU > 70% from read load
- Read QPS approaching single DB limit (~5K QPS for MySQL)

---

## 🧩 Sharding — Key Facts

**Sharding = splitting rows across multiple DB instances by a shard key.**

```
hash(user_id) % 4:

user_id=1001 → Shard 0    user_id=1002 → Shard 2
user_id=1003 → Shard 3    user_id=1004 → Shard 0
```

### Shard Key Selection Rules

| Rule | ✅ Good | ❌ Bad |
|---|---|---|
| Even distribution | `hash(user_id)` | `created_at` (time-based hotspot) |
| Query locality | `conversation_id` for chat | Splitting a user's data across shards |
| Avoid cross-shard JOINs | Keep related data on same shard | Sharding on a column you filter by AND join on |
| Low cardinality risk | High-cardinality key (user_id) | `country_code` (200 values, uneven traffic) |

### Hotspot Problem

```
❌ Shard by date:
   All new writes → today's shard (overwhelmed)
   Old shards → idle

✅ Shard by hash(user_id):
   New writes spread evenly across all shards
```

---

## 🔄 Consistent Hashing (for Dynamic Shard Counts)

```
Normal hashing:  shard = user_id % N
  Problem: add 1 shard (N+1) → almost every key remaps → massive migration

Consistent hashing:
  Keys and shards placed on a ring
  Each key maps to the nearest shard clockwise
  Add 1 shard → only ~1/N keys remaps ✅

Used by: Amazon DynamoDB, Apache Cassandra, CDN routing
```

---

## 📝 Sharding Math — How Many Shards?

```
Given:
- Total write QPS needed: 20,000 writes/sec
- Single primary DB write capacity: 5,000 writes/sec

Shards needed = ceil(20,000 ÷ 5,000) = 4 shards

With 2× headroom for growth: 8 shards (provision for scale)
```

---

## ❓ Common Interview Questions on This Topic

- *"When would you choose NoSQL over SQL?"*
  → Schema flexibility, massive write scale, no JOINs needed, horizontal partitioning natural

- *"How do you handle a hot shard?"*
  → Re-shard with better key, add virtual nodes, or application-level load balancing

- *"What's the difference between replication and sharding?"*
  → Replication = same data on multiple nodes (redundancy/reads); Sharding = different data on each node (scale/writes)

- *"Can you shard a relational database?"*
  → Yes (Vitess for MySQL, Citus for PostgreSQL), but JOINs across shards are expensive/impossible

---

*Day 9 · Week 2 · System Design Series*
