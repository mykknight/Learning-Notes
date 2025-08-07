# ⚖️ Load Balancer

A Load Balancer is a system that distributes incoming network or application traffic across multiple backend servers to ensure no single server becomes overloaded.

## 🎯 Purpose
- Ensure high availability and reliability by distributing requests evenly.

- Prevent server overload by spreading traffic based on smart algorithms.

- Improve scalability, response time, and fault tolerance.

## 🧱 Types of Load Balancers
🔹 1. Application Layer Load Balancer (Layer 7)
- Makes decisions based on:

  - Headers

  - Cookies

  - Session data

  - URLs

  - Responses

- Can support caching, SSL termination, and sticky sessions.

🧠 Example: AWS ALB, Nginx, HAProxy (as reverse proxy)

🔹 2. Network Layer Load Balancer (Layer 4)
- Operates based on:

  - Source IP

  - Destination IP

  - TCP/UDP ports

- Offers fast routing, with lower latency.

🧠 Example: AWS NLB, IPVS, LVS

> 🔍 Most load balancing algorithms (like Round-Robin, Least Connection) are implemented at the network level.

## ⚙️ Load Balancing Algorithms
Algorithms decide how traffic is distributed. These are categorized into:

## 📌 Static Algorithms
-  These don't rely on real-time server load or health—they use fixed logic.

### 1️⃣ Round Robin
- Sequentially assigns requests to servers in a circular order.

```
          +-------------------+
User Req →|   Load Balancer   |
          +-------------------+
                  |
     ----------------------------
     |            |            |
+---------+  +---------+  +---------+
| Server1 |  | Server2 |  | Server3 |
+---------+  +---------+  +---------+
   ↑          ↑          ↑
   |__________|__________|
      Requests distributed
        in round-robin
            fashion

```


```c
Clients  →  S1  →  S2  →  S3  →  S1  →  S2  →  S3
            ↑     ↑     ↑     ↑     ↑     ↑
         Request 1 to 6 assigned cyclically

Request 1 → Server A  
Request 2 → Server B  
Request 3 → Server C  
Request 4 → Server A (again)
```

#### ✅ Advantages:

- Simple and easy to implement

- Equal distribution of requests

#### ❌ Disadvantages:

- Doesn’t consider server capacity

- May overload weaker servers

### 2️⃣ Weighted Round Robin
- Assigns weight to each server based on capacity.

- A server with higher weight receives more requests.

```
          +-------------------+
User Req →|   Load Balancer   |
          +-------------------+
                  |
     ------------------------------------
     |                 |                |
+---------+       +---------+       +---------+
| ServerA |       | ServerB |       | ServerC |
+---------+       +---------+       +---------+
[Weight-3]        [weight-1]        [weight-1]

→ Server1 gets 3x more requests than Server2
```


```java
Server A (Weight 3): Request 1, 2, 3  
Server B (Weight 1): Request 4 
Server C (Weight 1): Request 5   
Server A (Weight 3): Request 6, 7, 8
```

#### ✅ Advantages:

- Protects low-capacity servers

- Still simple and static

#### ❌ Disadvantages:

- Doesn’t handle request processing time

- Long-running requests may still overload smaller servers


### 3️⃣ IP Hash

```text
          +-------------------+
User Req →|   Load Balancer   |
          +-------------------+
                  |
       Hash(Client IP)
                  ↓
      +------------------------+
      |   Hash(IP) → ServerN   |
      +------------------------+
                  ↓
           +-------------+
           |  Server-N   |
           +-------------+

→ Same IP → Same server (sticky session)

```
- Uses a hash function on the client’s IP to pick a server.

- Ensures the same client gets routed to the same server (useful for session persistence).

#### ✅ Advantages:

- Sticky sessions without cookies

- Easy to implement

#### ❌ Disadvantages:

- Clients behind the same proxy (e.g., in a café) may hash to the same server, causing imbalance

- Can’t ensure even distribution

## ⚡ Dynamic Algorithms
> These evaluate real-time server metrics (e.g., connections, response time) to make decisions.

### 1️⃣ Least Connections
- Routes new requests to the server with the fewest active connections.

```text
          +-------------------+
User Req →|   Load Balancer   |
          +-------------------+
                  |
     ---------------------------------
     |               |               |
+-----------+   +-----------+   +-----------+
| Server-A  |   | Server-B  |   | Server-C  |
| Conn: 5   |   | Conn: 2   |   | Conn: 7   |
+-----------+   +-----------+   +-----------+

→ Next request goes to Server-2
```

```sql
Server A → 5 active connections  
Server B → 2 active connections
Server C → 7 active connections    
→ New request goes to Server B
```
#### ✅ Advantages:

- Dynamically balances based on real-time load
- Works well when all requests have similar resource usage.


#### ❌ Disadvantages:

- Assumes all connections are equal, which is often false in real-world traffic.

- Can't detect whether a server is overloaded due to high-bandwidth or long-running connections.

#### 🧠 Example Pitfall:
> Server A has 5 inactive clients, Server B has 2 high-traffic clients — LB still sends more requests to B!

### 2️⃣ Weighted Least Connections
- Similar to LC, but adds weight to each server to reflect its capacity. A more powerful server (e.g., with more CPU/RAM) can handle more connections and will be favored.

```text
          +-------------------+
User Req →|   Load Balancer   |
          +-------------------+
                  |
     ------------------------------------------
     |                    |                   |
+-----------+     +-----------+       +-----------+
| Server-1  |     | Server-2  |       | Server-3  |
| Conn: 5   |     | Conn: 2   |       | Conn: 6   |
| Weight:10 |     | Weight:1  |       | Weight:5  |
+-----------+     +-----------+       +-----------+

→ Selects server with lowest (Conn/Weight) ratio
```

#### Improvement over LC:

- Adds awareness of server capability through manual weights.

- Useful in heterogeneous environments (different server specs).

#### Still shares LC's core limitation:

- ❗ Connection count ≠ actual load
A server with few but heavy connections (e.g., video streams) can still be picked, while another with many light or idle connections might be ignored.


```javascript
Active Connections ÷ Weight
```
#### 🔢 Example:

- Server A → 2 connections, weight = 10 → 2/10 = 0.2

- Server B → 1 connection, weight = 1 → 1/1 = 1
  - → Request goes to Server A (lower ratio)

#### ✅ Advantages:

- Accounts for server capacity and real-time load

#### ❌ Disadvantages:

- Slightly more complex to compute
  
- Connection count ≠ actual load:
    >The algorithm assumes each active connection uses similar resources. In reality, a server with fewer connections could be handling high-traffic workloads (e.g., video streaming), while another server with more connections might be mostly idle. This leads to misleading load distribution.

Example:

```text
┌────────────┬────────────────────┬─────────────┬────────────────────┐
│ Server     │ Active Connections │ Real Load   │ Algorithm Thinks   │
├────────────┼────────────────────┼─────────────┼────────────────────┤
│ Server-1   │ 5 (idle)           │ Low         │ High load          │
│ Server-2   │ 1 (heavy traffic)  │ High        │ Low load           │
└────────────┴────────────────────┴─────────────┴────────────────────┘
```

#### Result:
- Even with proper weights, WLC may route new requests to a server that is already resource-heavy, just because it appears to have fewer connections.

### 3️⃣ Least Response Time
- Picks the server with the lowest combination of:

  - Number of active connections

  - Time to First Byte (TTFB)

#### TTFB:
- ⏱️ The time it takes for a server to start responding to a client request — i.e., the time between the client sending a request and receiving the first byte of the response.

```text
          +-------------------+
User Req →|   Load Balancer   |
          +-------------------+
                  |
     ---------------------------------------------------
     |                         |                         |
+-------------+       +-------------+         +-------------+
|  Server-1   |       |  Server-2   |         |  Server-3   |
| Conn: 3     |       | Conn: 2     |         | Conn: 1     |
| TTFB: 200ms |       | TTFB: 100ms |         | TTFB: 400ms |
+-------------+       +-------------+         +-------------+

→ Picks server with lowest (Conn × TTFB)
```

```javascript
Active Connections × TTFB
```
#### ✅ Advantages:

- Smartest load balancing in terms of end-user performance

#### ❌ Disadvantages:

- Requires precise measurement of server response metrics

## 🔄 Summary of Algorithms
| Algorithm            | Type    | Load Awareness | Use Case                          |
| -------------------- | ------- | -------------- | --------------------------------- |
| Round Robin          | Static  | ❌ No           | Simple, even distribution         |
| Weighted Round Robin | Static  | ❌ No           | Account for server size           |
| IP Hash              | Static  | ❌ No           | Sticky sessions                   |
| Least Connections    | Dynamic | ✅ Yes          | Real-time load distribution       |
| Weighted Least Conn. | Dynamic | ✅ Yes          | Real-time + server capacity       |
| Least Response Time  | Dynamic | ✅ Yes          | Optimized for fastest performance |

## 🧠 Real-World Analogy
Imagine a pizza delivery system:

- 🍕 Round Robin: Every delivery guy gets one order in turn, no matter how far or busy they are.

- 🏋️‍♂️ Weighted Round Robin: Stronger guys get more deliveries.

- 📍 IP Hash: Same customer always gets the same delivery guy.

- 📉 Least Connections: Guy with the fewest deliveries gets the next one.

- ⚖️ Weighted Least Connections: Combines delivery guy strength + current load.

- ⏱️ Least Response Time: Fastest and least busy delivery guy gets the next job.