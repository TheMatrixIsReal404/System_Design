# 📝 Day 16 Practice — HTTP/HTTPS Protocols

> Linked from: `notes.md` (Day 16)
> Attempt these from memory first. Don't peek at the main notes until you're stuck.

---

### 1. Draw the 4 steps of the TLS handshake

Sketch it out (boxes + arrows are fine). Make sure you cover:
- What the client sends first
- What the server sends back
- Where the public key is used
- Where the private key is used
- At what point the symmetric session key gets established

---

### 2. Why does HTTPS switch from asymmetric to symmetric encryption mid-handshake?

Answer in your own words:
- What problem does asymmetric encryption solve?
- What's the downside of using it for *all* traffic?
- What does symmetric encryption give you in exchange?

---

### 3. What's the difference between HTTP/2 multiplexing and HTTP/1.1 head-of-line blocking?

- Describe what happens to Request B if Request A is slow, under HTTP/1.1.
- Describe what happens to the same scenario under HTTP/2.
- Bonus: why do browsers open ~6 parallel connections under HTTP/1.1, and why is that no longer necessary under HTTP/2?

---

## Self-check (don't read until you've attempted all 3)

<details>
<summary>Click to reveal answer key reference</summary>

See `notes.md` sections:
- Q1 → Section 4 (TLS Handshake Walkthrough)
- Q2 → Section 4, "Why switch from asymmetric → symmetric mid-handshake?"
- Q3 → Section 5 (HTTP/1.1 vs HTTP/2)

</details>
