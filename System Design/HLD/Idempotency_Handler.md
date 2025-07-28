# Idempotency Handler

An idempotency handler is a mechanism used to ensure that a particular operation can be safely retried without causing unintended effects, like duplicate data creation or repeated payment.

## 🔁 What is Idempotency?
Idempotency means:

No matter how many times you perform the same operation, the result will always be the same.

For example:

Calling GET /user/123 multiple times should always return the same user data.

Calling POST /payments multiple times should not charge the user multiple times if it's the same payment.

## 🧠 Why Do We Need an Idempotency Handler?
In distributed systems or network environments:

The client may retry a request if it gets a timeout.

Servers may process the same request more than once if not handled properly.

This can cause problems like:

Duplicate orders

Double payments

Redundant entries in the database

To avoid side effects, we need an idempotency handler.

## ⚙️ How Does an Idempotency Handler Work?
Typically, it's implemented by requiring a unique idempotency key with requests that mutate data (POST, PUT, etc.).

Flow:
The client generates a unique idempotency-key and sends it with the request.

The server:

Checks if a request with this key was already processed.

If yes, it returns the cached response.

If no, it processes the request, stores the response and associates it with the key.

Subsequent retries with the same key get the same stored result.

### DB Structure (simplified):
```json
{
  "idempotencyKey": "abc123",
  "status": "completed",
  "response": { "paymentId": "xyz456", "status": "success" }
}
```
## ✅ Benefits
Prevents duplicate operations

Improves reliability in retry scenarios

Ensures safe retries in distributed systems

## 🧱 Where It's Used
Payment Gateways (Stripe, Razorpay, etc.)

Order creation APIs

Email sending systems

Message queues and job schedulers

## 👨‍💻 Common Tech Implementations
Store idempotency keys in:

Redis (for fast lookup and TTL)

SQL/NoSQL DB

Keyed by:

User ID + Endpoint + Payload Hash

## How can it handle sequential and parallel requests❓

### ✅ Sequential Requests
This is simple:

First request hits the server.

Server processes it, stores the response tied to the idempotency key.

If the client retries after some time (e.g., due to timeout), the same key is sent.

Server sees it already processed → returns same response → ✅ No side effect.

### 🔄 Parallel Requests (same idempotency key)
This is more complex because:

Multiple requests with the same idempotency key hit the server at the same time.

Server must prevent race conditions.

#### 💡 How it's handled:
Database or Redis lock on the idempotency key.

First request acquires the lock, processes and stores the result.

Other parallel requests:

Wait for the lock to release

OR see the key already exists and return the cached result

This ensures only one request is processed, others return the same response.

## 🔐 Key point:
Idempotency only works if the same idempotency key is reused for the same logical operation.

To share the same idempotency key across devices, the app must:

Save the idempotency key on backend (e.g., after "start payment") and return it

Clients (phones) then use that same key when doing the actual action (e.g., payment)