# System Design Day 1 — Fundamentals

## 1. What is System Design?

System Design is the process of translating business, functional, non-functional, scale, and operational requirements into a technical architecture.

A strong design explains not only **what components exist**, but also:

- why each component is needed,
- what problem it solves,
- how the system behaves as load grows,
- what happens during failures,
- what alternatives exist,
- and what trade-offs are being made.

A useful mental flow is:

```text
Business Need
    ↓
Requirements
    ↓
Constraints + Scale
    ↓
Technical Decisions
    ↓
Architecture
```

System Design is therefore more than drawing boxes such as Load Balancer, Redis, Kafka, or Database. The reasoning behind those choices is the important part.

---

## 2. Why Do We Need System Design?

A small application may work with:

```text
Client
  ↓
Application
  ↓
Database
```

But as users, traffic, data, geography, and reliability expectations increase, additional concerns appear:

- CPU, memory, disk, and network limits
- database bottlenecks
- server failures
- traffic spikes
- latency
- data durability
- consistency
- security
- operational complexity
- cost

There is no universally best architecture. The correct design depends on the requirements and constraints.

### Core principle

> Architecture should come from requirements, not from technology preferences.

Bad approach:

```text
I know Kafka
→ Let's use Kafka
```

Better approach:

```text
We need asynchronous/event processing
→ evaluate possible solutions
→ decide whether Kafka is appropriate
```

---

## 3. Functional Requirements

Functional requirements describe **what the system must do**.

For a simple URL shortener:

- accept a long URL,
- generate a short URL,
- redirect a short URL to the original URL.

Example:

```text
Input:
https://example.com/products/mobile/iphone

Output:
https://short.ly/aB7x9Q
```

Later:

```text
GET short.ly/aB7x9Q
→ redirect to the original URL
```

### Requirement vs implementation

Requirement:

> Generate a short URL.

Implementation idea:

> Use Base62 encoding with a database-generated identifier.

Do not confuse the two. Requirement gathering should identify **what** must happen before deciding **how** to implement it.

### Questions to identify functional requirements

Ask:

- Who uses the system?
- What actions can they perform?
- What data enters the system?
- What output should they receive?
- What happens after each action?

### Core vs optional scope

For a URL shortener:

**Core**
- Create short URL
- Redirect short URL

**Optional**
- Custom aliases
- Expiration
- Analytics
- QR codes
- User accounts

During an interview, explicitly limiting scope prevents the design from becoming too broad.

---

## 4. Non-Functional Requirements

Non-functional requirements describe **how well the system must perform its functions**.

Common categories:

- latency
- throughput
- availability
- reliability
- durability
- scalability
- consistency
- security
- maintainability
- cost efficiency

### Latency

Latency is the time taken by one operation.

Example:

```text
p95 redirect latency < 100 ms
```

### Throughput

Throughput is the amount of work processed per unit time.

Examples:

```text
20,000 requests/sec
1 million events/minute
```

### Availability

Availability asks:

> Can users access the service when they need it?

### Reliability

Reliability asks:

> Does the system continue to behave correctly over time and during failures?

A service can be available but unreliable—for example, if it responds successfully while returning incorrect data.

### Durability

Durability asks:

> Once data has been acknowledged as saved, will it survive failures?

### Scalability

Scalability asks:

> Can the system continue operating as load grows?

### Consistency

Consistency asks:

> What data versions or states are different readers allowed to observe after updates?

### Security

Examples include:

- authentication
- authorization
- HTTPS
- encryption
- abuse prevention
- rate limiting

### Maintainability

A maintainable system should be understandable, deployable, observable, testable, and changeable without unnecessary risk.

### Cost efficiency

The technically most sophisticated architecture is not automatically the best. Infrastructure and operational cost matter.

### Make NFRs measurable

Weak:

> The system should be fast.

Better:

> p95 latency should remain below 200 ms.

Weak:

> The system should support many users.

Better:

> The service should sustain 50,000 requests/sec at peak.

---

## 5. Important Quality-Attribute Distinctions

### Latency vs Throughput

```text
Latency    = time taken by one operation
Throughput = operations handled per unit time
```

### Availability vs Reliability

```text
Availability = Can I access it?
Reliability  = Can I trust it to behave correctly?
```

### Availability vs Durability

```text
Availability = Is the service reachable?
Durability   = Does acknowledged data survive?
```

### Durability vs Consistency

```text
Durability  = Is saved data preserved?
Consistency = What state/version do readers observe?
```

---

## 6. Constraints

Constraints are boundaries that restrict the design choices available to us.

Examples:

- Must use PostgreSQL
- Must deploy on Azure
- Budget is limited
- Only four engineers are available
- Data must remain in a specific country
- Must integrate with an existing legacy API
- Only approved managed services may be used

### Requirement vs constraint

```text
Requirement:
Support 20,000 requests/sec.

Constraint:
The company requires PostgreSQL.
```

### Constraint vs assumption

Constraint:

> The organization requires deployment on Azure.

Assumption:

> Assume peak traffic is 50,000 requests/sec.

A constraint comes from reality or stakeholders. An assumption is introduced when information is missing so that we can continue reasoning.

### Hard vs soft constraints

Hard:

> Financial data must remain in India.

Soft:

> Prefer PostgreSQL because the team already operates it.

---

## 7. Scale Assumptions

Scale assumptions answer:

> How big is the system?

Typical estimates include:

- total users
- daily active users
- requests per second
- peak requests per second
- reads vs writes
- data generated per day
- storage growth
- bandwidth
- geographic distribution
- retention period

### URL shortener example

Assume:

```text
100 million new URLs/month
10 billion redirects/month
```

Read/write ratio:

```text
10,000M : 100M
≈ 100 : 1
```

This tells us the workload is heavily read-oriented.

---

## 8. Average and Peak QPS

For 10 billion redirects/month:

```text
30 × 24 × 60 × 60
≈ 2.6 million seconds/month
```

```text
10,000,000,000 / 2,600,000
≈ 3,850 requests/sec
≈ 4K average QPS
```

Real traffic is not uniform.

If peak traffic is assumed to be 5× average:

```text
~4K average QPS
→ ~20K peak QPS
```

Systems should not be sized only for average load.

Examples of traffic spikes:

- flash sales
- sports finals
- election results
- concert ticket launches
- stock-market opening
- breaking news

---

## 9. Read-Heavy vs Write-Heavy Systems

Read-heavy examples:

- URL shortener redirects
- news sites
- product catalogs

Write-heavy examples:

- log ingestion
- IoT telemetry
- analytics event pipelines
- market-data ingestion

The workload shape later influences caching, database, replication, and partitioning decisions.

---

## 10. Storage Estimation

Assume:

```text
100M new URLs/month
500 bytes per stored mapping
```

Monthly raw storage:

```text
100,000,000 × 500 bytes
≈ 50 GB/month
```

Yearly:

```text
≈ 600 GB/year
```

Five years:

```text
≈ 3 TB raw data
```

Real storage will usually be higher because of:

- indexes
- replication
- metadata
- backups
- database overhead
- analytics

---

## 11. Bandwidth Estimation

If a response is approximately 1 KB and peak traffic is 20,000 requests/sec:

```text
20,000 × 1 KB
≈ 20 MB/sec
```

Back-of-the-envelope calculations do not need perfect precision. Their purpose is to establish the correct order of magnitude.

Useful shortcuts:

```text
1 day ≈ 100,000 seconds
1 million ≈ 10^6
1 billion ≈ 10^9

1 KB ≈ 10^3 bytes
1 MB ≈ 10^6 bytes
1 GB ≈ 10^9 bytes
1 TB ≈ 10^12 bytes
```

---

## 12. FNCS — Day 1 Mental Model

Before architecture, think:

```text
F → Functional Requirements
N → Non-Functional Requirements
C → Constraints
S → Scale Assumptions
```

Or:

```text
WHAT?
→ Functional Requirements

HOW WELL?
→ Non-Functional Requirements

WHAT LIMITS US?
→ Constraints

HOW BIG?
→ Scale
```

Example URL shortener:

```text
Functional
- Create short URL
- Redirect short URL

Non-Functional
- Low redirect latency
- High availability
- Durable mappings
- Large read throughput

Constraints
- Example: approved cloud/database
- Example: 5-year retention

Scale
- 100M URLs/month
- 10B redirects/month
- ~100:1 reads:writes
- ~4K average redirect QPS
- higher peak QPS
```

Only after this phase should the interview move toward APIs, data model, architecture, databases, caches, messaging, and scaling.

---

## 13. Common Mistakes

Avoid:

- starting with technologies,
- trying to support every possible feature,
- confusing a requirement with a solution,
- ignoring non-functional requirements,
- ignoring peak traffic,
- using precise-looking numbers without explaining assumptions,
- ignoring business, regulatory, operational, and team constraints,
- over-engineering small-scale systems.

---

## 14. Senior-Level Thinking

A senior engineer should ask not only:

> Which database is technically best?

but also:

- What does the team already operate?
- What is the operational burden?
- What does it cost?
- What are the migration implications?
- What happens if the dependency fails?
- What happens at 10× traffic?
- Can the design be simplified?
- What trade-off are we making?

Use this reasoning chain:

```text
Requirement
    ↓
Problem
    ↓
Possible Solutions
    ↓
Trade-offs
    ↓
Decision
```

Not:

```text
Interesting Technology
    ↓
Find somewhere to use it
```

---

# Interview Tips

- Start by clarifying the problem before naming technologies.
- Identify only the core 2–5 functional requirements initially.
- Explicitly place optional features out of scope when necessary.
- Make vague NFRs measurable.
- State assumptions when scale information is missing.
- Estimate average and peak traffic separately.
- Explain why a component is needed before choosing the technology.
- Prefer the simplest architecture that satisfies the stated needs.
- Discuss alternatives and trade-offs.
- Include operational cost and maintainability at senior level.

A strong opening is:

> Before designing the architecture, I’d like to clarify the main use cases, non-functional expectations, constraints, and expected scale.

---

# Interview Questions

## What is System Design?

System Design is the process of translating functional, non-functional, scale, and operational requirements into an architecture composed of appropriate components and interfaces while considering scalability, reliability, failures, and trade-offs.

## Functional vs non-functional requirements?

Functional requirements describe **what the system does**.

Non-functional requirements describe **how well it must do it**.

## Requirement vs implementation?

A requirement defines the expected behavior or outcome. An implementation choice is one technical way to satisfy that requirement.

## What is a constraint?

A constraint is a real boundary that limits design choices, such as technology mandates, cloud restrictions, compliance, budget, or organizational limitations.

## Constraint vs assumption?

A constraint is imposed by the environment or stakeholders. An assumption is introduced when information is unavailable so that design reasoning can continue.

## Latency vs throughput?

Latency is the time required for an operation. Throughput is how many operations can be processed per unit time.

## Availability vs reliability?

Availability is whether the service can be accessed. Reliability is whether it behaves correctly over time, including during failures.

## Availability vs durability?

Availability concerns service access. Durability concerns survival of successfully stored data.

## Durability vs consistency?

Durability concerns persistence. Consistency concerns which state/version readers observe.

## Why estimate scale before architecture?

Because traffic, peak load, read/write ratio, storage growth, bandwidth, and geography directly influence later architecture decisions.

## Why is average QPS not enough?

Because real systems experience bursts and peaks. Peak capacity frequently determines whether a service remains healthy during its most important periods.

## What does FNCS stand for?

Functional Requirements, Non-Functional Requirements, Constraints, Scale Assumptions.

---

# Practical Interview Exercises

1. **Design WhatsApp** — Give the first five clarification questions before drawing architecture.
2. **Design a Notification System** — Define only FNCS.
3. **Design Food Delivery** — Choose 3–5 core functional requirements and explicitly defer the rest.
4. A payment service is always reachable but sometimes charges twice. Which quality attribute is poor?
5. A system acknowledges a write and loses the record after restart. Which quality attribute failed?
6. A system has 1 billion reads/month and 10 million writes/month. Estimate the read/write ratio.
7. A system receives 10 million requests/day. Estimate average QPS using 1 day ≈ 100K seconds.

---

# Final Recall Test

Without referring to the notes, answer:

1. What is System Design?
2. Why should architecture come after requirements?
3. What is a functional requirement?
4. What is a non-functional requirement?
5. Give three measurable non-functional requirements.
6. What is a constraint?
7. What is the difference between a constraint and an assumption?
8. Why do we estimate scale?
9. What is the difference between latency and throughput?
10. What is the difference between availability and durability?
11. What is the difference between availability and reliability?
12. What is the difference between durability and consistency?
13. Why is average QPS not enough?
14. What is a read/write ratio and why does it matter?
15. What does FNCS stand for?

---

# Whiteboard Exercise

Draw:

```text
Problem Statement
      ↓
Functional Requirements
      ↓
Non-Functional Requirements
      ↓
Constraints
      ↓
Scale Assumptions
      ↓
Architecture
```

Stop before architecture and use a URL shortener to provide one simple example for each FNCS category.

---

# Day 1 Completion Checklist

Day 1 is complete when you can explain from memory:

- what System Design is,
- why requirements come before architecture,
- functional requirements,
- non-functional requirements,
- constraints,
- assumptions,
- scale estimation,
- latency vs throughput,
- availability vs reliability vs durability,
- consistency at a basic level,
- read/write ratios,
- average vs peak QPS,
- basic storage and bandwidth estimation,
- FNCS,
- and the common interview mistakes above.
