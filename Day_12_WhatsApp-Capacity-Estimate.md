# WhatsApp — Back-of-Envelope Capacity Estimation

## Assumptions
- DAU: 500 million
- Messages per user per day: 40
- 10% of messages include media (avg 200 KB)
- Message retention: 30 days on servers (media references retained much longer — see notes)
- Replication factor: 3×
- Average text message size (incl. metadata): 100 bytes
- Peak traffic factor: 3× average
- Read:Write ratio for message delivery: ~1:1 (each message delivered to ~1 recipient on average)

## Calculations

### Daily Active Users & Messages
- DAU = 500,000,000
- Total messages/day = 500M × 40 = **20,000,000,000** (20 billion)
- Text messages/day (90%) = 18B
- Media messages/day (10%) = 2B

### QPS
- Write QPS (avg) = 20B / 86,400s ≈ **231,500 QPS**
- Write QPS (peak, 3×) ≈ **694,500 QPS**
- Read QPS (avg) ≈ 231,500 QPS (delivery is ~1:1)
- Read QPS (peak) ≈ 694,500 QPS
- **Combined peak QPS (read + write) ≈ 1.4M QPS**

### Storage

**Per-day raw storage:**
- Text storage/day = 18B × 100 bytes ≈ **1.8 TB**
- Media storage/day = 2B × 200 KB ≈ **400 TB**
- Total raw/day ≈ **401.8 TB**
- With 3× replication ≈ **~1.2 PB/day**

**30-day retention (active message store, replicated):**
- ~1.2 PB/day × 30 ≈ **~36 PB**

**5-year total (media — links/forwards must remain accessible long-term):**
- Media raw accumulation = 400 TB/day × 365 × 5 ≈ 730,000 TB ≈ **730 PB**
- With 3× replication ≈ **~2,190 PB (~2.2 EB)**

### Bandwidth
- Text ingress = (18B/86,400s) × 100 B ≈ **20.8 MB/s**
- Media ingress = (2B/86,400s) × 200 KB ≈ **4.63 GB/s**
- **Total ingress (avg) ≈ 4.65 GB/s; peak (3×) ≈ ~14 GB/s**
- Egress ≈ ~4.65 GB/s avg / ~14 GB/s peak (1:1 delivery assumption)
  - In practice, CDN/edge caching for shared media drastically cuts origin egress for popular/forwarded content.

## Summary Table

| Metric | Value |
|---|---|
| DAU | 500M |
| Messages/day | 20B |
| Write QPS (avg / peak) | 231.5K / 694.5K |
| Read QPS (avg / peak) | 231.5K / 694.5K |
| Storage/day (raw) | ~402 TB |
| Storage/day (3× replicated) | ~1.2 PB |
| 30-day retention storage | ~36 PB |
| 5-year media storage (replicated) | ~2.2 EB |
| Ingress (avg / peak) | 4.65 GB/s / ~14 GB/s |
| Egress (avg / peak) | 4.65 GB/s / ~14 GB/s |

## Optimizations Worth Mentioning in an Interview
- **Text compression** (~50% reduction on message bodies)
- **CDN caching for media** — huge egress savings on forwarded/viral content
- **Tiered storage**: hot tier (last 30 days, SSD-backed) vs cold tier (media, object storage e.g. S3/Glacier)
- **Erasure coding for cold media** instead of full 3× replication (~1.4–1.5× overhead vs 3×)
- **Deduplication** for repeatedly forwarded media (very common on WhatsApp — same image/video shared across many chats)
- **Separate lightweight pub-sub** for read receipts, typing indicators, presence — kept out of the main message-store QPS budget
