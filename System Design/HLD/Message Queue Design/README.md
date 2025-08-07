# 📦 Message Queue Design

Basic Architecture

## Producer --->> [Message]---> Queue Server---> Consumer

## ✅ Why Message Queues Are Needed

### 1. Asynchronous Processing
   Offload heavy or slow tasks (like email/SMS sending, notifications) from the main flow.

Example:
```md
E-Commerce Website --> Queue --> Notification Service / Email Service
```

#### ❓ Why not just use async/await or background tasks?

- If the service crashes, messages are lost.

- Queue systems ensure reliability through persistence and retry logic.
   
### 2. Pace Matching (Load Balancing)
- Services may operate at different speeds.
- Example:
```bash
Inventory: 15 req/sec
Product: 25 req/sec
Order: 10 req/sec
Notification Service: Can handle only 20 req/sec
```
- Without a queue, this creates backpressure or overload on slower services.

### 3. Location/Tracking Use Case
- For delivery apps, frequent location updates can overwhelm the server.

- A queue buffers high-volume updates:
```bash
  Driver Location Updates --> Queue --> Location Processor
```
## 🔁 Messaging Patterns

### ▶️ Point-to-Point (P2P)
- One producer → queue → one random consumer from a consumer group.

- Each message is consumed once.

### 📣 Publish/Subscribe (Pub/Sub)
One producer → exchange → multiple subscribed queues.

Each queue gets a copy of the message.

## ⚙️ How Messaging Queues Work

## Apache Kafka

### 🔍 Visual Diagram: Kafka Flow

```
             +-------------+
             |  Producer   |
             +------+------+
                    |
                    | [Key, Value]
                    v
              +-----+-----+
              |   Kafka   |
              |  Broker   |
              +-----+-----+
                    |
           +--------+--------+
           |                 |
     +-----+-------+     +-----+-------+
     | Partition 0 |     | Partition 1 |
     +-----+-------+     +-----+-------+
           |                 |
     Offset: 0-1-2      Offset: 0-1-2
           |                 |
   +-------+------+  +-------+------+
   |   Consumer A  |  |  Consumer B  |
   | (Group Alpha) |  | (Group Alpha)|
   +--------------+  +--------------+

```

### 🧱 Core Components:

| Component          | Description                                        |
| ------------------ | -------------------------------------------------- |
| **Producer**       | Sends messages to Kafka                            |
| **Consumer**       | Reads messages from partitions                     |
| **Consumer Group** | Group of consumers for load balancing              |
| **Topic**          | Logical category to group messages                 |
| **Partition**      | Splits a topic for parallelism                     |
| **Offset**         | Unique message position in partition               |
| **Broker**         | Kafka server node                                  |
| **Cluster**        | Group of brokers                                   |
| **Zookeeper**      | Manages cluster metadata (in older Kafka versions) |


### 📤 Message Structure
- Key (optional): Used for partitioning

- Value (required): Actual message payload

- Partition (optional): Specific partition to send to

- Topic (required): Destination topic

### 📈 Message Routing
```
If key exists:
    --> Hash(key) % NumPartitions --> Selected Partition
If key not present:
    --> Round Robin Partition Assignment
```

### ⛓️ Offset Management
- Each consumer tracks offset (last read index) in a partition.

- Stored in Kafka or Zookeeper (based on version).

- Example:
```
Consumer Group A --> Topic A --> Partition 0 --> Committed Offset = 3
```
### 🔁 Retry & DLQ (Dead Letter Queue)
- If a consumer fails to process a message:
  - Kafka retries as per configuration.
  - If retry limit exceeds, move message to Dead Letter Queue for manual inspection.
### 🔄 Resilience
- Broker Down: Leader partition is replaced by a replica.

- Consumer Down: Another consumer in the group takes over.

- Queue Full: Kafka uses disk storage, not memory, so messages are retained as per retention policy.

## 🐇 RabbitMQ

### 🔍 Visual Representation (RabbitMQ)
```

                Routing      +-----------+    +--------------+
                 Key 1  +--->| Queue - 1 |--->| Consumer - 1 |
                        |    +-----------+    +--------------+
        +-----------+   |
        | Exchange-1|----
        +----+------+   |
             ↑  Routing |    +-----------+    +--------------+
             |   Key 2  +--->| Queue - 2 |--->| Consumer - 2 |
             |          ↑    +-----------+    +--------------+
Producer ____|          |
             |          |
             |          |
             |          |
             v          |
      +-----------+     |    +-----------+    +--------------+
      | Exchange-2|-----+--->| Queue - 3 |--->| Consumer - 3 |   
      +----+------+          +-----------+    +--------------+
                   Routing
                    Key 3

```

### 🧱 Core Flow
```
Producer --> Exchange --[Routing Key]--> Queue --> Consumer
```
### 🔀 Exchange Types
1. Fanout
   1. Broadcast to all queues bound to the exchange.

2. Direct (Routing Key Based)
   1. Exact match between message’s routing key and queue binding key.

3. Topic
   1. Uses wildcards like order.* or *.notification.
   2. Flexible and scalable routing.

4. Headers (less used):
   1. Routes based on message headers instead of keys.

### 🔁 Retry in RabbitMQ
- No offset tracking.

- If consumer fails to acknowledge:
  - Message is requeued.
  - After x attempts, it can be sent to a Dead Letter Exchange (DLX).


## 🔄 Kafka vs RabbitMQ
| Feature                | Kafka                                      | RabbitMQ                             |
| ---------------------- | ------------------------------------------ | ------------------------------------ |
| **Model**              | Pull-based                                 | Push-based                           |
| **Storage**            | Disk-based, persistent log                 | Memory-first (disk-backed)           |
| **Ordering**           | Per partition                              | No guarantee unless in single queue  |
| **Delivery Semantics** | At least once / Exactly once (with config) | At most once / At least once         |
| **Replay**             | Supported via offset                       | Not natively supported               |
| **Throughput**         | Very high (designed for big data)          | Medium (application-level messaging) |
| **Best For**           | Real-time analytics, stream processing     | Task queues, transactional messaging |

## 🧠 Bonus: When to Use What?
| Use Case                             | Recommended Tool |
| ------------------------------------ | ---------------- |
| High-throughput event streaming      | Kafka            |
| Simple background jobs / async tasks | RabbitMQ         |
| Ordering within jobs                 | RabbitMQ         |
| Real-time analytics / log ingestion  | Kafka            |
| Request retry with manual inspection | Both (with DLQ)  |

### Q. What happens when Queue Size limit exceed?
- So kafka have many brokers (servers) who serves the incoming requests.
### Q. What happened to messages when queue goes down?
- Lets' Assume Architecture as 

Cluster: 
- Group of Brokers
- for example have three brokers Br1, Br2, Br3
- We have one topic Topic 1 and have two partition P0, P1

Now Br1 ---> Topic-1 ---> P0 and Br2 ---> Topic-1 ---> P1
- So event single topic partitions may have two different brokers
- now the P0 in Br1 Goes down then there is replica for it inside another brokers Br2 who replace the leader partition (Br1)
- So every partition (Leader) have their replicas inside different brokers so whenever leader goes down replica take over their actions
- replica partition continuously will read from the leader partition for any changes
- All Read/Write operation will perform on the leader partition until it goes down

### Q. What happens when Consumer goes down?
- Another consumer from that consumer group will take over place

### Q. What happens when consumer failed to processed message? Or How retry works different ways to retry
- Example
  - The partition have 7 committed offset values and consumer is on 6 now when it reads 7 which is faulty message then consumer will unable to perform action then it retry the execution and after some of retry still not able to perform then it transfer the message to the Dead Letter Queue and move forward

## RabbitMq vs Kafka

- Push Based Approach vs Pull Based Approach (kafka continuously) pulling from the queue about any new message

## RabbitMq

Producer ---> Exchange-1 ---> Binding of Routing Key-1  ---> [ Queue 1] ---> Consumer-1
Producer ---> Exchange-1 ---> Binding of Routing Key-2  ---> [ Queue 2] ---> Consumer-2
Producer ---> Exchange-2 ---> Binding of Routing Key-2  ---> [ Queue 2] ---> Consumer-2
Producer ---> Exchange-2 ---> Binding of Routing Key-3  ---> [ Queue 3] ---> Consumer-3

Above the diagram of working RabbitMq

### Types of Exchanges;
- Fan-out:
  - When Message comes with message key from the producer to the exchange this method with transfer the message to all the Queues that are associated with this exchange
- Fixed Exchange:
  - In this type message comes with the message key and routing key based on routing key that exchange will pass the message to the particular queue
- Topic Exchange: This is also called wild card where we define rule for ex msg-123 something comes from the message then we define in that case it goes to selected queue only

### Retry Capability:
- In RabbitMQ where wo don't have offset concept so after failing the message will requeue again inside the queue
- After multiple retry if still doesn't perform the it will pass to the dead queue