#Problem Statement - Piyush Garg - Youtube

Scenario: Constant database updates (e.g., Zomato delivery driver location changes) can overwhelm the database.
-High Operations Per Second (OPS)
-Low throughput databases struggle with high-frequency updates
-Similar issues in real-time applications like group chats
How Kafka Solves These Problems
-Kafka offers high throughput but isn't a database alternative due to lower storage capacity.
-Database: Low throughput, high storage
-Kafka: High throughput, low storage

Kafka enables bulk data insertion with high throughput.

![Kafka working](<./images/Screenshot (59).png>)

#Kafka Architecture

Topics:
-Logical partitioning of messages; Kafka topics are specific to servers.
Example: Zomato might have separate topics for rider updates and restaurant updates.

![kafka](<./images/Screenshot (61).png>) ![kafka](<./images/Screenshot (60).png>)

#Partitions:

-Topics are partitioned to manage data, similar to sharding.
-Example: Data distribution based on regions (North, South, East, West)

-Producers specify the partition for data storage.
Single consumer setup:
-Multiple partitions with more consumers than partitions lead to some consumers not having partitions to consume.
-A single partition can only be consumed by one consumer.

-So basically kafka have
Kafka --> Topics --> Partitions

![Main Architect](<./images/Screenshot (62).png>)
![consumer-architect](<./images/Screenshot (64).png>)

#Consumer Group

![consumer_group](<./images/Screenshot (65).png>)

-Allow multiple consumers to balance partition consumption.
Example: One partition, one consumer group for queueing; one partition, multiple groups for pub/sub.

#Kafka with NodeJS

-Kafka load balancing is managed by Zookeeper, which handles replicas and self-balancing.

#In Terminal

docker run -p 2181:2181 zookeeper // For running or starting the zookeeper

//To run the kafka in our terminal
docker run -p 9092:9092 ^ // port mapping where we are defining the the default port of kafka 9092 will run on our system 9092
-e KAFKA_ZOOKEEPER_CONNECT=192.168.34.125:2181 ^ //connect system IP with zookeeper port
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.34.125:9092 ^ //providing the listener for the kafka port
-e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 ^ // for replication factor
confluentinc/cp-kafka

Code Structure
-Admin: Sets up infrastructure topics.
-Producers: Generate messages.
-Consumers: Consume partitions.

In Project terminal add the kafkajs library
-yarn add kafkajs

#Admin.js file

in admin file we are defining the connection between project and kafka and also creating the topics for our project

#producer.js

-make connection with producer
-send message as per our topic

#consumer.js

-make connection with consumer
-subscribe with our topic

#PROCESS_FLOW

![process_flow](<./images/Screenshot (66).png) ![process_flow](<./images/Screenshot (67).png>)

-In our example we are creating one Topic with two partition
-In producer we will define in which partition we want to store the data
-In Consumer we listening the port for any data available from the producer
-If i run the two consumer portal with same group id then auto self balancing will be occur and each consumer will listen from their partition message
-If i create another consumer with different group id then group 1 will be im auto balance and group 2 will listen all the partitions
