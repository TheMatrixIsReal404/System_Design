# SD Reference — Powers of Two & Back-of-Envelope Cheat Sheet

> **Purpose:** Quick-reference card for system design interviews and back-of-envelope calculations.
> Use these numbers to estimate storage, throughput, and latency without a calculator.

---

## 📐 Powers of Two — Storage Sizes

| Power | Exact Value | Approximate | Name |
|-------|-------------|-------------|------|
| 2^10  | 1,024       | ~1 thousand | **1 KB** (Kilobyte) |
| 2^20  | 1,048,576   | ~1 million  | **1 MB** (Megabyte) |
| 2^30  | 1,073,741,824 | ~1 billion | **1 GB** (Gigabyte) |
| 2^40  | ~1 trillion | ~1 trillion | **1 TB** (Terabyte) |
| 2^50  | —           | ~1 quadrillion | **1 PB** (Petabyte) |

> **Trick:** Every +10 in power = ×1000 in value. So 2^30 ≈ 10^9 = 1 billion.

---

## ⏱️ Latency Numbers Every Engineer Should Know

| Operation | Latency | Relative Feel |
|-----------|---------|---------------|
| L1 cache reference | ~0.5 ns | 1x (baseline) |
| L2 cache reference | ~7 ns | 14x slower than L1 |
| RAM read | ~100 ns | 200x slower than L1 |
| SSD random read | ~100 µs (0.1 ms) | 200,000x slower than L1 |
| HDD seek | ~10 ms | 20,000,000x slower than L1 |
| Redis read (same datacenter) | ~0.5–1 ms | In-memory over LAN |
| Database query (PostgreSQL) | ~10–100 ms | Disk I/O + parsing |
| Round trip within same datacenter | ~0.5 ms | LAN |
| Round trip US → Europe | ~150 ms | WAN |
| Round trip US → India | ~200 ms | WAN |

> **Interview tip:** Cache = ~1ms, DB = ~10–100ms, Cross-continent = ~150–200ms. Memorise these three bands.

---

## 🗄️ Common Data Sizes (Typical Estimates)

| Data Type | Size Estimate |
|-----------|---------------|
| ASCII character | 1 byte |
| Unicode character (UTF-8) | 1–4 bytes |
| Integer (int32) | 4 bytes |
| Timestamp (Unix epoch) | 8 bytes |
| UUID / GUID | 16 bytes |
| Tweet / short text (280 chars) | ~300 bytes |
| Small row in a relational DB | ~100–500 bytes |
| Average webpage (HTML only) | ~50–100 KB |
| Thumbnail image (JPEG) | ~50–200 KB |
| Full-resolution photo (JPEG) | ~2–5 MB |
| 1-minute audio (MP3, 128kbps) | ~1 MB |
| 1-minute SD video (compressed) | ~10–50 MB |
| 1-minute HD video (compressed) | ~100–500 MB |

---

## 🔢 Time Conversions (for QPS Calculations)

| Period | Seconds |
|--------|---------|
| 1 minute | 60 s |
| 1 hour | 3,600 s |
| 1 day | 86,400 s (~100,000 s is a useful approximation) |
| 1 month | ~2.5 million s |
| 1 year | ~31.5 million s (~30 million for estimation) |

> **Trick:** Use 100,000 seconds/day (instead of 86,400) for easy mental division.
> Example: 10 million requests/day ÷ 100,000 = **100 QPS**

---

## 📊 Throughput Benchmarks (Rule-of-Thumb)

| Component | Throughput Estimate |
|-----------|---------------------|
| Single PostgreSQL primary (writes) | ~2,000–5,000 writes/sec |
| Single PostgreSQL read replica (reads) | ~5,000–10,000 reads/sec |
| Redis (single node, reads) | ~100,000–500,000 ops/sec |
| Nginx / Load Balancer | ~10,000–100,000 req/sec |
| Single App Server (Flask/Node, simple API) | ~1,000–5,000 req/sec |
| Kafka topic (single partition) | ~50,000–100,000 msg/sec |
| 1 Gbps network link | ~125 MB/sec |
| 10 Gbps datacenter link | ~1.25 GB/sec |

---

## 🧮 Back-of-Envelope Template (Use in Interviews)

```
Step 1 — Traffic (QPS)
  Daily Active Users (DAU) × actions/day ÷ 86,400 = Average QPS
  Peak QPS ≈ Average QPS × 2–3 (rule of thumb for peaks)

Step 2 — Storage
  Items written/day × item size = Storage/day
  Storage/day × 365 × years = Total storage needed

Step 3 — Bandwidth
  QPS × average response size = Bandwidth/sec

Step 4 — Servers needed
  Peak QPS ÷ QPS per server = Number of servers
```

### Example: Zomato Order System

```
DAU: 10 million users
Orders per user per day: 0.5 (not every user orders daily)
Total orders/day: 5 million

QPS (orders): 5M ÷ 86,400 ≈ 58 QPS average
Peak QPS: 58 × 3 = ~175 QPS (dinner rush)

Order record size: ~500 bytes
Storage/day: 5M × 500B = 2.5 GB/day
Storage/year: 2.5 GB × 365 ≈ ~900 GB/year → ~1 TB/year

DB sizing: 175 writes/sec peak → well within 1 PostgreSQL primary (cap: ~5,000/sec)
Read traffic (browse menus): 10× writes → 1,750 read QPS → 1–2 read replicas + Redis cache
```

---

## 💡 Quick Approximations Cheat Sheet

| What you need | Shortcut |
|---------------|----------|
| Seconds per day | **86,400** (use 100K for estimation) |
| 1 million users × 1 req/day | **~12 QPS** |
| 1 billion req/day | **~11,600 QPS** |
| 1 TB of text (1 KB avg) | **~1 billion records** |
| 1 GB RAM for Redis | **~1 million entries at 1KB avg** |

---

## 🏷️ Availability "Nines" Reference

| Availability | Downtime / Year | Downtime / Month | Use Case |
|---|---|---|---|
| 99% (two nines) | 3.65 days | ~7.3 hours | Internal tools, low priority |
| 99.9% (three nines) | ~8.7 hours | ~43 minutes | Most web apps (Zomato) |
| 99.99% (four nines) | ~52 minutes | ~4.4 minutes | Payments, banking |
| 99.999% (five nines) | ~5 minutes | ~26 seconds | Hospital systems, aviation |

> **Formula:** Availability = Uptime ÷ (Uptime + Downtime) × 100%

---

*Reference compiled: Week 1 — Day 1, System Design Fundamentals*
*Used for: Back-of-envelope calculations in interviews and architecture planning*
