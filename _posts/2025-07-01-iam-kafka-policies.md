---
layout: post
title: IAM policies for Kafka
subtitle: What is going on?
date: 2025-07-01
background: /images/yosef-futsum-ZAvhxLTcSok-unsplash.jpg
---

Apache Kafka is a widely used event streaming system built for large scale data pipelines and streaming applications.
AWS offers a managed service for Kafka clusters (MSK) which handles cluster maintenance and operation. The permissions to these two worlds are handled separately in AWS. This separation might not be very surprising but it took surprisingly long for me to realize because the documentation is not clear about it.

## Kafka's own configuration

Kafka clusters provide a set of APIs to operate the cluster and use it in applications. Typically, access control lists (ACLs) are used to configure access, topics are created and configured, partitions are managed, and clients can connect to topics.
The popular Java client library wraps it in Consumer, Producer and Admin APIs ([docs](https://kafka.apache.org/documentation/#api)).

When the Kafka cluster is an MSK cluster, access to its API is governed by IAM (if that is the selected authentication mechanism).
For each operation there is a specific action that needs to be allowed, similar to how ACLs are used. For exampl, the user/role needs to have the `kafka-cluster:CreateTopic` action allowed to create a new topic. These actions or APIs are grouped as [`Apache Kafka APIs for Amazon MSK cluster`](https://docs.aws.amazon.com/service-authorization/latest/reference/list_apachekafkaapisforamazonmskclusters.html) and the actions have the prefix `kafka-cluster`.

## AWS Kafka cluster configuration

When a cluster is managed by AWS as an MSK cluster, its configuration is also managed via APIs. These support things like cluster creation, getting central information about the cluster, or updating something like storage.
The access to these APIs is also managed by IAM. For example, to create a new cluster the user/role needs to have the `kafka:CreateCluster` action allowed in its policy. These actions and APIs are named [`Amazon Managed Streaming for Apache Kafka`](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonmanagedstreamingforapachekafka.html) and have the action prefix `kafka`.

## Naming still problem number one

Again, the separation of the two APIs makes sense. Apache Kafka is an open source system and has its own way of being managed.
What was troubling for me was that it is not clear from the official AWS documentation that there are these two separate APIs and when to use which prefix. There are numerous examples on the internet that use either one or the other without explanation. It is not very clear to me what the difference between `kafka:DescribeCluster` and `kafka-cluster:DescribeCluster` is.

Finally, I would have chosen the prefixes the other way around. Or maybe not? Still confused.
