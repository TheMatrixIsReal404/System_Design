# ⚡ SD Reference — Scaling & Load Balancers

> Quick-review cheat sheet. Read this before any system design interview.

---

## 📏 Vertical vs Horizontal — At a Glance

| | Vertical (Scale Up) | Horizontal (Scale Out) |
|---|---|---|
| How | Bigger machine (more CPU/RAM) | More machines |
| Ceiling | Hard ceiling (~192-core, ~24 TB RAM) | No ceiling |
| Downtime | Yes (resize requires restart) | No (add nodes live) |
| Cost curve | Exponential | Linear |
| Failure mode | Single machine = SPOF | N-1 redundancy |
| State handling | Easy (one machine) | Must be stateless |
| Use when | Early stage, simple stack, < ~100K users | At scale, multi-AZ, high availability needed |

---

## ⚖️ Load Balancer Algorithms — When to Use Each

| Algorithm | How it Works | Best For |
|---|---|---|
| **Round Robin** | Requests cycle evenly across servers | Stateless servers, uniform request cost |
| **Weighted Round Robin** | Heavier servers get proportionally more traffic | Mixed instance sizes |
| **Least Connections** | Route to server with fewest active connections | Long-lived connections (WebSockets, streaming) |
| **IP Hash** | Hash(client IP) → always same server | Sticky sessions (legacy apps with server-side state) |
| **Random** | Random server selection | Simple, low-overhead, similar to Round Robin |

---

## 🧱 Layer 4 vs Layer 7 Load Balancer

| | L4 (Transport Layer) | L7 (Application Layer) |
|---|---|---|
| Operates on | TCP/UDP (IP + port) | HTTP/HTTPS (URL, headers, cookies, body) |
| Speed | Faster (less inspection) | Slower (full packet inspection) |
| Routing flexibility | Low — only by IP/port | High — by path, host, cookie, method |
| TLS termination | No | Yes |
| Examples | AWS NLB, HAProxy (TCP mode) | AWS ALB, Nginx, Traefik |
| When to use | Ultra-low latency, non-HTTP (game servers, IoT) | Web APIs, microservices, A/B routing |

---

## 🔄 Eliminating the Load Balancer as SPOF

```
❌ Single LB:
  Clients → [LB] → Servers    ← LB goes down = total outage

✅ Active-Passive pair (Failover):
  Clients → [LB Primary] → Servers
              ↕ heartbeat
            [LB Standby]      ← Takes over IP if primary dies

✅ Active-Active pair (DNS round-robin):
  Clients → DNS → [LB-1] → Servers-A
                → [LB-2] → Servers-B
```

---

## 📌 Stateless Server Rule

For horizontal scaling to work, **app servers must be stateless.**

| State Type | Wrong place | Right place |
|---|---|---|
| User sessions | App server memory | Redis / Memcached |
| Uploaded files | App server disk | S3 / object storage |
| Scheduled jobs state | App server cron | Distributed job queue (SQS, Celery) |
| WebSocket connections | App server | Sticky LB + Redis pub/sub for broadcast |

---

## 🔢 Capacity Rule of Thumb

```
Single commodity app server:   ~1,000–2,000 HTTP QPS
Single Redis node:             ~100,000 ops/sec
Single LB (software, e.g. Nginx): ~50,000–100,000 req/sec

At 10,000 QPS → need ~5–10 app servers + 1 LB (with standby)
At 100,000 QPS → need ~50–100 app servers + LB cluster
```

---

## ❓ Common Interview Questions on This Topic

- *"How would you scale this service from 1K to 10M users?"*
  → Vertical first, then stateless app + horizontal + LB + shared session store

- *"What happens if your load balancer goes down?"*
  → Active-passive pair with virtual IP failover (e.g. AWS ELB is managed & redundant)

- *"Why can't you just keep vertically scaling forever?"*
  → Cost becomes exponential; single point of failure; hardware ceiling exists

---

*Day 8 · Week 2 · System Design Series*
