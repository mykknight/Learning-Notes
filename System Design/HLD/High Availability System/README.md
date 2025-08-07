# 🏗️ Design a High Availability System

Also Known As:

- Avoiding Single Points of Failure

- Active-Passive / Active-Active Architecture

- Designing for 99.99% System Uptime

## Problem Statement


        ┌─────────────┐
        │   Client    │
        └─────┬───────┘
              │
              ▼
       ┌───────────────┐
       │     Load      │
       │   Balancer    │
       └─────┬─────────┘
             │
             ▼
       ┌──────────────────────────┐
       │  AP.S-1   AP.S-2         │ Micro Service
       │         AP.S-3           │ Architecture
       └─────┬────────────────────┘
             │
             ▼
      ┌────────────────┐
      │   DataBase     │◄──── Primary DB
      └─────┬──────────┘    

## ❗ Issue:
If the Primary Database fails, the entire application or service goes down — this is known as a Single Point of Failure (SPOF) and should be avoided in any highly available system.


## ✅ Solution: Multi-Node Architecture

### 1️⃣ Active-Passive Architecture
Every production-grade system typically runs on at least two data centers (DCs).

📌 Primary Data Center (DC-1)

📌 Secondary/Backup Data Center (DC-2)

![Active-Passive Architecture](<./active-passive.png>)

- only one could be the primary/Live/read/Write
- Other one is for replica/disaster-recovery
  
#### 🔄 Process Flow:
- Write Request:
  - First POST request → goes through DC-1 → handled by AP.S-1 → hits Primary DB
- Read Request:
  - A GET request can go to either DC-1 or DC-2.
  - If it reaches DC-2 (Passive), the read will happen from the replica (read-only DB).
- What if a POST hits DC-2?
  - DC-2's application layer forwards the write request to DC-1's Primary DB, since DB-2 is not writable.


#### 🔍 Key Considerations:

❓ Why can't we write to the Replica DB?
- Traditional relational databases like Oracle, MySQL, and PostgreSQL allow writes only to one primary node.

- Replica nodes are read-only and eventually consistent — not suitable for direct write operations.

❓ What about reads from Replica?
- ✅ Yes, replicas can and should be used for read operations to distribute load.

❓ What happens if Primary DB fails?
- The replica DB (e.g., in DC-2) will be promoted to Primary, and both app layers switch their reference to the new primary.


#### ⚠️ Disadvantages of Active-Passive

1. Latency in Passive Requests:
   - Example:
      - DC-1 → AP.S-1 → DB-1 → ~1 ms
      - DC-2 → AP.S-2 → DB-1 (cross-region) → Higher latency due to network hops
2. Failover Time:
   - There is a delay during failover — the system is momentarily unavailable while promoting passive DB to active.
   - This may affect zero-downtime guarantees if not handled with instant failover strategies.


### 🚦 2. Active-Active Architecture
- This architecture ensures high availability and zero downtime by running multiple active nodes that can serve traffic simultaneously.

- Commonly used in distributed systems, microservices, and NoSQL databases like Cassandra, DynamoDB, Couchbase, etc.

- Both databases (or services) are kept in bidirectional sync, meaning each can handle reads and writes, and changes are replicated across.

- Load is distributed using a Load Balancer or DNS round-robin.

- Eliminates single point of failure (SPOF) — if one node fails, others take over without interruption.

- Requires conflict resolution strategies (e.g., last-write-wins, vector clocks) to handle data consistency.

- Ideal for stateless services, or services with replicated state via queues, caches, or distributed consensus algorithms (e.g., Raft, Paxos).

- Generally more complex to design and maintain, especially for write-heavy workloads where consistency is critical.

![Active-Active Architecture](<./active-active.png>)


## 🧠 When to Use Which Architecture?

| Criteria                          | Active-Passive                      | Active-Active                          |
| --------------------------------- | ----------------------------------- | -------------------------------------- |
| **System Criticality**            | Tolerates minor failover time       | Needs zero downtime                    |
| **Traffic Load**                  | Moderate or predictable             | High or globally distributed           |
| **Consistency Needs**             | Strong consistency preferred        | Eventual consistency acceptable        |
| **Complexity**                    | Simpler setup and maintenance       | More complex (conflict handling)       |
| **Resource Utilization**          | Passive node sits idle              | All nodes fully utilized               |
| **Use Case Examples**             | RDBMS failover, backup systems      | Cassandra clusters, load-balanced APIs |
| **Recovery Time Objective (RTO)** | Higher (manual or semi-auto switch) | Lower (auto-redundancy, failover)      |
| **Data Synchronization**          | Unidirectional replication          | Bidirectional replication              |


## ✅ Choose Active-Passive if:
- You prioritize simplicity and strong consistency.

- System can tolerate short downtime during failover.

- You're using relational DBs (e.g., MySQL, PostgreSQL) with master–replica setup.

- You want easier conflict-free replication.

- Real-world fit: Banking systems, GitHub's internal DB setup, on-prem enterprise software.

## ✅ Choose Active-Active if:
- You need high availability (99.99% or more) and fault tolerance.

- Your app is distributed across regions/zones.

- You're using NoSQL databases or stateless services.

- You can handle eventual consistency and resolve conflicts smartly.

- Real-world fit: Netflix, Uber, Amazon, Cloudflare, global SaaS platforms.

   

