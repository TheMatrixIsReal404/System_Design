# ⚡ SD Reference — Caching Strategies, Redis & CDN

> Quick-review cheat sheet. Read this before any system design interview.

---

## 🗺️ Cache Placement — Layer by Layer

```
[User's Browser]
  └── HTTP cache (Cache-Control, ETag, Last-Modified)
        ↓  miss
[CDN Edge Node]  (Cloudflare, CloudFront, Fastly)
  └── Static assets, full pages (SSG), API responses with long TTL
        ↓  miss
[App Server — Local Cache]
  └── In-process memory (e.g. Guava Cache, LRU dict) — per-instance, not shared
        ↓  miss
[Distributed Cache]  (Redis, Memcached)
  └── Shared across all app servers; user profiles, sessions, feeds, counters
        ↓  miss
[Primary Database]
  └── Source of truth
```

**Golden Rule:** Cache as close to the user as possible. Every layer missed adds latency.

---

## 📝 4 Cache Write Strategies

### 1. Cache-Aside (Lazy Loading)
```
Read:
  App → check cache
    HIT  → return cached value
    MISS → read from DB → write to cache → return value

Write:
  App → write to DB → invalidate (delete) cache key
```
- ✅ Only caches what's actually requested
- ✅ Cache failure doesn't break reads (just slower)
- ❌ Cache miss = 3 round trips (check, DB read, cache write)
- ❌ Brief stale window between DB write and cache invalidation
- **Use for:** User profiles, social feeds, product listings

---

### 2. Write-Through
```
Write:
  App → write to cache AND DB simultaneously (synchronous)

Read:
  App → check cache (almost always HIT — cache is always fresh)
```
- ✅ Cache always consistent with DB
- ✅ No stale reads
- ❌ Write latency doubled (two synchronous writes)
- ❌ Cache fills with data that may never be read (write amplification)
- **Use for:** Inventory counts, financial balances, anything needing strong read consistency

---

### 3. Write-Back (Write-Behind)
```
Write:
  App → write to cache only (async) → cache flushes to DB later

Read:
  App → check cache (HIT — cache is always fresh)
```
- ✅ Lowest write latency (only one write in hot path)
- ✅ Batches DB writes (reduces DB load)
- ❌ Data loss risk if cache crashes before flush
- ❌ Complexity: need reliable async flush mechanism
- **Use for:** Analytics counters, view counts, like counts, leaderboard scores

---

### 4. Read-Through
```
Read:
  App → ask cache library
    HIT  → cache returns value
    MISS → cache library fetches from DB → populates cache → returns value
           (app doesn't manage this logic)
```
- ✅ Cleaner app code (cache handles DB fallback)
- ✅ Same consistency as Cache-Aside
- ❌ First request is always slow (cold cache)
- ❌ Less control over what gets cached
- **Use for:** Same as Cache-Aside; used when cache supports read-through (e.g. NCache, Ehcache)

---

## ⚙️ Cache Eviction Policies

| Policy | Full Name | How it Works | Use When |
|---|---|---|---|
| **LRU** | Least Recently Used | Evict the item accessed least recently | General purpose — most common default |
| **LFU** | Least Frequently Used | Evict the item accessed least often over time | Hot vs cold content (videos, articles) |
| **TTL** | Time-To-Live | Evict after a fixed time regardless of access | Data that goes stale (stock prices, sports scores, weather) |
| **FIFO** | First In First Out | Evict oldest inserted item | Rarely used; ignores access patterns |
| **Random** | Random | Evict a random item | Simple, low-overhead, approximates LRU |

---

## 🔥 Cache Stampede (Thundering Herd) — Problem & Fix

```
Problem:
  Popular cache key expires →
  1,000 simultaneous requests all get cache MISS →
  All 1,000 hit DB simultaneously →
  DB overwhelmed, cascading failure

Fixes:
  1. Mutex Lock       → first thread fetches from DB, others wait
  2. Probabilistic Early Expiry → randomly refresh before TTL expires
  3. Stale-While-Revalidate → serve stale data while refreshing async
  4. Cache Warming    → pre-populate cache on deploy, before traffic hits
```

---

## 🌐 CDN — Key Facts

| | CDN Pull | CDN Push |
|---|---|---|
| **How** | CDN fetches from origin on first request; caches at edge | You upload content to CDN proactively |
| **Good for** | Dynamic/unpredictable content, long-tail URLs | Large static files (videos, software downloads) |
| **Cache miss behavior** | Origin handles it; brief latency on first request | No misses — content is pre-loaded |
| **Examples** | Cloudflare, CloudFront (default) | CloudFront with S3 pre-upload |

**What to put on CDN:**
- ✅ Images, videos, fonts, CSS, JS bundles
- ✅ Publicly cacheable API responses (e.g. news feeds, product listings)
- ✅ Entire HTML pages (for statically generated sites)
- ❌ Authenticated responses (user-specific data)
- ❌ Real-time data (prices, availability) — use short TTL if needed

---

## 🔢 Redis Quick Reference

| Data Structure | Commands | Use Case |
|---|---|---|
| **String** | GET, SET, INCR, EXPIRE | Sessions, counters, simple key-value cache |
| **Hash** | HGET, HSET, HGETALL | User profile object (field-level updates) |
| **List** | LPUSH, RPOP, LRANGE | Message queues, activity feeds (ordered) |
| **Set** | SADD, SMEMBERS, SISMEMBER | Unique visitors, tags, follower lists |
| **Sorted Set** | ZADD, ZRANGE, ZRANK | Leaderboards, priority queues, rate limiting |
| **TTL** | EXPIRE, TTL | Any key — auto-eviction after N seconds |

---

## 📊 Cache Sizing Formula

```
Cache RAM needed = (number of unique hot keys) × (avg value size)

Example — Twitter timeline cache:
  Active users at peak: 50M
  Avg timeline payload: 2 KB
  Cache RAM = 50M × 2 KB = 100 GB

Redis nodes needed (32 GB each, 80% fill):
  100 GB ÷ (32 GB × 0.8) ≈ 4 nodes
  → 2 primary + 2 replica (for HA) = 4 Redis nodes total
```

---

## ❓ Common Interview Questions on This Topic

- *"Where would you add a cache in this system?"*
  → Identify the hottest read path; add Redis between app servers and DB; CDN for static assets

- *"What happens when the cache goes down?"*
  → Requests fall through to DB; need to handle gracefully (circuit breaker, rate limit DB); cache-aside is resilient

- *"How do you handle cache invalidation?"*
  → On write: delete the key (Cache-Aside) or update it (Write-Through); use TTL as safety net

- *"How do you prevent cache stampede?"*
  → Mutex lock on cache miss, or probabilistic early expiry, or stale-while-revalidate

- *"What's the difference between Redis and Memcached?"*
  → Redis: rich data structures, persistence, replication, pub/sub; Memcached: simpler, multi-threaded, pure caching only

---

*Day 10 · Week 2 · System Design Series*
