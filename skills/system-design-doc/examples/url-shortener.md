**Test prompt:** *"Design a URL shortener like bit.ly"

---

# System Design: URL Shortener

## 1. Problem Statement
Users need a way to convert long URLs into short, shareable links that redirect to the original destination. The system must handle massive read traffic (redirects vastly outnumber writes), generate globally unique short codes, and deliver redirects with minimal latency.

*Assumptions: 10M users, ~500M redirects/day (~6K RPS peak reads), ~1M new links/day (~12 RPS writes). Read:write ratio ~500:1.*

---

## 2. Requirements

**Functional**
- Shorten a URL → return a short code (e.g. `sho.rt/aB3xZ`)
- Redirect short code → original URL
- Optional: custom aliases, expiry dates, click analytics

**Non-Functional (NFRs)**
- Latency: p99 redirect < 50ms
- Availability: 99.99% (redirects are customer-facing)
- Durability: no link loss once created
- Scale: 500M redirects/day, 1M new links/day
- Consistency: eventual is fine (new links may take ~1s to propagate globally)

---

## 3. Capacity Estimation

| Metric | Estimate |
|--------|----------|
| Write RPS | ~12 RPS |
| Read RPS | ~6,000 RPS (peak) |
| Avg URL size | ~200 bytes |
| New links/year | ~365M → ~73 GB/year |
| Cache hit target | 95% (top 10% links = 90% traffic) |

Short code space: 7 chars from base-62 = 62⁷ ≈ 3.5 trillion codes. Plenty.

---

## 4. High-Level Architecture
<img width="1066" height="412" alt="image" src="https://github.com/user-attachments/assets/443a0792-ac57-4871-bf20-65ae9b4f4632" />

```
flowchart LR
  Client["Client"] --> LB["Load Balancer"]
  LB --> WS["Write Service"]
  LB --> RS["Redirect Service"]
  WS --> IDGen["ID Generator\n(Snowflake)"]
  WS --> DB[("PostgreSQL")]
  RS --> Cache["Redis Cache"]
  Cache -->|miss| DB
  DB --> RS
```

**Components:**
- **Write Service** — accepts long URL, generates short code, writes to DB
- **Redirect Service** — looks up short code, serves 301/302 redirect
- **ID Generator** — produces unique IDs using Snowflake or a counter-based approach
- **Redis Cache** — caches hot short-code → URL mappings; primary read path
- **PostgreSQL** — source of truth for all mappings

---

## 5. Data Modeling

**`url_mappings` table (PostgreSQL)**

| Field | Type | Notes |
|-------|------|-------|
| `short_code` | `VARCHAR(8)` PK | indexed |
| `long_url` | `TEXT` | |
| `user_id` | `UUID` FK | nullable (anonymous) |
| `created_at` | `TIMESTAMP` | |
| `expires_at` | `TIMESTAMP` | nullable |
| `click_count` | `BIGINT` | updated async |

Index on `short_code` (primary key lookup only — no range scans needed).
Click counts are updated by async workers to avoid write amplification on the hot path.

**Data store choice:** PostgreSQL over NoSQL here because the data is relational (users → links), the write volume is low, and we want ACID on creation. Redis for cache; no Cassandra needed at this scale.

---

## 6. API Design

```
POST /shorten
  Request:  { long_url, custom_alias?, expires_at? }
  Response: { short_code, short_url }

GET /{short_code}
  Response: HTTP 301 redirect → long_url
            HTTP 404 if not found or expired

GET /api/links/{short_code}/stats     (optional)
  Response: { click_count, created_at, expires_at }
```

Use **301 (permanent)** redirect for cacheability at browser level; use **302 (temporary)** if you need accurate click analytics (browsers won't re-hit the server on 301).

---

## 7. Scalability & NFR Mapping

| NFR | Decision |
|-----|----------|
| p99 redirect < 50ms | Redis cache in front of DB; in-memory lookup ~1ms |
| 99.99% availability | Multi-AZ Redis + PostgreSQL with read replicas; stateless redirect servers scale horizontally |
| Horizontal scale | Redirect Service is stateless — add instances behind load balancer |
| Durability | PostgreSQL with WAL + daily snapshots; Redis is cache-only (loss is acceptable) |
| Eventual consistency | Cache-aside pattern; new links visible within seconds of DB write |

---

## 8. Trade-off Analysis

**301 vs 302 redirects** — 301 is browser-cached (faster UX, no repeat traffic) but makes click analytics impossible for repeat visits. 302 hits your server every time (accurate analytics, higher load). Most link shorteners use 302 for analytics and eat the extra traffic.

**Random code generation vs counter-based** — Random base-62 codes are simple but risk collisions (need retry logic). Counter-based with Snowflake IDs are monotonically unique but expose link volume. Chosen: counter-based encoded to base-62, collision-free and fast.

**PostgreSQL vs Cassandra** — Cassandra would handle 10× the write scale but adds operational complexity and loses transactions. At 12 write RPS, PostgreSQL is the right call. Revisit at 10K write RPS.

---

## 9. Gotchas & Failure Modes

**Cache stampede on popular links** — A viral link that expires from cache causes thousands of simultaneous DB lookups. Fix: probabilistic early expiry refresh or a "lock and load" pattern where one request fetches from DB while others wait briefly.

**Hot partition in Redis** — A single extremely viral short code can hammer one Redis node. Fix: replicate hot keys across multiple Redis nodes or use client-side key sharding with a prefix.

**Custom alias conflicts** — Two users requesting the same alias simultaneously can both succeed without proper locking. Fix: unique constraint on `short_code` in DB + optimistic retry in the Write Service.

**301 redirect poisoning** — Browsers cache 301s forever. If a link is updated or deleted, users with a cached redirect are stuck. Never use 301 unless links are truly immutable.

**URL validation gap** — Users can shorten URLs pointing to malware or phishing pages. You need async URL scanning (e.g. Google Safe Browsing API) and a blocklist, or you'll become a malware distribution vector. This is a compliance landmine.

**Click count precision** — If you update `click_count` synchronously on every redirect at 6K RPS, the DB will buckle. Write counts to Redis incr() and flush to DB in batches every 60 seconds.

---

## 10. Out of Scope / Future Considerations

**Not designed here:** analytics dashboard, geographic redirect (route to nearest server), QR code generation, link bundles, A/B testing via redirects.

**v2 additions:** Multi-region active-active with CRDTs for eventual consistency on click counts; rate limiting on the Write Service to prevent link spam; a Link Preview service that generates OG metadata for social sharing.

---
