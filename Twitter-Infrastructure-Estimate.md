# Twitter — Back-of-Envelope Infrastructure Estimation

## Assumptions
- DAU: 200 million
- Tweets per user per day: 2 (most users read far more than they post)
- % tweets with media: 20%
- Read:Write ratio: ~1000:1 (extremely read-heavy)
- Tweet text size (incl. metadata): ~300 bytes (280-char limit + metadata)
- Average media size: 1 MB
- Replication factor: 3×
- Peak traffic factor: 3× average
- Average API response size for a timeline read (text/JSON): 5 KB
- CDN cache hit rate for media: 90%

## Calculations

### Total Tweets/Day
- Total tweets/day = 200M × 2 = **400,000,000** (400 million)
- Text-only tweets (80%) = 320M
- Media tweets (20%) = 80M

### Write QPS
- Write QPS (avg) = 400M / 86,400s ≈ **4,630 QPS**
- Write QPS (peak, 3×) ≈ **~13,900 QPS**

### Read QPS (timeline loading, search)
- Read QPS (avg) = Write QPS × 1000 ≈ **4,630,000 QPS (~4.63M)**
- Read QPS (peak, 3×) ≈ **~13.9M QPS**

### Storage Per Day
- Text storage/day = 400M × 300 bytes ≈ **120 GB**
- Media storage/day = 80M × 1 MB ≈ **80 TB**
- Raw storage/day ≈ **~80.12 TB**
- With 3× replication ≈ **~240.4 TB/day**

### 5-Year Total Storage
- 5-year raw = 80.12 TB × 365 × 5 ≈ **~146,200 TB (~146.2 PB)**
- With 3× replication ≈ **~438.7 PB (~0.44 EB)**

### Ingress / Egress Bandwidth
- **Ingress (avg)** = 80.12 TB/day ÷ 86,400s ≈ **0.93 GB/s**
- **Ingress (peak, 3×)** ≈ **~2.8 GB/s**

- **API egress** (timeline reads, text/JSON) = Read QPS × 5 KB ≈ 4.63M × 5 KB ≈ **23.15 GB/s (avg)**
  - Peak (3×) ≈ **~69.5 GB/s**

- **Media egress** (before CDN):
  - Edge-facing media reads ≈ Read QPS × 20% × 1 MB ≈ **~926 GB/s**
  - With 90% CDN cache hit rate, origin egress (10% miss) ≈ **~92.6 GB/s (avg)**

### CDN Needs for Media
- At ~926 GB/s of media requests at peak from clients, a CDN is **mandatory**, not optional.
- Need multi-region edge PoPs close to users to minimize latency for images/video.
- Target 90%+ cache hit rate — viral tweets create extreme "hot key" access patterns, so popular media must be aggressively cached at the edge.
- An origin shield layer absorbs cache misses before they hit object storage, preventing thundering-herd effects.
- Tiered storage for media: hot tier (recent/trending) vs cold tier (older, infrequently accessed, cheaper object storage).

## Summary Table

| Metric | Value |
|---|---|
| DAU | 200M |
| Tweets/day | 400M |
| Write QPS (avg / peak) | 4.63K / 13.9K |
| Read QPS (avg / peak) | 4.63M / 13.9M |
| Storage/day (raw) | ~80.1 TB |
| Storage/day (3× replicated) | ~240.4 TB |
| 5-year storage (3× replicated) | ~438.7 PB |
| Ingress (avg / peak) | 0.93 GB/s / ~2.8 GB/s |
| API egress (avg / peak) | 23.15 GB/s / ~69.5 GB/s |
| Media origin egress (avg, post-CDN) | ~92.6 GB/s |

## Sanity Check vs ByteByteGo's Twitter Design
- ByteByteGo-style designs typically don't push raw read:write ratios all the way to the database — they rely on **fanout-on-write** (push tweets to follower timelines at write-time for normal users) and **fanout-on-read** (compute on the fly for celebrities with huge follower counts) to keep actual DB read QPS far below the 13.9M user-facing request rate.
- Storage estimates broadly line up with typical media-heavy social platforms: media dominates total storage by 2–3 orders of magnitude over text, which matches the pattern seen in WhatsApp's estimate too.
- The 1000:1 read:write ratio assumption is on the aggressive end of real-world figures, but it's a reasonable "worst case" starting point for an interview — the key takeaway is that **read-path caching (CDN + application cache + timeline cache)** is the dominant design concern, not the write path.
