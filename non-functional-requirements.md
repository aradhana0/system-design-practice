# Non-Functional Requirements — Complete Module

#system-design #nfr #capacity-estimation #storage #interview

---

## 🔑 What Are Requirements?

When someone asks you to build a system, there are always two kinds of questions hidden inside:

| Question | Type | Example |
|----------|------|---------|
| What should the system DO? | Functional Requirement (FR) | "User can shorten a URL" |
| How WELL should it do it? | Non-Functional Requirement (NFR) | "Redirect must happen in < 100ms" |

> **NFRs are the forces that shape your architecture.**
> Same functional requirement, different NFRs → completely different architecture.

**Proof:**
- System A: 1,000 users, stale data OK → SQLite on a single server
- System B: 500M users, real-time → sharding, replication, caching, CDNs
- Same FR: "store user data." Completely different architecture. The difference = NFRs.

---

## 🧩 The 5 Core NFR Categories (STAMP)

| Letter | NFR | Key Metric |
|--------|-----|------------|
| **S** | Scalability | DAU, RPS, WPS, Storage |
| **T** | Throughput / Latency | RPS, p99 latency (ms) |
| **A** | Availability | Nines (99.9%, 99.99%) |
| **M** | Maintainability | (less common in interviews) |
| **P** | Partition tolerance / Durability | Replication factor, RPO |

---

## 📌 NFR 1: Scalability

**Definition:** Ability to handle growing load by adding resources.

| Type | Meaning | Has a ceiling? |
|------|---------|---------------|
| Vertical | Bigger single machine (more RAM/CPU) | Yes — hardware limit |
| Horizontal | More machines | No — theoretically limitless |

- Big tech uses **horizontal scaling**
- Requires **stateless services** + **load balancer**
- Express scale as: DAU, MAU, RPS, WPS, storage over 5 years

---

## 📌 NFR 2: Throughput vs Latency

### Throughput
- How many requests handled **per second**
- Unit: RPS (requests/sec), TPS (transactions/sec), MB/s
- Highway analogy: **width of road** (lanes = throughput)

### Latency
- How long **one request** takes end-to-end
- Unit: milliseconds (ms)
- Highway analogy: **travel time** from A to B

### The Conflict
| Approach | Effect on Throughput | Effect on Latency |
|----------|---------------------|-------------------|
| Batching requests | ↑ Higher | ↑ Worse per item |
| One-at-a-time | ↓ Lower | ↓ Better |

### Real-World Latency Targets

| System | Target |
|--------|--------|
| Real-time chat (WhatsApp) | < 100ms end-to-end |
| Search (Google) | < 200ms |
| Payment processing | < 500ms |
| Video buffering | Seconds are OK |

### Latency → Architecture
- Low latency needed → **Cache** (Redis, Memcached)
- High throughput needed → **Message Queue** (Kafka, RabbitMQ, async processing)

---

## 📌 NFR 3: Availability

**Definition:** % of time the system is operational and serving requests.

### The Nines Table — Memorise This

| Availability | Downtime per year | Architecture needed |
|-------------|------------------|---------------------|
| 99% | ~3.65 days | Single server + restarts |
| 99.9% | ~8.7 hours | Redundancy + health checks |
| 99.99% | ~52 minutes | Multi-server + auto-failover |
| 99.999% | ~5 minutes | Multi-region + active-active |

### Formula
```
Availability = (Total Time − Downtime) / Total Time × 100
```

### ⚠️ Availability ≠ Durability

| Term | Meaning | Example |
|------|---------|---------|
| **Availability** | System is UP and responding | Server is running |
| **Durability** | Data is NOT LOST | Data survives disk failure |

> AWS S3 = 99.99% available + 99.999999999% durable (11 nines for durability!)
> A system can be available but not durable (data lost on crash), or durable but unavailable (backup exists but server is down).

---

## 📌 NFR 4: Consistency

**Definition:** All users see the same data at the same time.

| Type | Meaning | Use case |
|------|---------|---------|
| **Strong** | Read always returns latest write | Bank balance |
| **Eventual** | Reads may be stale, will converge | Instagram like count |
| **Read-your-writes** | Creator always sees their own write; others may lag | URL shortener redirect |

> Consistency connects directly to **CAP Theorem** — during a network partition, you choose consistency OR availability.

---

## 📌 NFR 5: Fault Tolerance & Durability

**Fault Tolerance:** System works even when components fail
**Durability:** Written data is never lost

### Mechanisms
- **Replication** — multiple copies of data on different nodes
- **Checksums** — detect data corruption at rest
- **Write-Ahead Log (WAL)** — log before committing (used in PostgreSQL, Kafka)
- **Heartbeats** — detect node failures in distributed systems

### RPO and RTO (Know These for Architect Roles)

| Term | Full Form | Meaning |
|------|-----------|---------|
| **RPO** | Recovery Point Objective | How much data loss is acceptable? (e.g., RPO = 0 means zero data loss) |
| **RTO** | Recovery Time Objective | How long can the system be down during recovery? (e.g., RTO = 30s) |

---

## 🏷️ SLI, SLO, SLA — Three Terms, One Concept

This is high-signal knowledge. Most candidates only know "SLA."

| Term | Full Form | Who sets it | What it is |
|------|-----------|-------------|-----------|
| **SLI** | Service Level Indicator | Engineering | The **actual measured metric** (raw data) |
| **SLO** | Service Level Objective | Engineering | The **internal target** — stricter than SLA |
| **SLA** | Service Level Agreement | Business/Legal | The **external promise** with consequences |

### Real Example — Zomato Order Tracking API

**SLI** (measurement):
> "Our API responded in under 200ms for 97.3% of requests last week."
> → Raw factual observation. Just data.

**SLO** (internal goal):
> "We want the API to respond in under 200ms for 99% of requests."
> → What engineering aims for. Stricter than the SLA intentionally.

**SLA** (external promise):
> "We guarantee under 200ms for 98% of requests. If we fail, restaurants get a 10% fee waiver."
> → Public commitment with a financial consequence.

### The Relationship
```
SLI (actual measurement)
    ↓ compared against
SLO (internal target — always stricter than SLA)
    ↓ if SLO is repeatedly missed, you risk breaching
SLA (external promise — consequence if broken)
```

> **SLO is always stricter than SLA.** The gap between them is your breathing room.

---

## 💰 Error Budget

**Error budget = 100% − SLO**

If SLO = 99.9%, error budget = 0.1% of time = ~8.7 hours/year.

**What it means in practice:**
- How many risky deployments can you do this month?
- If you burned your error budget in January → freeze feature releases, focus on reliability
- Google SRE literally pauses feature work when error budget is spent

> **This is real at Google, Meta, Amazon.** Knowing this puts you ahead of most candidates.

---

## 📊 Percentiles — The Right Way to Measure Latency

### ⚠️ Never use averages for latency in an interview

**Why averages lie:**
```
9 requests at 10ms + 1 request at 10,000ms
Average = (9×10 + 10,000) / 10 = ~1,009ms

But 90% of users had a great experience!
The average makes it look terrible for everyone.
```

### Percentile Meanings

| Metric | Meaning |
|--------|---------|
| **p50** (median) | 50% of requests are faster than this |
| **p95** | 95% of requests are faster than this |
| **p99** | 99% of requests are faster than this — the standard SLA metric |
| **p99.9** | 99.9% of requests faster — "tail latency" |

### Tail Latency
- p99 and p99.9 are called **tail latency**
- Root causes: GC pauses, hot partitions, lock contention, slow queries
- At 1M requests/day → p99 = 10,000 users affected by slow response
- At 10M requests/day → p99 = 100,000 users affected

> **Rule:** Always specify latency as pXX. "API response < 200ms at p99" — not "API response < 200ms."

---

## 🔢 How to Arrive at NFR Metrics (Back-of-Envelope)

### Step 1: Understand the Product
Ask these before writing a single number:
- DAU / MAU?
- Read-heavy or write-heavy?
- Global or regional?
- What's the SLA requirement?

### Step 2: Estimate Load

**Formula:**
```
RPS = (DAU × actions per user per day) ÷ 86,400
```

**86,400 = seconds in a day. Memorise this.**

**Twitter Example:**
```
DAU = 100M
Tweets/day: 100M × 2 = 200M writes/day
WPS: 200M ÷ 86,400 ≈ 2,300 writes/sec → ~2.5K WPS

Feed reads/day: 100M × 100 = 10B reads/day
RPS: 10B ÷ 86,400 ≈ 115,000 reads/sec → ~100K RPS

Read:Write ratio = 100K : 2.5K = 40:1 → VERY read-heavy
```

**Shortcut for ratio:** Derive directly from assumptions, not from RPS.
> "10 reads and 2 writes per user" → ratio = 10:2 = 5:1. No math needed.

### Step 3: Estimate Storage

**Formula:**
```
Storage = (size of one record in bytes) × (number of records per day) × (time period)
```

**4-Step Process:**
```
1. Sum all fields → size of one record
2. DAU × writes/day → records per day
3. records × record size → storage per day
4. × 365 × 5 → 5-year projection
```

### Step 4: Define the SLA
```
- Latency: "Feed load < 200ms at p99"
- Availability: "99.99% availability SLO, 99.9% SLA"
- Consistency: "Eventual consistency — tweets may appear with 2s delay"
```

---

## 📦 Storage Unit Ladder

```
1 Byte   (B)
1 KB     = 10³ bytes   = 1,000 B
1 MB     = 10⁶ bytes   = 1,000 KB
1 GB     = 10⁹ bytes   = 1,000 MB
1 TB     = 10¹² bytes  = 1,000 GB
1 PB     = 10¹⁵ bytes  = 1,000 TB
1 EB     = 10¹⁸ bytes  = 1,000 PB
```

### Converting UP (to larger unit) → divide by 1,000
### Converting DOWN (to smaller unit) → multiply by 1,000

### Powers of 10 Shortcut
```
200M records × 200 bytes
= 2 × 10⁸ × 2 × 10²
= 4 × 10¹⁰ bytes
= 40 GB  (since 1 GB = 10⁹, subtract 9 from exponent: 10¹⁰⁻⁹ = 10¹ = 10)
```

---

## 📐 Record Size Cheat Sheet

| Field type | Size |
|------------|------|
| Integer / numeric ID | 4–8 bytes |
| Timestamp | 8 bytes |
| Boolean / status flag | 1 byte |
| UUID | 16 bytes |
| Average URL | ~100 bytes |
| Short string (username ~20 chars) | 20 bytes (1 byte/char) |
| IPv4 address | 4 bytes |
| IPv6 address | 16 bytes |

> **Rounding rule:** Always add 20–30% overhead. Real records have padding, indexes, metadata.

---

## 🌍 Real World Size Anchors

| Thing | Size |
|-------|------|
| One character | 1 byte |
| A tweet (280 chars) | ~280 bytes |
| Small compressed photo | ~100 KB |
| High quality photo | ~3–5 MB |
| 1 hour HD video | ~1–2 GB |
| DVD movie | ~4–8 GB |
| All text on Wikipedia | ~20 GB |

---

## 🧮 Worked Storage Examples

### URL Shortener
```
Record: short_url(7) + long_url(100) + user_id(8) + timestamps(16) + overhead(20) = ~150 → 200 bytes
Volume: 100M DAU × 2 writes = 200M records/day
Per day: 200M × 200 = 40 GB/day
5 years: 40 × 365 × 5 = 73 TB
→ Needs distributed DB + sharding
```

### Instagram Profiles
```
Record: username(20) + bio(150) + pic_url(100) + follower_count(8) + created_at(8) = 286 → 300 bytes
Volume: 2 billion total users
Total: 300 × 2 × 10⁹ = 6 × 10¹¹ bytes = 600 GB = 0.6 TB
→ Fits on one server technically, but still needs distribution (see below)
```

### WhatsApp Messages
```
Record: message_id(16) + sender_id(8) + receiver_id(8) + content(100) + timestamp(8) + status(1) = ~141 → 150 bytes
Volume: 100M DAU × 50 messages/day = 5B messages/day
Per day: 5 × 10⁹ × 150 = 750 GB/day ≈ 0.75 TB/day
5 years: 0.75 × 365 × 5 = ~1.4 PB
→ Petabyte scale → distributed storage across thousands of servers
```

### Instagram Photos
```
One photo (compressed): ~200 KB
Volume: 500M DAU × 0.1 photos/day = 50M photos/day
Per day: 50M × 200 KB = 5×10⁷ × 2×10⁵ = 10 × 10¹² = 10 TB/day
5 years: 10 × 365 × 5 = 18,250 TB ≈ 18 PB
→ Object storage (AWS S3)
```

---

## 🏗️ Architectural Decision Rules

### How much can one server hold?

| Server type | Disk capacity |
|-------------|--------------|
| Commodity server | 1–4 TB SSD, 4–20 TB HDD |
| High-end DB server | 8–16 TB SSD, up to 50 TB HDD |
| RAM | 64 GB – 512 GB |

### Storage → Architecture

```
< 1 TB    → Fits on one server — still distribute for availability
1–100 TB  → Multiple servers + sharding
> 100 TB  → Distributed DB (Cassandra, HBase, HDFS)
> 1 PB    → Object storage + data lakes (S3, GCS, Azure Blob)
```

### ⚠️ "Can Hold" ≠ "Should Hold"

Even 0.6 TB on one server is wrong. Three reasons:

| Reason | Explanation |
|--------|-------------|
| **Single Point of Failure** | Server crashes → everything down |
| **Read traffic** | Storage fits ≠ 2B concurrent reads handled |
| **Growth** | Design for 3–5 years, not today |

> **Always state:** "Raw storage X TB. With 3× replication factor = 3X TB actual storage needed."

---

## ✍️ NFR Template for Interviews

```
1. SCALE
   - DAU: X million, MAU: Y million
   - Reads: X RPS  (DAU × reads/day ÷ 86,400)
   - Writes: Y WPS (DAU × writes/day ÷ 86,400)
   - Read:Write ratio: X:1 → read-heavy / write-heavy → implication
   - Record size: Z bytes
   - Storage/day: N GB/day
   - 5-year storage: X TB → architectural implication
   - With 3× replication: 3X TB

2. LATENCY
   - [Operation A]: p99 < Xms
   - [Operation B]: p99 < Yms, p50 < Zms (with cache)

3. AVAILABILITY
   - SLO: 99.99% (internal)
   - SLA: 99.9%  (external — gives ~8.7 hrs error budget/year)
   - Justification: [why this level is needed for this system]

4. CONSISTENCY
   - Writes: Strong/Eventual — [why]
   - Reads: Strong/Eventual/Read-your-writes — [why]

5. DURABILITY
   - RPO = 0 (zero data loss after write ack) or RPO = X minutes
   - RTO = Y seconds (recovery time)
   - Replication factor: 3
   - [Any immutability constraints]
```

---

## ⚖️ Core Trade-offs — Know All 5 Cold

| Trade-off | Explanation |
|-----------|-------------|
| **Latency vs Consistency** | Cache = faster reads but may return stale data |
| **Availability vs Consistency** | CAP theorem — can't have both fully during partition |
| **Throughput vs Latency** | Batching = more throughput, worse per-item latency |
| **Durability vs Cost** | More replicas = more durable, more expensive |
| **Strong consistency vs Scalability** | Consensus (Raft/Paxos) is expensive to scale horizontally |

---

## 🎯 What Makes a Metric "Good"?

1. **Measurable** — you can observe and alert on it
2. **Tied to user experience** — p99 > average
3. **Achievable** — know the cost of what you're promising
4. **Drives architectural decisions** — if it doesn't change your design, it's not worth stating

---

## 📝 Practice: URL Shortener NFRs (Solved)

```
1. SCALE
   - DAU: 100M, MAU: 500M
   - Writes: 100M × 2 ÷ 86,400 = 2,314 WPS ≈ 2.3K WPS
   - Reads:  100M × 10 ÷ 86,400 = 11,574 RPS ≈ 11.5K RPS
   - Read:Write ratio = 5:1 → read-heavy → cache + read replicas
   - Record: ~200 bytes
   - Storage: 200M × 200 bytes = 40 GB/day
   - 5-year: 40 × 365 × 5 = 73 TB → distributed DB + sharding
   - With 3× replication: ~220 TB

2. LATENCY
   - URL generation (write): p99 < 200ms
   - Redirect (read): p99 < 100ms, p50 < 20ms (cached)

3. AVAILABILITY
   - SLO: 99.99% | SLA: 99.9%
   - Justification: redirects are in critical path of marketing
     campaigns and QR codes — downtime breaks real-world links

4. CONSISTENCY
   - Writes: Strong — short URL must be globally unique, enforced atomically
   - Reads: Read-your-writes for creator; eventual consistency for
     global propagation (< 2 second delay acceptable)

5. DURABILITY
   - RPO = 0 — zero data loss after write ack
   - Replication factor = 3
   - URL mappings are immutable after creation
   - No expiry by default; optional TTL at creation time
```

---

## 🧠 Interview Q&A

**Q: What is the difference between SLA, SLO, and SLI?**
> SLI is the raw measured metric (e.g., actual p99 latency observed). SLO is the internal engineering target (e.g., p99 < 200ms for 99% of requests). SLA is the external business commitment with consequences (e.g., 98% of requests under 200ms; breach = refund). SLO is always stricter than SLA — the gap is the error budget.

**Q: What is an error budget?**
> Error budget = 100% − SLO. If SLO is 99.9%, error budget is 0.1% of time (~8.7 hours/year). It governs how many risky deployments you can do. When the budget is spent, feature work stops and reliability work takes priority. This is the foundation of Google SRE.

**Q: What is p99 latency and why does it matter more than average?**
> p99 means 99% of requests complete faster than this value. Averages hide outliers — a few very slow requests can mislead. At 1M requests/day, p99 affects 10,000 users. At 10M requests/day, it's 100,000 users. p99 reflects real user pain; averages don't.

**Q: What is the difference between availability and durability?**
> Availability = system is up and responding. Durability = data is not lost. A system can be available but not durable (server responds but lost your data on crash), or durable but unavailable (data safely backed up but server is down for maintenance).

**Q: Can I have high availability AND strong consistency?**
> Not fully during a network partition — this is the CAP theorem. You must choose. For payments: prefer consistency (wrong balance is worse than brief downtime). For social feeds: prefer availability with eventual consistency (a 2-second tweet delay is harmless).

**Q: How do you calculate storage?**
> Sum the field sizes to get record size → multiply by records per day → multiply by time period (5 years) → round up 20–30% for overhead → multiply by replication factor (3) for actual hardware needed.

**Q: What storage architecture is needed at different scales?**
> Under 1 TB: single server possible but still distribute for availability. 1–100 TB: multiple servers with sharding. Over 100 TB: distributed DB (Cassandra, HBase). Over 1 PB: object storage (S3, GCS).

**Q: What is RPO and RTO?**
> RPO (Recovery Point Objective) = maximum acceptable data loss. RPO = 0 means no data loss tolerated after a write acknowledgement. RTO (Recovery Time Objective) = maximum acceptable downtime during recovery. These drive backup frequency and failover speed decisions.

---

## 🪤 Interview Traps

| Trap | Correct Response |
|------|-----------------|
| "High availability AND strong consistency" | CAP theorem — explain the trade-off, choose based on use case |
| "Zero latency" | Physically impossible — speed of light is a real constraint. Give realistic numbers |
| "Just cache everything" | Cache has invalidation cost and staleness risk. Ask: what's the TTL? write-through or write-around? |
| Using average latency | Always p99 or p95 for SLA definitions |
| "0.6 TB fits on one server, done" | Still need 3 replicas + read replicas + 5-year growth |
| Using 86,700 as seconds/day | It's 86,400 — signals sloppiness |
| Forgetting replication in storage | State raw storage AND ×3 replication total |
| Confusing availability with reliability | Availability = up. Reliability = correct results over time |

---

## 🔗 Related Topics
- [[CAP Theorem]] — Module 13: consistency vs availability during partition
- [[Consistent Hashing]] — Module 11: distributing data across nodes
- [[Caching Strategies]] — cache-aside, write-through, write-around, TTL
- [[Database Sharding]] — horizontal partitioning when storage exceeds one node
- [[Replication]] — leader-follower, multi-leader, quorum writes
- [[Object Storage vs Block Storage]] — S3 vs EBS, when to use which
- [[Message Queues]] — Module 20: Kafka, RabbitMQ, throughput vs latency
