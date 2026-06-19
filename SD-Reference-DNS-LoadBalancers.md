# SD-Reference-DNS-LoadBalancers.md

> Reference: DNS record types, TTL, L4 vs L7 load balancing, algorithms, health checks.
> Created: Day 18 | Related daily notes: `notes.md → Day 18`

---

## DNS Record Types — Quick Reference

| Record | Maps                          | Use case                                      | Example                                   |
|--------|-------------------------------|-----------------------------------------------|-------------------------------------------|
| `A`    | domain → IPv4 address         | Standard name resolution                      | `zomato.com → 54.239.28.85`              |
| `AAAA` | domain → IPv6 address         | IPv6 resolution                               | `zomato.com → 2600:1f18:4b4f:b800::1`   |
| `CNAME`| subdomain → another domain    | Aliases; don't repeat IPs across subdomains   | `www.zomato.com → zomato.com`            |
| `MX`   | domain → mail server hostname | Email routing (with priority number)          | `zomato.com MX 10 mail.zomato.com`       |
| `TXT`  | domain → arbitrary text       | SPF, DKIM, domain ownership verification      | `"v=spf1 include:_spf.google.com ~all"`  |

### CNAME Constraint (important for interviews)

```
✅ VALID:    www.zomato.com  CNAME  zomato.com
❌ INVALID:  zomato.com      CNAME  anything     ← apex/root domain cannot be CNAME
```

---

## TTL — Time To Live

TTL = seconds a cached DNS record is considered valid before re-querying.

| Scenario                          | Recommended TTL  | Reason                                      |
|-----------------------------------|------------------|---------------------------------------------|
| Stable production record          | 3600s (1 hr)     | Reduces resolver load                       |
| Pre-migration (lower in advance)  | 60–300s          | Fast propagation when you flip the IP       |
| Post-migration (stable)           | 3600s+           | Caches benefit, stable now                  |
| Disaster recovery / failover DNS  | 60s              | Must propagate fast when switching          |

**Migration playbook:**
1. 24h before migration → lower TTL to 60s
2. Perform migration → update A record to new IP
3. Monitor → 1h after stable → raise TTL back to 3600s

---

## L4 vs L7 Load Balancing

### OSI Layer Context

```
Layer 7 — Application  ← L7 LB operates here (reads HTTP)
Layer 4 — Transport    ← L4 LB operates here (reads TCP/UDP)
Layer 3 — Network      (IP addresses)
```

### Feature Comparison

| Feature                          | L4 Load Balancer       | L7 Load Balancer              |
|----------------------------------|------------------------|-------------------------------|
| Routing basis                    | IP + port              | URL path, headers, cookies    |
| Protocol awareness               | TCP/UDP only           | HTTP, HTTPS, gRPC, WebSocket  |
| SSL termination                  | ❌ (pass-through)       | ✅ (terminates TLS at LB)     |
| Speed                            | ⚡ Faster               | Slightly slower (parses HTTP) |
| A/B testing                      | ❌                      | ✅                             |
| Path-based routing               | ❌                      | ✅                             |
| Host-based routing               | ❌                      | ✅                             |
| Header injection                 | ❌                      | ✅                             |
| Examples                         | AWS NLB, HAProxy (L4)  | AWS ALB, Nginx, Envoy         |

### L7 Routing Rules Example (Nginx-style)

```nginx
# Route by path
location /api/     { proxy_pass http://api_pool; }
location /static/  { proxy_pass http://cdn_pool; }
location /admin/   { proxy_pass http://admin_server; }

# Route by header
if ($http_x_mobile = "true") {
    proxy_pass http://mobile_pool;
}
```

---

## Load Balancing Algorithms

### 1. Round Robin

```
Incoming:  R1   R2   R3   R4   R5   R6
Assigned:  S1   S2   S3   S1   S2   S3
```

- ✅ Simple, stateless
- ❌ Ignores server load; a slow server gets same traffic as a fast one
- **Use when:** all servers are homogeneous and requests are short-lived (REST APIs)

### 2. Weighted Round Robin

```
S1 weight=3, S2 weight=1
Pattern: S1 S1 S1 S2 S1 S1 S1 S2 ...
```

- ✅ Handles heterogeneous servers
- ❌ Static; doesn't auto-adapt to real-time load spikes
- **Use when:** servers have different hardware specs

### 3. Least Connections

```
Current state:
  S1: 15 connections
  S2: 4 connections   ← next request goes here
  S3: 9 connections

After assigning:
  S2: 5 connections
```

- ✅ Adapts to actual load dynamically
- ✅ Best for long-lived connections
- **Use when:** WebSockets, file uploads, streaming (connections stay open for a while)

### 4. IP Hash (Sticky Sessions)

```
hash(client_IP) % 3 = server index

192.168.1.1  → always S1
192.168.1.2  → always S2
192.168.1.3  → always S0
```

- ✅ Same client always hits same server (session affinity without cookies)
- ❌ Uneven distribution if many users share an IP (corporate NAT, CGN)
- ❌ When a server dies, all its "stuck" clients reroute and lose session state
- **Use when:** legacy apps that store session in-memory on the server

### Algorithm Selection Cheatsheet

```
Short stateless REST calls, same hardware  →  Round Robin
Different server sizes                     →  Weighted Round Robin
Long-lived connections (WS, uploads)       →  Least Connections
Must hit same server per user (legacy)     →  IP Hash
Modern stateless + microservices           →  Least Connections or Round Robin
```

---

## Health Checks

### Active Health Check Flow

```
Every 10 seconds:

LB ──GET /health──► Server
      ◄── 200 OK ──        → HEALTHY: keeps receiving traffic
      ◄── 503 / timeout ── → UNHEALTHY: removed from pool
                              (LB keeps checking; adds back on recovery)
```

### Key Parameters

| Parameter           | Typical Value | Meaning                                         |
|---------------------|---------------|-------------------------------------------------|
| Interval            | 10s           | How often to probe                              |
| Timeout             | 5s            | Max wait for response before counting as failed |
| Healthy threshold   | 2             | How many consecutive successes to mark healthy  |
| Unhealthy threshold | 3             | How many consecutive failures to mark unhealthy |

### Good /health Endpoint

```python
# ✅ Good: checks actual dependencies
@app.route("/health")
def health():
    try:
        db.ping()           # check DB connection
        cache.ping()        # check Redis connection
        return {"status": "ok"}, 200
    except Exception as e:
        return {"status": "error", "reason": str(e)}, 503

# ❌ Bad: always returns 200 — pointless
@app.route("/health")
def health():
    return {"status": "ok"}, 200  # even if DB is down!
```

### Passive vs Active Health Checks

| Type    | How it works                                              | When to use                      |
|---------|-----------------------------------------------------------|----------------------------------|
| Active  | LB sends dedicated probe requests on a schedule           | Always prefer; catches issues early |
| Passive | LB watches real traffic error rates; marks bad if >X% fail| Good supplement, not a replacement |

---

## Practice — Answer Key

*(You should fill this in yourself before checking here)*

| Question | Answer |
|---|---|
| Map `api.zomato.com` to an IP | `A` record |
| Alias `www.zomato.com` → `zomato.com` | `CNAME` record |
| Pre-migration TTL action | Lower TTL to 60s at least 1 TTL-period before migration |
| Route `/restaurant/*` vs `/user/*` differently | L7 load balancer |
| Best algorithm for WebSocket connections | Least Connections |
| Health check endpoint path | `GET /health` returning 200 OK |

**Request Flow Answer:**

```
User types zomato.com
         │
         ▼
[ DNS Resolver ] ──── queries DNS server ────► A record: 54.239.28.85 (LB's IP)
         │
         ▼
[ L7 Load Balancer ] ── reads HTTP path ──► routes to correct server pool
         │
    ┌────┴────┐
    ▼         ▼
[Server 1] [Server 2]
```

Why L7? Because Zomato needs to route `/api/`, `/restaurant/`, `/user/`, `/static/`
to different backend pools — that requires reading the HTTP path, which only L7 can do.

---

## Interview One-Liners

- **"L4 routes packets; L7 routes requests."**
- **"Lower TTL before migrations; raise it after stability is confirmed."**
- **"Least connections wins for WebSockets; round robin wins for short REST calls."**
- **"A health check that always returns 200 is not a health check."**
- **"CNAME is an alias — it points to a name, not an IP. And never on the apex domain."**
