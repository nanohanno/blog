---
layout: post
title: Kafka replication
subtitle: Topic copies
date: 2024-06-21
background: /images/default_post.jpg
---

Kafka is built for scale and reliability. It is at the center of many distributed systems that use it for asynchronous event processing which can accomodate for failures, spikes in load and latency requirements.

## Producer time outs

However, the system is only reliable and scalable when it is configured properly. Central parameters that need to tuned to the demands of the producers and consumers as well as the general setup layout are:

1. Partition count
1. Replication factor

## Partition count

A topic holds all events that are similar to each other and therefore written to the same topic. That topic lives as logs on brokers - say servers - of the Kafka cluster. If a topic is accessed heavily by producers and/or consumers, it can be broken up into multiple partitions. That means multiple brokers hold a subset of the events and can serve them to clients which improves throughput.

## Replication factor

The partitions of a topic are distributed over the brokers of a Kafka cluster. Additionally, the partitions can be copied to multiple brokers which is called replication. Here, it is important that only one broker is the leader for a partition which means it receives all new events that are written to the partition. The other replicas then copy the log of the leader and can receive read requests on that partition. This configuration helps in reliability because when one of the brokers goes down for maintenance reason or technical failures, there are still other brokers that have the data and can serve incoming requests. Additionally, it can help in read throughput because events from a partition can now be read by clients from multiple brokers.

An important addition to the replication factor is the `min.insync.replicas` setting on the broker or topic level and the `ack` setting on the consumer side. The `ack` configuration parameter defines the nature of a confirmation that a producer needs to receive from the kafka cluster before it can assume produce to be successful. By default, many clients set it to `all` which means all in-sync replicas need to receive the new event before the producer can assume it to be succesffully written. In-sync replicas means the number of replicas that are available and are not lagging behind by too much. The `min.insync.replicas` setting on the cluster can now restrict this process and define the minimum required number of insync replicas.

As an example, we could have a replication factor of 3 and and `min.insync.replicas` of 1 with a producer setting of `ack=all`. Up to two replicas can be down and the new event can still be written succesfully. However, if that replica goes down and cannot be recovered we loose data. To prevent such cases, we can increase the `min.insync.replicas` factor to 2. Now, we need at least two replicas to be available to write a new event. If one of those goes down we still have one replica that holds the correct data and can serve it. As soon as the other brokers come up again they will synchronize the topic and no data is lost. On the other hand, if we set the `min.insync.replicas` setting to the number of replicas (3), we cannot accept any downtime of a broker because if one of the replicas is down no new events can be written.

## Conclusion

As said in the beginning, Kafka is built for high reliability and availability. There is however always a trade-off with resource consumption and performance which depend on a correct configuration of the complete system.

## Resources

https://www.conduktor.io/kafka/kafka-topics-choosing-the-replication-factor-and-partitions-count/
https://www.conduktor.io/kafka/kafka-topic-configuration-min-insync-replicas/
https://repost.aws/knowledge-center/msk-avoid-disruption-during-patching
