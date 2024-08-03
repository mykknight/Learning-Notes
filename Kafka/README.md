#Problem Statement - Piyush Garg - Youtube

- If we are inserting/updating database after every second in a case where zomato delivery guy changing their location so our database goes down
- In that case we have higher OPS(operation per second)
- Database have very low throughput(maximum operation allowed per operation)
- same case will happend in group chat applications also (Real time Applications)

#How Kafka solve problems

-kafka have high throughput but it is not alternative of DB because have low storage
-Database:- Low Throughput and High Storage
-Kafka- High Throughput and Low Storage

![Kafka working](<./Screenshot (59).png>)

-Kafka is a service which have high throughput and can insert data in bulk

#Kafka Architecture

@Topic:-
-logical partitioning of the messages & kafka have different topic for one server,

- for example in zomato kafka has 2 topics one for rider updates and another for restaurant updates

![kafka](<./Screenshot (61).png>) ![kafka](<./Screenshot (60).png>)

@partition:-

- Each topic have a partition to manage the database like shrading if i want to distribute the data as per the location wise like north, south, east and west

-So basically kafka have
Kafka --> Topics --> Partitions

![Main Architect](<./Screenshot (62).png>)

- So basically in producer we will define in which partition we want to store data and if have only one consumer then they will consume by only one consumer
- what if we have 4 partition and have 5 consumer then each will consume by one partition and one who left doesn't have any partiton to follow

- SO In kafka any consumer can consume many partitions but each partition can consume by only one consumer
  ![consumer-architect](<./Screenshot (64).png>)

@So if we want to consume partition by multi consumer then the concept comes as consumer group

#Consumer Group

- Here Self Balancing will be done by the groups

![consumer_group](<./Screenshot (65).png>)

-So using groups we can manage to create queue and pub/sub, where queue will be created by one partition have one consumer group and pub/sub by one partition have multiple groups

#Kafka with NodeJS

- kafka load balancing will be managed by the zookeeper
