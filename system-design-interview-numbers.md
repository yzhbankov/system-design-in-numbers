# The System Design Interview Numbers Cheat Sheet: Every Technology and Its Figure of Merit

> *"What's your estimated QPS?" — the question that separates a candidate who has operated systems from one who has only read about them.*

Every system design interview eventually reaches the same moment. You have drawn boxes and arrows, you have named your technologies, and then the interviewer leans in and asks: **"How many of those do you need?"**

At that point, hand-waving stops working. You need numbers — not memorized trivia, but a working set of **figures of merit**: the one or two quantities that characterize what a technology can actually do. How many requests can one application server absorb? How fast is a Redis read, really? At how many rows does PostgreSQL start hurting? How many partitions can a Kafka broker hold? How many shards should an Elasticsearch index have?

This article is the reference I wish I had. It collects the figures of merit for essentially every technology class that shows up in a system design interview: compute, load balancing, API rate limiting, auth and ID generation, caching, relational databases, consistency and quorums, NoSQL stores, search, streaming and messaging, stream processing, workflow engines, object storage and CDN, analytics and lakehouses, observability, vector search, ML and LLM serving, graph and geospatial indexes, real-time media, coordination, storage hardware, serverless, Kubernetes, serialization, probabilistic structures, client-side budgets, availability, multi-region DR and cost. Alongside the numbers sits the back-of-the-envelope math that turns them into an architecture.

**How this is organized.** Nine parts, ~23,000 words, a little over two hours end to end — but it is built to be read in pieces. Part 1 is the foundation; Parts 2–7 walk the stack from the load balancer down to the disk and out to the bill; Part 8 works nine complete capacity estimates; Part 9 is drills. Every section opens with a one-line takeaway tagged Tier 1, 2 or 3, so you can tell at a glance what to memorize and what to merely recognize.

A word on precision before we start. **These are order-of-magnitude numbers.** Your hardware, your payload sizes, your access patterns and your tuning will move any of them by 2–5×. That is fine. Interviews reward *calibration*, not precision: knowing that a Redis GET is ~0.2 ms and not 20 ms, that a Postgres node does ~10k writes/sec and not 10M, that a Kafka broker holds ~4,000 partitions and not 4 million. Being right to within an order of magnitude, and being explicit that you are estimating, is exactly the behavior a strong interviewer is looking for.

---

## How to use this guide

There are ~250 numbers in here. **You do not need to memorize them all** — and trying to is the wrong preparation. They fall into three tiers, and every section below is tagged with the tier it belongs to.

| Tier | What it means | How many | What to do |
|---|---|---|---|
| **Tier 1** | You should be able to say it without thinking | **~40 numbers** | **Memorize.** These come up in almost every interview |
| **Tier 2** | You should know the shape and be within 10× | ~80 numbers | Read twice, recognize when they matter |
| **Tier 3** | You should know that a limit *exists* and roughly where | the rest | Look up when the question calls for it |

Tier 1 is short enough to fit on one page: the cheat sheet right after the contents compresses the whole article to a screen, and the flashcard list at the very end is exactly those forty numbers. Inside the tables, a **bold value** marks one of those Tier 1 numbers; everything in plain text is Tier 2 or 3.

### Three ways to read this

**The 45-minute cram** (interview tomorrow): read the cheat sheet below, then Part 1 (foundations), then §10 (relational), §9 (caching), §15 (streaming), §12 (sharding thresholds), then the worked examples in Part 8. Skip everything else.

**The one-week prep** (interview next week): one part per day, in order. After each part, close the page and try to re-derive the Tier 1 numbers from memory. Finish with the drills in Part 9 — do them cold, not while reading.

**The reference** (on the job, or mid-loop): jump straight to the section you need from the table of contents. Every section leads with its single most important number so you can grab it in five seconds.

### Why this works better than memorizing

Interviewers are not testing recall — they are testing whether your architecture is *physically possible*. Almost every design question reduces to the same five-step chain, and each step needs exactly one or two numbers:

```
users → requests/second → bytes/second and bytes/year → machines → what breaks first
```

If you can run that chain out loud with roughly correct constants, you will outperform a candidate who knows twice as many technologies but cannot size any of them.

---

## Table of Contents

- [How to use this guide](#how-to-use-this-guide)
- [The cheat sheet: every number on one page](#the-cheat-sheet-every-number-on-one-page)

**[Part 1 — Foundations](#part-1--foundations)**

1. [Why numbers win interviews](#1-why-numbers-win-interviews)
2. [The latency ladder every engineer should know](#2-the-latency-ladder-every-engineer-should-know)
3. [The back-of-the-envelope toolkit](#3-the-back-of-the-envelope-toolkit)
4. [Network, geography and protocol latency](#4-network-geography-and-protocol-latency)

**[Part 2 — The request path](#part-2--the-request-path)**

5. [Application servers: requests per second per box](#5-application-servers-requests-per-second-per-box)
6. [Load balancers, proxies and API gateways](#6-load-balancers-proxies-and-api-gateways)
7. [API design: rate limits, quotas, pagination and timeouts](#7-api-design-rate-limits-quotas-pagination-and-timeouts)
8. [Identity, auth, ID generation and clocks](#8-identity-auth-id-generation-and-clocks)

**[Part 3 — Data at rest](#part-3--data-at-rest)**

9. [Caching: Redis, Memcached and friends](#9-caching-redis-memcached-and-friends)
10. [Relational databases: PostgreSQL and MySQL](#10-relational-databases-postgresql-and-mysql)
11. [Consistency, quorums and distributed transactions](#11-consistency-quorums-and-distributed-transactions)
12. [When to shard: the thresholds table](#12-when-to-shard-the-thresholds-table)
13. [NoSQL: DynamoDB, Cassandra, ScyllaDB, MongoDB, HBase](#13-nosql-dynamodb-cassandra-scylladb-mongodb-hbase)
14. [Search engines: Elasticsearch and OpenSearch](#14-search-engines-elasticsearch-and-opensearch)

**[Part 4 — Data in motion](#part-4--data-in-motion)**

15. [Streaming and messaging](#15-streaming-and-messaging)
16. [Stream processing: Flink, Spark, Kafka Streams](#16-stream-processing-flink-spark-kafka-streams)
17. [Workflow orchestration and background jobs](#17-workflow-orchestration-and-background-jobs)
18. [Object storage and CDN](#18-object-storage-and-cdn)
19. [OLAP and warehouses](#19-olap-and-warehouses)
20. [Time series and observability](#20-time-series-and-observability)

**[Part 5 — Specialized workloads](#part-5--specialized-workloads)**

21. [Vector databases and ANN search](#21-vector-databases-and-ann-search)
22. [ML and LLM serving](#22-ml-and-llm-serving)
23. [Graph databases](#23-graph-databases)
24. [Geospatial indexing](#24-geospatial-indexing)
25. [Real-time media, live streaming and notifications](#25-real-time-media-live-streaming-and-notifications)

**[Part 6 — The infrastructure underneath](#part-6--the-infrastructure-underneath)**

26. [Coordination: ZooKeeper, etcd, Consul, Raft](#26-coordination-zookeeper-etcd-consul-raft)
27. [Storage hardware](#27-storage-hardware)
28. [Serverless and edge](#28-serverless-and-edge)
29. [Kubernetes and orchestration limits](#29-kubernetes-and-orchestration-limits)
30. [Serialization, compression and protocol overhead](#30-serialization-compression-and-protocol-overhead)
31. [Probabilistic data structures](#31-probabilistic-data-structures)
32. [Client-side, mobile and frontend budgets](#32-client-side-mobile-and-frontend-budgets)

**[Part 7 — Running it in production](#part-7--running-it-in-production)**

33. [Availability, reliability and error budgets](#33-availability-reliability-and-error-budgets)
34. [Multi-region, disaster recovery, RPO and RTO](#34-multi-region-disaster-recovery-rpo-and-rto)
35. [Cost figures of merit](#35-cost-figures-of-merit)

**[Part 8 — Putting it together](#part-8--putting-it-together)**

36. [Nine worked capacity estimates](#36-nine-worked-capacity-estimates)
37. [Interview mechanics: where the numbers go](#37-interview-mechanics-where-the-numbers-go)

**[Part 9 — Practice](#part-9--practice)**

38. [Drills: twenty questions, cold](#38-drills-twenty-questions-cold)
- [The Tier 1 flashcard list](#the-tier-1-flashcard-list)

---

## The cheat sheet: every number on one page

*Skim it now, come back to it the night before. Everything below this is the explanation.*

| Latency | Number |
|---|---|
| L1 cache | 1 ns |
| RAM | 100 ns |
| NVMe random read | 50 µs |
| Same-DC round trip | 0.5 ms |
| Cross-AZ round trip | 1 ms |
| US → Europe round trip | 90 ms |
| HDD seek | 8 ms |
| TLS 1.3 handshake | 1 RTT |

| Throughput | Per box |
|---|---|
| App server, real business logic | 1k–5k rps |
| NGINX / reverse proxy | 50k–500k rps |
| Redis | 100k ops/s (1M pipelined) |
| PostgreSQL | 20k reads/s, 10k writes/s |
| MySQL | 50k reads/s |
| Cassandra | 20k writes/s |
| ScyllaDB | 200k+ writes/s |
| Elasticsearch indexing | 20k docs/s |
| Kafka broker | 100+ MB/s |
| S3, per prefix | 3,500 writes/s |
| SFU (video) | 1k streams |
| bcrypt | 10 hashes/s per core |

| Capacity threshold | Number |
|---|---|
| PostgreSQL | partition at 100M rows, shard at 1–2 TB |
| MySQL | shard at 500 GB–1 TB |
| Redis | shard at 25–50 GB |
| Cassandra | partition < 100 MB, node 1–2 TB |
| DynamoDB | 3,000 RCU / 1,000 WCU / 10 GB per partition, 400 KB item |
| Elasticsearch | shard 10–50 GB, heap ≤ 31 GB |
| Kafka | ≤ 4,000 partitions per broker, message ≤ 1 MB |
| MongoDB | document ≤ 16 MB |
| etcd | ≤ 8 GB |
| Prometheus | ≤ 10M series |
| Temporal | history ≤ 50 MB |
| Lambda | 15 min / 10 GB |

| Namespace | Capacity |
|---|---|
| Kafka | 200k partitions (ZooKeeper), millions (KRaft) |
| Pulsar | millions of topics |
| RabbitMQ | 10k–100k queues per node |
| Kinesis | 500 shards per stream |
| Pub/Sub | 10k topics per project |
| SNS | 100k topics |

| IDs, clocks, geo | Number |
|---|---|
| Snowflake | 64 bits = 41 timestamp + 10 node + 12 sequence → 4M IDs/s per node |
| UUIDv7 | time-sortable, index-friendly |
| NTP accuracy | ±1–50 ms |
| Geohash length 6 | 1.2 km |
| H3 resolution 8 | 0.74 km² |
| S2 level 30 | 1 cm² |

| Consistency | Number |
|---|---|
| Strong reads | R + W > N |
| Quorum write | 2–5 ms in-region, 60–100 ms cross-region |
| Two-phase commit | 2 RTT, blocking |
| Async replica lag | 1 ms – 1 s |
| Read-your-writes | pin to primary for 1–5 s |

| Conversion | Result |
|---|---|
| 1M per day | ≈ 12 rps |
| 1B per day | ≈ 11.6k rps |
| Peak | 3× average |
| Seconds per day / year | 86,400 / 31.5M |
| Fiber round trip | 1 ms per 100 km |
| 1 token | ≈ 4 characters |
| Polling load | clients ÷ interval = rps |

| Law | Statement |
|---|---|
| Little | concurrency = rps × latency |
| Queueing | delay explodes past 70–80% utilization |
| Serial availability | five 99.9% services = 99.5% |
| Four nines | 53 min downtime per year |

| Resilience | Setting |
|---|---|
| Retry budget | ≤ 10% of requests, full jitter |
| Circuit breaker | open at 50% errors over 10 s |
| RPO / RTO | backup hours/hours · warm standby seconds/minutes · active-active ~0/seconds |
| Autoscaling | 1–5 min to react |

**Design ladder (say it in this order)**
`Index → cache → read replica → CDN → async/queue → partition → archive → shard → multi-region`

---

## Part 1 — Foundations

*Sections 1–4 · ~10 min · **Read this part even if you skip everything else.***

Four sections that everything else is built on: why numbers matter in the room, the latency ladder from CPU cache to the far side of the planet, the arithmetic that turns "10 million users" into "how many machines", and what the network and the speed of light cost you.

If you only ever internalize one part of this article, make it this one. Every later number is a consequence of these.

---

### 1. Why numbers win interviews

> **Tier 1 — learn by heart.** The habit that scores: **say the number, say the assumption, say the tolerance**, all in one sentence.

A system design interview measures four things: structured problem solving, breadth of technology knowledge, depth in at least one area, and **quantitative judgment**. The last one is where most candidates lose points, and it is the easiest to fix.

Numbers do three jobs in an interview:

**They size the system.** "10 million daily active users, 20 actions each, so 200M requests/day, which is ~2,300 requests/sec average and maybe 7,000 at peak" instantly tells the interviewer whether you need one box or a fleet. Without it, every later decision is arbitrary.

**They justify choices.** "I'll shard the orders table" is an opinion. "Each order row is ~500 bytes, we write 50M/day, that's 25 GB/day and 9 TB/year — well past what one PostgreSQL primary handles comfortably, so I'll shard by customer_id" is an argument.

**They expose bottlenecks.** Capacity numbers are how you find the single component that will fail first. If your design needs 500,000 writes/sec into one relational primary, the number tells you before the interviewer does.

The failure modes to avoid are equally clear: inventing suspiciously precise figures ("this handles 47,300 QPS"), quoting benchmark numbers as production numbers (a "1M ops/sec" benchmark is a pipelined, single-key, no-network number), and forgetting the **peak-to-average ratio** — real traffic peaks at 2–10× the daily average, and you provision for peak.

> **Interview technique:** say the number, say the assumption, say the tolerance. *"I'll assume ~1 KB per message — if it's 10 KB, storage goes up 10× but the QPS math is unchanged."* This single habit reads as seniority.

---

### 2. The latency ladder every engineer should know

> **Tier 1 — learn by heart.** RAM 100 ns · NVMe SSD 50–100 µs · datacenter round trip 0.5 ms · cross-continent 100 ms. Every other latency in this article is a consequence of these four.

This is the foundation. Jeff Dean's "Latency Numbers Every Programmer Should Know" is still the single most valuable table in system design, modernized here for NVMe, current CPUs and cloud networks.

| Operation | Latency | Relative to L1 |
|---|---|---|
| L1 cache reference | 0.5–1 ns | 1× |
| Branch mispredict | 3–5 ns | ~5× |
| L2 cache reference | 4–7 ns | ~10× |
| Atomic increment / uncontended mutex lock-unlock | 15–25 ns | ~25× |
| L3 cache reference | 20–40 ns | ~40× |
| Main memory (DRAM) reference | **80–120 ns** | ~100× |
| Syscall (light, e.g. `getpid` with vDSO bypassed) | 50–500 ns | ~500× |
| Send 1 KB over 10 Gbps network (serialization only) | ~1 µs | ~1,000× |
| Context switch (thread) | 1–3 µs | ~2,000× |
| Compress 1 KB with Snappy / LZ4 | 1–3 µs | ~2,000× |
| NVMe SSD random 4 KB read | **20–100 µs** | ~50,000× |
| Read 1 MB sequentially from RAM | 40–100 µs (10–25 GB/s per core) | ~50,000× |
| SATA SSD random 4 KB read | 100–200 µs | ~150,000× |
| Round trip within the same rack | 100–250 µs | ~200,000× |
| Read 1 MB sequentially from NVMe (3–7 GB/s) | 150–350 µs | ~250,000× |
| Round trip within the same AZ / datacenter | **0.25–0.5 ms** | ~500,000× |
| Round trip across AZs in one region | **0.5–2 ms** | ~1,000,000× |
| Network-attached block storage (EBS) read | 0.5–2 ms | ~1,000,000× |
| Read 1 MB sequentially from SATA SSD (500 MB/s) | ~2 ms | ~2,000,000× |
| HDD seek | **4–10 ms** | ~8,000,000× |
| Read 1 MB sequentially from HDD (150 MB/s) | 6–10 ms | ~8,000,000× |
| Cross-region round trip (US-East → US-West) | 60–80 ms | ~70,000,000× |
| Cross-continent round trip (US → Europe) | **75–100 ms** | ~90,000,000× |
| Intercontinental round trip (US → Asia / Australia) | 120–250 ms | ~200,000,000× |

**The five ratios to memorize:**

- Memory is ~**100×** faster than NVMe SSD, which is ~**100×** faster than an HDD seek.
- A same-datacenter round trip (~0.5 ms) costs as much as **5,000 memory reads**.
- A cross-continent round trip (~100 ms) costs as much as **200 same-DC round trips**.
- **Disk is not the enemy any more; the network and the speed of light are.**
- Anything a human perceives as instant is **~100 ms**; anything over **1 s** feels slow; over **10 s** they leave.

#### Human perception budgets

| Target | Budget | Implication |
|---|---|---|
| Feels instantaneous | < 100 ms | Everything must be in-memory / edge-cached |
| Keeps flow of thought | < 1 s | One or two DB round trips + render |
| Keeps attention | < 10 s | Needs a progress indicator |
| API p99 SLO (typical internal) | 100–300 ms | Budget across 3–5 hops |
| API p99 SLO (typical public) | 200 ms–1 s | Includes TLS + internet RTT |
| Search-as-you-type | < 100 ms | Prefix index in memory |
| Video start (join time) | < 2 s | CDN + adaptive bitrate |

---

### 3. The back-of-the-envelope toolkit

> **Tier 1 — learn by heart.** **1M requests/day ≈ 12 rps**, peak is 3× average, and concurrency = throughput × latency.

#### Time and scale constants

| Constant | Value | Use |
|---|---|---|
| Seconds per day | **86,400 (~10⁵)** | Convert daily volume → QPS |
| Seconds per month | 2.6M | Monthly billing/storage math |
| Seconds per year | **31.5M (π × 10⁷)** | Annual storage growth |
| 1 M/day | ≈ **12 requests/sec** | The most useful conversion |
| 1 B/day | ≈ **11,600 requests/sec** | Web-scale conversion |
| Peak-to-average ratio | 2–10× (use **3×** by default) | Provisioning |
| Read:write ratio, consumer apps | 10:1 to 1000:1 | Justifies caches and replicas |

**The one conversion to never forget:** *X million requests per day ÷ 100,000 ≈ X × 10 requests/second.* (86,400 ≈ 10⁵ makes the mental arithmetic trivial.)

#### Powers of two and data sizes

| Power | Approx | Name | Bytes |
|---|---|---|---|
| 2¹⁰ | 1 thousand | Kilobyte | 1 KB |
| 2²⁰ | 1 million | Megabyte | 1 MB |
| 2³⁰ | 1 billion | Gigabyte | 1 GB |
| 2⁴⁰ | 1 trillion | Terabyte | 1 TB |
| 2⁵⁰ | 1 quadrillion | Petabyte | 1 PB |

| Data item | Typical size |
|---|---|
| `char` / ASCII byte | 1 B |
| `int32` / `float32` | 4 B |
| `int64` / `double` / timestamp | 8 B |
| UUID (binary / string) | 16 B / 36 B |
| Row overhead (Postgres tuple header) | ~24–28 B |
| Typical DB row (a few columns) | 100–500 B |
| Tweet / short post (text + metadata) | 300 B – 1 KB |
| JSON API response (small) | 1–10 KB |
| Web page HTML | 50–100 KB |
| Thumbnail image | 5–50 KB |
| Photo (JPEG, phone) | 200 KB – 5 MB |
| Minute of 1080p video (5 Mbps) | ~37 MB |
| Minute of 4K video (20 Mbps) | ~150 MB |
| Embedding vector (768 dims, fp32) | ~3 KB |

#### The three laws you can apply out loud

**Little's Law — `L = λ × W`** (concurrency = throughput × latency). This sizes thread pools, connection pools and queue depths.
*Example:* 2,000 rps at 50 ms average latency ⇒ 100 requests in flight ⇒ you need ~100 worker threads or ~100 DB connections' worth of concurrency (before adding headroom).

**Queueing (M/M/1) — `W = S / (1 − ρ)`.** At 50% utilization latency is 2× service time; at 80% it is 5×; at 90% it is 10×; at 95% it is 20×.
*Practical rule:* **plan for 60–70% steady-state CPU utilization.** Above ~80% your p99 becomes unpredictable. This is *the* reason "just add more traffic to the existing box" fails.

**Amdahl / Universal Scalability Law.** Speedup is capped by the serial fraction, and *worsens* with coherency cost. A system with 5% serial work caps at 20× no matter how many machines you add — the practical justification for sharding (no coordination) over a single coordinated cluster.

#### Storage estimation recipe

```
daily_bytes   = writes_per_day × bytes_per_record
yearly_bytes  = daily_bytes × 365
provisioned   = yearly_bytes × replication_factor × (1 + index_overhead) × (1 / compression) × years_retained / 0.7
```

Use **replication factor 3**, **index overhead 20–100%** for relational/search, **compression 3–10×** for columnar/log data, and divide by **0.7** because you never run a disk past 70% full.

#### Bandwidth estimation recipe

```
bandwidth = qps × avg_response_bytes × 8   (bits/sec)
```
*Example:* 10,000 rps × 50 KB responses = 500 MB/s = **4 Gbps** — already more than a single 1 Gbps NIC and a strong argument for a CDN.

---

### 4. Network, geography and protocol latency

> **Tier 1 — learn by heart.** **1 ms of round trip per 100 km** of fiber. Never put a cross-region call in a user-facing path.

Physics sets the floor. Light travels at ~300,000 km/s in vacuum and roughly **200,000 km/s in fiber**, and real routes are 1.5–2× longer than the great-circle distance.

> **The rule:** ~**5 µs per km one way**, ~**10 µs per km round trip** → **1 ms of RTT per 100 km of fiber**, before any router, firewall or middlebox.

| Path | Distance | Theoretical RTT | Real-world RTT |
|---|---|---|---|
| Same rack | — | — | 0.1–0.25 ms |
| Same availability zone | < 5 km | ~0.05 ms | 0.25–0.5 ms |
| Cross-AZ, same region | 10–100 km | 0.1–1 ms | 0.5–2 ms |
| US-East ↔ US-West | ~4,000 km | ~40 ms | 60–80 ms |
| US-East ↔ Europe (N. Virginia ↔ Frankfurt) | ~6,600 km | ~66 ms | 80–95 ms |
| Europe ↔ Asia (Frankfurt ↔ Singapore) | ~10,000 km | ~100 ms | 150–180 ms |
| US-West ↔ Asia (Oregon ↔ Tokyo) | ~8,000 km | ~80 ms | 100–130 ms |
| US ↔ Australia | ~16,000 km | ~160 ms | 180–220 ms |
| Mobile 4G LTE first hop | — | — | 30–80 ms |
| Mobile 5G first hop | — | — | 10–30 ms |
| Home broadband first hop | — | — | 5–20 ms |
| Satellite (LEO, Starlink) | — | — | 25–60 ms |
| Satellite (geostationary) | 35,786 km × 2 | — | 500–650 ms |

#### Protocol handshake costs (in RTTs — multiply by the RTT above)

| Step | Cost | Notes |
|---|---|---|
| DNS lookup (uncached) | 1–3 RTT, 20–120 ms | Cached at OS/browser for the TTL (typ. 30–300 s) |
| TCP handshake | 1 RTT | SYN, SYN-ACK, ACK |
| TLS 1.2 handshake | 2 RTT | On top of TCP |
| TLS 1.3 handshake | 1 RTT | Default in modern stacks |
| TLS 1.3 session resumption (0-RTT) | 0 RTT | Replay-unsafe for non-idempotent requests |
| QUIC / HTTP/3 first connect | 1 RTT (0-RTT resumed) | TCP+TLS combined |
| Total cold HTTPS request (80 ms RTT) | ~300–400 ms | DNS + TCP + TLS + request |
| Warm HTTPS request on a keep-alive connection | 1 RTT + service time | Why connection pooling matters so much |

**Interview-grade consequences:**
- A cold mobile HTTPS request costs ~0.5 s before your server does any work. Keep-alive, HTTP/2 multiplexing and TLS session resumption are the fixes.
- **Never make a synchronous cross-region call in a user-facing path.** One US↔EU round trip (~90 ms) blows most p99 budgets on its own.
- N+1 query patterns are lethal at scale: 100 sequential queries at 0.5 ms intra-DC RTT = 50 ms of pure network time.
- Cross-AZ calls are ~1 ms and **cost money** (see the cost section); cross-AZ *chattiness* is a common real-world cost bug.

#### Throughput per host

| Link | Throughput | 1 GB transfer takes |
|---|---|---|
| 1 Gbps | 125 MB/s | 8 s |
| 10 Gbps (typical cloud VM) | 1.25 GB/s | 0.8 s |
| 25 Gbps | 3.1 GB/s | 0.3 s |
| 100 Gbps (large instances) | 12.5 GB/s | 0.08 s |
| Single TCP stream, cross-region | 20–100 MB/s (window/RTT limited) | — |

*Bandwidth-delay product:* a single TCP stream is capped at `window / RTT`. With a 64 KB window and 80 ms RTT that is **~800 KB/s**. Bulk cross-region transfer needs parallel streams (or a bigger window) — this is why S3 clients use 8–32 parallel parts.

---

## Part 2 — The request path

*Sections 5–8 · ~12 min*

Follow a single request from the client to your business logic: through the load balancer, past the rate limiter, through authentication, into an application server. Each hop has a capacity ceiling and a latency cost, and interviewers love to ask about exactly the ones people skip — how many requests a server *really* handles, what a TLS handshake costs, and why the login endpoint is the most expensive route in most products.

---

### 5. Application servers: requests per second per box

> **Tier 1 — learn by heart.** A typical API server doing real work handles **1,000–5,000 rps**. Size it at 70% CPU, not 100%.

The single most-asked number: **how much traffic does one server handle?** The honest answer is "it depends on what it does per request," so carry a ladder.

#### The mental model

```
rps_per_server ≈ cores × (1 / cpu_seconds_per_request) × 0.7 utilization headroom
```

If a request burns 5 ms of CPU, one core does 200 rps, so a 16-core box does ~3,200 rps at 100% and ~**2,200 rps at a safe 70%**.

| Work per request | CPU time | rps per core | rps on a 16-core box (at 70%) |
|---|---|---|---|
| Static file / health check | 0.05–0.1 ms | 10,000–20,000 | 100k–200k |
| Simple JSON echo, no I/O | 0.2–0.5 ms | 2,000–5,000 | 25k–55k |
| Cache read + serialize | 0.5–1 ms | 1,000–2,000 | 12k–22k |
| Typical CRUD: 1–3 DB queries + JSON | 2–10 ms | 100–500 | **1,500–5,500** |
| Heavy business logic / aggregation | 20–50 ms | 20–50 | 200–500 |
| Image resize / thumbnail | 50–200 ms | 5–20 | 50–200 |
| ML inference (small CPU model) | 10–100 ms | 10–100 | 100–1,000 |

> **The number to quote when in doubt: a typical API server handles ~1,000–5,000 rps of real business logic.** Modern hardware is fast; the classic "500 rps per server" figure is a decade out of date, but so is assuming 50k.

#### By runtime (16-core box, "hello world" vs. realistic CRUD)

| Stack | Trivial response | Realistic request (DB + JSON) | Concurrency model |
|---|---|---|---|
| Rust (Actix, Axum) | 200k–1M rps | 10k–50k rps | async, work-stealing |
| Go (net/http, Fiber) | 100k–500k rps | 8k–40k rps | goroutines (M:N) |
| Java / Kotlin (Netty, Vert.x, Spring WebFlux) | 100k–400k rps | 8k–30k rps | event loop / virtual threads |
| Java Spring Boot (blocking MVC) | 30k–100k rps | 3k–15k rps | thread per request |
| C# / .NET (ASP.NET Core, Kestrel) | 100k–400k rps | 8k–30k rps | async |
| Node.js (single process) | 8k–20k rps | 1k–4k rps | single-threaded event loop |
| Node.js (cluster, 16 workers) | 60k–150k rps | 6k–20k rps | process per core |
| Python (FastAPI + uvicorn, async) | 5k–20k rps | 1k–5k rps | async |
| Python (Django/Flask + gunicorn sync) | 2k–8k rps | 300–2,000 rps | process/thread per request |
| Ruby on Rails (Puma) | 2k–8k rps | 300–2,000 rps | thread/process |
| PHP (PHP-FPM, 8.x + OPcache) | 5k–20k rps | 500–3,000 rps | process per request |

**Interpretation for interviews:** language choice moves throughput by ~10× on trivial work but only ~3× on realistic work, because realistic work is dominated by I/O waits and serialization. Say this out loud — it shows you know where the time actually goes.

#### Connections, threads and memory

| Quantity | Typical value | Notes |
|---|---|---|
| Idle TCP connection kernel memory | 4–16 KB | Plus app buffers |
| Connection with app buffers | 20–100 KB | Node/Java per-socket overhead |
| Max concurrent connections per box | 100k–1M | The "C10M" problem; needs tuning of `somaxconn`, file descriptors, ephemeral ports |
| Practical WebSocket connections per node | **50k–200k** | 500k–1M achievable with lean runtimes (Go/Erlang/Rust) |
| Ephemeral port exhaustion | ~28k–64k per (src IP, dst IP, dst port) tuple | Why outbound connection pooling matters |
| OS thread stack (default) | 1–8 MB virtual, ~8–64 KB resident | Limits thread-per-request models to ~a few thousand |
| Goroutine / virtual thread | 2–8 KB | Enables 100k+ concurrent handlers |
| Java G1 GC pause | 50–200 ms | Often the real cause of p99 spikes |
| Java ZGC / Shenandoah pause | < 1–10 ms | Choose for latency-sensitive services |
| JVM heap sweet spot | 8–31 GB | Stay under 32 GB for compressed OOPs |
| Container startup (image pulled) | 0.5–3 s | Affects autoscaling reaction time |
| Autoscaling reaction time (end to end) | 1–5 min | Why you provision for peak, not average |

---

### 6. Load balancers, proxies and API gateways

> **Tier 2 — know the rough value.** One proxy box does **50k–500k rps**; the expensive part is TLS handshakes at **1–2k/core/s** with RSA.

| System | Throughput | Latency added | Concurrent connections |
|---|---|---|---|
| NGINX (static/reverse proxy) | 50k–500k rps per box; 10k–50k rps per core | **0.1–1 ms** | 100k–1M |
| HAProxy | 100k–500k rps per box | 0.1–0.5 ms | 1–2M |
| Envoy (sidecar/mesh) | 10k–50k rps per core | 0.3–2 ms per hop | 100k+ |
| Traefik / Caddy | 20k–100k rps per box | 0.5–2 ms | 50k+ |
| AWS NLB (L4) | Millions of rps, tens of Gbps | ~0.1 ms | Millions of flows |
| AWS ALB (L7) | 100k+ rps (scales automatically, pre-warm for spikes) | 1–5 ms | Tens of thousands per LCU |
| AWS API Gateway (REST) | 10,000 rps default quota per region | 10–50 ms | 29 s max integration timeout |
| Cloudflare / CDN edge | Effectively unbounded | 5–30 ms to edge | — |
| Service mesh (2 sidecars per call) | — | +1–4 ms p99 | The classic mesh tax |

#### TLS termination — the CPU cost people forget

| Operation | Per core |
|---|---|
| RSA-2048 handshakes | **1,000–2,000 /s** |
| ECDSA P-256 handshakes | 5,000–15,000 /s |
| Resumed sessions (ticket/PSK) | 20,000–50,000 /s |
| AES-GCM bulk encryption (AES-NI) | 1–10 GB/s |

A fleet taking 50,000 new HTTPS connections per second with RSA certs needs **~30 cores of pure handshake work**. Keep-alive and session resumption are not micro-optimizations; they are capacity decisions.

#### Load balancing algorithms — when each wins

| Algorithm | Best for | Caveat |
|---|---|---|
| Round robin | Uniform requests, uniform servers | Ignores slow servers |
| Least connections | Variable request duration | Needs shared state or per-LB view |
| Power of two choices | Large fleets | Near-optimal with O(1) state — the modern default |
| Consistent hashing | Cache affinity, sticky sessions | 100–200 virtual nodes per physical node to balance within ±5% |
| Latency/EWMA weighted | Heterogeneous fleets | Can oscillate without damping |

---

### 7. API design: rate limits, quotas, pagination and timeouts

> **Tier 2 — know the rough value.** Token bucket for limits, **retry budget ≤ 10%** with full jitter, and cursor pagination past a few thousand rows.

Every design that exposes an API gets asked "how do you stop one client from taking the system down?" These are the numbers behind the answer.

#### Rate limiter algorithms

| Algorithm | Memory per key | Accuracy | Burst behavior |
|---|---|---|---|
| Fixed window counter | ~16 B (one int + expiry) | Allows 2× the limit at a window boundary | Spiky |
| Sliding window log | 16 B × requests in window (100 req/min ≈ 1.6 KB) | Exact | Smooth, expensive |
| Sliding window counter | ~32 B | < 1% error | Smooth — the usual production choice |
| Token bucket | ~32 B (tokens + last refill ts) | Exact | Burst = bucket size, sustained = refill rate |
| Leaky bucket (queue) | Queue depth × item | Exact | Smooths output completely, adds latency |

**Distributed limiter cost:** a Redis-backed limiter is **1 round trip (0.2–1 ms) and ~1–2 Redis ops per request**. At 100k rps that is 100k–200k Redis ops/s — one node, but now every request depends on Redis. Above ~1M rps use **local token buckets per instance with a global budget divided by instance count**, resynced every 1–10 s (accept ~5–10% overshoot).

#### Typical published API limits (useful as calibration)

| Service | Limit |
|---|---|
| GitHub REST API | 5,000 requests/hour authenticated (60/h anonymous) |
| Stripe | 100 requests/second per account (live mode) |
| Google Maps Platform | 50 QPS per API by default |
| AWS API Gateway | 10,000 rps per region, 5,000 burst |
| Twilio (long code SMS) | 1 message/second per number |
| Typical SaaS free tier | 10–100 requests/minute per user |
| Typical internal service quota | 1,000–10,000 rps per caller |
| Login endpoint (anti-credential-stuffing) | 5–10 attempts/minute per account, 100/hour per IP |

#### Request-shaping numbers

| Knob | Typical value |
|---|---|
| Page size (list endpoints) | 20–100 items default, 1,000 max |
| Offset pagination cost | `OFFSET 100000` scans 100k rows — use cursor/keyset pagination past a few thousand |
| Max request body | 1–10 MB (413 beyond); uploads go to presigned S3 URLs |
| Connect timeout | 1–3 s |
| Read timeout | p99.9 × 1.5, typically 1–10 s |
| Total client timeout budget | Must be less than the caller's timeout, or you retry a request that is still running |
| Retry policy | 2–3 attempts, exponential backoff with full jitter, base 100 ms, cap 10–30 s |
| Retry budget | ≤ 10% of requests — otherwise a brownout becomes an outage (3 retries = 4× load) |
| Circuit breaker | Open at 50% errors over a 10 s / ≥20 request window, half-open probe after 5–30 s |
| Bulkhead (max concurrent per dependency) | `rps × p99_latency` from Little's Law, + 20–50% |
| Idempotency key record | ~200–500 B, 24 h TTL; SQS FIFO dedup window is 5 minutes |
| Load shedding threshold | Shed at > 80% CPU or when queue wait > 1× timeout (fail fast beats timing out) |

---

### 8. Identity, auth, ID generation and clocks

> **Tier 2 — know the rough value.** **bcrypt is ~10 hashes/second per core**, which makes login your most expensive endpoint. Snowflake gives 4M IDs/s per node.

#### Password hashing — the most-forgotten capacity number

| Algorithm | Cost per hash | Hashes per core per second |
|---|---|---|
| bcrypt, cost 10 | 50–100 ms | 10–20 |
| bcrypt, cost 12 (OWASP default) | 200–400 ms | 2.5–5 |
| Argon2id (19 MB, t=2, p=1) | 50–100 ms + 19 MB RAM | 10–20 (memory-bound) |
| PBKDF2-HMAC-SHA256, 600k iterations | ~100 ms | ~10 |
| scrypt (N=16384, r=8) | ~100 ms + 16 MB | ~10 |
| SHA-256 alone (never use for passwords) | ~1 µs | 1,000,000+ |

**This is a real capacity constraint:** 1,000 logins/second at bcrypt cost 12 needs **200–400 CPU cores** of pure hashing. Login is the most expensive endpoint in most products. Interview-grade answers isolate auth into its own autoscaled service, or move to a token-based flow so hashing happens once per session rather than once per request.

#### Tokens and sessions

| Item | Value |
|---|---|
| JWT size (HS256, small claims) | 300–800 B; with roles/permissions 1–4 KB |
| JWT overhead at scale | 1 KB × 10k rps = 10 MB/s of pure header traffic |
| HMAC-SHA256 verify | 1–5 µs (essentially free) |
| RS256 sign / verify | 1–2 ms / 50–100 µs — verify is cheap, signing is not |
| Session cookie + Redis lookup | 50–200 B cookie, 0.2–1 ms lookup |
| Session store size | 100M sessions × 200 B = 20 GB |
| Access token TTL | 5–60 minutes |
| Refresh token TTL | 7–90 days |
| Revocation lag with stateless JWTs | = token TTL (the core JWT tradeoff — quote this) |
| TLS certificate lifetime | 90 days (Let's Encrypt), 398 days max public CA |

#### ID generation — know these cold

| Scheme | Bits | Rate | Properties |
|---|---|---|---|
| DB auto-increment | 64 | 1 per insert, needs coordination | Sequential (index-friendly), leaks volume, single point |
| UUIDv4 | 128 (122 random) | Unlimited, no coordination | Random → clustered-index fragmentation; 2–5× slower inserts in MySQL |
| UUIDv7 / ULID | 128 (48-bit ms timestamp + random) | Unlimited | Time-sortable and index-friendly — the modern default |
| Snowflake | 64 = 1 sign + 41-bit ms timestamp (69 years) + 10-bit machine (1,024 nodes) + 12-bit sequence (4,096/ms) | 4.096M IDs/sec/node, ~4B/sec cluster-wide | Sortable, compact, needs clock discipline |
| Ticket server / range allocation | 64 | 1 DB round trip per 1,000–10,000 IDs | Batch amortizes coordination |
| Base62 short code | 6 chars = 56.8B, 7 = 3.5T, 8 = 218T | — | For URL shorteners |
| Content hash (SHA-256 / blake3) | 256 | Hash speed-bound | Natural dedup key |

**Collision math worth quoting:** UUIDv4 needs ~**2.7 × 10¹⁸** IDs for a 50% collision chance — you will never hit it. A 12-bit Snowflake sequence *will* be exhausted if one node exceeds 4,096 IDs in a single millisecond, so the generator spins to the next ms.

#### Clocks — why "just use the timestamp" is a trap

| Property | Value |
|---|---|
| NTP sync accuracy over WAN | 1–50 ms |
| NTP sync accuracy on LAN | 0.1–1 ms |
| PTP (hardware timestamping) | sub-microsecond |
| Google TrueTime uncertainty ε | ~1–7 ms — Spanner *waits out* ε on commit |
| Untuned crystal drift | 50–200 ppm ≈ 5–17 seconds per day |
| AWS Time Sync Service | ~1 ms, microsecond-accurate on supported instances |

Consequences: **last-write-wins by wall clock can silently lose a write** when clocks differ by more than the inter-write gap; Snowflake IDs need drift protection (refuse to emit IDs if the clock goes backwards); and any "ordering" guarantee across machines needs **logical clocks** — Lamport timestamps, vector clocks (size grows with the number of writers), or hybrid logical clocks (HLC: physical ms + logical counter, bounded by NTP error).

---

## Part 3 — Data at rest

*Sections 9–14 · ~18 min · **The heart of most interviews.***

This is where the interview usually lands and where candidates usually lose points. Cache first; then the relational databases and the question every interview asks, *when do you shard?*; then the consistency model that decision commits you to; then the NoSQL stores and search engines you would reach for instead.

Section 12 (the sharding thresholds table) is the single most useful page in this article. If you print one thing, print that.

---

### 9. Caching: Redis, Memcached and friends

> **Tier 1 — learn by heart.** Redis is **0.2–1 ms and ~100k ops/s per node**; shard past 25–50 GB. A 95% hit rate is a **20× cut** in origin load.

#### Redis / Valkey figures of merit

| Metric | Value |
|---|---|
| Server-side latency, simple GET/SET | 0.05–0.2 ms (p50 ~0.1 ms) |
| **Client-observed latency, same AZ** | **0.2–1 ms** (p99 ~1–2 ms) — dominated by network RTT |
| Client-observed latency, cross-AZ | 1–3 ms |
| Throughput, single instance (single-threaded core) | **80k–150k ops/sec** |
| Throughput with pipelining (batches of 10–100) | 500k–1.5M ops/sec |
| Throughput with I/O threads (Redis 6+, 4–8 threads) | 200k–500k ops/sec |
| Memcached (multithreaded, 8+ cores) | 500k–1M+ ops/sec |
| Network is the limit at | ~100k ops/s × 1 KB = 800 Mbps |
| Max value size (hard limit) | 512 MB — keep values < 10 KB, ideally < 1 KB |
| "Big key" warning threshold | > 10 KB value or > 5,000 collection elements |
| Practical memory per node/shard | **10–50 GB** (up to 100–256 GB, but failover/RDB fork/replication get painful) |
| Memory overhead per key | ~50–100 B (plus key and value) |
| Redis Cluster hash slots | 16,384 (fixed) |
| Redis Cluster recommended max nodes | ~1,000 |
| Replication lag (async) | < 1–10 ms same AZ |
| Failover time (Sentinel/Cluster) | 1–30 s (dominated by failure detection timeout) |
| `KEYS *` on 1M keys | Blocks for ~100 ms–1 s — never do it; use `SCAN` |
| Persistence: RDB fork on 20 GB dataset | 100 ms–1 s latency spike (copy-on-write) |
| AOF `everysec` write overhead | < 1 ms typical, fsync spikes |
| HyperLogLog: cardinality of billions | 12 KB, 0.81% standard error |
| Redis Streams / Pub-Sub throughput | 100k–1M msgs/sec (no durability guarantees in Pub/Sub) |

#### The cache math you should show on the whiteboard

```
effective_latency = hit_ratio × cache_latency + (1 − hit_ratio) × origin_latency
```
With a 95% hit ratio, 0.5 ms cache and 20 ms DB: `0.95 × 0.5 + 0.05 × 20 = 1.5 ms` — a **13× improvement**, and the backend sees **20× less load**.

| Cache metric | Healthy target |
|---|---|
| Hit ratio, hot-key workload | 90–99% |
| Hit ratio, long-tail workload | 50–80% |
| Cache size needed for 80% hit rate | Often only 10–20% of the dataset (Zipf/Pareto) |
| TTL for volatile data | 30 s – 5 min |
| TTL for reference data | 1–24 h |
| CDN/browser TTL for static assets | 1 year + content hash in filename |

**Failure modes with numbers:** a **thundering herd** on one expired hot key can send 10,000 simultaneous requests to a database that handles 500. The fix is request coalescing / single-flight or probabilistic early expiry. A **hot key** taking more than ~20–25% of a shard's traffic will saturate one core regardless of cluster size. The fix is client-side local caching or key splitting (`key:shard:0..N`). A **cold-start** cache after a deploy can take the origin down; warm it or ramp traffic.

#### Other cache tiers

| Layer | Latency | Capacity | Notes |
|---|---|---|---|
| CPU/local process cache (Caffeine, LRU map) | ~100 ns – 1 µs | MBs–GBs | Fastest, but per-instance inconsistency |
| Redis / Memcached (same AZ) | 0.2–1 ms | 10s of GB per node, TBs per cluster | Shared, consistent |
| CDN edge | 10–50 ms (to user) | Effectively unlimited | 85–95% offload target |
| Database buffer pool / page cache | 1–100 µs | RAM-sized | Free and often forgotten |
| Materialized view / precomputed table | 1–10 ms | Disk-sized | Trades freshness for latency |

#### Caching strategies and what each costs

| Strategy | Read path | Write path | Staleness | When to use |
|---|---|---|---|---|
| Cache-aside (lazy loading) | Miss → DB → populate | Write DB, invalidate cache | Until TTL or invalidation | The default. Simple, resilient to cache loss |
| Read-through | Cache library fetches on miss | Same as cache-aside | Same | Cleaner code, ties you to the cache client |
| Write-through | Always fresh | Write DB and cache (+0.2–1 ms) | None | Read-heavy data that's always re-read |
| Write-behind (write-back) | Fast | Write cache, flush to DB in 1 s – 1 min batches | Risk of data loss on cache failure | Counters, metrics, high-write low-value data |
| Refresh-ahead | Never misses on hot keys | Async refresh at ~75% of TTL | Small | Predictable hot keys, expensive recomputation |

| Guard | Value |
|---|---|
| Negative caching (cache the miss) | 30–60 s TTL — stops a nonexistent key from hammering the DB |
| TTL jitter | ±10–20% — prevents synchronized mass expiry |
| Single-flight / request coalescing | Collapses N concurrent misses into 1 DB query |
| Probabilistic early expiration | Refresh when `now > expiry − β × log(rand()) × recompute_time` |
| Stale-while-revalidate | Serve stale up to 60 s while refreshing in the background |

---

### 10. Relational databases: PostgreSQL and MySQL

> **Tier 1 — learn by heart.** One node: **20k reads/s, 10k writes/s, 1–2 TB**. Partition at 100M rows, shard past 1–2 TB.

This is where interviews live or die, because "when do I shard?" is the single most common follow-up question in the entire format.

#### PostgreSQL figures of merit

| Metric | Value |
|---|---|
| Point lookup by primary key (warm, in buffer cache) | 0.1–0.5 ms |
| Point lookup requiring disk (NVMe) | 0.5–2 ms |
| Simple indexed query returning 100 rows | 1–5 ms |
| Unindexed scan of 1M rows | 200 ms – 2 s |
| Single-row INSERT/UPDATE commit (fsync, NVMe) | 0.5–2 ms |
| Read throughput, single node (in-memory working set) | **20k–100k QPS** (point reads, prepared statements) |
| Write throughput, single node | **5k–20k TPS** (up to 50k+ with batching/group commit and fast WAL disk) |
| pgbench TPC-B-like on a big box | 10k–30k TPS |
| `max_connections` default | 100 |
| Memory per backend connection | 5–15 MB |
| Practical connection limit without pooling | **200–500** (performance degrades past ~2–4× core count of *active* connections) |
| PgBouncer pool size rule of thumb | `(cores × 2) + effective_spindles`, typically 20–100 server connections |
| PgBouncer capacity | 10k+ client connections per instance, ~1–2 ms added |
| `shared_buffers` | 25% of RAM (rest to OS page cache) |
| Page size | 8 KB |
| Row overhead (tuple header) | ~24–28 B |
| Max table size | 32 TB (default block size) |
| Max row size | 1.6 TB (via TOAST); values > 2 KB TOASTed out-of-line |
| Max columns per table | 250–1,600 |
| **Partition a table at** | **> 100 GB or > 100M rows** (declarative partitioning; keep partitions < 100 and ideally < 1,000) |
| **Shard the database at** | **> 1–2 TB of hot data, > 10–20k write TPS, or when the working set stops fitting in RAM** |
| Practical max data on one node | 2–10 TB (backups, `VACUUM`, index rebuilds, failover all get slow past this) |
| Read replicas per primary | 5–15 (cascade beyond that) |
| Streaming replication lag (async, same region) | 1–100 ms; seconds under heavy write load |
| Synchronous replication cost | +1 RTT per commit (+0.5–2 ms same-region, +60–100 ms cross-region) |
| Logical replication / CDC lag | 10 ms – 1 s |
| Index size | ~10–40% of table size per B-tree index |
| B-tree depth for 100M rows | 3–4 levels (≈ 3–4 page reads worst case) |
| Autovacuum threshold | 20% of rows changed (default); tune to 2–5% for hot tables |
| Transaction ID wraparound | 2.1 B transactions — must vacuum before this or the DB shuts down |
| Bloat if vacuum lags | 20–200% table growth |
| Failover time (Patroni/RDS Multi-AZ) | 30–120 s |
| Backup/restore of 1 TB | 30 min – 4 h (physical); 4–24 h logical (`pg_dump`) |
| Connection establishment cost | 5–20 ms (process fork) — always pool |

#### MySQL / InnoDB figures of merit

| Metric | Value |
|---|---|
| Point lookup by PK (in buffer pool) | 0.1–0.3 ms |
| Read throughput, single node | 30k–150k QPS (point reads; sysbench on modern hardware) |
| Write throughput, single node | 5k–30k TPS (`innodb_flush_log_at_trx_commit=1`); 50k–100k+ with `=2` (risk of losing ~1 s on crash) |
| `max_connections` default | 151 (practical 500–2,000 with thread pool) |
| Memory per connection | 256 KB – 1 MB (much lighter than Postgres) |
| `innodb_buffer_pool_size` | 70–80% of RAM — the single most important setting |
| Page size | 16 KB |
| Clustered index | PK is the clustered index → use a monotonic PK; random UUID PKs cause page splits and can cut insert throughput by 2–5× |
| Secondary index lookup | Index → PK → row (2 B-tree traversals) |
| **Partition a table at** | **> 50–100M rows or > 50–100 GB** |
| **Shard at** | **> 500 GB – 1 TB per instance, or > 10–20k write TPS** |
| Vitess/PlanetScale shard size guidance | ≤ 250–500 GB per shard (for fast backup, restore and resharding) |
| Replication lag (single-threaded legacy) | seconds to minutes under load |
| Replication lag (parallel replication, MySQL 8) | 10–500 ms typical |
| Group Replication / InnoDB Cluster write cost | +1 RTT for consensus |
| Online DDL on 100M-row table | minutes to hours (use `gh-ost` / `pt-online-schema-change`) |
| Failover (Orchestrator / RDS) | 30–120 s |
| Aurora MySQL/Postgres | Up to 15 read replicas, < 100 ms replica lag, 128 TB volume, 6 copies across 3 AZs |

#### Cloud managed limits worth quoting

| Service | Figure |
|---|---|
| Amazon RDS max storage | 64 TB (Postgres/MySQL), 128 TB Aurora |
| Aurora replica lag | typically < 20–100 ms (shared storage, not log shipping) |
| Google Cloud Spanner | Externally consistent; ~10 ms commit latency regional, 5–10× that multi-region; splits at ~a few GB; 10k+ QPS per node |
| CockroachDB | ~2–10 ms reads regional, ranges split at 512 MB, linear scaling to 100s of nodes |
| Vitess | Powers YouTube-scale MySQL; 10k+ shards possible |
| Citus (distributed Postgres) | Shard count typically 32–128 shards per node group; 1 GB–50 GB per shard |

#### The five signals that you have outgrown one relational node

1. **Working set > RAM.** Buffer cache hit ratio drops below ~98% and p99 explodes (0.2 ms → 5 ms).
2. **Write throughput > ~10–20k TPS**, or WAL/binlog writes saturating the disk.
3. **Data > 1–2 TB**, making backups, restores, index builds and failover operationally painful.
4. **Vertical scaling exhausted** — you are already on the biggest instance and it is 70% utilized.
5. **Replica lag persistently > 1 s**, breaking read-after-write expectations.

> The correct interview ordering is: **index → cache → read replicas → partition → archive cold data → shard.** Sharding is the last resort because it costs you cross-shard joins, distributed transactions, and rebalancing. Say the ladder out loud before you jump to sharding.

---

### 11. Consistency, quorums and distributed transactions

> **Tier 2 — know the rough value.** **R + W > N** for strong reads, and strong consistency across regions costs **60–100 ms per write**.

#### Quorum math

With `N` replicas, `W` write acks and `R` read responses, **strong consistency requires `R + W > N`**.

| Config (N=3) | Semantics | Write latency | Read latency | Tolerates |
|---|---|---|---|---|
| W=1, R=1 | Eventual, fastest | 1 replica RTT (~1 ms) | ~1 ms | 2 failures |
| W=2, R=2 (quorum) | Strong-ish (read-your-writes) | 2nd-fastest replica (~2–5 ms) | 2–5 ms | 1 failure |
| W=3, R=1 | Fast reads, fragile writes | slowest replica (tail-dominated) | ~1 ms | 0 write failures |
| W=1, R=3 | Fast writes, slow reads | ~1 ms | slowest replica | 0 read failures |

**Tail-at-scale:** waiting for the slowest of N replicas means your p99 becomes roughly the p99 of the *slowest* of N: with N=3 and per-replica p99 of 10 ms, "wait for all" lands near 20–30 ms. **Hedged requests** (send a duplicate after p95 elapses) cut p99 by **2–10×** for about **5% extra load**, a great detail to mention.

#### Consistency models and what each costs

| Model | Latency cost | Example |
|---|---|---|
| Linearizable / strong | **1 RTT to a quorum** + leader hop: 1–5 ms same region, **60–100 ms cross-region** | etcd, Spanner, single-leader RDBMS |
| Sequential / bounded staleness | Read from replica with a lag bound (e.g. < 1 s) | Aurora replicas, Cosmos DB |
| Read-your-writes | Free if you pin reads to the primary for 1–5 s after a write (sticky routing) | The standard practical fix |
| Monotonic reads | Pin a user to one replica | Session consistency |
| Causal | Track dependencies (vector clocks, ~100 B metadata) | COPS, MongoDB causal sessions |
| Eventual | 0 extra — converges in 1 ms – 1 s in-region, < 1 s cross-region for DynamoDB global tables, minutes worst case | DynamoDB, Cassandra, S3 CRR |

#### Distributed transactions

| Approach | Latency | Throughput cost | Failure behavior |
|---|---|---|---|
| Two-phase commit (2PC) | 2 RTT + 2 durable log writes ≈ 4–10 ms same region | 2–5× slower than a local transaction | Blocking if the coordinator dies — recovery takes seconds to minutes |
| Three-phase commit | 3 RTT | Worse | Non-blocking but rarely used in practice |
| Saga (choreographed or orchestrated) | Sum of steps (100 ms – minutes) | Near-zero overhead | Needs compensating transactions; intermediate states are visible |
| Outbox + CDC | 10 ms – 1 s to propagate | Negligible | At-least-once → consumers must be idempotent |
| Deterministic / Calvin-style | 1 consensus round | High throughput | Requires pre-declared read/write sets |

**The interview line:** *"I'd avoid a distributed transaction. I'll keep the invariant inside one shard, use the outbox pattern for the cross-service event, and make consumers idempotent — a saga with compensation if the business flow truly spans services."*

#### Conflict resolution

| Mechanism | Metadata cost | Note |
|---|---|---|
| Last-write-wins | 8 B timestamp | Silently drops writes under clock skew |
| Vector clocks | ~16 B per writer — grows unboundedly | Requires client-side merge (Dynamo-style siblings) |
| CRDTs | 2–10× payload overhead | Converge without coordination; ideal for collaborative editing and counters |
| Operational transform | Server-coordinated | Google Docs heritage |
| Read repair / anti-entropy | Merkle tree comparison, minutes–hours | How Cassandra self-heals; run repair within `gc_grace_seconds` (10 days) |

---

### 12. When to shard: the thresholds table

> **Tier 1 — learn by heart.** This table is the answer to the most common follow-up question in the format. Know the rows for your stack cold.

A compact answer to "at what point do you split this?" for every storage technology.

| Technology | Unit | Comfortable | Act at | Hard limit |
|---|---|---|---|---|
| PostgreSQL table | rows | < 100M | 100M+ → partition | 32 TB/table |
| PostgreSQL instance | data size | < 1 TB | 1–2 TB → shard | 2–10 TB practical |
| MySQL table | rows | < 50M | 50–100M → partition | ~64 TB/table |
| MySQL instance | data size | < 500 GB | 500 GB–1 TB → shard | ~2 TB practical |
| MySQL/Postgres writes | TPS | < 5k | 10–20k → shard | ~50k with tricks |
| Redis node | memory | < 25 GB | > 50 GB → cluster | 256 GB+ (painful) |
| Cassandra partition | bytes/rows | < 10 MB / 100k rows | > 100 MB → change partition key | 2 GB (fails) |
| Cassandra node | data | 1–2 TB | > 2 TB → add nodes | 4–10 TB (Scylla higher) |
| MongoDB shard | data | < 1–2 TB | > 2–3 TB → add shards | — |
| MongoDB document | bytes | < 100 KB | approaching 16 MB → restructure | 16 MB |
| DynamoDB partition | throughput | < 2,000 RCU | 3,000 RCU / 1,000 WCU → split key | 10 GB per partition |
| DynamoDB item | bytes | < 4 KB (1 RCU) | > 100 KB → offload to S3 | 400 KB |
| Elasticsearch shard | bytes | 10–30 GB | > 50 GB → more shards | 2.1B docs (Lucene) |
| Kafka partition | throughput | < 10 MB/s | > 10–30 MB/s → more partitions | — |
| Kafka broker | partitions | < 2,000 | > 4,000 → more brokers | ~4,000 (ZK), far more with KRaft |
| Kafka message | bytes | < 100 KB | > 1 MB → claim-check pattern | 1 MB default (`message.max.bytes`) |
| etcd | DB size | < 2 GB | > 2 GB → reduce/compact | 8 GB |
| ZooKeeper znode | bytes | < 1 KB | > 100 KB → move to a DB | 1 MB |
| S3 prefix | rps | < 3,000 writes/s | → add prefixes | 3,500 PUT / 5,500 GET per prefix |
| Prometheus server | active series | < 2M | > 5–10M → federate/remote-write | ~10M per server |

---

### 13. NoSQL: DynamoDB, Cassandra, ScyllaDB, MongoDB, HBase

> **Tier 1 — learn by heart.** DynamoDB: **3,000 RCU / 1,000 WCU / 10 GB per partition, 400 KB items.** Cassandra: keep partitions **under 100 MB**.

#### DynamoDB — the limits *are* the design

| Metric | Value |
|---|---|
| Latency (single-item read/write) | p50 2–5 ms, p99 10–20 ms |
| With DAX cache | microseconds–1 ms for reads |
| 1 RCU | 1 strongly consistent read of 4 KB/s (2 eventually consistent reads) |
| 1 WCU | 1 write of 1 KB/s |
| Per-partition ceiling | **3,000 RCU** and **1,000 WCU** |
| Per-partition storage | **10 GB** |
| Max item size | **400 KB** |
| Query/Scan page size | 1 MB |
| BatchGetItem | 100 items / 16 MB |
| BatchWriteItem | 25 items / 16 MB |
| TransactWriteItems | 100 items, 4 MB |
| Global Secondary Indexes | 20 per table (soft), LSIs 5 (hard) |
| Default table throughput quota | 40,000 RCU / 40,000 WCU per table (soft) |
| Adaptive capacity / burst | 300 s of unused capacity bursted |
| Global tables replication lag | < 1 s typical (last-writer-wins) |
| Streams retention | 24 h |

**Design consequence:** a hot partition is the #1 DynamoDB failure. If one key takes > 1,000 writes/s you must add a write-sharding suffix (`pk#0..N`). Quote this — it is the expected answer.

#### Cassandra / ScyllaDB

| Metric | Cassandra | ScyllaDB |
|---|---|---|
| Writes per node | 10k–50k /s | 100k–1M /s |
| Reads per node | 5k–30k /s | 100k–500k /s |
| Write latency p99 | 5–15 ms | < 1–5 ms |
| Read latency p99 | 10–50 ms | 1–10 ms |
| Data per node | 1–2 TB | 4–30 TB (shard-per-core, no JVM GC) |
| Partition size target | **< 10 MB, hard warn 100 MB** | same |
| Rows per partition | < 100,000 | same |
| Tombstone warn / fail thresholds | 1,000 / 100,000 per query | same |
| Compaction space overhead | up to 50% free disk (STCS); LCS ~10% | similar |
| Typical topology | RF=3, `LOCAL_QUORUM` | same |
| Cluster size in production | 10s–1,000+ nodes | 3–100s |
| Repair cadence | Every `gc_grace_seconds` (default 10 days) | same |

**The Cassandra interview answer:** the data model is the query. Writes are cheap (LSM-tree, append-only ≈ memtable write ~µs), reads across partitions are expensive, deletes create tombstones that make reads *slower*, and a partition key with unbounded growth (e.g. `user_id` for an event log) is the classic mistake — bucket it by time (`user_id + yyyymm`).

#### MongoDB

| Metric | Value |
|---|---|
| Max document size | 16 MB (nesting depth 100) |
| Read/write throughput per node | 10k–50k ops/s |
| Latency (indexed query) | 0.5–5 ms |
| WiredTiger cache | 50% of RAM − 1 GB |
| Shard chunk size | 128 MB default |
| Shard when | data per shard > 2–3 TB, or working set > RAM |
| Replica set members | 50 max (7 voting) |
| Typical replication lag | 10 ms – 1 s |
| Index limit | 64 per collection |
| Aggregation pipeline stage memory | 100 MB (spills to disk when allowed) |
| Change streams lag | < 100 ms – 1 s |

#### HBase / Bigtable

| Metric | Value |
|---|---|
| Latency (point read) | 1–10 ms (Bigtable p50 ~2–6 ms) |
| Throughput per node | 10k QPS per Bigtable node (rule of thumb: 10,000 reads or writes/s per node at 1 KB rows) |
| Region/tablet size | 5–10 GB (HBase); Bigtable tablets split automatically |
| Row size guidance | < 10 MB per row, < 100 MB per row hard |
| Scaling | Linear with nodes — quote "add a node, get 10k QPS" |

#### Choosing between them — the number-driven answer

| If you need… | Choose | Because |
|---|---|---|
| Transactions, joins, ad-hoc queries, < 1–2 TB | PostgreSQL/MySQL | 10k–20k TPS is plenty for most products |
| Predictable single-digit-ms KV at any scale, managed | DynamoDB | Partition-level limits are explicit |
| Huge write volume, multi-region active-active | Cassandra/Scylla | 100k+ writes/s per cluster, tunable consistency |
| Flexible documents, moderate scale | MongoDB | 16 MB docs, easy sharding |
| Massive sorted scans, time series over rows | HBase/Bigtable | Range scans are native |

---

### 14. Search engines: Elasticsearch and OpenSearch

> **Tier 2 — know the rough value.** Shard size **10–50 GB**, heap **≤ 31 GB**, and oversharding costs more than it saves.

| Metric | Value |
|---|---|
| **Shard size sweet spot** | **10–50 GB** — 20–25 GB for search workloads, 30–50 GB for logs |
| Absolute max docs per shard | 2,147,483,519 (2³¹ − 1) — a Lucene limit |
| Shards per node | ≤ 20 shards per GB of heap → ~600 shards on a 30 GB heap; aim much lower |
| JVM heap | **50% of RAM, never above 31 GB** (compressed object pointers) |
| RAM per node | 32–64 GB typical (half heap, half OS page cache for Lucene) |
| Indexing throughput per node | 10k–50k docs/s (small docs, bulk) |
| Bulk request size | 5–15 MB per request |
| Query latency (simple term/match, warm) | 5–50 ms |
| Query latency (aggregations over 100M docs) | 100 ms – 2 s |
| `refresh_interval` (near-real-time visibility) | 1 s default; raise to 30 s for ingest-heavy workloads (2–5× indexing gain) |
| Replica count | 1 (i.e. 2 copies) typical |
| Index size vs raw data | 1.1–2× raw JSON (or 0.3–0.6× with `_source` disabled / best_compression) |
| Cluster size | 3 dedicated masters + N data nodes; 10s–100s of nodes |
| Deep pagination limit | `from + size ≤ 10,000` → use `search_after` / PIT |
| Scroll / PIT context | Keep alive 1–5 min |
| Recovery/rebalance of a 50 GB shard | Minutes (network-bound) — the reason to cap shard size |

**Shard count formula to state in an interview:**
```
primary_shards ≈ ceil(expected_index_size_GB / 30 GB)
total_shards   = primary_shards × (1 + replicas)
nodes          ≈ total_shards / shards_per_node_budget
```
For 3 TB of logs: 3,000 / 30 = **100 primaries**, ×2 with replicas = 200 shards, over nodes holding ~50 shards each ⇒ **~4–8 data nodes** (disk will probably decide it first).

**Oversharding is the classic mistake.** Each shard is a full Lucene index with its own heap, file handles and merge threads. 1,000 tiny shards cost more than 50 healthy ones. Use ILM/rollover on time-based indices at a fixed size (e.g. roll at **50 GB** or **1 day**).

#### Other search options

| System | Figure of merit |
|---|---|
| Apache Solr | Comparable to ES; shard 10–50 GB; strong for faceting |
| Typesense / Meilisearch | Sub-50 ms search, in-memory, best under ~10–50M docs |
| Algolia (hosted) | 1–20 ms search latency, scales by managed replicas |
| PostgreSQL full-text (`tsvector` + GIN) | Fine up to 1–10M docs; GIN index build is slow; no relevance tuning depth |
| OpenSearch k-NN / ES dense_vector | Adds vector search — see the vector-database section |

---

## Part 4 — Data in motion

*Sections 15–20 · ~18 min*

Nothing stays in one database. This part covers the pipes: message brokers and event streams, the processors that read them, the workflow engines that coordinate long-running work, object storage and CDNs for bulk bytes, analytical stores for the queries your OLTP database must never run, and the observability stack that tells you any of it is working.

The recurring theme: **throughput is easy, ordering and retention are what cost you.**

---

### 15. Streaming and messaging

> **Tier 1 — learn by heart.** **≤ 4,000 partitions per broker, ~10 MB/s per partition, RF 3, 1 MB messages.** Partitions are your consumer parallelism.

#### Apache Kafka — the numbers interviewers probe

| Metric | Value |
|---|---|
| Throughput per broker | 100 MB/s – 1 GB/s (commodity: ~100–300 MB/s write; NVMe + 25 GbE far higher) |
| Messages per second per broker | 100k–1M+ (1 KB messages, batched) |
| Throughput per partition (planning number) | **~10 MB/s write, 10–30 MB/s read** |
| End-to-end latency, `acks=all`, `linger.ms=0` | 5–20 ms p99 (tuned clusters < 10 ms) |
| Producer batch/linger tradeoff | `linger.ms=5–100` buys 2–10× throughput for that much latency |
| **Partitions per broker** | **≤ 4,000** (ZooKeeper era); **1,000–2,000 is the comfortable planning number** |
| Partitions per cluster | ≤ 200,000 with ZooKeeper; millions with KRaft (tested to ~2M) |
| Topics per cluster | Practically bounded by partitions: 10k–100k topics is achievable; each topic ≥ 1 partition × RF files on disk |
| Partition count formula | `max(target_in_throughput / per_partition_write, target_out_throughput / per_consumer_rate)` |
| Consumer parallelism | 1 consumer per partition per group — partitions are the parallelism cap |
| Replication factor | 3 (with `min.insync.replicas=2`) |
| Max message size | **1 MB** default (`message.max.bytes` = 1,048,588) — keep < 100 KB, use claim-check in S3 beyond |
| Retention | 7 days default; sizing = `throughput × retention × RF` |
| Disk for 100 MB/s × 7 days × RF3 | 100 MB/s × 604,800 s × 3 ≈ 180 TB |
| Consumer rebalance time | seconds to minutes (cooperative/incremental rebalancing reduces it dramatically) |
| Broker restart / recovery | 1–10 min with large partition counts (log recovery) |
| Typical cluster size | 3–30 brokers; LinkedIn-scale: 100s of brokers, 7 trillion msgs/day |
| Zero-copy `sendfile` path | Why Kafka reads are near disk speed |
| Exactly-once (transactions) overhead | 10–30% throughput cost, +few ms latency |

> **The Kafka sizing sentence to memorize:** *"I'll size partitions as max(in-throughput / 10 MB/s, out-throughput / consumer-rate), keep it under ~2,000 per broker with RF 3, and size disk as throughput × retention × 3."*

#### The rest of the messaging landscape

| System | Throughput | Latency | Key limits |
|---|---|---|---|
| Apache Pulsar | 100k–1M+ msgs/s per broker | p99 5–20 ms | Millions of topics (tiered by BookKeeper); compute/storage separated; geo-replication built in |
| Redpanda | 1–3× Kafka per core (no JVM, thread-per-core) | p99 < 10 ms | Kafka API compatible; fewer GC-induced tail spikes |
| RabbitMQ (classic queue) | 20k–50k msgs/s per queue, 100k+ per node across queues | < 1–5 ms | Single queue is a single Erlang process → one queue is the bottleneck |
| RabbitMQ (quorum queue) | 5k–20k msgs/s per queue | 5–20 ms | Raft-replicated; keep queues short (< 10k messages) or memory alarms fire at 40% RAM |
| Amazon SQS (standard) | Effectively unlimited (scales automatically) | 10–100 ms | 256 KB message (2 GB via extended client), retention 4 d default / 14 d max, visibility timeout 30 s default / 12 h max, at-least-once |
| Amazon SQS (FIFO) | 300 msg/s, 3,000/s with batching of 10; high-throughput mode up to 9,000+/s per API action | 10–100 ms | Exactly-once processing within a 5-min dedup window |
| Amazon Kinesis Data Streams | Per shard: 1 MB/s or 1,000 records/s in; 2 MB/s out (5 `GetRecords`/s) | 70 ms – 200 ms (enhanced fan-out ~70 ms) | Record 1 MB max, retention 24 h default → 365 days max, 20 consumers with EFO |
| Google Pub/Sub | Unlimited (auto-scaling) | 100 ms p50 publish→deliver | 10 MB message, 7-day retention, at-least-once (exactly-once opt-in) |
| Azure Event Hubs | 1 TU = 1 MB/s in, 2 MB/s out, 1,000 events/s | 10–100 ms | 32 partitions per hub (standard), 1 MB event |
| NATS Core | 1M–10M msgs/s per server | < 1 ms (µs in-memory) | Fire-and-forget; JetStream adds persistence (100k–1M msgs/s) |
| ActiveMQ / Artemis | 10k–100k msgs/s | 1–10 ms | JMS semantics |
| AWS EventBridge | 10,000 events/s default (raisable) | 0.5–2 s typical | 256 KB events, rule-based routing |
| AWS SNS | Effectively unlimited fan-out | 10–100 ms | 256 KB message, 12.5M subscriptions/topic |
| MQTT (EMQX/Mosquitto) | 100k–1M+ connections per node | < 10 ms | IoT: QoS 0/1/2, small payloads |

**How to choose, with numbers:**
- Need **replay, ordering per key, multiple independent consumers, > 100k msgs/s** → **Kafka/Pulsar/Redpanda**.
- Need **per-message routing, priorities, delays, complex topologies, < 50k msgs/s** → **RabbitMQ**.
- Need **zero operations** and traffic is spiky → **SQS/Pub/Sub** (and accept 10–100 ms latency and at-least-once).
- Need **µs latency, no durability** → **NATS core / Redis Pub-Sub**.

**Delivery semantics cost:** at-most-once is free; at-least-once costs you idempotency handling at the consumer; exactly-once costs **10–30% throughput** and only works within the system's transactional boundary. Say this — interviewers love it.

#### How many topics, queues and streams can each system hold?

A frequently asked, rarely answered question — the namespace limit is often what forces a redesign (e.g. "a topic per user" almost never works).

| System | Topic / queue / stream capacity | What actually constrains it |
|---|---|---|
| Kafka (ZooKeeper) | ~10k–50k topics, ≤ 200,000 partitions per cluster | Partitions, not topics: metadata in ZK, controller failover time, open file handles (3+ files per partition) |
| Kafka (KRaft) | Millions of partitions tested (~2M); 100k+ topics | Controller metadata log; per-partition memory on brokers |
| Apache Pulsar | Millions of topics per cluster; ~10k–50k per broker | Topics are cheap (BookKeeper ledgers); designed for topic-per-user patterns |
| RabbitMQ | 10k–100k queues per node | Each queue is an Erlang process (~10–50 KB idle); a million queues needs sharding across nodes |
| NATS / NATS JetStream | Millions of subjects (core NATS keeps no per-subject state); JetStream streams in the thousands | Subject wildcards make "subject per user" natural |
| Amazon SQS | 1M queues per account (soft); throughput per queue unlimited (standard) | Account quota, not performance |
| Amazon SNS | 100,000 topics per account; 12.5M subscriptions per topic | Account quota |
| Amazon Kinesis | 500 shards per stream default (raisable to 10k+); 50–500 streams per account | Shard quota per region |
| Google Pub/Sub | 10,000 topics per project, 10,000 subscriptions per topic | Project quota |
| Azure Event Hubs | 10 hubs per namespace (Standard), 32 partitions per hub | Namespace tier |
| Redis Pub/Sub / Streams | Millions of channels (no persistent state per channel) | Memory for Streams; channels are free |

**Design rule:** if your design wants **one topic per user or per entity**, only Pulsar, NATS and SQS are comfortable there. On Kafka you use **one topic partitioned by key** and let the key provide per-entity ordering — that is almost always the expected answer.

---

### 16. Stream processing: Flink, Spark, Kafka Streams

> **Tier 2 — know the rough value.** Flink is **10–100 ms**; your checkpoint interval is also your worst-case replay window.

| Framework | Throughput | Latency | State | Notes |
|---|---|---|---|---|
| Apache Flink | 1M+ events/s per cluster, 100k–500k/s per task slot | 10–100 ms (true streaming) | RocksDB backend, GBs–TBs of state | Checkpoint interval 1 s – 5 min; exactly-once via barriers; the default choice for stateful streaming |
| Kafka Streams | 100k–1M events/s | 10–100 ms | RocksDB local state, sized per partition | No separate cluster — it is a library; scales by adding instances (capped by partitions) |
| Spark Structured Streaming | Very high throughput (batch-based) | 100 ms – seconds (micro-batch); continuous mode lower | Checkpoint to HDFS/S3 | Best when you already run Spark; latency floor is the batch interval |
| Apache Beam | Engine-dependent | Engine-dependent | — | Portability layer over Flink/Dataflow |
| Materialize / RisingWave | 100k+ updates/s | < 100 ms incremental view maintenance | — | SQL-native streaming views |

**Numbers that matter in design answers:**
- **Watermarks / allowed lateness:** typical 1 s – 1 min; anything later goes to a side output.
- **Checkpoint interval vs recovery time:** a 1-minute interval means up to ~1 minute of reprocessing on failure; checkpointing 100 GB of state to S3 takes minutes — use incremental checkpoints.
- **Window sizes:** tumbling 1 min – 1 h for analytics; session gaps 5–30 min.
- **Backpressure:** if consumer lag grows monotonically, you are under-provisioned; Kafka consumer lag is the canonical alert (**alert at > 1–5 min of lag**).
- **Shuffle in Spark:** target **128 MB per partition**; more than ~200 partitions per core creates scheduling overhead.

---

### 17. Workflow orchestration and background jobs

> **Tier 3 — know that a limit exists.** **Workers = arrival rate × job duration.** Airflow schedules in minutes, Temporal in milliseconds.

Almost every design ends up with "and then we process it asynchronously." These are the engines and their limits.

| System | Throughput | Scheduling latency | Hard limits |
|---|---|---|---|
| Celery (Python) | 1k–10k tasks/s per cluster; ~100–1,000/s per worker | 10–100 ms via Redis/RabbitMQ | Task payload should stay < 100 KB — pass IDs, not blobs |
| Sidekiq (Ruby) | 1k–10k jobs/s | 10–50 ms | Redis-backed; 25 threads/process typical |
| BullMQ / RQ / Resque | 1k–20k jobs/s | 10–50 ms | Redis sorted sets for delayed jobs |
| Apache Airflow | 1k–10k DAGs, 10k+ tasks/day | 1–30 s per task (scheduler loop) | Batch tool — not for sub-second work; realistic min schedule ~1–5 min; default parallelism 32 |
| Temporal / Cadence | 1k–100k workflow starts/s (cluster) | 10–100 ms | Workflow history capped at 51,200 events / 50 MB → `continue-as-new`; activity/task queue ~10k/s per partition |
| AWS Step Functions (Standard) | 2,000 state transitions/s per account (soft) | 50–500 ms | 25,000 events per execution, 1-year max duration |
| AWS Step Functions (Express) | 100,000 executions/s | ~10 ms | 5 minute max duration, at-least-once |
| Kubernetes CronJob / cron | — | 1-minute granularity, ±seconds of skew | No exactly-once guarantee — jobs must be idempotent |
| Quartz / db-backed schedulers | 100–10k jobs/s | 100 ms–1 s | Lock contention on the jobs table is the usual bottleneck |

**Worker-pool sizing (Little's Law again):**
```
workers = arrival_rate × job_duration
```
1,000 jobs/s × 2 s each = **2,000 concurrent workers**; at 4 jobs per core that is ~500 cores. If jobs take 30 s, the same rate needs 30,000 concurrent workers — which is when you redesign, not scale.

**Queue health numbers:** alert when queue depth exceeds **1–5 minutes of drain time** (`depth / drain_rate`), not on an absolute count. Retries: **3–5 attempts** with exponential backoff, then a **dead-letter queue**; DLQ growth is the canary. Delayed jobs at scale: a Redis sorted set keyed by run-at timestamp handles **100k+ scheduled jobs/s**; beyond a few million pending, use a time-bucketed table.

---

### 18. Object storage and CDN

> **Tier 2 — know the rough value.** **3,500 writes / 5,500 reads per second per prefix**, and 100–200 ms to first byte. Bulk bytes, never a hot path.

#### S3 / GCS / Azure Blob

| Metric | Value |
|---|---|
| **Request rate per prefix (S3)** | **3,500 PUT/COPY/POST/DELETE per second** and **5,500 GET/HEAD per second** — unlimited prefixes, so parallelize across prefixes |
| First-byte latency (S3 Standard) | **100–200 ms** |
| First-byte latency (S3 Express One Zone) | < 10 ms |
| Throughput per connection | 50–100 MB/s |
| Throughput with parallelism | Up to 100 Gbps per instance with multipart + many connections |
| Max object size | 5 TB |
| Multipart part size | 5 MB – 5 GB, max 10,000 parts |
| Single-PUT max | 5 GB |
| Durability | **99.999999999% (11 nines)** |
| Availability SLA | 99.9% (Standard); 99.99% design |
| Consistency | Strong read-after-write (since Dec 2020) |
| Listing | 1,000 keys per `ListObjectsV2` page |
| Glacier restore | Expedited 1–5 min, Standard 3–5 h, Bulk 5–12 h |
| Versioning/lifecycle transitions | Per-object, minimum storage durations (30 d IA, 90 d Glacier) |

**Interview consequence:** object storage is for **bulk, immutable, large objects** — never for a hot key-value path (100 ms p50 vs Redis 0.3 ms). The standard pattern is *metadata in a DB, bytes in S3, access via presigned URLs served through a CDN*.

#### CDN

| Metric | Value |
|---|---|
| Edge RTT to user | 10–50 ms (vs 50–150 ms to origin) |
| Cache hit ratio target (static assets) | 90–99% |
| Cache hit ratio target (dynamic/API) | 50–80% with careful keys |
| Origin offload at 95% hit rate | 20× reduction in origin traffic |
| Number of PoPs (major CDNs) | 200–400+ |
| Purge propagation | seconds to minutes globally |
| TLS termination at edge | Saves origin ~1–2 RTT + handshake CPU |
| Typical asset TTL | 1 year (immutable, hashed filenames) |
| Cost | $0.02–0.09/GB egress (usually cheaper than origin egress) |

#### File sync, chunking, dedup and erasure coding

The Dropbox/Google Drive family of questions lives on these numbers.

| Metric | Value |
|---|---|
| Fixed chunk size (Dropbox-style) | 4 MB |
| Content-defined chunking (rolling hash) | Average 1–8 MB, min 256 KB, max 16 MB — survives byte insertions |
| Chunk fingerprint | SHA-256 (32 B) per chunk → ~8 B of index per MB of data |
| Cross-user dedup ratio | 2–10× for typical corpora (much higher for shared corporate files) |
| Delta sync saving | Uploads only changed chunks — often 1–5% of file size |
| Metadata per file | 1–2 KB (name, version, chunk list, ACL) → 1B files ≈ 1–2 TB of metadata |
| Sync conflict window | Detect via version vector; resolve LWW or keep both ("conflicted copy") |
| Replication (3×) | 200% storage overhead, fastest recovery |
| Erasure coding RS(6,3) | 50% overhead, tolerates 3 losses, reconstruct = read 6 chunks |
| Erasure coding RS(10,4) | 40% overhead, tolerates 4 losses — typical for cloud object stores |
| EC reconstruct cost | Network-heavy: rebuilding 1 TB reads several TB — why EC suits cold data, replication suits hot data |
| HDFS block size | 128 MB (small files are the classic HDFS killer) |
| S3 optimal object size | > 1 MB; millions of tiny objects waste request budget (see the per-prefix limits above) |

---

### 19. OLAP and warehouses

> **Tier 2 — know the rough value.** Columnar gives you **5–10× compression × 10–100× column pruning**, which is why a 1 TB scan becomes a 10 GB scan.

| System | Scan rate | Query latency | Key numbers |
|---|---|---|---|
| ClickHouse | 100M–2B rows/s per server (multi-core, columnar, vectorized) | 10 ms – 1 s on billions of rows | Compression 5–10×; insert in batches of 10k–100k rows, ideally ≤ 1 insert/s per partition (too many small parts = merge storm); 1 node handles TBs |
| Apache Druid | 100M+ rows/s | sub-second | Real-time + historical; segment target 300–700 MB / ~5M rows; ingest 100k–1M events/s |
| Apache Pinot | 100M+ rows/s | < 100 ms p99 | Built for user-facing analytics (LinkedIn); upserts supported |
| Apache Doris / StarRocks | 100M+ rows/s | sub-second | MPP, MySQL protocol |
| BigQuery | TB/s aggregate (thousands of slots) | 1–30 s | Serverless; $5–6.25 per TB scanned; partition + cluster to cut scan cost 10–100× |
| Snowflake | Scales with warehouse size (XS→4XL = 1→128 nodes) | 1–60 s | Credits per hour per size; auto-suspend at 60 s idle |
| Redshift | 10–100M rows/s per node | seconds | Sort/dist keys matter enormously |
| Presto / Trino | Depends on source | seconds–minutes | Federated query; 10s–100s of workers |
| Apache Spark (batch) | GB/s per executor | minutes | 128 MB partitions, 2–5× cores in tasks |
| DuckDB (single node) | 100M–1B rows/s | ms–s | Embedded OLAP; up to ~100s of GB on a laptop-class machine |

**Columnar rules of thumb:** columnar formats (Parquet/ORC) compress **5–10×** vs raw JSON, and column pruning + predicate pushdown often reads **1–10%** of the data. That combination is why a query over 1 TB of JSON becomes a query over 10 GB of Parquet — a **100× difference** worth stating explicitly.

**OLTP vs OLAP separation:** you run analytics on the primary until roughly **100M rows / a few hundred GB**; beyond that, an analytical query that scans 10M rows will hold locks/IO and hurt your 1 ms transactional p99. Ship to a warehouse via CDC (10 ms–1 s lag) or batch ETL (5 min–24 h).

#### Lakehouse table formats

| Format | Figure of merit |
|---|---|
| Parquet | Row group 128 MB, page 1 MB; compresses 5–10×; column pruning often reads 1–10% of bytes |
| Apache Iceberg | Target file size 128 MB – 1 GB; snapshot-based time travel; metadata scales to millions of files; compaction needed when small files accumulate |
| Delta Lake | Same file-size targets; `OPTIMIZE` + Z-order clustering; transaction log checkpoints every 10 commits |
| Apache Hudi | Copy-on-write (read-optimized) vs merge-on-read (write-optimized, 5–10× faster upserts) |
| The small-file problem | 100k files of 1 MB scan 10–100× slower than 1k files of 100 MB — schedule daily compaction |
| Partitioning granularity | Aim for ≥ 1 GB per partition; date-partitioning at hourly granularity on low volume is a common mistake |

---

### 20. Time series and observability

> **Tier 3 — know that a limit exists.** **Cardinality is the killer.** 1M active series ≈ 4–8 GB of RAM; bound every label.

| System | Ingest | Cardinality limit | Storage per sample |
|---|---|---|---|
| Prometheus (single server) | 100k–1M samples/s | 1M–10M active series (practical: keep under 2–5M) | 1.3–2 bytes compressed (Gorilla/XOR + delta) |
| VictoriaMetrics | 1M–10M samples/s per node | 10M–100M+ series | ~0.4–1.2 bytes |
| Thanos / Cortex / Mimir | Horizontally scaled | 100M+ series | Object storage backed, unlimited retention |
| InfluxDB | 500k–1M points/s per node | Cardinality-sensitive (the classic failure) | 2–3 bytes |
| TimescaleDB | 100k–1M rows/s | Postgres semantics | Compressed hypertables 10–20× |
| Datadog/New Relic (SaaS) | Unlimited | Billed per custom metric | $$ per host/metric |

**Prometheus sizing math to quote:**
```
memory ≈ active_series × 4–8 KB          (≈ 4–8 GB RAM per 1M series)
disk   ≈ active_series × samples_per_s × bytes_per_sample × retention
       = 1M × (1/15s) × 1.5 B × 15 d ≈ 130 GB
```
- Default scrape interval **15 s**, retention **15 days**.
- **Cardinality is the killer:** a label with user IDs on a metric with 1M users is 1M series *per metric*. Bound every label; keep cardinality per metric under ~10k.
- **Logs:** raw application logs run **1–10 KB per event**; at 10k events/s that is 10–100 MB/s = **1–8 TB/day**. Sample aggressively, and keep hot retention to 7–30 days.
- **Traces:** one span ≈ 300 B–1 KB; sample at **0.1–10%** for high-traffic services (tail-based sampling to keep the errors).
- **Metrics vs logs vs traces cost ratio:** metrics are ~100–1,000× cheaper per unit of insight than raw logs. Design accordingly.

---

## Part 5 — Specialized workloads

*Sections 21–25 · ~14 min*

Five domains that show up when the question is not a generic CRUD app: vector search and RAG, model and LLM serving, graph traversal, geospatial lookup, and real-time media. You will not get all five in one interview — but you will get one, and generic answers do not survive there. Each has a handful of numbers that immediately mark you as someone who has built it.

---

### 21. Vector databases and ANN search

> **Tier 2 — know the rough value.** 768 dims fp32 = **3 KB per vector**; 1M vectors ≈ 4–5 GB in RAM and **1–10 ms** per query.

| Metric | Value |
|---|---|
| Embedding dimensions | 384 (MiniLM), 768 (BERT-base), 1,536 (OpenAI ada-002/3-small), 3,072 (3-large) |
| Memory per vector (fp32) | `dims × 4 B` → 1.5 KB (384d), 3 KB (768d), 6 KB (1536d) |
| HNSW graph overhead | `M × 8–16 B` per vector → +0.2–1 KB (M = 16–64) |
| RAM for 1M × 768d vectors (HNSW, fp32) | ~4–5 GB |
| RAM for 100M × 768d vectors | ~400–500 GB → shard across nodes |
| Scalar quantization (int8) | 4× smaller, ~1–2% recall loss |
| Product quantization | 8–32× smaller, 5–15% recall loss (rerank to recover) |
| Binary quantization | 32× smaller, needs reranking |
| Query latency (1M vectors, HNSW, top-10) | 1–10 ms |
| Query latency (100M vectors, sharded) | 10–100 ms |
| QPS per node | 1,000–10,000 at recall ≈ 0.95 (single-threaded per query, scales with cores) |
| HNSW build parameters | `M = 16–64`, `efConstruction = 100–500` |
| HNSW search parameter | `efSearch = 50–400` (higher = better recall, linearly more latency) |
| IVF parameters | `nlist ≈ sqrt(N)` (e.g. 4,000 for 16M), `nprobe = 8–64` |
| Index build time, 1M vectors | 1–10 min (HNSW, multi-threaded) |
| Recall target | 0.90–0.99 — below 0.9 users notice |
| Brute force (exact) feasible up to | ~100k–1M vectors (linear scan: 1M × 768d ≈ 3 GB of dot products ≈ 100 ms–1 s) |

| Engine | Figure of merit |
|---|---|
| FAISS (library) | Fastest raw; 1M–10M vectors per node in RAM; GPU support |
| pgvector | Good to 1–10M vectors; HNSW index; keeps data next to your relational data |
| Qdrant / Weaviate / Milvus | 10M–1B+ vectors, distributed, filtering + hybrid search |
| Pinecone (hosted) | p95 < 100 ms; pods sized by vectors × dims |
| Elasticsearch/OpenSearch k-NN | Convenient if you already run it; slower than specialists at scale |

**RAG sizing sentence:** *"1M documents × 3 chunks each = 3M vectors at 1,536 dims fp32 ≈ 18 GB of raw vectors plus graph — one 64 GB node holds it in RAM, and I'd quantize to int8 before I need two."*

---

### 22. ML and LLM serving

> **Tier 2 — know the rough value.** TTFT **200 ms–2 s**, generation **20–100 tokens/s per stream**, KV cache **0.1–0.5 MB per token**.

Modern interviews increasingly include a model in the request path. These numbers keep that realistic.

#### Classical ML serving

| Stage | Latency | Throughput |
|---|---|---|
| Logistic regression / linear model | 10–100 µs | 100k+ QPS per core |
| Gradient-boosted trees (XGBoost/LightGBM, 500 trees) | 0.1–1 ms | 5k–50k QPS per core |
| Small neural net on CPU | 1–10 ms | 500–5,000 QPS per core |
| Deep model on GPU (batch 1) | 1–10 ms | Batching 8–64 gives 5–20× throughput for +10–50 ms latency |
| Online feature fetch (the usual bottleneck) | 1–10 ms p99 (Redis/DynamoDB/Feast) | Often 50–80% of total inference latency |
| Two-stage recommender | Candidate gen (ANN) 5–20 ms → ranking of 100–1,000 candidates 10–50 ms | Total budget 50–150 ms |

#### Embeddings and LLMs

| Metric | Value |
|---|---|
| Sentence-embedding throughput (MiniLM-class, GPU) | 1,000–10,000 texts/s |
| Same on CPU | 50–300 texts/s |
| Embedding API latency | 20–100 ms per batch |
| LLM time-to-first-token (TTFT) | 200 ms – 2 s (grows with prompt length; prefill is compute-bound) |
| LLM generation speed | 20–100 tokens/s per stream; 500–5,000 tokens/s aggregate per GPU with continuous batching |
| Tokens per word (English) | ~1.3 tokens/word; 1 token ≈ 4 characters |
| Model weights memory (fp16) | 2 bytes × parameters → 7B ≈ 14 GB, 70B ≈ 140 GB (2× 80 GB GPUs) |
| Quantization (int8 / int4) | 2×/4× smaller, 1–3× faster, small quality loss |
| KV cache per token | ~0.1–0.5 MB for 7B–70B models with GQA → a 32k-token context is 3–16 GB per concurrent request |
| Context windows (current generation) | 128k–1M tokens |
| GPU cost | $1–4/hour (A10/L4) to $2–10/hour (A100/H100) on demand |
| Hosted LLM API pricing | ~$0.25–15 per million tokens depending on model tier |
| Guardrail / moderation pass | +50–200 ms |
| RAG end-to-end budget | embed query 10–50 ms + vector search 10–50 ms + rerank 20–100 ms + generation 1–3 s |

**The sizing sentence:** *"At 100 requests/second with 500 output tokens each, that's 50,000 tokens/s. One H100 with continuous batching does roughly 2,000–5,000 tokens/s for a 70B model, so I need ~10–25 GPUs — or a smaller/quantized model, aggressive caching of common prompts, and streaming so TTFT is what the user feels."*

**Cache the model layer too:** semantic caching of prompts (embedding similarity > 0.95) commonly deflects **20–40%** of LLM traffic, and prompt-prefix caching cuts prefill cost by 50–90% for long system prompts.

---

### 23. Graph databases

> **Tier 3 — know that a limit exists.** Two hops is milliseconds; **four-plus hops is a precompute problem**, not a query problem.

| Metric | Value |
|---|---|
| Neo4j traversal rate | 1M+ relationship hops/s per core (index-free adjacency) |
| Single node capacity | 10s of billions of nodes/relationships (disk bound); practical working set in RAM |
| 2-hop neighborhood query | 1–10 ms |
| 4+ hop query on a dense graph | 100 ms – seconds (combinatorial explosion: 1,000 friends² = 1M paths) |
| Write throughput | 10k–50k tx/s |
| When a relational DB is fine | ≤ 2–3 joins deep; beyond that, graph wins by orders of magnitude |
| Alternatives | JanusGraph/Neptune (distributed, 10s of thousands QPS), TigerGraph (parallel, deep-link analytics), or adjacency lists in Cassandra/Redis for simple social graphs |

**The social-graph interview answer:** a friends-of-friends query on 1,000-friend users touches ~1M edges. At scale you do not traverse live — you precompute and cache, exactly as feed systems do.

---

### 24. Geospatial indexing

> **Tier 3 — know that a limit exists.** **H3 resolution 8 ≈ 0.74 km²**, so a 5 km radius is ~100 cells: a pipelined cache lookup, not a database scan.

Ride-hailing, delivery, "find nearby", geofencing — all of these need one of three encodings.

#### Geohash precision

| Length | Cell size (approx) | Use |
|---|---|---|
| 4 | 39 × 20 km | City |
| 5 | 4.9 × 4.9 km | District |
| 6 | 1.2 × 0.6 km | Neighborhood — typical "nearby" bucket |
| 7 | 153 × 153 m | Block |
| 8 | 38 × 19 m | Building |
| 9 | 4.8 × 4.8 m | Doorway |
| 12 | 3.7 × 1.9 cm | Surveying |

Geohash is a string prefix, so **prefix match = bounding box** — it works directly as a key in Redis, DynamoDB or any B-tree index. Its weakness: cells are rectangular and **neighbors can differ entirely in prefix at boundaries**, so you always query the cell plus its 8 neighbors.

#### S2 (Google) and H3 (Uber)

| S2 level | Average cell area |
|---|---|
| 8 | ~1,300 km² |
| 10 | ~81 km² |
| 12 | ~5 km² |
| 13 | ~1.3 km² |
| 16 | ~20,000 m² |
| 20 | ~80 m² |
| 30 | ~1 cm² |

| H3 resolution | Average hexagon area | Edge length |
|---|---|---|
| 5 | 252 km² | 8.5 km |
| 6 | 36 km² | 3.2 km |
| 7 | 5.2 km² | 1.2 km |
| 8 | 0.74 km² | 461 m |
| 9 | 0.105 km² | 174 m |
| 10 | 0.015 km² | 66 m |
| 12 | 307 m² | 9 m |

Resolutions 7 to 9 are the ones you will actually use for "nearby" queries; resolution 8 is the usual default.

**H3's advantage:** hexagons have **6 equidistant neighbors** (squares have 4 near + 4 diagonal at 1.41× the distance), which makes ring queries and smoothing uniform. `k-ring(1) = 7 cells`, `k-ring(2) = 19`, `k-ring(k) = 3k² + 3k + 1`.

**Worked query:** "drivers within 5 km" at H3 resolution 8 covers `π × 5² / 0.74 ≈ 106 cells`. With driver positions in Redis as one set per cell, that is ~106 pipelined lookups ≈ **2–10 ms**. Drop to resolution 7 and it is ~15 cells but coarser filtering.

#### Engine numbers

| Engine | Figure of merit |
|---|---|
| Redis GEO (sorted set of 52-bit geohashes) | `GEOSEARCH` 0.5–5 ms for thousands of results; 100k+ ops/s; ~100 B per member |
| PostGIS (GiST index) | `ST_DWithin` 1–20 ms over millions of points; 10M+ rows per node fine |
| Elasticsearch geo_point | 10–50 ms, combines with text filters |
| MongoDB 2dsphere | 1–10 ms |
| Quadtree (in memory) | O(log n) lookup, ~50–100 B per point; rebalancing on dense cities is the cost |
| DynamoDB + geohash prefix as PK | Single-digit ms, but you manage the neighbor fan-out yourself |

---

### 25. Real-time media, live streaming and notifications

> **Tier 3 — know that a limit exists.** **SFU, not MCU** (0.1–0.5 vs 1–2 vCPU per participant), and 150 ms mouth-to-ear is the whole budget.

#### WebRTC and video conferencing

| Metric | Value |
|---|---|
| Peer-to-peer glass-to-glass latency | 50–200 ms |
| Via SFU (selective forwarding — forwards streams, no transcode) | +20–50 ms, ~0.1–0.5 vCPU per participant |
| Via MCU (mixes streams, transcodes) | +100–300 ms, 1–2 vCPU per participant — 10× the cost of an SFU |
| SFU capacity per 16-core node | 500–2,000 concurrent streams |
| Bandwidth per HD stream | 1–2 Mbps → 1,000 streams ≈ 1–2 Gbps per node |
| Audio (Opus) | 24–64 kbps, 20 ms frames |
| Video bitrates | 360p 0.4 Mbps · 720p 1–2 Mbps · 1080p 2–4 Mbps · 4K 15–25 Mbps |
| Jitter buffer | 20–200 ms adaptive |
| Acceptable packet loss | < 1% good, 1–3% degraded, > 5% unusable (FEC/NACK/RED mitigate) |
| Mouth-to-ear target (ITU) | < 150 ms one way for natural conversation; > 400 ms is unusable |
| NAT traversal | STUN works for ~80–85% of peers; TURN relays the remaining 15–20% — budget relay bandwidth for 1 in 5 users |
| Simulcast | 3 layers (e.g. 180p/360p/720p) ≈ 1.5–2× uplink, lets the SFU adapt per receiver |

#### Live and on-demand streaming

| Protocol | Latency | Note |
|---|---|---|
| HLS / DASH (standard) | 6–30 s (segments of 2–10 s × ~3 buffered) | Universally supported, CDN-friendly |
| LL-HLS / LL-DASH | 2–5 s | Chunked transfer |
| WebRTC | < 500 ms | Interactive; expensive to scale |
| RTMP (ingest) | 2–5 s | Legacy ingest standard |
| SRT / RIST (contribution) | < 1 s | Lossy-network contribution links |

| Video pipeline metric | Value |
|---|---|
| Transcoding cost | 1–5 core-hours per hour of video per rendition (CPU); 10–20× faster on GPU/ASIC |
| Renditions per title (ABR ladder) | 4–8 |
| Storage per hour of content, all renditions | ~10 GB |
| Startup (join) time target | < 2 s; rebuffer ratio target < 0.5% |
| Segment size | 2–10 s (smaller = lower latency, more requests) |

#### Push notifications, email and SMS

| Channel | Throughput | Payload | Notes |
|---|---|---|---|
| APNs (Apple) | Thousands/s per HTTP/2 connection; scale with connections | 4 KB | Feedback/unregistered tokens must be pruned |
| FCM (Android/Web) | ~600,000 messages/minute per project default (≈ 10k/s) | 4 KB | Topic fan-out handled server-side |
| Mass fan-out reality | 10M notifications ÷ 10k/s = ~17 minutes — parallelize workers and prioritize | | Stagger to avoid a thundering herd back into your API |
| WebSocket push (own infra) | 50k–200k connections/node (see the app-server section) | — | Lowest latency, highest operational cost |
| Email (SES) | 14–50 msg/s default, ramps to thousands | 10 MB | Warm up IPs or land in spam |
| SMS (Twilio) | 1 msg/s per long code, ~100/s short code, 10-DLC tiers | 160 chars | Cost $0.005–0.08 per message dominates design |

**The notification fan-out trap:** sending 10M pushes in one burst brings 10M users back to your API within ~60 seconds — a **~170k rps spike**. Always quote the return-traffic number, not just the send rate, and stagger delivery over 5–30 minutes.

---

## Part 6 — The infrastructure underneath

*Sections 26–32 · ~16 min*

The layer most candidates wave at and strong candidates quantify: consensus systems, the physical storage your database sits on, serverless and Kubernetes limits, the serialization format on the wire, the probabilistic structures that turn impossible memory into kilobytes, and the client-side budget that frames your entire latency story.

These sections are mostly Tier 2 and 3. Skim them now; come back when a design touches one.

---

### 26. Coordination: ZooKeeper, etcd, Consul, Raft

> **Tier 2 — know the rough value.** These are **metadata stores: kilobytes, not gigabytes**. A consensus write is one round trip to a majority.

| System | Read throughput | Write throughput | Limits |
|---|---|---|---|
| ZooKeeper | 50k–100k reads/s (served by any node) | 10k–20k writes/s (leader-bound) | znode max 1 MB (keep < 1 KB); ensemble 3, 5 or 7 nodes; watches are one-shot |
| etcd | 30k–90k reads/s (linearizable reads cost a quorum round) | 5k–15k writes/s | DB size 2 GB default quota, 8 GB max; keep < 1.5 GB; value 1.5 MB max; compaction/defrag required |
| Consul | 10k–50k reads/s | 5k–10k writes/s | Adds service discovery + health checks (typical check interval 10 s) |
| Raft/Paxos commit | — | 1 RTT to a majority | Same-AZ ~1 ms, cross-region 60–100 ms per commit |

**Rules to say out loud:**
- Coordination systems are **metadata stores, not data stores.** Kilobytes, not gigabytes.
- Quorum size is `(N/2) + 1`: a 5-node cluster tolerates 2 failures; a 3-node cluster tolerates 1. **Even-numbered clusters buy nothing.**
- Leader election / failure detection takes **1 × session timeout** (typically **5–30 s** for ZooKeeper, **1–2 s** election timeout for etcd/Raft).
- A **distributed lock** costs at least one consensus round trip (1–10 ms same region). If you need 100k locks/s, you do not need locks — you need partitioning.
- Cross-region consensus (Spanner-style) costs **~50–100 ms per write**. That is the entire reason "just make it globally strongly consistent" is not free.

---

### 27. Storage hardware

> **Tier 2 — know the rough value.** Local NVMe **50 µs** vs network-attached storage **1–2 ms**, a 20–40× gap that decides your database p99.

| Device | Random IOPS (4 KB) | Sequential throughput | Latency | Capacity |
|---|---|---|---|---|
| HDD 7,200 rpm | 80–200 | 100–250 MB/s | 4–10 ms | 1–24 TB |
| SATA SSD | 50k–100k | 500–550 MB/s | 100–200 µs | 0.5–8 TB |
| NVMe SSD (PCIe 3) | 300k–600k | 2–3.5 GB/s | 50–100 µs | 0.5–8 TB |
| NVMe SSD (PCIe 4/5) | 600k–1.5M | 5–14 GB/s | 20–80 µs | 1–30 TB |
| Persistent memory / Optane | 1M+ | 6+ GB/s | < 10 µs | 128–512 GB |
| RAM | ~10M+ "IOPS" | 10–50 GB/s per socket | ~100 ns | 64 GB–24 TB |

| Cloud block storage | Figure |
|---|---|
| AWS EBS gp3 | Baseline 3,000 IOPS / 125 MB/s, up to 16,000 IOPS / 1,000 MB/s; latency 1–2 ms |
| AWS EBS io2 Block Express | Up to 256,000 IOPS / 4,000 MB/s, sub-millisecond, 99.999% durability |
| Instance store (NVMe local) | 100k–3M IOPS, ephemeral — lost on stop |
| AWS EFS / NFS | 10s of thousands of IOPS, 1–10 ms latency, elastic throughput |
| GCP pd-ssd | 30 IOPS/GB up to 100k, ~1 ms |

**Design consequences worth stating:**
- Network-attached storage costs **~1–2 ms per I/O** vs **~50 µs** local NVMe — a 20–40× difference that dominates database p99. This is why high-performance databases use local NVMe plus replication instead of EBS.
- **Sequential is 100–1,000× faster than random on HDDs, ~2–5× on NVMe.** That asymmetry is why LSM-trees (Cassandra, RocksDB, ClickHouse) convert random writes into sequential ones.
- **Write amplification:** LSM compaction amplifies writes **5–30×**; B-trees amplify by page size (a 100-byte update rewrites an 8–16 KB page = **80–160×**). Pick the tree for your read/write ratio.
- Never fill a disk past **70–80%** — SSD garbage collection and LSM compaction both need headroom.

---

### 28. Serverless and edge

> **Tier 2 — know the rough value.** **Concurrency = rps × duration**, and the default ceiling is 1,000. Cold starts: 100 ms for Node/Python, seconds for the JVM.

| Metric | AWS Lambda | Notes |
|---|---|---|
| Cold start (Node.js / Python) | 100–400 ms | Add 0.5–2 s inside a VPC in the old ENI model (now ~ms) |
| Cold start (Java / .NET) | 1–5 s (SnapStart: ~200–400 ms) | Why JVM serverless needs provisioned concurrency |
| Warm invocation overhead | 1–10 ms | Plus your code |
| Default concurrency limit | 1,000 per region (raisable to 10k+) | Burst 500–3,000 depending on region |
| Scaling rate | +1,000 concurrent executions per 10 s (per function) | Fast, but not instant |
| Max memory / vCPU | 10 GB (~6 vCPU, CPU scales with memory) | 1,769 MB ≈ 1 vCPU |
| Max execution time | 15 minutes | Longer work → Step Functions / ECS / Batch |
| Payload | 6 MB sync, 256 KB async | Bigger → S3 |
| `/tmp` storage | 512 MB – 10 GB | |
| Pricing | ~$0.20 per 1M requests + $0.0000166667 per GB-second | Crossover vs. always-on EC2/containers at roughly 30–50% sustained utilization |

| Edge platform | Figure |
|---|---|
| Cloudflare Workers | < 5 ms cold start (V8 isolates), 128 MB memory, 10–50 ms CPU per request |
| Lambda@Edge / CloudFront Functions | CF Functions < 1 ms, sub-ms JS; Lambda@Edge 5–50 ms |
| Edge KV stores | Read 5–30 ms at edge, eventual consistency (propagation seconds–60 s) |

**The serverless number that decides architectures:** concurrency = `rps × duration`. 1,000 rps × 200 ms = **200 concurrent executions**. 1,000 rps × 3 s = **3,000 concurrent** — over the default limit, and a throttling incident waiting to happen.

---

### 29. Kubernetes and orchestration limits

> **Tier 3 — know that a limit exists.** **110 pods per node**, and autoscaling takes **1–5 minutes** end to end, which is why you keep 30–50% headroom.

| Metric | Value |
|---|---|
| Pods per node (default) | 110 (configurable to 250+; kubelet-tested) |
| Nodes per cluster | 5,000 |
| Total pods per cluster | 150,000 |
| Total containers per cluster | 300,000 |
| Services per cluster (iptables mode) | degrades past ~5,000; IPVS mode for more |
| etcd backing store | keep under 2 GB (see the coordination section) |
| Pod startup (image cached) | 1–5 s; with image pull 10–60 s |
| HPA reaction time | 15–60 s (metrics window + stabilization) |
| Cluster autoscaler node provisioning | 1–5 min |
| Readiness probe default period | 10 s |
| Graceful termination grace period | 30 s default |
| Sidecar overhead (Envoy) | 50–100 MB RAM, 0.1–0.5 vCPU, +1–4 ms p99 per hop |
| Recommended requests:limits | Requests = p50 usage, limits = 2–4× (avoid CPU limits on latency-sensitive pods) |

**Consequence for design answers:** autoscaling takes **1–5 minutes end-to-end** (metrics → HPA → scheduler → node → image → readiness). Traffic spikes arrive in seconds. Therefore: keep **30–50% headroom**, use queues to absorb bursts, and pre-scale for known events. Saying this is the difference between "we'll autoscale" and an actual capacity plan.

---

### 30. Serialization, compression and protocol overhead

> **Tier 3 — know that a limit exists.** Protobuf is **2–5× faster and 3–10× smaller** than JSON; zstd gives 3–5× compression at 500 MB/s.

| Format | Encode/decode speed | Size vs JSON | Notes |
|---|---|---|---|
| JSON (text) | 100–500 MB/s parse | 1× (baseline) | Human readable, schema-less |
| JSON (simdjson) | 1–3 GB/s | 1× | If parsing is your bottleneck |
| Protocol Buffers | 500 MB–2 GB/s | 0.2–0.5× | Schema, backward compatible, the RPC default |
| Avro | similar to protobuf | 0.2–0.4× | Schema registry; standard in Kafka pipelines |
| Thrift | similar | 0.2–0.5× | |
| MessagePack | 300 MB–1 GB/s | 0.6–0.8× | Schema-less binary |
| FlatBuffers / Cap'n Proto | zero-copy reads (~0 parse) | 0.5–1× | When you read a few fields from big messages |
| Parquet / ORC (columnar, at rest) | GB/s scan | 0.1–0.2× | Analytics storage |

| Compression | Compress speed | Decompress speed | Ratio (text) |
|---|---|---|---|
| LZ4 | 400–800 MB/s | 2–4 GB/s | 2–3× |
| Snappy | 250–500 MB/s | 1–2 GB/s | 2–3× |
| Zstd (level 3) | 300–600 MB/s | 1–1.5 GB/s | 3–5× |
| Zstd (level 19) | 5–20 MB/s | 1 GB/s | 5–7× |
| Gzip (level 6) | 30–100 MB/s | 300–500 MB/s | 3–5× |
| Brotli (level 11) | 1–5 MB/s | 300–500 MB/s | 4–6× (best for static web assets) |

**Protocol overhead:**

| Protocol | Per-request overhead | Latency vs REST/JSON |
|---|---|---|
| HTTP/1.1 + JSON | 200–800 B headers, one request per connection at a time | baseline |
| HTTP/2 + JSON | HPACK-compressed headers, multiplexed | −10–30% |
| gRPC (HTTP/2 + protobuf) | ~50–100 B | 2–5× lower latency and CPU, 3–10× smaller payloads |
| GraphQL | Variable; solves over-fetching, risks N+1 | Add DataLoader batching or pay 10–100× DB queries |
| WebSocket | ~2–14 B per frame after handshake | Near-zero per message; ideal > 1 msg/s per client |
| Server-Sent Events | HTTP framing | One-way, auto-reconnect, simpler than WS |
| Long polling | Full HTTP request per event | Use only as a fallback |

**Rule to quote:** switching a chatty internal service from REST/JSON to gRPC/protobuf typically cuts p99 by **30–50%** and CPU by **2–3×**. Switching a public API for the same reason is usually not worth the ecosystem cost.

#### Hash function throughput

| Function | Speed (per core) | Use |
|---|---|---|
| xxHash3 / wyhash | 10–30 GB/s | Hash tables, checksums, sharding |
| CRC32C (hardware) | 10–20 GB/s | Storage/network integrity |
| MurmurHash3 | 3–6 GB/s | Consistent hashing, bloom filters |
| SipHash-1-3 | 1–3 GB/s | Hash-flooding-resistant keys |
| SHA-256 (with SHA-NI) | 1–2 GB/s | Content addressing, signatures |
| MD5 | ~500 MB/s | Legacy checksums only — broken for security |
| bcrypt / Argon2 | ~10 hashes/s (by design) | Passwords only (see the auth section) |

The 8-order-of-magnitude gap between xxHash and bcrypt is the entire point: **fast hashes for data structures, deliberately slow hashes for secrets.**

---

### 31. Probabilistic data structures

> **Tier 2 — know the rough value.** Bloom filter: **10 bits per element at 1% false positives**. HyperLogLog: **12 KB** counts billions.

These earn a lot of credit because they turn "impossible memory" into "kilobytes."

| Structure | Memory | Error | Use case |
|---|---|---|---|
| Bloom filter | ~10 bits per element for 1% FPR (`m/n = 1.44 × log₂(1/p)`); 1.2 MB per 1M elements | False positives only, never false negatives | "Have I seen this URL?", LSM SSTable skip |
| Counting Bloom / Cuckoo filter | ~1.5–2× Bloom | Supports deletes | Deletable membership |
| HyperLogLog | 12 KB (Redis) for billions of items | 0.81% standard error | Unique visitors, distinct counts |
| Count-Min Sketch | KBs–MBs | Overestimates | Heavy hitters, hot-key detection, rate limiting |
| t-digest / DDSketch | ~KBs | ~1% quantile error | p99 latency aggregation across hosts |
| MinHash / SimHash | 100s of bytes per doc | Jaccard estimate | Near-duplicate detection |
| Consistent hashing ring | 100–200 vnodes per node | ±5% balance | Sharding with minimal reshuffle: adding the Nth node moves only 1/N of keys |

*Example to cite:* one global unique-visitor counter is **12 KB** with 0.8% error, where an exact set would be gigabytes. Per-page counters for 1M pages are 12 KB × 1M = **12 GB**: still far smaller than exact sets, but large enough that you also bucket by time or drop cold pages. Showing both sides of that trade is the kind of reasoning interviewers reward.

---

### 32. Client-side, mobile and frontend budgets

> **Tier 3 — know that a limit exists.** **Active clients ÷ polling interval = your rps.** 1M clients polling every 30 s is 33,000 rps of "anything new?".

Backend-heavy candidates forget that the user's latency budget starts in the browser.

| Metric | Target |
|---|---|
| Largest Contentful Paint (LCP) | < 2.5 s (Core Web Vital) |
| Interaction to Next Paint (INP) | < 200 ms |
| Cumulative Layout Shift (CLS) | < 0.1 |
| Time to First Byte | < 800 ms |
| JS bundle for "interactive in 5 s on 3G" | < 170 KB gzipped |
| Median real-world page weight | ~2 MB (about half of it images) |
| Requests per page | 50–100 is fine over HTTP/2; each costs ~0 RTT after the first |
| Image formats | WebP/AVIF are 25–50% smaller than JPEG at equal quality |
| Critical CSS inline budget | < 14 KB (fits the first TCP congestion window) |
| Mobile app cold start | < 2 s (users abandon past ~3 s) |
| localStorage quota | 5–10 MB |
| IndexedDB quota | 50 MB – several GB (browser/disk dependent) |
| Mobile background sync interval | 15 min minimum on iOS/Android schedulers |
| Battery-friendly polling | ≥ 5 min, or push instead |
| Offline conflict strategy | LWW for simple fields; CRDT for collaborative documents |

**Why it belongs in a backend interview:** a 200 ms API improvement is invisible if the client ships a 3 MB bundle, and conversely, a client that polls every 5 seconds turns 1M users into **200k rps** of pure heartbeat traffic. Quote that conversion from polling interval to server load whenever the design involves mobile clients:

```
server_rps = active_clients / polling_interval_seconds
1M clients ÷ 30 s = 33,000 rps just to say "anything new?"
```
The same 1M clients on WebSockets cost **~5–20 connection nodes** and near-zero request volume.

---

## Part 7 — Running it in production

*Sections 33–35 · ~10 min*

Availability arithmetic, disaster recovery tiers, and money. This is the part that separates "I can design it" from "I have operated it" — and it is where senior-level interviews spend their last ten minutes. Knowing that five serial 99.9% services give you 99.5%, or that egress costs $0.09/GB, changes designs in ways that no amount of architecture vocabulary does.

---

### 33. Availability, reliability and error budgets

> **Tier 1 — learn by heart.** **Five serial 99.9% services give you 99.5%**, or 43 hours a year. Four nines is 53 minutes.

| Availability | Downtime/year | Downtime/month | Downtime/week |
|---|---|---|---|
| 99% ("two nines") | 3.65 days | 7.3 h | 1.7 h |
| 99.9% ("three nines") | **8.77 h** | **43.8 min** | 10.1 min |
| 99.95% | 4.38 h | 21.9 min | 5 min |
| 99.99% ("four nines") | **52.6 min** | **4.4 min** | 1 min |
| 99.999% ("five nines") | 5.26 min | 26 s | 6 s |

**Composition math (the part candidates miss):**
- **Serial dependencies multiply:** a request touching 5 services at 99.9% each ⇒ 0.999⁵ = **99.5%** (43 h/year). This is why you need redundancy *and* graceful degradation.
- **Parallel redundancy:** two independent 99% components in active-active ⇒ 1 − 0.01² = **99.99%** — if failures are truly independent (they rarely are: shared config, shared deploy, shared DNS).
- **A single AZ** typically buys ~99.9%; **multi-AZ** ~99.99%; **multi-region** is what you need for 99.999% — at 2× cost and a large consistency tax.

| Reliability figure | Typical value |
|---|---|
| Annual failure rate, commodity server | 2–5% (in a 1,000-node fleet: ~1 failure every 3–7 days) |
| Disk AFR | 1–3% |
| Rack failure | ~1/year per rack |
| AZ outage | a few per year across a large cloud |
| Region outage | rare, but plan for it (hours) |
| MTTR target | < 30 min (detect < 5 min, mitigate < 15 min) |
| Deploy-related incidents | ~50–70% of all incidents → canary, feature flags, fast rollback (< 5 min) |

**SLO practice:** pick an SLO (e.g. 99.9% of requests < 300 ms over 28 days), derive the **error budget** (0.1% of 100M requests = 100,000 failed requests/month), and spend it deliberately. Retries with **exponential backoff + full jitter**, **circuit breakers** (open after ~50% errors in a 10 s window, half-open probe after 5–30 s), timeouts set at **p99.9 × 1.5**, and bulkheads are the standard toolkit.

**Retry math to be careful with:** naive retries multiply load during an incident. If every client retries 3×, a degraded service sees **4× traffic** at exactly the worst moment. Quote *retry budgets* (cap retries at ~10% of requests) and *jitter*.

#### Security and abuse figures worth knowing

| Metric | Value |
|---|---|
| Largest observed volumetric DDoS | several Tbps; HTTP-layer attacks exceeding 100M+ requests/second |
| Practical origin capacity vs. that | Any single origin is ~4–6 orders of magnitude short → absorption must happen at the edge/CDN/scrubbing tier |
| SYN flood defense | SYN cookies (no per-connection state until ACK) |
| WAF rule evaluation | 0.1–1 ms per request |
| Bot traffic share of the internet | ~40–50% of all requests — size capacity for it or filter it |
| Credential-stuffing defense | Rate limit 5–10 login attempts/min per account, exponential lockout, device fingerprinting |
| Secrets rotation | 30–90 days; TLS certs 90 days automated |
| Encryption overhead (AES-GCM with AES-NI) | 1–10 GB/s per core — TLS in transit is essentially free; it is the *handshake* that costs (see the load-balancer section) |
| Encryption at rest (disk/KMS) | < 5% throughput overhead |
| Field-level encryption | Breaks indexes and range queries — call this out before proposing it |

---

### 34. Multi-region, disaster recovery, RPO and RTO

> **Tier 2 — know the rough value.** Pick an RPO/RTO tier and price it. **DNS failover is 1–5 minutes**, not instant.

| Strategy | RPO (data loss) | RTO (downtime) | Relative cost |
|---|---|---|---|
| Backup and restore | hours–24 h | hours | 1.0× (cheapest) |
| Pilot light (data replicated, compute off) | minutes | 10–60 min | 1.1–1.3× |
| Warm standby (scaled-down live stack) | seconds | 5–15 min | 1.3–1.7× |
| Hot standby / active-passive | < 1 s | 1–5 min (mostly DNS + health checks) | ~2× |
| Active-active multi-region | ~0 (or conflict-resolved) | seconds | 2–2.5× + consistency complexity |

| Mechanism | Number |
|---|---|
| Cross-region async replication lag | 100 ms – 1 s typical (DynamoDB global tables < 1 s, Aurora Global ~1 s) |
| S3 Cross-Region Replication | Minutes (99.99% within 15 min with RTC) |
| Cross-region *synchronous* write cost | **+60–100 ms per commit** — the reason active-active writes usually go last-writer-wins or partitioned-by-region |
| DNS failover | TTL 30–60 s + client cache → 1–5 min real-world; health check detection 30–90 s |
| Anycast / global load balancer failover | seconds (no DNS dependency) |
| Database promotion (managed) | 30–120 s |
| Full restore of 1 TB | 30 min – 4 h |
| Backup schedule (typical) | Full weekly + incremental daily + 5–15 min transaction log shipping |
| Backup retention | 30 days operational, 1–7 years compliance |
| DR drill cadence | Quarterly — an untested backup is not a backup |

**Cell / blast-radius numbers:** cell-based architectures cap a single failure at **1/N of users** (typical N = 5–20 cells). Shuffle sharding with 8 nodes and 2 per customer gives **28 unique combinations** — one bad tenant affects ~7% of others instead of 100%. Deployment safety: canary at **1% → 5% → 25% → 100%** with 10–30 min bake times, and a rollback that completes in **< 5 minutes**.

**Data residency:** GDPR/regional rules often force a region-pinned partition key (`region + user_id`), which conveniently also removes most cross-region write conflicts. Say that out loud — it turns a compliance constraint into an architectural simplification.

---

### 35. Cost figures of merit

> **Tier 2 — know the rough value.** **Egress $0.05–0.09/GB and cross-AZ $0.01–0.02/GB** are the line items that surprise everyone.

Money is a figure of merit too, and mentioning it distinguishes senior candidates.

| Resource | Typical cloud price (2025-ish, on-demand US) |
|---|---|
| vCPU-hour (general purpose) | $0.03–0.05 (≈ $25–40/month per vCPU) |
| GB RAM-hour | $0.004–0.006 (≈ $3–5/month per GB) |
| 16 vCPU / 64 GB instance | ~$400–600/month on-demand; ~$150–250 with 3-year commitment |
| Spot instances | 60–90% discount, can be reclaimed in 2 minutes |
| EBS gp3 | $0.08/GB-month + IOPS/throughput above baseline |
| S3 Standard | $0.023/GB-month (~$23/TB-month) |
| S3 Glacier Deep Archive | $0.00099/GB-month (~$1/TB-month) |
| **Internet egress** | **$0.05–0.09/GB** — the line item that surprises everyone ($90/TB) |
| Cross-AZ traffic | $0.01–0.02/GB each direction |
| CDN egress | $0.02–0.085/GB (cheaper than origin egress) |
| RDS/managed DB premium | ~2× the raw instance cost |
| DynamoDB on-demand | $1.25 per million writes, $0.25 per million reads, $0.25/GB-month |
| Managed Kafka (MSK) | ~$0.05–0.25 per broker-hour + storage |
| Lambda | $0.20/1M requests + $16.67 per million GB-seconds |
| Load balancer | ~$16–25/month + traffic-based units |

**Cost sentences that land:** "Serving 1 PB/month of video from origin at $0.085/GB is **$85,000/month** — a CDN at 95% offload plus committed pricing cuts that by 5–10×." Or: "That microservice chatter is 3 cross-AZ calls per request; at 10k rps and 5 KB per call that's 150 MB/s ≈ 390 TB/month = **$4–8k/month in cross-AZ fees alone**, for traffic that never leaves the region."

---

## Part 8 — Putting it together

*Sections 36–37 · ~14 min*

Nine complete capacity estimates, worked end to end from assumptions to machine counts to the thing that breaks first — URL shortener, social feed, chat, video streaming, ride-hailing, checkout, file sync, video conferencing and autocomplete. Then a map of where each kind of number belongs in the 45 minutes you actually get.

Read one worked example a day and try to reproduce it on paper before looking. That single habit is worth more than re-reading the tables.

---

### 36. Nine worked capacity estimates

> **Tier 1 — learn by heart.** The pattern, every time: **DAU → QPS → bytes → machines → what breaks first.**

#### 36.1 URL shortener (Bitly-scale)

```
Assumptions: 100M new URLs/day, 10:1 read:write
Writes  : 100M / 86,400        ≈ 1,160 writes/s (peak 3×  ≈ 3,500/s)
Reads   : 1B  / 86,400         ≈ 11,600 reads/s (peak     ≈ 35,000/s)
Storage : 500 B/record × 100M  = 50 GB/day = 18 TB/year
Keyspace: base62, 7 chars      = 62⁷ ≈ 3.5 trillion codes
```
**Design implications:** 3,500 writes/s at peak fits comfortably inside one Postgres primary (5k–20k TPS), so write rate is *not* what forces the split. Storage is: 18 TB/year is past the 1–2 TB-per-node comfort zone within weeks, so you shard by hash of the short code from day one. 35,000 reads/s is a **cache problem, not a database problem**. With a 95% hit rate Redis serves 33k/s (well within one cluster) and the DB sees only ~1,700/s. Counters/analytics go to a stream, not to a synchronous `UPDATE` (which would create a hot row).

#### 36.2 Twitter-like feed

```
500M tweets/day       → 5,800 writes/s, peak 15,000/s
200M DAU × 20 reads   → 4B reads/day = 46,000 reads/s, peak 150,000/s
Tweet payload         ≈ 300 B text + metadata → 150 GB/day text, ~30 TB/day with media
Average fanout        ≈ 200 followers → 5,800 × 200 = 1.2M timeline writes/s
```
**Design implications:** pure fanout-on-write costs 1.2M writes/s into a timeline store (feasible with Redis lists: 1.2M ops/s = ~10–20 Redis shards), but a celebrity with 100M followers would need 100M writes for one tweet, so you use the **hybrid**: fanout-on-write for normal users, fanout-on-read (merge at query time) for accounts above ~**100k followers**. Timeline cache: 200M users × 800 tweet IDs × 8 B ≈ **1.3 TB** of Redis, roughly 30–60 nodes.

#### 36.3 Chat / messaging (WhatsApp-scale)

```
100B messages/day     → 1.2M messages/s, peak 3M/s
500M concurrent connections
Per node: 200k WebSockets → 2,500 connection nodes
Message size ~200 B   → 20 TB/day, 7.3 PB/year (before replication)
```
**Design implications:** connections dominate, not CPU. You need a connection tier (Go/Erlang, 200k–1M sockets per node) separate from a message tier, a session registry (which user is on which node — Redis, 500M entries × ~100 B = **50 GB**), and Cassandra/Scylla for message history (1.2M writes/s ÷ 100k writes/s per Scylla node ≈ **12–30 nodes with RF 3**). Kafka between the tiers absorbs bursts at ~1.2M msgs/s = **2–5 brokers of headroom, 120+ partitions**.

#### 36.4 Video streaming (Netflix/YouTube-scale)

```
Bitrates: 480p 1 Mbps | 720p 2.5 Mbps | 1080p 5 Mbps | 4K 15–25 Mbps
10M concurrent viewers × 5 Mbps = 50 Tbps
Storage: 1 h of 1080p ≈ 2.25 GB; 5 renditions ≈ 10 GB per hour of content
Transcoding: ~0.2–1× realtime per CPU core → 1 h video ≈ 1–5 core-hours per rendition (10–20× faster on GPU/ASIC)
```
**Design implications:** 50 Tbps is **impossible from origin** (a 100 Gbps datacenter uplink is 0.2% of it), so the answer is a CDN/edge-cache fleet — thousands of PoPs, 95%+ hit rate, HLS/DASH segments of **2–10 s**, and pre-positioning popular content. Origin only serves the long tail.

#### 36.5 Ride-hailing / location tracking

```
1M active drivers, location ping every 4 s → 250,000 writes/s
Payload ~100 B → 25 MB/s, 2 TB/day raw
Nearby-driver query: p99 < 100 ms, radius 5 km
```
**Design implications:** 250k writes/s rules out a relational primary (10–20k/s). Use an in-memory geospatial index (Redis GEO, S2 cells or Uber's H3 hexagons) keyed by cell ID, with **last-write-wins** and TTL. History goes to Kafka → Cassandra/S3 asynchronously. Quote the resolution choice: H3 resolution 8 ≈ 0.7 km² hexagons, so a 5 km radius touches ~100 cells → ~100 Redis lookups ≈ 5–10 ms with pipelining.

#### 36.6 E-commerce checkout (correctness over scale)

```
1M orders/day → 12 orders/s average, 100–500/s on Black Friday
Inventory decrement must be exact; payment must be idempotent
```
**Design implications:** this is the case where you *should not* reach for eventual consistency. 500 TPS is trivially within a single PostgreSQL primary (5k–20k TPS), so use transactions, `SELECT … FOR UPDATE` or optimistic concurrency with a version column, an **idempotency key** per payment request (stored with a 24 h TTL), and the **outbox pattern** (write the event in the same transaction, ship via CDC with 10 ms–1 s lag) rather than a distributed transaction. Recognizing that the scale does *not* require exotic architecture is itself a senior signal.

#### 36.7 File sync / cloud drive (Dropbox-scale)

```
500M users, 50M DAU, 100 files changed/day each → 5B changes/day = 58,000 changes/s
Average file 1 MB, 4 MB chunks, dedup 3× → 5 PB/day raw → ~1.7 PB/day stored
Metadata: 100B files × 1.5 KB = 150 TB of metadata
```
**Design implications:** the *metadata* service is the hard part, not the bytes — 58k changes/s of small transactional updates is a sharded relational workload (by `user_id`), while the bytes go to object storage with content-addressed chunks and erasure coding at **1.4×** rather than 3× replication (saving ~1 PB/day of overhead). Notification of other devices goes over long-lived connections; delta sync means the average change uploads **~10–50 KB (1–5% of the file), not 1 MB**.

#### 36.8 Video conferencing (Zoom-scale)

```
10M concurrent participants, average meeting 5 people
Each sends 1 stream at 1.5 Mbps, receives 4 → 15 Tbps into the SFUs, 60 Tbps out
Stream legs the SFU tier forwards: 1 in + 4 out = 5 per participant → 50M legs
SFU capacity ~1,000 legs/node → ~50,000 SFU nodes (~200 participants, ~1.5 Gbps each)
```
**Design implications:** SFU, never MCU — mixing 10M streams would cost **10–20M vCPUs**. Note that the SFU count is driven by *outbound* legs: every participant receives four streams, so egress is 4× ingress and the fleet is 5× what "one stream per participant" suggests. Route participants to the nearest regional SFU (mouth-to-ear budget is **150 ms**, so a cross-continent hop of 100 ms is already the whole budget), use simulcast so each receiver gets a layer matching its bandwidth, and expect **15–20% of users to need TURN relays**, which is ~2–3 Tbps of relay capacity on the uplink alone that you must actually pay for.

#### 36.9 Search autocomplete / typeahead

```
100M searches/day → 1,160 QPS, peak 3,500 QPS
Each keystroke queries: 5 keystrokes per search → 17,500 QPS of prefix lookups
p99 budget: < 100 ms end to end, so < 20 ms server-side
Corpus: 100M distinct queries, average 20 bytes
```
**Design implications:** 17,500 QPS at sub-20 ms rules out hitting a database per keystroke. A trie of the top few million prefixes with the top-10 completions precomputed at each node is **~1–5 GB in memory** — it fits on every application server, so the query never leaves the box. Rebuild the trie offline from query logs **hourly**, and debounce the client at **50–100 ms** so you cut keystroke traffic by 2–3× before it ever arrives.

---

### 37. Interview mechanics: where the numbers go

> **Tier 1 — learn by heart.** Estimation belongs in minutes 5–10; thresholds belong in the deep dive. Numbers without a decision score nothing.

A 45–60 minute system design interview has a predictable shape, and the numbers belong in specific places.

| Phase | Time | What you produce | Numbers to state |
|---|---|---|---|
| Requirements and scope | 5–10 min | Functional list, 2–3 non-functional targets, explicit out-of-scope | DAU, latency SLO, availability target, read:write ratio |
| Back-of-envelope estimation | 3–5 min | QPS, storage/year, bandwidth, memory for cache | Average *and* peak (3×), bytes per record |
| API and data model | 5 min | 3–5 endpoints, core entities, keys | Payload sizes, page sizes, partition/shard key |
| High-level design | 10–15 min | The box diagram, data flow for the main path | Instance counts, replication factor, cache hit rate |
| Deep dive | 10–15 min | One or two components at real depth | Thresholds: when to shard, partition counts, timeouts |
| Bottlenecks, failure, wrap-up | 3–5 min | What breaks first, how you'd detect and mitigate it | Utilization headroom, error budget, failover times |

**Scorecard behaviors that numbers unlock:**
- *Scope control:* "10M DAU, not 10B — so one region and a single sharded database is honest."
- *Justified tradeoffs:* "Strong consistency here costs 60–100 ms cross-region, so I'll keep writes region-pinned."
- *Bottleneck identification:* "The first thing to fail is the 10k-writes/s primary at ~2× growth."
- *Headroom thinking:* "I size for 3× peak and 60–70% utilization, so 16 shards, not 6."
- *Knowing when not to scale:* "500 TPS fits one Postgres node — the complexity isn't justified."

**Three phrases to keep in your pocket:**
1. *"Let me state my assumptions so you can correct them."*
2. *"That's roughly X — within an order of magnitude, which is all I need to pick the architecture."*
3. *"At 3× growth this component breaks first, and here's the migration path."*

---

## Part 9 — Practice

*Section 38 and the flashcard list · ~15 min, done properly*

Twenty drills that take a few seconds each, and a flashcard list of the Tier 1 numbers. Do these **cold** — closed page, out loud, writing the arithmetic down. Recognizing a number when you read it and producing it under interview pressure are different skills, and only the second one gets scored.

---

### 38. Drills: twenty questions, cold

Cover the answers. Say each one out loud, with the arithmetic. If it takes you more than ~20 seconds, that number belongs on a flashcard.

1. 5M DAU, 30 requests each, peak is 3× average. What QPS do you provision for?
2. Your app server is in the same AZ as Redis. What p50 does a `GET` add to your request?
3. At what size do you partition a PostgreSQL table? At what size do you shard the database?
4. You need to ingest 500 MB/s into Kafka. How many partitions, minimum?
5. A service handles 2,000 rps with 40 ms average latency. How many threads/connections does it need in flight?
6. What is the maximum write rate DynamoDB will give a single partition key?
7. You are indexing a 2 TB Elasticsearch index. How many primary shards?
8. Three services at 99.95% in series. What is the end-to-end availability, in hours of downtime per year?
9. 1M concurrent WebSocket clients. How many connection nodes?
10. 50M embeddings at 768 dimensions, fp32, HNSW. How much RAM?
11. Kafka at 200 MB/s with 7-day retention and RF 3. How much disk?
12. You egress 500 TB/month from the origin at $0.085/GB. What is the bill?
13. What does a synchronous cross-region write cost, in milliseconds?
14. A 16-core box hashes passwords with bcrypt cost 12. How many logins per second?
15. You need to cut database read load by 10×. What cache hit ratio do you need?
16. Your servers run at 85% CPU. Roughly how much worse is queueing delay than at 50%?
17. A job queue takes 500 jobs/s, each job runs 4 seconds. How many concurrent workers?
18. You need 20,000 PUT/s into S3. How do you get it?
19. A single Postgres primary is taking 25,000 writes/s. What breaks first?
20. You send 10M push notifications in one minute. What does your API see?

---

#### Answers

1. **~1,750 rps average, ~5,200 rps peak.** 5M × 30 = 150M/day; ÷ 86,400 ≈ 1,736; × 3 ≈ 5,200. Provision for the peak.
2. **~0.2–0.5 ms.** ~0.1 ms server-side plus same-AZ network. If you said "microseconds" you forgot the network; if you said "5 ms" you are thinking of a database.
3. **Partition at ~100 GB or 100M rows. Shard at 1–2 TB, or above ~10–20k writes/s, or when the working set stops fitting in RAM.**
4. **~50 partitions minimum** (500 ÷ 10 MB/s per partition), realistically **100–150** for consumer parallelism and headroom. Stay under ~4,000 per broker.
5. **~80 in flight** (Little's Law: 2,000 × 0.04), so size the pool at ~100 with headroom.
6. **1,000 WCU — 1,000 writes/s of 1 KB.** Past that you must shard the key with a suffix.
7. **~65–70 primaries** (2,000 GB ÷ 30 GB per shard), doubled to ~135 shards with one replica.
8. **99.85%**, which is **~13 hours/year**. Serial dependencies multiply: 0.9995³.
9. **5–10 nodes** at 100k–200k connections each — plus headroom, so call it 12–15.
10. **~175–200 GB.** 768 × 4 B = 3 KB per vector plus ~0.5–1 KB of HNSW graph; × 50M. Shard it, or quantize to int8 and cut it 4×.
11. **~363 TB.** 200 MB/s × 604,800 s × 3 replicas. (And then divide by 0.7 for disk headroom: ~520 TB provisioned.)
12. **~$42,500/month.** 500,000 GB × $0.085. This is why the answer is a CDN.
13. **+60–100 ms per commit** — a full round trip to the other region. It is the reason "just make it globally consistent" is never free.
14. **~40–80 logins/s.** bcrypt cost 12 is 200–400 ms per hash, so 2.5–5 per core, × 16 cores. Login is your most expensive endpoint.
15. **90%.** Origin load is `1 − hit_ratio`. For 20× you need 95%.
16. **About 3× worse.** M/M/1 queueing: 1/(1−0.85) ≈ 6.7× service time vs 1/(1−0.5) = 2×. This is why you provision to 60–70%.
17. **2,000 concurrent workers** (500 × 4). At 4 jobs per core that is ~500 cores — the number that should make you redesign the job, not the fleet.
18. **Spread across at least 6 prefixes** (20,000 ÷ 3,500 per prefix), realistically 10+. Prefixes are unlimited; per-prefix rate is not.
19. **WAL/fsync throughput and replication lag.** You are ~2× past a single node's comfort zone, so replicas fall behind and p99 commit latency climbs. Shard, or batch writes.
20. **A ~170k rps spike** within about a minute, as recipients open the app. Stagger delivery over 5–30 minutes, and size the read path for the return traffic — not just the send rate.

---

### The Tier 1 flashcard list

Forty numbers. If you can produce all of them cold, you are calibrated for any system design interview. Cover the right column.

| Ask yourself | Answer |
|---|---|
| Main memory reference | **100 ns** |
| NVMe SSD random read | **50–100 µs** |
| Round trip inside a datacenter | **0.5 ms** |
| Round trip cross-AZ | **1 ms** |
| Round trip US → Europe | **~90 ms** |
| HDD seek | **~8 ms** |
| RTT per 100 km of fiber | **1 ms** |
| 1M requests/day in rps | **~12 rps** |
| 1B requests/day in rps | **~11,600 rps** |
| Seconds in a day / year | **86,400 / 31.5M** |
| Peak-to-average ratio | **3×** (range 2–10×) |
| Safe CPU utilization target | **60–70%** |
| Concurrency formula | **rps × latency** (Little's Law) |
| App server, realistic business logic | **1,000–5,000 rps** |
| NGINX / reverse proxy per box | **50k–500k rps** |
| RSA-2048 TLS handshakes per core | **1,000–2,000/s** |
| Concurrent WebSockets per node | **100k–200k** |
| Redis latency (client-observed, same AZ) | **0.2–1 ms** |
| Redis ops/sec per node | **~100k** (1M pipelined) |
| Redis shard size ceiling | **25–50 GB** |
| Cache hit ratio for 20× origin offload | **95%** |
| PostgreSQL reads / writes per node | **20k/s / 10k/s** |
| Partition a relational table at | **100 GB or 100M rows** |
| Shard a relational database at | **1–2 TB or 10–20k writes/s** |
| Postgres connections before pooling | **200–500** |
| Quorum rule | **R + W > N** |
| Cross-region synchronous write | **+60–100 ms** |
| DynamoDB partition ceiling | **3,000 RCU / 1,000 WCU / 10 GB** |
| DynamoDB max item size | **400 KB** |
| Cassandra partition size limit | **< 100 MB** (target < 10 MB) |
| Elasticsearch shard size | **10–50 GB**, heap ≤ **31 GB** |
| Kafka partitions per broker | **≤ 4,000** (plan ~2,000) |
| Kafka throughput per partition | **~10 MB/s** |
| Kafka max message size | **1 MB** |
| S3 request rate per prefix | **3,500 PUT / 5,500 GET per second** |
| S3 first-byte latency | **100–200 ms** |
| Object storage durability | **11 nines** |
| 99.9% / 99.99% downtime per year | **8.8 h / 53 min** |
| Five serial 99.9% services | **99.5%** |
| Internet egress cost | **$0.05–0.09/GB** |

**How to drill these:** read the left column, say the answer out loud, then check. Three passes of ten minutes each, spread across three days rather than one, beat an hour of re-reading.

---

## Closing: how to actually use these in the room

Nobody expects you to recite this table. What a strong interviewer wants to hear is a **short chain of numbers that leads to a decision**:

> *"20M DAU, 10 writes each, so 200M writes/day ≈ 2,300/s, peak ~7,000/s. Each record is ~500 bytes, so 100 GB/day, 36 TB/year. 7,000 writes/s is above one Postgres primary's comfort zone of 5–10k, and 36 TB is far past the 1–2 TB I'd want on one node, so I'll shard by user_id into, say, 16 shards. That's ~440 writes/s and ~2 TB each, which leaves room to double before resharding. Reads are 20:1, so 140k reads/s, and that's a cache tier: Redis at a 95% hit rate serves 133k/s across ~4–6 shards and the databases see only 7k/s."*

That paragraph contains maybe a dozen of the numbers above. It takes ninety seconds. It demonstrates estimation, capacity planning, technology knowledge and an awareness of headroom — which is, more or less, the entire rubric.

If that paragraph does not come naturally yet, the drills in Part 9 are the fastest way to get there — twenty of them, a few seconds each.

Three habits to practice:

1. **Always state the assumption with the number.** "Assuming 1 KB records" costs you two seconds and buys you the right to be wrong.
2. **Round aggressively.** 86,400 is 100,000. 365 is 400. Nobody wants to watch you do long division.
3. **Know the thresholds, not just the maxima.** The valuable knowledge is not "Postgres tables can be 32 TB," it is "I'd partition at 100 GB and shard at 1 TB" — that is the number that changes the design on the whiteboard.

Bookmark this, benchmark your own systems against it, and update the numbers as hardware moves — because it does. NVMe made "disk is slow" half-wrong; KRaft made "Kafka can't do a million partitions" wrong; ZGC made "JVM means 200 ms pauses" wrong. The figures of merit are a living document; the habit of reaching for them is the permanent skill.
