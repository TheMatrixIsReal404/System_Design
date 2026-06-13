# 📱 WhatsApp — Capacity Estimation

> Part of **Day 11: System Design — Storage & Bandwidth Estimation**
> Back to notes → [Day11-SystemDesign-Storage-Bandwidth.md](./Day11-SystemDesign-Storage-Bandwidth.md)

---

## 🏁 Assumptions (Always State These First!)

```
- 2 billion total users
- 1 billion Daily Active Users (DAU)
- Each user sends ~20 messages/day
- 10% of messages contain images
- Average text message  = 100 bytes
- Average image         = 200 KB
- Data retained for     = 5 years
- Replication factor    = 3×
```

---

## 📦 Storage Calculation

### Step 1 — Daily Text Messages

```
Messages/day       = 1B users × 20 msgs/user
                   = 20 billion msgs/day

Text storage/day   = 20B msgs × 100 bytes
                   = 2,000,000,000,000 bytes
                   = 2 TB/day
```

### Step 2 — Daily Image Messages

```
Image msgs/day     = 10% of 20B
                   = 2 billion images/day

Image storage/day  = 2B images × 200 KB
                   = 400,000,000,000 KB
                   = 400 TB/day
```

### Step 3 — 5-Year Text Storage (with Replication)

```
5-year text = 2 TB/day × 365 days × 5 years × 3 (replication)
            = 2 × 365 × 5 × 3
            = ~10,950 TB
            ≈ ~10 PB
```

### Step 4 — 5-Year Image Storage

```
5-year images = 400 TB/day × 365 days × 5 years
              = 400 × 365 × 5
              = 730,000 TB
              ≈ ~730 PB
```

> 🤯 **Total: ~740 PB over 5 years** — this is why WhatsApp needs massive infrastructure!

---

## 🌐 Bandwidth Calculation

### Step 1 — Calculate Write QPS (text messages)

```
Msgs/day  = 20 billion
QPS       = 20,000,000,000 ÷ 86,400
          ≈ 231,000 QPS
          = ~231K QPS
```

### Step 2 — Ingress Bandwidth (text only)

```
Ingress = 231K QPS × 100 bytes
        = 23,100,000 bytes/s
        ≈ 23 MB/s
```

### Step 3 — Egress Bandwidth

```
Assumption: every message is read by at least 2 people (sender + receiver)

Egress  = Ingress × 2
        = 23 MB/s × 2
        = ~46 MB/s
```

> 📌 Note: This is **text only**. Adding image bandwidth would dramatically increase these numbers.

---

## 📊 Final Summary Table

| Metric | Value |
|--------|-------|
| Daily Active Users | 1 billion |
| Daily text messages | 20 billion |
| Daily text storage | 2 TB/day |
| Daily image storage | 400 TB/day |
| 5-year text (with 3× replication) | ~10 PB |
| 5-year images | ~730 PB |
| **Total storage (5 years)** | **~740 PB** |
| Write QPS | ~231K/s |
| Ingress bandwidth (text) | ~23 MB/s |
| Egress bandwidth (text) | ~46 MB/s |

---

## 💡 Optimizations Worth Mentioning

- **Compression** — Applying 0.5× compression to text could halve the ~10 PB text footprint
- **CDN** — Serve images from edge nodes to cut egress costs for popular media
- **Tiered storage** — Move messages older than 1 year to cold/archival storage (e.g. AWS Glacier)
- **Deduplication** — Identical images sent to group chats can be stored once and referenced many times
