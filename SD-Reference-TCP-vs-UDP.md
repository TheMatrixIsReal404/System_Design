# 🔄 SD Reference — TCP vs UDP

## 🎯 Why This Matters
Every protocol you'll design around (HTTP, DNS, video streaming, gaming, gRPC, QUIC) sits on top of either TCP or UDP. Interviewers ask "why TCP here and not UDP?" constantly — knowing the trade-offs cold is what separates a memorized answer from an understood one.

---

## 📊 Quick Comparison Table

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Handshake | 3-way (SYN → SYN-ACK → ACK) | None — just send |
| Reliability | Guaranteed delivery | No guarantee |
| Ordering | In-order delivery | No ordering |
| Speed | Slower (overhead) | Faster, lightweight |
| Header size | ~20 bytes | 8 bytes |
| Congestion control | Yes (slow start, AIMD, etc.) | None |
| Flow control | Yes (sliding window) | None |
| Error handling | Checksum + retransmission | Checksum only, no retry |
| Typical use | HTTP, file transfer, email | DNS, video streaming, gaming, VoIP |

---

## 🤝 TCP — Connection-Oriented Protocol

### The 3-Way Handshake
```
Client                          Server
  |------ SYN ----------------->|   "I want to connect, here's my seq#"
  |<----- SYN-ACK ---------------|   "Got it, here's mine too"
  |------ ACK ------------------>|   "Confirmed, let's talk"
  |      Connection Established  |
```

**Why the handshake exists:**
- Confirms both sides are alive and ready before any data flows.
- Synchronizes sequence numbers so packets can be reordered correctly on arrival.
- Lets both sides estimate round-trip time (RTT) for setting retransmission timeouts.

### Key TCP Mechanisms
- **Guaranteed delivery** — every byte sent is acknowledged; unacknowledged packets get retransmitted.
- **Ordered delivery** — sequence numbers let the receiver reorder out-of-sequence packets before handing data to the application.
- **Congestion control** — TCP starts slow (slow start), ramps up, and backs off when it detects packet loss (a signal of network congestion).
- **Flow control** — sliding window mechanism stops a fast sender from overwhelming a slow receiver.

### Connection Teardown (4-way close)
```
Client                    Server
  |------ FIN ----------->|   "I'm done sending"
  |<----- ACK -------------|
  |<----- FIN -------------|   "I'm done too"
  |------ ACK ------------>|
```

---

## 📡 UDP — Connectionless Protocol

- **No handshake** — sender just fires packets ("fire and forget").
- **No delivery guarantee** — packets can be lost, duplicated, or arrive out of order, and UDP won't fix any of that.
- **No congestion/flow control** — if you need it, the application has to build it itself.
- **Lower overhead → lower latency** — this is the entire reason UDP exists.

---

## 📦 Header Overhead

**TCP header (~20 bytes, no options):** source port, dest port, sequence number, ack number, flags, window size, checksum, urgent pointer.

**UDP header (8 bytes):** source port, dest port, length, checksum.

The 12-byte difference itself rarely matters. What matters is the *cost of the guarantees*: TCP's handshake adds round-trip latency before any data even moves, and its retransmission/ordering logic can stall delivery while waiting for a lost packet (head-of-line blocking). For something like a multiplayer game sending 60 position updates/sec, a stale packet is worse than a missing one — so paying for TCP's guarantees is actively counterproductive.

---

## 🧭 When to Use Which

| Use Case | Protocol | Why |
|---|---|---|
| HTTP/HTTPS (web pages) | TCP | Need complete, ordered, error-free data |
| File transfer (FTP, SCP) | TCP | Can't tolerate lost or corrupted bytes |
| Email (SMTP) | TCP | Correctness matters far more than speed |
| DNS queries | UDP | Single small request/response; just retry if lost |
| Live video streaming | UDP | A dropped frame beats a frozen, buffering stream |
| Online gaming | UDP | Stale state is worse than missing state — latency is everything |
| VoIP / video calls | UDP | Real-time audio tolerates loss but not delay |
| DNS zone transfers | TCP | Response exceeds UDP's safe size, reliability needed |

**One-liner to remember it by:** TCP optimizes for *correctness*, UDP optimizes for *speed*.

---

## 🧠 Mental Model

- **TCP** is registered mail — the sender waits for confirmation of receipt, resends if it doesn't arrive, and delivery is in order.
- **UDP** is a postcard — you drop it in the mailbox and move on. It might arrive late, out of order, or not at all, and nobody's checking.

---

## ❓ Common Interview Questions

- **Why does DNS use UDP but sometimes fall back to TCP?** Most queries are small enough for a single UDP packet; TCP kicks in for large responses like DNSSEC-signed records or zone transfers.
- **Why does QUIC (HTTP/3) build on UDP instead of TCP?** To avoid TCP's head-of-line blocking and get faster connection setup, while still implementing reliability and congestion control at the application layer.
- **What is TCP head-of-line blocking?** A single lost packet blocks delivery of all later, already-arrived packets to the application until the lost one is retransmitted and received.
- **How does TCP detect packet loss?** Either a retransmission timeout expires, or the receiver sends duplicate ACKs, triggering fast retransmit.

---

## ✅ Key Takeaways

- TCP trades latency and overhead for guaranteed, ordered delivery.
- UDP trades reliability for speed and simplicity.
- The right choice depends on whether your system needs *correctness* (TCP) or *real-time responsiveness* (UDP).
- Modern protocols like QUIC/HTTP-3 take a hybrid approach: built on UDP, but with reliability and congestion control re-implemented at the application layer to dodge TCP's head-of-line blocking.

---

## 🔗 Related Notes
- SD-Reference-Scaling-and-LB.md
- SD-Reference-Caching-Strategies.md
