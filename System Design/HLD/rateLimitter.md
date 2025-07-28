# Rate Limiter

## Flow Diagram

        ┌─────────────┐
        │   Client    │
        └─────┬───────┘
              │
              ▼
     ┌──────────────────┐
     │ Application/API  │◄──── Reads Config (limits)
     │   Server         │
     └─────┬────────────┘
           │
           ▼
     ┌──────────────┐
     │ Rate Limiter │
     └─────┬────────┘
           │
           ▼
    ┌──────────────────┐
    │  In Memory       │◄──── fast local checks
    │   Cache          │    |
    └─────┬────────────┘    |
          │                 |
          ▼                 |
     ┌──────────┐           |
     │  Redis   │<---------- Central store for counts
     └─────┬────┘




### 🌐 Request Flow Walkthrough
Let’s say:

```json

User ID: user_123

Limit: 5 requests per minute

Time: 12:00:00 PM

Request: POST /login
```

### 🔁 First Request (12:00:00)
Step-by-step:
🧑 Client sends POST /login

🚪 API Server receives it

📁 Loads rate limit config:

```json
{
  "login": { "limit": 5, "window": 60 }
}
```
🎯 Calls RateLimiter.check('user_123')

🗃️ Cache lookup: Not found

⚡ Redis query: No key yet → count = 0

✅ Count is under limit → allow request

➕ Redis INCR key ratelimit:user_123:login → now 1

EXPIRE set to 60s

🗃️ Cache updated: { count: 1, ttl: 60 }

✅ Final: Request is allowed

### 🔁 5th Request (12:00:30)
Cache says: count = 4 → ok

Redis INCR → now 5

✅ Still allowed

✅ Final: Allowed

### 🔁 6th Request (12:00:45)
🗃️ Cache says: count = 5

⚡ Redis count = 5
```json
if (count >= limit) return 429
```
🚫 Final: Request is rejected with 429 Too Many Requests

### 🔁 7th Request (12:01:01)
Time has moved past 60s window

Cache is expired

Redis key ratelimit:user_123:login expired

#### 🆕 New window starts
Redis count reset → 0

✅ Final: Request is allowed

| Component    | What it does                                                                 |
|--------------|------------------------------------------------------------------------------|
| Client       | Sends the request to backend                                                  |
| API Server   | Hosts the login route, passes request to rate limiter before logic            |
| RateLimiter  | Central logic: checks limits, increments, and decides allow/deny              |
| Redis        | Stores request counts per user per endpoint → accurate, atomic, shared        |
| Cache        | Optional local memory to avoid hitting Redis for every request                |
| Config File  | Tells RateLimiter how many requests per window are allowed                    |


## 🧠 Problem Highlighting
In a distributed system (multiple servers/microservices/load-balanced nodes), you might have:

 Client Requests → Load Balancer -->

     ┌───────────────|───────────────┐
     │ RateLimiter 1 │ RateLimiter 2 │
     └─────┬─────────┘─────┬─────────┘
                     │
                     ▼
                  Redis

### ❓ question is: 
How does rate limiting work correctly and atomically, when multiple rate limiter nodes (1, 2, 3...) are reading/writing to the same Redis?

### ✅ The Answer: Because Redis operations are atomic
Even if 10 servers hit Redis at the same time, Redis ensures correctness, because:

🔐 Redis guarantees:
INCR is atomic

INCRBY, EXPIRE, SET, GET, etc. are all atomic

Even if multiple clients send commands in parallel, Redis processes one command at a time internally

📌 Example – INCR in Distributed System:
All servers run this:
```javascript
redis.incr("ratelimit:user_123");
```
Even if RateLimiter1 and RateLimiter2 hit Redis at the exact same time, Redis will:

Increment from 4 → 5

Then 5 → 6

Not both think it was 5

⚠️ So no race conditions — Redis handles this internally.

