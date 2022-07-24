---
layout: post
title: Kafka vs. RabbitMQ   —  My Top 10 Differences
---

Just wanted to publish a top 10 list differences between these two messaging platforms which are both awesome in my opinion. So let’s jump in.

### [1] Dumb or Smart?

Kafka follows the Dumb-Broker / Smart Clients concept. RabbitMQ adopts the opposite and has many built-in broker features making it intelligent.   
AMQP protocol that RabbitMQ adopts requires the broker to be more intelligent about routing. Kafka offloads all of important routing and message distribution decisions to client side and acts on behalf of them. Ex: Kafka Partition Assignment, Rebalancing, Messaging Routing are all driven by the client.

### [2] Efficiency of Writes

Kafka adopts the concept of Partition Logs a Write Ahead Log pattern which makes the write highly performant and also durable (persistent).

RabbitMQ does not have this concept. It has the Durable Topics and Transient (In-Memory) topics. In-memory makes this faster for RabbitMQ but durable guarantees come from Disk Writes but they are slower.

### [3] How Message Publication Works

Kafka Topics has the concept of partitions and therefore it can be perform uni-cast, multi-cast and broadcast publications based on the configuration. The topic-partition configuration and consumer group configurations drive the way the messages are published amongst consumers.

RabbitMQ has the concept of exchanges that comes inherently from AMQP protocol. How the exchange is configured by the publisher determines whether it is a broadcast, multi-cast or p2p messaging (ex: direct, fan-out, header based etc).

### [4] Retention Policy

Kafka Topics can be retained in disk as per the requirement. It won’t delete messages upon delivery making it a durable message storage. This opens up the possibilities to support many use-cases that desires durability and the re-playability.

RabbitMQ does not have this concept. Instead messages are not persisted beyond successful delivery.

### [5] Durability

Kafka can keeps a copy of partition in multiple brokers. This can be tuned to fit the needs of the use-case. We can determine the number of replicas to be maintained per partition and those number of replicas are spread across the number of brokers. This minimizes the risk of data loss and the ability to failover and be available in case of broker failures.

RabbitMQ only stores messages of a particular queue in one node. If the node goes down, the messages could be lost. Data corruption could mean a significant data loss.

### [6] Distributed System Design

Kafka follows fully distributed system architecture. The concept of Partition can be considered as the most atomic unit of Kafka. The partitions are distributed across nodes and the system is build to serve client cohesively across all nodes without having a tight dependency on each node with the partition. Yes partitions do have a leader node, but it is elected based on quorum and can be re-elected if the leader goes down. So essentially, at its’s core, Kafka avoids single point of failure from the ground-up in the way it is designed.

RabbitMQ is not a fully distributed system like Kafka. Only a single node in RabbitMQ will have the full topic content (metadata and topic data) which makes nodes independently specialized, meaning a downtime of that node could lead to unavailability to part of the overall system.

### [7] Streaming Support

Kafka has first class support to be a streaming platform with Kafka Streams. Kafka Streams is built on Kafka’s basic constructs and java libraries are there to build streaming topologies.

RabbitMQ does not have streaming support.

### [8] Data Integration Support

Kafka supports ETL (Extraction and Load) capabilities using Kafka Connect which runs outside Kafka broker which again keeps the broker dumb.

RabbitMQ can support data sourcing and sinking but it is mostly done by custom plugins configured on the broker.

### [9] Delivery Guarantee

Kafka now provides configurable delivery guarantees — At most once, At least once and Exactly once and they are configurable based on replication settings, acknowledgment model and idempotence settings.

With RabbitMQ, we can achieve at most once and at least once based on our consumer acknowledgement and publisher confirms settings.

### [10] Use-case Driven

Because RabbitMQ has the concept of Plugins and it natively supports many features and support specific use-cases. For example, you can build a web-socket architecture easily with RabbitMQ.

Kafka does not provide any such features out of the box within the broker and any such designs has to be custom built within our architecture using the basic constructs of Kafka.

### References

- RabbitMQ in Action: [https://www.amazon.com/RabbitMQ-Action-Distributed-Messaging-Everyone/dp/1935182978](https://www.amazon.com/RabbitMQ-Action-Distributed-Messaging-Everyone/dp/1935182978)  
- Apache Kafka — The Definitive Guide: [https://www.confluent.io/resources/kafka-the-definitive-guide/?test=hushlylp&var=original](https://www.confluent.io/resources/kafka-the-definitive-guide/?test=hushlylp&var=original)

and Many Other Comparison Blog articles that are awesome, informative.